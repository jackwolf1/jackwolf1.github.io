---
layout: post
title: jekyll注意事项
tags:
- Jekyll
categories: Jekyll
description: jekyll使用时遇到的一些问题
---

#错误分析：


在运行 jekyll serve 的时候

**问题1**：

```
Dependency Error: Yikes! It looks like you don't have redcarpet or one of its
dependencies installed. In order to use Jekyll as currently configured, you'll n
eed to install this gem. The full error message from Ruby is: 'cannot load such
file -- redcarpet' If you run into trouble, you can find helpful resources at ht
tp://jekyllrb.com/help/!
  Conversion error: Jekyll::Converters::Markdown encountered an error while conv
erting '_posts/2014-12-28-first-blog.md':
                    redcarpet
             ERROR: YOUR SITE COULD NOT BE BUILT:
                    ------------------------------------
                    redcarpet

```
需要把`_config.yml`中的redcarpet改为kramdown

**问题2**：

```

Deprecation: You appear to have pagination turned on, but you haven’t included the jekyll-paginate gem. Ensure you 
have gems: [jekyll-paginate] in your configuration file.

```
因为我们的配置文件`_config.yml`使用了 paginate 配置项，所以需要添加一行：

```
# Gems
gems: [jekyll-paginate]
```