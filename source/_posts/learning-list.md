---
title: 待探究问题清单
date: 2018-06-7 22:36:49
cdn: "header-off"
header-img: "/img/issue.jpeg"
---
## [2017.9.4 hexo正确使用方式](http://riveryellow.github.io/2017/09/10/buid-blog/)
```
通过hexo发布blog至github时，发布了整个文件脚手架到github上，
导致访问404(npm install hexo-deployer-git --save)，每次删除原来脚手架重新拉取时都要安装发布插件
若默认发布分支上有非web结构的目录结构，要先清空再发布
```


## [2017.9.4 新版Struts2调用java静态方法](https://riveryellow.github.io/2017/09/20/struts-ognlstack-bug/)
```
项目升级struts2，从2.1.0.2升级到2.3.0.15后，freemarker中
  stack.findValue("@com.aa.bb.classC@getJSONValue()")
返回为null。
```

## 2017.9.4 Struts为何禁止通过值栈调用外部静态方法
```
学习OGNL、struts值栈的实现原理
http://struts.apache.org/docs/s2-009.html
```

## 2017.9.5 fetch如何模仿jsonp实现跨域请求
```
项目重构改造，为简化代码，将原有jsonp请求通过fetch获取，遇到报错如下：

>  fetch("//cart.yhd.com/cart/info/minicart.do")
<· Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
·  Fetch API cannot load http://cart.yhd.com/cart/info/minicart.do. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.yhd.com' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
·  Uncaught (in promise) TypeError: Failed to fetch
```

## 2017.9.7 jQuery call() 、apply()方法学习
```
融合项目中出现很多callback.call()、callback.apply()(dom.apply()?)的调用方式，搞清楚为什么
CitySelected ——> addToCartNew()
```

## 2017.9.8 埋点系统学习——JD子午线、JD通天塔、Google Tag Manager
```
当前需求：希望将订单中的商品归类到各个活动中去。
痛点：由于商品加车大多数在详情页，根据当前的打点规则，很难简单的追溯商品的活动来源。
```

## 2017.9.9 jquery Object添加元素
```
如何向已存在的jquery对象中新增元素，push？obj[]?
```

## 2017.9.13 如何正确使用github开源代码--fork
```
用hexo搭建了github pages之后，每次拉取最新的scaffolds后，
hexo generate后都会出现WARN NO LAYOUT的警告，hexo server之
后页面空白。仔细一看，clone到本地的Anisina主题无法提交到自己的
github。
```
参考：[如何把别人项目代码修改后 提交到github](http://yijiebuyi.com/blog/9c00641126e41779ef38cafb9c6aad67.html)

## 2017.9.14 struts如何引用jar包中的struts配置
```
融合项目中，服务引用了jar包中的错误页面struts配置，访问时却无法访问到action指向页面，
状态码200，页面空白，后台无报错。
是否和引用jar包中的struts配置有关？
```

## [2017.9.22 如何使用spring配置在不同环境获取不同的配置文件](https://riveryellow.github.io/2017/09/25/property-placeholder/)
```
开发时，往往本地、测试、预发布、线上使用的一些系统参数都是不一样的。
参考：GlobalPropertyConfigurer
```

## 2017.9.25 图片懒加载
```
简单实现图片懒加载，使页面上的图片能够按需加载，降低页面加载时间。
参考： lazyImg
```

## 2017.9.28 Spring Security单点登录实现
```
今天被要求给后台系统做一个简单的登陆，于是使用了spring security。
搞清楚原理：https://docs.spring.io/spring-security/site/docs/current/guides/html5//helloworld-javaconfig.html#security-config-java
```

## 2017.10.1 浏览器url缓存机制及JSONP使用缓存实例（cache-control）
```
对于返回结果变动不大的请求，可以使用浏览器的缓存机制。
参考：getCityById、JsonpInterceptor
```

## 2017.10.9 sitemesh以及decorate模式
```
学习sitemesh的配置以及实现
```

## 2017.10.10 ABTEST分桶实现
```
回顾ABTEST的分桶实现，以及后台新增AB实验后的消息通知机制（Zookeeper）
```

## 2017.10.10 jsonp请求的返回值中出现"$ref": "$.objCitys.T[0]"
```
在jsonp请求后返回的结果中，出现了"$ref": "$.objCitys.T[0]"，导致解析成undefined.
原因：大概是后台返回的数组中有重复的元素
探究fastjson如何解析json对象
```

## 2017.10.12 Spring boot
```
熟悉新项目框架（特性）
```

## 2017.10.12 OpenResty
```
熟悉新项目技术
```

## 2017.10.16 groovy基础学习
Groovy是一种基于JVM（Java虚拟机）的敏捷开发语言，它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。由于其运行在 JVM 上的特性，Groovy 可以使用其他 Java 语言编写的库。
很厉害的样子，学习一下也无妨。

## 2017.10.16 js自动混淆实现
```
当本地修改js时，不需要重新package就可获取到新的混淆js
参考：MergeStaticServlet
```

## 2018.1.3 spring BeanUtil使用方法及原理学习
```
今天做网关接口时，用到了spring的BeanUtil进行对象拷贝。
学习其原理。（顺带学习对象vo的遍历）
```

## 2018.1.9 @Autowired 与 @Resouce的区别

## 2018.1.9 Spring3.1与Tomcat8兼容性问题
```
Spring3.1与tomcat8组合时，@RequestMapping无法完成链接映射
```

## 2018.6.12 HikariCP连接池学习(与Spring整合)

