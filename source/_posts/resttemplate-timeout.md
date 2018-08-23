---
title: RestTemplate连接池、超时时间配置
cdn: header-off
date: 2018-08-23 11:29:03
header-img: "/img/resttemplate.jpg"
tags:
	- Java
	- Spring
---
   RestTemplate是Spring Framework提供的比HttpClient更优雅的进行HTTP交互的模板类。Java Doc如是说：
> Spring's central class for synchronous client-side HTTP access. It simplifies communication with HTTP servers, and enforces RESTful principles. It handles HTTP connections, leaving application code to provide URLs (with possible template variables) and extract results.


## 默认的RestTemplate配置
   RestTemplate默认的ClientHttpRequestFactor为SimpleClientHttpRequestFactory，其中的createRequest方法，每次创建request都是通过jdk的URL类new一个新的连接，每个连接都占用一个端口，若资源没有及时释放，就会导致无法建立新的连接：
``` java
	protected HttpURLConnection openConnection(URL url, Proxy proxy) throws IOException {
        URLConnection urlConnection = proxy != null ? url.openConnection(proxy) : url.openConnection();
        if (!HttpURLConnection.class.isInstance(urlConnection)) {
            throw new IllegalStateException("HttpURLConnection required for [" + url + "] but got: " + urlConnection);
        } else {
            return (HttpURLConnection)urlConnection;
        }
    }
```
   从SimpleClientHttpRequestFactory中还能看出，
``` java
	private int connectTimeout = -1;
    private int readTimeout = -1;
```
   默认的超时时间设置为-1，若出现服务器宕机的情况，该连接将永远不会被释放。

## 配置RestTemplate线程池、超时时间
   为RestTemplate创建线程池，其中一个思路就是将HTTP Lib换成带有线程池功能的HttpComponents，设置线程池的同时，设置连接超时时间。
   RestTemplate文档中说到， RestTemplate默认使用标准的JDK工具进行HTTP连接，若想使用别的HTTP库（HttpComponents, Netty, OkHttp...）,可以通过InterceptingHttpAccessor.setRequestFactory(org.springframework.http.client.ClientHttpRequestFactory)属性进行修改。
   具体实现如下：
``` java
@Configuration
public class WebConfiguration extends WebMvcConfigurerAdapter {

    @Bean
    public RestTemplate restTemplate() {
        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
        connectionManager.setDefaultSocketConfig(SocketConfig.custom().setTcpNoDelay(true).build());
        connectionManager.setDefaultMaxPerRoute(20); // 每个路由(host)的最大连接数
        connectionManager.setMaxTotal(200); // 总的最大连接数

        HttpClient httpClient = HttpClientBuilder.create().setConnectionManager(connectionManager).build();

        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
		// 超时时间设置
        factory.setReadTimeout(30 * 1000);
        factory.setConnectTimeout(30 * 1000);
        factory.setConnectionRequestTimeout(30 * 1000);

        return new RestTemplate(factory);
    }
}
```

## HttpComponentsClientHttpRequestFactory实现原理

   以RestTemplate#getForObject(URI url, Class<T> responseType)为例，以下是调用栈：

	- RestTemplate#getForObject(URI url, Class<T> responseType)
	- HttpComponentsClientHttpRequest#executeInternal(HttpHeaders headers, byte[] bufferedOutput)
	// CloseableHttpClient是HttpComponentsClientHttpRequestFactory默认的HttpClient（HttpClients.createSystem()）
	- CloseableHttpClient#execute(HttpHost target, HttpRequest request, HttpContext context) 
	- InternalHttpClient#doExecute( HttpHost , HttpRequest ,HttpContext ) 
	- this.execChain.execute(route, wrapper, localcontext, execAware) 
	- MainClientExec#execute*(HttpRoute,HttpRequestWrapper,HttpClientContext,HttpExecutionAware) 
	// HttpClientConnectionManager就是在Bean中set到HttpClient中的PoolingHttpClientConnectionManager
	- HttpClientConnectionManager#requestConnection(HttpRoute,Object):ConnectionRequest 
	// 这里是lease时机
	- ConnectionRequest#get(long,TimeUnit):HttpClientConnection
	// 这是release时机 
	- CloseableHttpResponse#close()
	- ConnectionHolder#close() 
	- ConnectionHolder#releaseConnection() 
	- releaseConnection$releaseConnection(boolean)

   连接池的核心应该是HttpClientConnectionManager中的属性cPool的父类AbstractConnPool：
