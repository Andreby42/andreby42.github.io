---
title: Mybatis源码分析(一)
date: 2018-03-10 13:10:07
tags: [转载]
categories: [Mybatis]
---

​	Mybatis源码分析<!--more-->

## 核心接口（对象）


#### sqlSessionFactory 对象

sqlSessionFactory 是 mybatis 核心配置类，管理 mybatis 全局配置。

#### sqlSession 接口

一个或者多个sql操作的执行单元。实现一个完整的sql操作。依赖sqlSessionFactory创建

#### Executor 接口

对应 jdbc 底层一个完整的 sql 操作。sqlSession 实际通过 Executor 执行 sql 操作

#### MapperProxy 类

代理实现 mybatis 的客户端 mapper 接口

#### MapperMethod 类

对应客户端代码的 mapper 接口里面的一个方法。该实例缓存了。

#### StatementHandler 接口

jdbc Statement 的装饰器

#### ResultSetHandler 接口

jdbc resultSet 的装饰器

## mybaits 执行 orm 操作细节

![mybaits执行orm操作细节](Mybatis源码分析-一/20170308.png)

**sqlsession 的 close 方法会关闭底层 jdbc connection**

## mybatis 插件机制

mybatis 插件是基于代理实现的，具体支持一下四个接口

- Executor
- ParameterHandler


- ResultSetHandler


- StatemetHandler


[]: http://www.chenxun.wiki/2017/03/01/mybatis-01/

