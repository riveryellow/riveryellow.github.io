---
title: 待探究问题清单
date: 2017-09-08 22:36:49
cdn: "header-off"
header-img: "./img/issue.jpeg"
---
## [2017.9.4 hexo正确使用方式](http://riveryellow.github.io/2017/09/10/buid-blog/)
```
通过hexo发布blog至github时，发布了整个文件脚手架到github上，
导致访问404(npm install hexo-deployer-git --save)，每次删除原来脚手架重新拉取时都要安装发布插件
若默认发布分支上有非web结构的目录结构，要先清空再发布
```


## 2017.9.4 新版Struts2调用java静态方法
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

