---
title: Tomcat启动报错：java.io.IOException:invalid constant type:18
date: 2017-09-21 21:28:29
cdn: "header-off"
header-img: "/img/java-logo.png"
tags: 
	- jdk
	- Tomcat
	- Java
---
## 问题背景
测试环境的项目部署到Tomcat启动时，由于报错启动失败，报错如下：
``` shell
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'endUserService' defined in class path resource [spring-resources-service.xml]: Cannot resolve reference to bean 'endUserService-jsf' while setting bean property 'service'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'endUserService-jsf': FactoryBean threw exception on object creation; nested exception is java.lang.ExceptionInInitializerError
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:328)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:106)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1360)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1118)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:517)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:456)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:294)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:225)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:291)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.autowireResource(CommonAnnotationBeanPostProcessor.java:443)
	at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.getResource(CommonAnnotationBeanPostProcessor.java:417)
	at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor$ResourceElement.getResourceToInject(CommonAnnotationBeanPostProcessor.java:559)
	at org.springframework.beans.factory.annotation.InjectionMetadata$InjectedElement.inject(InjectionMetadata.java:150)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:87)
	at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessPropertyValues(CommonAnnotationBeanPostProcessor.java:304)
	... 26 more
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'endUserService-jsf': FactoryBean threw exception on object creation; nested exception is java.lang.ExceptionInInitializerError
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:149)
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.getObjectFromFactoryBean(FactoryBeanRegistrySupport.java:102)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance(AbstractBeanFactory.java:1442)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:305)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:193)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:322)
	... 41 more
Caused by: java.lang.ExceptionInInitializerError
	at com.jd.jsf.gd.codec.msgpack.MsgpackUtil.<clinit>(MsgpackUtil.java:32)
	at com.jd.jsf.gd.util.CodecUtils.registryService(CodecUtils.java:61)
	at com.jd.jsf.gd.config.ConsumerConfig.refer(ConsumerConfig.java:124)
	at com.jd.jsf.gd.config.spring.ConsumerBean.getObject(ConsumerBean.java:93)
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:142)
	... 46 more
Caused by: com.jd.jsf.gd.error.JSFCodecException: Cannot compile java.util.GregorianCalendar's template : 
	at com.jd.org.msgpack.template.TemplateRegistry.buildAndRegister(TemplateRegistry.java:666)
	at com.jd.org.msgpack.template.TemplateRegistry.register(TemplateRegistry.java:197)
	at com.jd.org.msgpack.template.TemplateRegistry.registerTemplatesWhichRefersRegistry(TemplateRegistry.java:192)
	at com.jd.org.msgpack.template.TemplateRegistry.<init>(TemplateRegistry.java:93)
	at com.jd.jsf.gd.codec.msgpack.JSFMsgPack.<clinit>(JSFMsgPack.java:38)
	... 51 more
Caused by: com.jd.org.msgpack.template.builder.TemplateBuildException: Cannot compile java.util.GregorianCalendar's template : 
	at com.jd.org.msgpack.template.builder.BuildContext.build(BuildContext.java:81)
	at com.jd.org.msgpack.template.builder.DefaultBuildContext.buildTemplate(DefaultBuildContext.java:54)
	at com.jd.org.msgpack.template.builder.JavassistTemplateBuilder.buildTemplate(JavassistTemplateBuilder.java:116)
	at com.jd.org.msgpack.template.builder.AbstractTemplateBuilder.buildTemplate(AbstractTemplateBuilder.java:60)
	at com.jd.org.msgpack.template.TemplateRegistry.buildAndRegister(TemplateRegistry.java:649)
	... 55 more
Caused by: java.lang.RuntimeException: java.io.IOException: invalid constant type: 18
	at javassist.CtClassType.getClassFile2(CtClassType.java:203)
	at javassist.CtClassType.subtypeOf(CtClassType.java:303)
	at javassist.CtClassType.subtypeOf(CtClassType.java:318)
	at javassist.compiler.MemberResolver.compareSignature(MemberResolver.java:247)
	at javassist.compiler.MemberResolver.lookupMethod(MemberResolver.java:119)
	at javassist.compiler.MemberResolver.lookupMethod(MemberResolver.java:144)
	at javassist.compiler.MemberResolver.lookupMethod(MemberResolver.java:96)
	at javassist.compiler.TypeChecker.atMethodCallCore(TypeChecker.java:704)
	at javassist.compiler.TypeChecker.atNewExpr(TypeChecker.java:148)
	at javassist.compiler.ast.NewExpr.accept(NewExpr.java:72)
	at javassist.compiler.CodeGen.doTypeCheck(CodeGen.java:241)
	at javassist.compiler.CodeGen.compileExpr(CodeGen.java:228)
	at javassist.compiler.CodeGen.atThrowStmnt(CodeGen.java:627)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:364)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:350)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atIfStmnt(CodeGen.java:390)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:354)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:350)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atIfStmnt(CodeGen.java:390)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:354)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:350)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.MemberCodeGen.atTryStmnt(MemberCodeGen.java:233)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:366)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:350)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:49)
	at javassist.compiler.CodeGen.atMethodBody(CodeGen.java:291)
	at javassist.compiler.Javac.compileBody(Javac.java:222)
	at javassist.CtBehavior.setBody(CtBehavior.java:401)
	at javassist.CtBehavior.setBody(CtBehavior.java:375)
	at javassist.CtNewMethod.make(CtNewMethod.java:137)
	at com.jd.org.msgpack.template.builder.BuildContext.buildReadMethod(BuildContext.java:152)
	at com.jd.org.msgpack.template.builder.BuildContext.build(BuildContext.java:72)
	... 59 more
Caused by: java.io.IOException: invalid constant type: 18
	at javassist.bytecode.ConstPool.readOne(ConstPool.java:1027)
	at javassist.bytecode.ConstPool.read(ConstPool.java:970)
	at javassist.bytecode.ConstPool.<init>(ConstPool.java:127)
	at javassist.bytecode.ClassFile.read(ClassFile.java:716)
	at javassist.bytecode.ClassFile.<init>(ClassFile.java:103)
	at javassist.CtClassType.getClassFile2(CtClassType.java:190)
	... 97 more
```

