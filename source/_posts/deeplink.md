---
title: H5唤醒app
cdn: header-off
date: 2017-10-29 11:33:35
header-img: /img/deeplink.jpg
tags:
	- deeplink
	- H5
---
最近做了两个app唤醒的需求，可以说是一路爬着做过来的，然而爬到最后发现，结果并不讨喜，各种环境的唤醒结果有所差异，有些甚至不能唤醒（坑爹鹅）。今天来就这段时间踩的坑做个总结。
## js实现的三种唤醒方式
目前，IOS和Android通常都使用scheme（协议）的方式进行唤醒，链接规范如下：
[scheme]://[host]/[path]?[query]
### iframe
#### 实现方法
iframe的实现方式也有好几种：
+ 在html页面中写入一个不可见的iframe：
``` html
<iframe id="deeplink" src="[scheme]://[host]/[path]?[query]" frameborder="0" scrolling="no" width="0%" style="display:none"></iframe>
```
+ 监听事件中，动态向html中插入iframe节点：
``` js
	window.addEventListener("onload", function(){
		var iframe = document.createElement("iframe");
		iframe.src = srcLink;
		iframe.style.display = "none";
		document.body.appendChild(iframe);
	});//方法执行完之后会尝试打开app
```
#### 验证结果
|   浏览器    |    Android   |  IOS  | 
|:----------:|:------------:|:----:|
|   UC       |   YES     |  YES  |
| Chrome     |  NO   |   NO  |
|   QQ浏览器  |  YES   |  YES    |
|  Safari   |   ——  |   NO |
|  微信    |   NO   |  NO   |

综上，iframe的方式适用于Android系统，在IOS下，UC、qq浏览器也可以唤醒，但是在Safari大佬中无法唤醒，所以在IOS系统下不考虑使用iframe。

### window.location.href
#### 实现方法
window.location.href通常适用于事件驱动的方法中，例如：
``` js
$('#goApp').on("click", function(){
	var srcLink = "[scheme]://[host]/[path]?[query]"；
	window.location.href = srcLink;	
});
```
``` js
window.onload = function(){
	window.location.href = 'toplife://topLifeCommodityDetail?skuId=5474432';
}
```
同样的，在a标签中直接设置href的链接，效果也是一样的：
``` html
<a class="goApp" href="[scheme]://[host]/[path]?[query]" />
```

#### 验证结果
|   浏览器    |    Android   |  IOS  | 
|:----------:|:------------:|:----:|
|   UC       |   YES     |  YES  |
| Chrome     |  YES   |   YES  |
|   QQ浏览器  |  YES   |  YES    |
|  Safari   |   —  |   YES |
|  微信    |   NO   |  NO   |

## 结论分析
1. 对于IOS来说，很明显的，location.href的方式更合适，主要是为了Safari；
2. Android系统中，若需要进入页面就直接唤醒，iframe和location.href都可以；而事件驱动的唤醒，则选用iframe。

## 关于坑鹅（微信）
由于微信将唤起本地APP的接口给禁了，所以原则上微信中是不能直接唤起APP的。但也有例外：
+ 在安卓系统中，只有机器上安装了应用宝，并将跳转链接跳到应用宝对应app的下载页面才能实现唤起（亲儿子的待遇）；
+ 而IOS，在IOS9以后的系统中，据说可以通过配置通用链接（Universal Links）实现微信中的唤醒，没试过，不做评价。
+ 坊间还有个说法，可以让微信团队将app加入白名单，接口放行。