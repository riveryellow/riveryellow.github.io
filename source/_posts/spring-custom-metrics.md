---
title: Spring Boot 1.5.X 自定义Metrics
cdn: header-off
date: 2018-06-09 11:00:14
header-img: "/img/springboot.jpg"
tags: 
	- Spring
	- Java
---

### 需求场景
   当我们使用Spring Boot Actuator对Spring项目进行监控时，我们可以得到大部分我们想要的监控数据，但是仍无法满足我们的监控场景需要。
   当前需要监控的场景之一是，在项目中通过RestTemplate调用远程接口时，监控调用的返回状态，以监控远程服务的状态。在一时间段内，若状态 > 400的次数过多，意味着远程服务出现问题，应当采取紧急维护措施。在[Spring Boot 1.5.X](https://docs.spring.io/spring-boot/docs/1.5.15.BUILD-SNAPSHOT/reference/htmlsingle/)中，并没有集成对RestTemplate的监控。因此，通过参考Spring文档中[52.5 Recording your own metrics](https://docs.spring.io/spring-boot/docs/1.5.15.BUILD-SNAPSHOT/reference/htmlsingle/#production-ready-recording-metrics)的方法，实现当前所需场景的监控。

### 实现方法

#### 1. 创建HTTP Request拦截器
```java
@Component
public class CustomRequestInterceptor implements ClientHttpRequestInterceptor {
	@Autowired
    private CounterService counterService;

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
    	long start = System.currentTimeMillis();
        String responseBody = "N/A";
        HttpStatus statusCode = null;
        ClientHttpResponse response;
        try {
        	response = execution.execute(request, body);
            if (response != null && (statusCode = response.getStatusCode()) != null) {
            	if (hasError(statusCode)) {
                	responseBody = getResponseBodyAsString(response);
                }
                counterService.increment(String.format("counter.status.code.%d-%s", statusCode.value(), request.getURI().getHost()));
            }
        } finally {
        	long end = System.currentTimeMillis();
            REMOTE_LOGGER.info("{}||{}||{}||{}||{}", request.getMethod(), request.getURI(), statusCode != null ? statusCode.value() : 0, responseBody, end - start);
        }

        return response;
    }

    private boolean hasError(HttpStatus statusCode) {
   		return (statusCode.series() == HttpStatus.Series.CLIENT_ERROR || statusCode.series() == HttpStatus.Series.SERVER_ERROR);
    }

    private String getResponseBodyAsString(ClientHttpResponse response) {
    	byte[] payload = getResponseBody(response);
        if (payload == null) {
        	return null;
        }

        Charset charset = getCharset(response);
        if (charset == null) {
        	charset = Constants.DEFAULT_CHARSET;
        }
        return new String(payload, charset);
    }

    protected Charset getCharset(ClientHttpResponse response) {
    	HttpHeaders headers = response.getHeaders();
        MediaType contentType = headers.getContentType();
        return (contentType != null ? contentType.getCharset() : null);
    }

    private byte[] getResponseBody(ClientHttpResponse response) {
    	try {
        	return FileCopyUtils.copyToByteArray(response.getBody());
        } catch (IOException ex) {
        	// ignore
        }
        return null;
    }

}
```
   其中，

```java
counterService.increment(String.format("counter.status.code.%d-%s", statusCode.value(), request.getURI().getHost()));
```
   对RestTemplate的返回结果状态进行计数，并将计数保存在内存中。

#### 2. 将HTTP返回状态计数加入到/health中
   查看counterService.increment()的源码，可以看到，
   DropwizardMetricServices.class
```java
public class DropwizardMetricServices implements CounterService, GaugeService {
	private final MetricRegistry registry;
    ...
	...
	private void incrementInternal(String name, long value) {
        if (name.startsWith("meter")) {
            Meter meter = this.registry.meter(name);
            meter.mark(value);
        } else {
            name = this.wrapCounterName(name);
            Counter counter = this.registry.counter(name);
            counter.inc(value);
        }

    }
	...
	...
}
```

   MetricRegistry.class
```java
public class MetricRegistry implements MetricSet {
	...
	...

	public Counter counter(String name) {
        return (Counter)this.getOrAdd(name, MetricRegistry.MetricBuilder.COUNTERS);
    }

	private <T extends Metric> T getOrAdd(String name, MetricRegistry.MetricBuilder<T> builder) {
        Metric metric = (Metric)this.metrics.get(name);
        if (builder.isInstance(metric)) {
            return metric;
        } else {
            if (metric == null) {
                try {
                    return this.register(name, builder.newMetric());
                } catch (IllegalArgumentException var6) {
                    Metric added = (Metric)this.metrics.get(name);
                    if (builder.isInstance(added)) {
                        return added;
                    }
                }
            }

            throw new IllegalArgumentException(name + " is already used for a different type of metric");
        }
    }

	public <T extends Metric> T register(String name, T metric) throws IllegalArgumentException {
        if (metric instanceof MetricSet) {
            this.registerAll(name, (MetricSet)metric);
        } else {
            Metric existing = (Metric)this.metrics.putIfAbsent(name, metric);
            if (existing != null) {
                throw new IllegalArgumentException("A metric named " + name + " already exists");
            }

            this.onMetricAdded(name, metric);
        }

        return metric;
    }
	
	...
	...

}
```

   自定义的计数器将在注册到MetricRegistry容器中，因此，在后续对计数的操作，都可以从MetricRegistry中获取数据进行操作。

+ 构建过滤器，获取计数

   要想批量获取MetricRegistry的metric信息，可以通过MetricFilter对Metric的key进行过滤。
```java
	private final static MetricFilter metricFilter = new MetricFilter() {
        @Override
        public boolean matches(String s, Metric metric) {
            if (s.startsWith("counter.status.code")) return true;
            return false;
        }
    };
```

+ 提取Metric供健康检查(/health)使用
```java
	Map metricCounters = metricRegistry.getCounters(metricFilter);
```

http://127.0.0.1:8088/health
``` json
{
  "status": "UP",
  "remoteHttpStatus": {
    "status": "UP",
    "detail": {
      "comment.social.3g.net.cn": {
        "counter.status.code.200": 229
      }
    },
    "failedCount": 0
  }
}
```