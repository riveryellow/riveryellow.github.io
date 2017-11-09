---
title: Spring bean的装配方案
cdn: header-off
date: 2017-11-09 15:53:26
header-img: "/img/spring.png"
tags:
	- Spring
	- Java
---
> Spring容器负责创建应用程序中的bean并通过DI来协调这些对象之间的关系。 但是， 作为开发人员，需要告诉Spring要创建哪些bean并且如何将其装配在一起。 当描述bean如何进行装配时， Spring具有非常大的灵活性， 它提供了三种主要的装配机制：
+ 在XML中进行显示配置
+ 在Java代码中进行显示配置
+ 隐式的bean发现机制和自动装配

## 自动化装配bean

## 通过Java代码装配bean

## 通过XML装配bean

## 导入和混合配置

## Summary
Spring的配置风格是可以互相搭配的，所以你可以选择使用XML装配一些bean，使用Spring基于Java的配置（JavaConfig）来装配另一些bean， 而将剩余的bean让Spring去自动发现。
即便如此，还是尽可能地使用自动配置的机制。 显式配置越少越好。 当你必须要显式配置bean的时候（ 比如，有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），我推荐使用类型安全并且比XML更加强大的JavaConfig。 最后，只有当你想要使用便利的XML命名空间， 并且在JavaConfig中没有同样的实现时，才应该使用XML。