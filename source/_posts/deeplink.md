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
+ 坊间还有个说法，可以让微信团队将app加入白名单，接口放行。本地做一下尝试，将应用的二级域名指定为jd.com，微信中就能唤醒了。
+ 向微信申请开通第三方app唤起接口权限——launchApplication，配置、调用方式如下：
【第一步】
如果业务方当前网站的公众号 appid 和需要拉起的移动应用已经做了 unionid 打通，那么可以跳过这一步。否则需进行二者关联，即登录 https://open.weixin.qq.com/ 网站-“ 管理中心” -“ 公众帐号” ， 将当前网站做 JSSDK config 的公众号 appid， 绑定上。 绑定完成该网站的公众号 appid 后， 再在“移动应用” 标签下，创建移动应用，将需要被拉起的移动 app 也做登记。 这样， 该公众号所属的网站，就可以拉起该移动应用了。

【第二步】
网站上的 JSSDK config 代码， 需新增 beta 参数， 并赋值为 true， 这样可以注入wx.invoke 和 wx.on 方法。 wx.invoke 用于调用由页面主动调用的 JS 接口， wx.on 用于调用监听类 JS 接口。 然后将launchApplication 接口加入 jsApiList，修正后代码类似如下。
```javasscript
wx.config({
	beta: true, // 必填，开启内测接口调用，注入 wx.invoke 和 wx.on 方法
	debug: true, // 选填，开启调试模式，默认为 false 不开启
	appId: '', // 必填，公众号的唯一标识
	timestamp: , // 必填，生成签名的时间戳
	nonceStr: '', // 必填，生成签名的随机串
	signature: '',// 必填，签名，详见微信 JS-SDK 说明文档
	http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html
	jsApiList: ['*******', 'launchApplication'] // 必填，需要使用的 jsapi 列表
});
```
【第三步】
调用 launchApplication 接口，可以在 wx.invoke 中调用，亦可以在 wx.on 中调用。
代码类似如下， 具体参数用法参考下方请求参数表：

|  参数 |  必填 |  说明  |
|:----:|:-----:|--------|
| appID | 是 | 要拉起的移动应用的 appid，注册在 open.weixin.qq.com 网站上 |
| extInfo | 否 | Android 使用此参数，第三方应用自定义简单数据，微信终端会回传给第三方应用处理。 这种要求 app接入了微信的 openSDK。|
| messageExt | 否 | iOS 使用此参数，第三方应用自定义简单数据，微信终端会回传给第三方程序处理。 这种要求 app接入了微信的 openSDK。|
| parameter | 否 | iOS 下使用此参数，微信会将 appID 参数与之组合成： appid://parameter 传给系统， 第三方应用可将 appID 注册为自己的 scheme 前缀， 然后在application handleOpenUrl 回调中读取parameter。（scheme://后的内容）|

```javascript
wx.ready(function () {
	wx.invoke('launchApplication', {"appID":"wxXXXXX","extInfo":"xx","parameter":"pageid=10101"},function(res) {
		//这里是回调函数
	});
});
```
【第四步】
微信通过该接口启动第三方 app 是通过 opensdk 拉起的方式，因此第三方 app需要处理 ShowMessageFromWX.req 的微信回调， iOS 则需要将 appID 添加到第三方 app 工程所属的 plist 文件 URL types 字段。
