---
title: grpc实践(二) — 消息字段不能为空处理
cdn: header-off
date: 2018-06-19 15:03:49
header-img: "/img/grpc.png"
tags:
	- rpc
---

## 问题背景

### 现象
   在使用gRPC做远程接口调用时，从本地的实体通过BeanUtils.copyProperties()对即将传给客户端的由proto3生成的对象进行拷贝时会出现如下报错：
```console
org.springframework.beans.FatalBeanException: Could not copy property 'firebaseToken' from source to target; nested exception is java.lang.reflect.InvocationTargetException
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:626) ~[spring-beans-4.3.14.RELEASE.jar:4.3.14.RELEASE]
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:538) ~[spring-beans-4.3.14.RELEASE.jar:4.3.14.RELEASE]
	at com.gomo.grpc.authorize.service.AuthorizeService.getAccountByAccountId(AuthorizeService.java:162) ~[classes/:na]
	at com.gomo.grpc.authorize.service.AuthorizeService.findUsersByAccountIds(AuthorizeService.java:146) ~[classes/:na]
	at com.gomo.grpc.authorize.gen.AuthorizationGrpc$MethodHandlers.invoke(AuthorizationGrpc.java:689) ~[classes/:na]
	at io.grpc.stub.ServerCalls$UnaryServerCallHandler$UnaryServerCallListener.onHalfClose(ServerCalls.java:171) ~[grpc-stub-1.12.0.jar:1.12.0]
	at io.grpc.internal.ServerCallImpl$ServerStreamListenerImpl.halfClosed(ServerCallImpl.java:283) ~[grpc-core-1.12.0.jar:1.12.0]
	at io.grpc.internal.ServerImpl$JumpToApplicationThreadServerStreamListener$1HalfClosed.runInContext(ServerImpl.java:706) ~[grpc-core-1.12.0.jar:1.12.0]
	at io.grpc.internal.ContextRunnable.run(ContextRunnable.java:37) ~[grpc-core-1.12.0.jar:1.12.0]
	at io.grpc.internal.SerializingExecutor.run(SerializingExecutor.java:123) ~[grpc-core-1.12.0.jar:1.12.0]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_162]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_162]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_162]
Caused by: java.lang.reflect.InvocationTargetException: null
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_162]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_162]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_162]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_162]
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:622) ~[spring-beans-4.3.14.RELEASE.jar:4.3.14.RELEASE]
	... 12 common frames omitted
Caused by: java.lang.NullPointerException: null
	at com.gomo.grpc.authorize.gen.AbstractAuthorizeService$Account$Builder.setFirebaseToken(AbstractAuthorizeService.java:3615) ~[classes/:na]
	... 17 common frames omitted
```

   查看copyProperties方法内部，可以看到如下调用过程:
   BeanUtils.class
``` java
private static void copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties) throws BeansException {
	...
	...
	if (sourcePd != null) {
		Method readMethod = sourcePd.getReadMethod();
        if (readMethod != null && ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
        	try {
            	if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
                	readMethod.setAccessible(true);
                }

                Object value = readMethod.invoke(source);
                if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
                	writeMethod.setAccessible(true);
                }

                writeMethod.invoke(target, value);
//public AbstractAuthorizeService$Account$Builder AbstractAuthorizeService$Account$Builder.setFirebaseToken(java.lang.String)
            } catch (Throwable var15) {
            	throw new FatalBeanException("Could not copy property '" + targetPd.getName() + "' from source to target", var15);
            }
        }
	...
	...
}

```

   AbstractAuthorizeService.class
```java
public Builder setFirebaseToken(java.lang.String value) {
	if (value == null) {
    	throw new NullPointerException();
    }
  
	firebaseToken_ = value;
	onChanged();
    return this;
}
```

### 问题原因
   __由此可见，proto生成的消息类中，不允许对对象的属性赋null值，并且，生成的int32、int64对象均为普通对象。__

## 解决办法

### 1. 使用proto消息属性的默认值
   修改copyProperties()，判断拷贝，为空时不进行拷贝：
``` java
private static void copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties) throws BeansException {
	...
	...
	if (sourcePd != null) {
		Method readMethod = sourcePd.getReadMethod();
        if (readMethod != null && ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
        	try {
            	if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
                	readMethod.setAccessible(true);
                }

                Object value = readMethod.invoke(source);
				if(value == null) {
					continue;
				}
                if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
                	writeMethod.setAccessible(true);
                }

                writeMethod.invoke(target, value);
//public AbstractAuthorizeService$Account$Builder AbstractAuthorizeService$Account$Builder.setFirebaseToken(java.lang.String)
            } catch (Throwable var15) {
            	throw new FatalBeanException("Could not copy property '" + targetPd.getName() + "' from source to target", var15);
            }
        }
	...
	...
}

```
   [Proto3 Default Value](https://developers.google.com/protocol-buffers/docs/proto3#default)
	When a message is parsed, if the encoded message does not contain a particular singular element, the corresponding field in the parsed object is set to the default value for that field. These defaults are type-specific:

		For strings, the default value is the empty string.
		For bytes, the default value is empty bytes.
		For bools, the default value is false.
		For numeric types, the default value is zero.
		For enums, the default value is the first defined enum value, which must be 0.
		For message fields, the field is not set. Its exact value is language-dependent. See the generated code guide for details.
		The default value for repeated fields is empty (generally an empty list in the appropriate language).

### 2. 修改copyProperties()，自定义默认值
   修改copyProperties()方法：
``` java
private static void copyProperties(Object source, Object target, Class<?> editable, String... ignoreProperties) throws BeansException {
	...
	...
	if (sourcePd != null) {
		Method readMethod = sourcePd.getReadMethod();
        if (readMethod != null && ClassUtils.isAssignable(writeMethod.getParameterTypes()[0], readMethod.getReturnType())) {
        	try {
            	if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
                	readMethod.setAccessible(true);
                }

                Object value = readMethod.invoke(source);
	// 处理拷贝值为空的情况
                if(value == null) {
                	if(readMethod.getReturnType() == Integer.class || readMethod.getReturnType() == Long.class) {
                    	// 自定义默认值
                        value = readMethod.getReturnType().getConstructor(String.class).newInstance("0");
                    } else {
                    	value = readMethod.getReturnType().newInstance();
                    }
                }
                if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
                	writeMethod.setAccessible(true);
                }

                writeMethod.invoke(target, value);
//public AbstractAuthorizeService$Account$Builder AbstractAuthorizeService$Account$Builder.setFirebaseToken(java.lang.String)
            } catch (Throwable var15) {
            	throw new FatalBeanException("Could not copy property '" + targetPd.getName() + "' from source to target", var15);
            }
        }
	...
	...
}

```



