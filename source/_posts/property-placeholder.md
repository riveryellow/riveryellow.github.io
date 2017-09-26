---
title: 通过PropertyPlaceholderConfigurer配置不同环境配置
date: 2017-09-25 14:46:15
cdn: "header-off"
header-img: "./img/configuration.jpg"
tags:
	- Spring
---
> 项目中，本地调试、测试环境、预发布环境、线上，不同环境有时需要不同的配置文件。原来的项目中，可以通过公共的配置管理服务为不同的环境在线配置不同的配置文件，然后在构建或服务器启动时去获取。
> 现在没有了这样的公共服务，将环境配置硬编码到代码中显然是不现实的。在Spring中，可以通过扩展PropertyPlaceholderConfigurer（bean factory post processor）来实现不同环境读取不同配置。

## 硬编码实现
java代码
``` java
public class UserServiceImpl implements UserService {
    private String couponSystem;
    private String couponLanguage;
	private String jpeaKey;
	...
	...
	public void setCouponSystem(String couponSystem) {
        this.couponSystem = couponSystem;
    }
    public void setCouponLanguage(String couponLanguage) {
        this.couponLanguage = couponLanguage;
    }
    public void setJpeaKey(String jpeaKey) {
        this.jpeaKey = jpeaKey;
    }
}
```
spring配置
``` xml
<bean id="userService" class="com.demo.service.impl.UserServiceImpl">
	<property name="couponSystem" value="H5"></property>
	<property name="couponLanguage" value="zh-CN"></property>
	<property name="jpeaKey" value="e3b95cec784745"></property>
</bean>
```
通过setter方法，spring将配置注入到类的属性中。但是如果环境变化，需要不同的配置值，就要通过修改代码来实现，非常不方便。

## 扩展PropertyPlaceholderConfigurer实现

### 使用PropertyPlaceholderConfigurer读取.properties配置文件
env-local.properties
```
coupon.config.system=H5
coupon.config.language=zh-CN
jpea.config.key=e3b95cec784745
```
在spring的配置文件中可以做如下修改：
``` xml
<bean id="propertyConfigurer" class="com.demo.tools.PropertyPlaceholderConfigurer">
	<property name="locations">
		<list>
			<value>classpath:env-local.properties</value>
		</list>
	</property>
</bean>
<bean id="userService" class="com.demo.service.impl.UserServiceImpl">
	<property name="couponSystem" value="${coupon.config.system}"></property>
	<property name="couponLanguage" value="${coupon.config.language}"></property>
	<property name="jpeaKey" value="${jpea.config.key}"></property>
</bean>
```
PropertyPlaceholderConfigurer通过Spring IoC容器去加载.properties配置文件，然后再使用通配符${...}通过java代码中的setter方法将.properties文件中的配置注入到类的属性中。
当然，也可以使用spring注解：
``` java
public class UserServiceImpl implements UserService {
	@Value("coupon.config.system")
    private String couponSystem;
	@Value("coupon.config.language")
    private String couponLanguage;
	@Value("jpea.config.key")
	private String jpeaKey
}
```

### 扩展PropertyPlaceholderConfigurer获取不同环境下的配置
+ 在classpath下配置各个环境所需要的配置文件：env-test.properties、env-stg.properties/env-prod.properties.
+ Spring配置文件中加入动态获取配置文件设置：
``` xml
<bean id="propertyConfigurer" class="com.demo.tools.PropertyPlaceholderConfigurer">
	<property name="locations">
		<list>
			<!-- 默认test -->
			<value>classpath:env-${run_env:test}.properties</value>
		</list>
	</property>
</bean>
```
+ 配置tomcat启动参数：
	+ 本地可以在Tomcat配置中的"VM options"中加入参数 -Drun_env="test"
	+ 远程服务器上，可以在启动脚本中（如：catalina.sh，当前容器中使用的是default_tomcat_env.sh）加入语句 export run_env="stg"
+ 服务器启动，就会根据启动参数获取不同的环境所需的配置。

## 忽略PlaceHolder Exception
当properties文件中不包含所引用的属性时，会产生如下报错：
``` console
Exception in thread "" org.springframework.beans.factory.BeanDefinitionStoreException: Invalid bean definition with name "userService" in class path resource [spring-resource-base.xml]:
	...
	...
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'jpea.config.value' in string value "${jpea.config.value}"
	...
	...
```
如果不想看到报错，可以在Spring配置文件中对propertyConfigurer做如下修改：
``` xml
<bean id="propertyConfigurer" class="com.demo.tools.PropertyPlaceholderConfigurer">
	<property name="locations">
		<list>
			<value>classpath:env-local.properties</value>
		</list>
	</property>
	<property name="ignoreUnresolvablePlaceholders" value="true"/>
</bean>
```