---
title: b00 写在 Spring Boot 学习之前
date: 2017-07-19 14:30:17
categories: 
- Spring
tags:
- learning
---
转载请注明来源 [赖赖的博客](http://laiyijie.me)

## 导语 

> 心急吃不了热豆腐，先应用后原理不是不可以，但是一定不要停在应用上


在前面的一些部分介绍了Spring Framwork的一些基本概念（IoC AOP）和基本的配置，以及其一些基本原理。那些是学习Spring Boot的基础，你有必要知道前面的一些基础知识，有利于使用Spring Boot，如果没有Spring的基础，建议可以先学习前面的课程。

后面以bxx开头的文章将介绍Spring Boot

<!-- more -->


## 一、学习Spring之前所需要的一些基础
	1. 熟悉Java语言（推荐 java编程思想）
	2. 熟悉Maven （推荐 MAVEN实战）
	3. 熟悉Spring IoC，AOP等基本概念

## 二、如何学习Spring Boot

### 2.1 Spring Boot 的出现为了解决什么问题

在之前使用Spring的时候需要非常多的配置，导致Spring的开发非常的繁琐，有大量的配置需要开发的时候进行配置，过于繁重

因此**为了解决繁重的配置问题**Spring Boot出现了，Spring Boot的核心思想是**约定优先配置**，也是效仿Ruby On Rails的核心理念

简单的说，就是**给Spring 配置了很多默认的配置以满足绝大部分程序的需要**，这样就可以减轻程序员繁重的配置工作。

### 2.2 核心需要抓住理解的地方

1. Spring Boot AutoConfig （自动配置是个啥？怎么用？怎么跑的？）
2. 默认配置是什么？如何修改默认配置？
3. Spring Boot Starter（如何快速开启项目？如何使用Spring？）

## 三、学习资料&参考文档

Spring Boot[官方文档](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)