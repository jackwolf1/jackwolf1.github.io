---
layout: post
title: Spring Boot中使用JdbcTemplate访问数据库
tags:
- SpringBoot
categories: SpringBoot
description: SpringBoot
---

之前介绍了很多Web层的例子，包括构建RESTful API、使用Thymeleaf模板引擎渲染Web视图，但是这些内容还不足以构建一个动态的应用。通常我们做App也好，做Web应用也好，都需要内容，而内容通常存储于各种类型的数据库，服务端在接收到访问请求之后需要访问数据库获取并处理成展现给用户使用的数据形式。

本文介绍在Spring Boot基础下配置数据源和通过JdbcTemplate编写数据访问的示例。


# 数据源配置 #

在我们访问数据库的时候，需要先配置一个数据源，下面分别介绍一下几种不同的数据库配置方式。

首先，为了连接数据库需要引入jdbc支持，在pom.xml中引入如下配置：
{% highlight null %}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
{% endhighlight %}

## 嵌入式数据库支持 ##
（不需要使用）

嵌入式数据库通常用于开发和测试环境，不推荐用于生产环境。Spring Boot提供自动配置的嵌入式数据库有H2、HSQL、Derby，你不需要提供任何连接配置就能使用。

比如，我们可以在pom.xml中引入如下配置使用HSQL
{% highlight null %}
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
{% endhighlight %}

## 连接生产数据源 ##

以MySQL数据库为例，先引入MySQL连接的依赖包，在pom.xml中加入：


{% highlight null %}
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>runtime</version>
</dependency>
{% endhighlight %}


在src/main/resources/application.properties中配置数据源信息
  
{% highlight null %}
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
{% endhighlight %}  

可以使用application.yml进行配置，代码如下：

{% highlight null %}
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&autoReconnect=true&failOverReadOnly=false&autoReconnectForPools=true
    username: root
    password: 1234567890
    tomcat:
      max-active: 100
      initial-size: 5
      max-idle: 10
      min-idle: 5
      test-while-idle: true
      test-on-borrow: true
      validation-query: SELECT 1
      time-between-eviction-runs-millis: 5000
      min-evictable-idle-time-millis: 60000
{% endhighlight %}  

## 连接JNDI数据源 ##

当你将应用部署于应用服务器上的时候想让数据源由应用服务器管理，那么可以使用如下配置方式引入JNDI数据源。

{% highlight null %}

spring.datasource.jndi-name=java:jboss/datasources/customer

{% endhighlight %}  

## 使用JdbcTemplate操作数据库 ##

Spring的JdbcTemplate是自动配置的，你可以直接使用@Autowired来注入到你自己的bean中来使用。

举例：我们在创建User表，包含属性name、age，下面来编写数据访问对象和单元测试用例。

定义包含有插入、删除、查询的抽象接口UserService
{% highlight java %}
package com.xuying.demo.service;

public interface UserService {
	/**
	 * 新增一个用户
	 * 
	 * @param name
	 * @param age
	 */
	void create(String name, Integer age);

	/**
	 * 根据name删除一个用户高
	 * 
	 * @param name
	 */
	void deleteByName(String name);

	/**
	 * 获取用户总量
	 */
	Integer getAllUsers();

	/**
	 * 删除所有用户
	 */
	void deleteAllUsers();
}
{% endhighlight %}  

通过JdbcTemplate实现UserService中定义的数据访问操作

{% highlight java %}
package com.xuying.demo.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {
	@Autowired
	private JdbcTemplate jdbcTemplate;

	@Override
	public void create(String name, Integer age) {
		jdbcTemplate.update("insert into USER(NAME, AGE) values(?, ?)", name, age);
	}

	@Override
	public void deleteByName(String name) {
		jdbcTemplate.update("delete from USER where NAME = ?", name);
	}

	@Override
	public Integer getAllUsers() {
		return jdbcTemplate.queryForObject("select count(1) from USER", Integer.class);
	}

	@Override
	public void deleteAllUsers() {
		jdbcTemplate.update("TRUNCATE table USER");
	}
}

{% endhighlight %}  

创建对UserService的单元测试用例，通过创建、删除和查询来验证数据库操作的正确性。

{% highlight java %}
package com.xuying;

import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;

import com.xuying.demo.DemoApplication;
import com.xuying.demo.service.UserService;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@SpringBootTest
@SpringApplicationConfiguration(DemoApplication.class)
public class ApplicationTestsJdbc {
	@Autowired
	private UserService userSerivce;

	@Before
	public void setUp() {
		// 准备，清空user表
		userSerivce.deleteAllUsers();
	}

	@Test
	public void test() throws Exception {
		// 插入5个用户
		userSerivce.create("a", 1);
		userSerivce.create("b", 2);
		userSerivce.create("c", 3);
		userSerivce.create("d", 4);
		userSerivce.create("e", 5);
		// 查数据库，应该有5个用户
		Assert.assertEquals(5, userSerivce.getAllUsers().intValue());
		// 删除两个用户
		userSerivce.deleteByName("a");
		userSerivce.deleteByName("e");
		// 查数据库，应该有5个用户
		Assert.assertEquals(3, userSerivce.getAllUsers().intValue());
	}
}
{% endhighlight %} 


通过上面这个简单的例子，我们可以看到在Spring Boot下访问数据库的配置依然秉承了框架的初衷：简单。我们只需要在pom.xml中加入数据库依赖，再到application.properties中配置连接信息，不需要像Spring应用中创建JdbcTemplate的Bean，就可以直接在自己的对象中注入使用。