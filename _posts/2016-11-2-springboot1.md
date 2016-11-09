---
layout: post
title: Spring Boot快速入门
tags:
- SpringBoot
categories: SpringBoot
description: SpringBoot
---

Spring Boot快速入门 

# 简介 #

Spring Boot的主要优点：


为所有Spring开发者更快的入门  
开箱即用，提供各种默认配置来简化项目配置  
内嵌式容器简化Web项目  
没有冗余代码生成和XML配置的要求  

# 快速入门 #
本章主要目标完成Spring Boot基础项目的构建，并且实现一个简单的Http请求处理，通过这个例子对Spring Boot有一个初步的了解，并体验其结构简单、开发快速的特性。

## 使用Maven构建项目 ##


通过SPRING INITIALIZR工具产生基础项目
访问：[http://start.spring.io/](http://start.spring.io/)
选择构建工具Maven Project、Spring Boot版本1.4.1以及一些工程基本信息,点击Generate Project下载项目压缩包 

解压项目包，并用STS以Maven项目导入

通过上面步骤完成了基础项目的创建，如上图所示，Spring Boot的基础结构共三个文件（具体路径根据用户生成项目时填写的Group所有差异）：

src/main/java下的程序入口：DemoApplication  
src/main/resources下的配置文件：application.properties  
src/test/下的测试入口：DemoApplicationTests

生成的DemoApplication和DemoApplicationTests类都可以直接运行来启动当前创建的项目，由于目前该项目未配合任何数据访问或Web模块，程序会在加载完Spring之后结束运行。


## 引入Web模块 ##

当前的pom.xml内容如下，仅引入了两个模块：

spring-boot-starter：核心模块，包括自动配置支持、日志和YAML  
spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito  
引入Web模块，需添加spring-boot-starter-web模块：  
{% highlight null %}
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
{% endhighlight %} 


## 编写HelloWorld服务 ##

创建package命名为com.xuying.web（根据实际情况修改）
创建HelloController类，内容如下  

{% highlight null %}
@RestController
public class HelloController {
   @RequestMapping("/hello")
   public String index() {
       return "Hello World";
   }
}
{% endhighlight %} 

启动主程序，打开浏览器访问[http://localhost:8080/hello](http://localhost:8080/hello)，可以看到页面输出Hello World
{% highlight java %}
# 编写单元测试用例 #

打开的src/test/下的测试入口DemoApplicationTests类。下面编写一个简单的单元测试来模拟http请求，具体如下：


package com.xuying;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import com.xuying.web.HelloController;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@SpringApplicationConfiguration(classes = MockServletContext.class)
@WebAppConfiguration
public class DemoApplicationTests {
	private MockMvc mvc;

	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
	}

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
				.andExpect(content().string(equalTo("Hello World")));
	}
}

{% endhighlight %} 

使用MockServletContext来构建一个空的WebApplicationContext，这样我们创建的HelloController就可以在@Before函数中创建并传递到MockMvcBuilders.standaloneSetup（）函数中。