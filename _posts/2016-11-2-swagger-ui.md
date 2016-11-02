---
layout: post
title: Swagger UI教程 API 文档神器 搭配Node使用
tags:
- swagger
categories: swagger
description: Swagger UI教程 API 文档神器 搭配Node使用

---

# 环境搭建 #

下载Swagger UI（也可以直接下载 zip 文件）
{% highlight ruby  %} 

    git clone https://github.com/swagger-api/swagger-ui.git

{% endhighlight %} 
安装 express  
创建一个空文件夹node_app

{% highlight java %} 
    mkdir node_app
{% endhighlight %} 
初始化 node ，创建package.json文件（）
{% highlight  %} 
    cd node_ap
    npm init
    // 下面的看你心情填写
    name: (node_app) node_app
    version: (1.0.0)
    description:
    entry point: (index.js)
    test command:
    git repository:
    keywords:
    author:
    license: (ISC)
{% endhighlight %} 
安装 express
{% highlight  %}
    npm install express --save
{% endhighlight %} 
创建 index.js
{% highlight  %}
    vim index.js
{% endhighlight %} 
把下面代码贴如 index.js 中
{% highlight  %}
    var express = require('express');
    var app = express();
    app.get('/', function (req, res) {
      res.send('Hello World!');
    });
    
    app.listen(3000, function () {
      console.log('Example app listening on port 3000!');
    });
{% endhighlight %} 
在 node_app 中创建空目录 public
{% highlight  %}

      mkdir public
      cd public
{% endhighlight %} 
修改路由
{% highlight  %}
    vim ../index.js
    //在文件第三行插入下面这句话
    app.use('/static', express.static('public'));
{% endhighlight %} 
把下载好的Swagger UI 文件中dist 目录下的文件全部复制到 public 文件夹下。
如下图：
![](http://i.imgur.com/jCmK5cx.png)


开启 node
{% highlight  %}
    node index.js
{% endhighlight %} 
打开浏览器，输入[http://localhost:3000/static/index.html](打开浏览器，输入http://localhost:3000/static/index.html)

到此为止，你已经把官方的 demo 在本地配置好了。当然你也可以吧这个搭建在服务器上

# 编写文档并发布 #

使用Swagger Editor编写 API 文档

Swagger Editor 上的是基于 yaml 的语法，但是不用害怕，看着官方的 demo 看个10分钟就会了。


导出 test.json 文档

![](http://upload-images.jianshu.io/upload_images/437742-77e0ea186ad1a01e.png?imageMogr2/auto-orient/strip%7CimageView2/2)


把 test.json 放到 node_app/public 目录下。  
利用编辑器修改 index.html文件

url = "http://petstore.swagger.io/v2/swagger.json";  
为  
url = "/static/test.json";  

重启 node 服务，浏览器中打开[http://localhost:3000/static/index.html](http://localhost:3000/static/index.html)就是你自己写的 api 文档了

