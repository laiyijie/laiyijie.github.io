---
title: 04 Spring 注解配置，简化xml配置，@Service、@Autowired简介
date: 2016-11-26 14:10:29
categories: 
- Spring
tags:
- learning
---
转载请注明来源 [赖赖的博客](http://laiyijie.me)
## 导语

> 简单的不一定是好的，复杂的也不一定是好的，合适的才是最好的。

通过增加Spring注解从而简化Xml配置，也简单介绍Java注解，并提供实例。

建议先了解Spring ApplicationContext的xml配置方式。可参考**03 Spring IoC之对象间的相互关系和 Spring 应用结构**

<!-- more -->

## 实例

如果看过 **03 Spring IoC之对象间的相互关系和 Spring 应用结构** 可以跳到**项目详解**这一节，因为结构和输出结构与上一节一致。

### 项目工程目录结构和代码获取地址

#### 获取地址（版本Log将会注明每一个版本对应的课程）
https://github.com/laiyijie/SpringLearning

#### 目录结构

![目录结构](spring-04/sayhello.png)

#### 运行工程
运行具有Main函数的 App.java
得到如下输出
> false

#### 工程简单分层（如果看过 **03 Spring IoC之对象间的相互关系和 Spring 应用结构** 可以略过到下一节）
现在工程有三个包，包之间的关系如下：

- 应用层：`me.laiyijie.demo` , 只调用**服务层**
- 服务层：`me.laiyijie.demo.service`，只调用**数据层**
- 数据层：`me.laiyijie.demo.dataaccess`，最底层，直接操作持久化数据（文件、数据库等）


### 项目详解

从App.java入手

#### App.java（与上一节比无变化）

	package me.laiyijie.demo;
	
	import org.springframework.context.support.ClassPathXmlApplicationContext;
	
	import me.laiyijie.demo.service.UserService;

	public class App {
		public static void main(String[] args) {
			ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("root-context.xml");
			
			UserService userService = context.getBean(UserService.class);
			
			System.out.println(userService.login("lailai", "laiyijie","127.0.0.1"));
			
			context.close();
		}
	}

Main函数中依旧只有四行代码，这里引用到了`UserService`，并且调用了`UserService`的`login`方法。

我们来看下`UserService`

#### UserService.java（与上一节比无变化）
	
	package me.laiyijie.demo.service;
	
	public interface UserService{
		
		boolean login(String username,String password,String ip);
		
	}

OK，非常简单，只是一个接口，定义了有这么一个方法，具体实现是在同一个包下面的另一个文件，UserServiceImpl.java中

#### UserServiceImpl.java（变化来了！）
	
	package me.laiyijie.demo.service;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;
	
	import me.laiyijie.demo.dataaccess.AccountAccess;
	import me.laiyijie.demo.dataaccess.LoginLogAccess;
	
	@Service
	public class UserServiceImpl implements UserService {
	
		@Autowired
		private AccountAccess accountAccess;
		@Autowired
		private LoginLogAccess loginLogAccess;
		
		public boolean login(String username, String password,String ip) {
			
			if (!accountAccess.isAccountExist(username)) {
				return false;
			}
			
			if (accountAccess.isPasswordRight(username, password)) {
				accountAccess.updateLastLoginTime(username);
				loginLogAccess.addLoginLog(username, ip);
				return true;
			}
			return false;
		}
	}

在这里**新增了两个注解** @Service、@Autowired ，**减少了getter和setter**

- @Service 在类前注释，将此类实例化，等同于在root-context中配置

		<bean class="me.laiyijie.demo.service.UserServiceImpl">
		</bean>
- @Autowired 在属性前配置，将属性对应的对象从工厂中取出并注入到该bean中，等同于

		<property ref="accountAccessImpl" name="accountAccess"></property>
		<property ref="loginLogAccessImpl" name="loginLogAccess"></property>

两个注解组合起来等同于：

	<bean class="me.laiyijie.demo.service.UserServiceImpl">
		<property ref="accountAccessImpl" name="accountAccess"></property>
		<property ref="loginLogAccessImpl" name="loginLogAccess"></property>
	</bean>

那么，让我们看一下root-context.xml是否可以去除掉这一部分的配置！

#### root-context.xml（有变化）

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
		<bean id="accountAccessImpl" class="me.laiyijie.demo.dataaccess.AccountAccessImpl"></bean>
		<bean id="loginLogAccessImpl" class="me.laiyijie.demo.dataaccess.LoginLogAccessImpl"></bean>
		
		<context:component-scan base-package="me.laiyijie.demo.service"></context:component-scan>
	
	</beans>

yes！你没看错，配置`me.laiyijie.demo.service.UserServiceImpl`这个bean的代码被两个注解代替了，而新增了一行：

	<context:component-scan base-package="me.laiyijie.demo.service"></context:component-scan>

从文字直译就可以看出，这一行代表了

- 从`me.laiyijie.demo.service`包下面扫描 `componet`

而@Service正是`componet` 中的一种

#### 进一步简化

是否可以更进一步简化root-context.xml的配置？！因为还有两个bean是用xml配置的！！

当然可以，通过如下步骤

1. 删除root-context.xml中的两行bean配置
2. 在AccountAccessImpl.java和LoginLogAccess.java的类定义前增加@Service注解
3. 修改root-context.xml 中componet-scan的 base-package属性为`me.laiyijie.demo`，如下所示

		<context:component-scan base-package="me.laiyijie.demo"></context:component-scan>

### 小结：

1. @Service可以将一个类定义成一个bean（也就是实例化并放入工厂）
2. @Autowired 可以根据**属性的类型**来注入工厂中存在的该类型的实例
3. 要使用注解，需要在ApplicationContext中增加 `<context:component-scan base-package="me.laiyijie.demo"></context:component-scan>`配置