``` java
	@Contract(
    threading = ThreadingBehavior.SAFE_CONDITIONAL
)
public abstract class AbstractConnPool<T, C, E extends PoolEntry<T, C>> implements ConnPool<T, E>, ConnPoolControl<T> {
    private final Lock lock;
    private final Condition condition;
    private final ConnFactory<T, C> connFactory;
    private final Map<T, RouteSpecificPool<T, C, E>> routeToPool;
    private final Set<E> leased;
    private final LinkedList<E> available;
    private final LinkedList<Future<E>> pending;
    private final Map<T, Integer> maxPerRoute;
    private volatile boolean isShutDown;
    private volatile int defaultMaxPerRoute;
    private volatile int maxTotal;
    private volatile int validateAfterInactivity;
    ...
	...
	...
}
``` 
   其中，routeToPool是各个路由对应的连接池，一个路由可以存在DefaultMaxPerRoute个连接，所有路由的连接总数不得超过MaxTotal，routeToPool的构成如下：
```java
abstract class RouteSpecificPool<T, C, E extends PoolEntry<T, C>> {
    private final T route;
    private final Set<E> leased;
    private final LinkedList<E> available;
    private final LinkedList<Future<E>> pending;

    RouteSpecificPool(T route) {
        this.route = route;
        this.leased = new HashSet();
        this.available = new LinkedList();
        this.pending = new LinkedList();
    }
	...
	...
	...
}
```

### lease connection

``` java
	public Future<E> lease(final T route, final Object state, final FutureCallback<E> callback) {
        Args.notNull(route, "Route");
        Asserts.check(!this.isShutDown, "Connection pool shut down");
        return new Future<E>() {
            private final AtomicBoolean cancelled = new AtomicBoolean(false);
            private final AtomicBoolean done = new AtomicBoolean(false);
            private final AtomicReference<E> entryRef = new AtomicReference((Object)null);

            public boolean cancel(boolean mayInterruptIfRunning) {
                if (this.cancelled.compareAndSet(false, true)) {
                    this.done.set(true);
                    AbstractConnPool.this.lock.lock();

                    try {
                        AbstractConnPool.this.condition.signalAll();
                    } finally {
                        AbstractConnPool.this.lock.unlock();
                    }

                    if (callback != null) {
                        callback.cancelled();
                    }

                    return true;
                } else {
                    return false;
                }
            }

            public boolean isCancelled() {
                return this.cancelled.get();
            }

            public boolean isDone() {
                return this.done.get();
            }

            public E get() throws InterruptedException, ExecutionException {
                try {
                    return this.get(0L, TimeUnit.MILLISECONDS);
                } catch (TimeoutException var2) {
                    throw new ExecutionException(var2);
                }
            }

            public E get(long timeout, TimeUnit tunit) throws InterruptedException, ExecutionException, TimeoutException {
                E entry = (PoolEntry)this.entryRef.get();
                if (entry != null) {
                    return entry;
                } else {
                    synchronized(this) {
                        try {
                            while(true) {
                                E leasedEntry = AbstractConnPool.this.getPoolEntryBlocking(route, state, timeout, tunit, this);
                                if (AbstractConnPool.this.validateAfterInactivity <= 0 || leasedEntry.getUpdated() + (long)AbstractConnPool.this.validateAfterInactivity > System.currentTimeMillis() || AbstractConnPool.this.validate(leasedEntry)) {
                                    this.entryRef.set(leasedEntry);
                                    this.done.set(true);
                                    AbstractConnPool.this.onLease(leasedEntry);
                                    if (callback != null) {
                                        callback.completed(leasedEntry);
                                    }

                                    PoolEntry var10000 = leasedEntry;
                                    return var10000;
                                }

                                leasedEntry.close();
                                AbstractConnPool.this.release(leasedEntry, false);
                            }
                        } catch (IOException var8) {
                            this.done.set(true);
                            if (callback != null) {
                                callback.failed(var8);
                            }

                            throw new ExecutionException(var8);
                        }
                    }
                }
            }
        };
    }
```
   寻找空闲连接的流程大致如下：
+ 通过routeToPool找到对应route的连接池；
+ 遍历连接池RouteSpecificPool中的available列表，查询到空闲连接，将该连接entry放入连接池的leased列表中，并返回；
+ 将返回的future存入pending列表中

### release connection
``` java
	public void release(E entry, boolean reusable) {
        this.lock.lock();

        try {
            if (this.leased.remove(entry)) {
                RouteSpecificPool<T, C, E> pool = this.getPool(entry.getRoute());
                pool.free(entry, reusable);
                if (reusable && !this.isShutDown) {
                    this.available.addFirst(entry);
                } else {
                    entry.close();
                }

                this.onRelease(entry);
                Future<E> future = pool.nextPending();
                if (future != null) {
                    this.pending.remove(future);
                } else {
                    future = (Future)this.pending.poll();
                }

                if (future != null) {
                    this.condition.signalAll();
                }
            }
        } finally {
            this.lock.unlock();
        }

    }
```
   和lease连接原理类似，将连接从leased列表中移除并存入available列表，再移除pending列表中的future.