## 报错原因
乍一看以为是spring的问题，查了半天的spring问题也没查出个所以然来，遂往下看cause，真正的问题应该是出在了这条报错上：
``` shell
Caused by: java.lang.RuntimeException: java.io.IOException: invalid constant type: 18
	at javassist.CtClassType.getClassFile2(CtClassType.java:203)
	... more
```
惯用套路Baidu、Google、StackOverFlow一番（以后这种问题真不想问百度了orz），原因是Javassist不支持java8导致的。
参考：[javassist does not work with java8 and lambda](https://issues.jboss.org/browse/JASSIST-217)

那问题又来了，Java8是从哪冒出来的？编译打包指定的jdk版本都是1.7，上传的war包都是本地用jdk1.7打好的。打包构建发布流程仔仔细细又过了一遍，终于找到了问题（内牛满面）...
![](img/build.png)
上传了war包之后，需要构建镜像，构建时使用了默认的jdk1.8，最终导致了惨剧的发生。

## Javassist是个啥？
看了下项目代码，并没有明显用到这个javassist的东西。看了下POM的继承关系，发现是ognl的jar包里依赖了Javassist。
> Javassist (JAVA programming ASSISTant) makes Java bytecode manipulation simple. It is a class library for editing bytecodes in Java; it enables Java programs to define a new class at runtime and to modify a class file when the JVM loads it. Unlike other similar bytecode editors, Javassist provides two levels of API: source level and bytecode level. If the users use the source- level API, they can edit a class file without knowledge of the specifications of the Java bytecode. The whole API is designed with only the vocabulary of the Java language. You can even specify inserted bytecode in the form of source text; Javassist compiles it on the fly. On the other hand, the bytecode-level API allows the users to directly edit a class file as other editors.

据说Javassist可以修改jar包源码，有空可以参考 [【Java】使用javaassist修改jar包](https://yq.aliyun.com/articles/53278) 学习一下。
