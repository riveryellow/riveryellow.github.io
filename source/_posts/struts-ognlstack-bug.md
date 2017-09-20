---
title: struts升级至2.3.20后无法使用stack.findValue调用java静态方法
date: 2017-09-20 20:56:43
cdn: "header-off"
header-img: "./img/struts-logo.png"
tags:
	- Struts
---
## 问题背景
项目融合之前，struts2一直使用的是2.0.11版本，前端页面使用freemarker模板引擎，通过值栈调用后台的Java静态方法。具体场景如下，
拦截器向值栈中存入多个键值对：
``` java
public class WebPageInterceptor extends AbstractInterceptor{
	private static final long serialVersionUID = 7150874893323446302L;
	@Override
	public String intercept(ActionInvocation invocation) throws Exception {

		String headType = WebPageUtils.getHeadType(invocation);		
		//默认为base头
		if (StringUtils.isEmpty(headType)) {
			headType = "base";
		}
		//PreResultListener是个好东西！
		invocation.addPreResultListener(new PageInfoPreResultListener());
		/**
		 * ...
		 * ...
		 * 中间还有很多操作
		 */	
		HashMap<String, Object> vars = new HashMap<String, Object>();
		vars.put("headerType", headType);
		vars.put("GlobalConfigFactory", ConfigFactory.getInstance());
		/**
		 * ...
		 * ...
		 * vars里面还有很多东西
		 */	
		ValueStack stack = invocation.getStack();
		stack.push(vars);
		return invocation.invoke();
	}

}
```
前台通过freemarker既获取值栈中的值，又通过值栈调用栈外的静态方法：
``` js
<#-- 值栈外静态方法调用 -->
var globalCoinFlag = "${(stack.findValue("@com.front.global.hessian.adaptor.GlobalConfigInfoFactoryAdaptor@getInstance()").getGlobalConfigInfo('globalCoinFlag').getExtendInfo())!'0'}";
<#-- 使用值栈中的参数 -->
var globalCouponPopupFlag = "${(GlobalConfigFactory.getConfig("globalSwitch","userCouponPopupFlag").getExtendInfo())!'0'}";

```
融合项目开始之后，struts要求升级到2.3.20，那么问题就来了，原先在模板引擎中通过值栈调用值栈外Java静态方法取值的变量统统无法取到值，WHY？怎么办？

## 官方漏洞修补WW-4429
百度、Google、stackoverflow了一番（不得不说，这种事情还是问墙外的大大比较管用），原来，使用值栈调用值栈之外的java静态方法实际上是一种很不安全的bug，struts在2.3.20已经修复了这个bug，即使在struts配置文件中设置：
``` xml
<constant name="struts.ognl.allowStaticMethodAccess" value="true"/>
```
``` js
<#-- not working -->
var value = ${stack.findValue("@com.your.full.package.Classname@methodName(optionalParameters)")}
<#-- working -->
var day = ${stack.findValue("@java.util.Calendar@DAY_OF_WEEK")}
```
也只能获取静态敞亮，而不能调用静态方法。
详见：[WW-4429](https://issues.apache.org/jira/browse/WW-4429) [WW-4348](https://issues.apache.org/jira/browse/WW-4348)

## 解决办法
将需要调用的静态方法事先存储到值栈中，通过模板引擎直接通过值栈调用即可。