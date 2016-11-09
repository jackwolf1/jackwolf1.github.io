---
layout: post
title: swagger-editor 快速REST-API 测试文档编写
tags:
- swagger
categories: swagger
description: swagger-editor 快速REST-API 测试文档编写
---

# 1. 在线使用 #

 [http://editor.swagger.io/#/](http://editor.swagger.io/#/)

# 2. 离线工具 #

 [https://github.com/swagger-api/swagger-editor]( https://github.com/swagger-api/swagger-editor)


# 3. 跨域访问问题： #
 添加一下：
{% highlight null %}
Access-Control-Allow-Origin: *  
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept
{% endhighlight %} 
#  4. docker 安装 #
{% highlight null %}
docker pull swaggerapi/swagger-editor
docker run -p 80:8080 swaggerapi/swagger-editor
{% endhighlight %} 