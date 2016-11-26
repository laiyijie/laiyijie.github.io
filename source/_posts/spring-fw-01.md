---
title: 番外：Spring IoC 实现原理简析，Java的反射机制，通过类名创建对象
date: 2016-11-26 12:59:26
categories: 
- Spring-fw
tags:
- learning
---

## 前景概要

在 **01 走进Spring，Context、Bean和IoC** 中，我们看到了强大的Spring通过ApplicationContext实现了bean工厂（也就是对象工厂），那究竟是怎么实现的呢，本次给大家写一个小Demo展现其原理；

<!-- more -->

### Spring bean的调用方式和配置方式
（详情可以查看 **01 走进Spring，Context、Bean和IoC** 这一课程）此处仅贴出代码，如果已经看过 **01 走进Spring，Context、Bean和IoC** 这一课，可以直接跳过前景概要。


运行App.java输出结果如下

> hello

#### App.java


	package me.laiyijie.demo;
	
	import org.springframework.context.support.ClassPathXmlApplicationContext;
	import me.laiyijie.demo.service.AccountService;
	
	/**
	 * Hello
	 *
	 */
	public class App {
		public static void main(String[] args) {
			ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("root-context.xml");
	
			AccountService accountService = context.getBean(AccountService.class);
	
			System.out.println(accountService.sayHello());
	
			context.close();
		}
	}

#### root-context.xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
		<bean class="me.laiyijie.demo.service.AccountService"></bean>
	
	</beans>

#### AccountService.java

	package me.laiyijie.demo.service;
	
	public class AccountService {
	
		public String sayHello() {
	
			return "hello";
		}
	}


#### pom.xml代码如下

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>
	
		<groupId>me.laiyijie</groupId>
		<artifactId>demo</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>jar</packaging>
	
		<dependencies>
		
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context</artifactId>
				<version>4.3.2.RELEASE</version>
			</dependency>
	
		</dependencies>
	</project>


## 通过类名创建对象

### 问题引出和分析

在App.java中，调出AccountService用了两步：

	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("root-context.xml");

	AccountService accountService = context.getBean(AccountService.class);

而root-context.xml中配置AccountService实例只用了一行配置

	<bean class="me.laiyijie.demo.service.AccountService"></bean>

那么问题来了，Spring是如何通过这一行配置文件来创建这个对象的呢？

此处配置文件只提供了一个类的全限定名`me.laiyijie.demo.service.AccountService`,那么问题就转化成：
> 如何通过类名创建对象

### 问题解决

要解决通过类名创建对象的问题就要引入Java的反射机制。

#### 实例：通过类名创建对象
	
##### App.java

	package me.laiyijie.demo;
	
	import java.lang.reflect.Constructor;
	import java.lang.reflect.InvocationTargetException;
	
	import me.laiyijie.demo.service.AccountService;
	
	public class App {
		public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, SecurityException,
				InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
			ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
	
			Class clazz = classLoader.loadClass("me.laiyijie.demo.service.AccountService");
	
			Constructor constructor = clazz.getConstructor();
	
			AccountService accountService = (AccountService) constructor.newInstance();
	
			System.out.println(accountService.sayHello());
		}
	}
	
没有用到 `new AccountService` 却成功创建了他的对象并调用其方法。

##### 调用过程如下

1. 通过Thread获取当前的类加载器（`ClassLoader`）
2. 通过`ClassLoader`获取`me.laiyijie.demo.service.AccountService`对应的`Class`对象
3. 通过`Class`对象获取构造函数对应的`Constructor`的对象
4. 通过`Contructor`对象创建`AccountService`对象
5. 调用`sayHello`方法

> hello


