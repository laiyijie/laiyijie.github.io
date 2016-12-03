---
title: 番外 02： Spring 之使用 JAVA 操作Mysql数据库（为何要用ORM）Spring整合 Mybatis前基础
date: 2016-12-02 20:50:32
categories: 
- Spring-fw
tags:
- learning
---

## 前景概要  

在**08 Spring 操作持久层 （融合 Mybatis）最简使用（使用 Mybatis Generator）** 对外依赖过大，对新手来说可能有跳跃性。  
特写此章做一下铺垫。  

<!-- more -->
## ORM的作用及Spring对数据库的优化  

现在我们都会看到网上流行各种ORM框架来操作数据库，例如`Mybatis`和`hibernate`等，那到底为何要用ORM框架呢？  
一开始的时候是没有ORM存在的，那么使用JAVA操作数据库步骤应该如下：  

### 原始方式操作数据库（直接获取数据库连接池） 

#### App.java  
	
	package me.laiyijie.demo;
	
	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	
	public class App {
	
		public static void main(String[] args) throws SQLException, ClassNotFoundException {
	
			Class.forName("com.mysql.jdbc.Driver");
			Connection conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1/myspring", "test", "o34rWayJPPHgudtL");
			ResultSet rs = conn.createStatement().executeQuery("select * from account");
	
			while (rs.next()) {
				System.out.println("username: " + rs.getString("username") + " password: " + rs.getString("password") + " name: "
						+ rs.getString("name") + " create_time: " + rs.getLong("createtime"));
			}
			conn.close();
		}
	}

步骤详解：  

- 加载数据库驱动  
> Class.forName("com.mysql.jdbc.Driver");  

- 获取数据库连接  
> Connection conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1/myspring", "test", "o34rWayJPPHgudtL");  

- 执行查询语句并获取结果集  
> ResultSet rs = conn.createStatement().executeQuery("select * from account");  

- 输出结果  
> while (rs.next()) {  
> 	System.out.println("username: " + rs.getString("username") + " password: " + rs.getString("password") + " name: "
> 			+ rs.getString("name") + " create_time: " + rs.getLong("create_time"));  
> }

- 关闭连接  
> conn.close();  

这种操作方式有几个弊端：

1. 需要自己管理数据库连接（又要开启又要关闭的，万一没关就泄露了，而且还没有线程池，效率堪忧）  
2. 每次都要自己组装SQL语句，很容易出错
3. 不是面向对象的方式，每次操作的时候都是要通过`getString`来获取，需要对数据库非常熟悉才行，也就是说耦合很深！


那么我们就来陆续解决这几个问题：  

### 问题解决及优化  
#### 解决数据库连接管理问题  

解决方式是引入`数据库连接池`管理数据库连接。  

在**08 Spring 操作持久层 （融合 Mybatis）最简使用（使用 Mybatis Generator）** 我们引入了：  

	<dependency>
		<groupId>commons-dbcp</groupId>
		<artifactId>commons-dbcp</artifactId>
		<version>1.4</version>
	</dependency>

这个依赖中的 `org.apache.commons.dbcp.BasicDataSource`就是数据库连接池的一个实现，可以有效管理数据库连接，对这类数据库连接资源我们称之为**数据源**，在Spring中的配置如下：  

	<bean id="mysqlDataSource" class="org.apache.commons.dbcp.BasicDataSource"
		p:driverClassName="com.mysql.jdbc.Driver"
		p:url="jdbc:mysql://127.0.0.1:3306/myspring"
		p:username="test" p:password="o34rWayJPPHgudtL" />  
也就是说，我们可以通过： 

	@Autowired
	private BasicDataSource basicDataSource;

	public void test() throws SQLException {
		Connection conn = basicDataSource.getConnection();
	}  

直接获取数据库连接，而不需要关心连接的关闭等问题！  

#### 解决需要一直直接写SQL和与对象映射问题（ORM出现）  

##### 对象映射  
想象现在如果从数据库取出的数据直接是一个对象！那多好！就不用重复写`rs.getString("uesrname")`这种既需要知道类型、又需要知道字段名称的**重复语句**，而是可以直接这样写`Account.getUsername()`。  
这样简直棒呆！  
而`Mybatis`正是为我们做了这样一件事情。  
**08 Spring 操作持久层** 这一章中，`MybatisGenerator`为我们做了如下的事情：  

- 创建数据表对应的对象（与数据表完全一样）  
		
		public class Account {
		    private String username;
		
		    private String password;
		
		    private String name;
		
		    private Long create_time;
		    
		}   

- 查询后将结果与对象映射（也就是查询完成直接返回对象）

		public interface AccountMapper {
		    long countByExample(AccountExample example);
		
		    int deleteByExample(AccountExample example);
		
		    int deleteByPrimaryKey(String username);
		
		    int insert(Account record);
		
		    int insertSelective(Account record);
		
		    List<Account> selectByExample(AccountExample example);
		
		    Account selectByPrimaryKey(String username);
		
		    int updateByExampleSelective(@Param("record") Account record, @Param("example") AccountExample example);
		
		    int updateByExample(@Param("record") Account record, @Param("example") AccountExample example);
		
		    int updateByPrimaryKeySelective(Account record);
		
		    int updateByPrimaryKey(Account record);
		}   
- 不用直接组装SQL语句  

		public List<Account> getAccountsByCreateTime(Long start, Long end) {
			AccountExample accountExample = new AccountExample();
			accountExample.or().andCreate_timeGreaterThan(start).andCreate_timeLessThanOrEqualTo(end);
			return accountMapper.selectByExample(accountExample);
		}   

有这么好的利器，为何不用？  

