---
layout: post
title: anroid下使用react-native
tags:
- android
- react-native
categories: react-native
description: react-native的使用说明
---

react-native使用说明

需要安装android sdk,node.js 

**快速开始：**

    npm install -g react-native-cli   //初始化配置
    react-native init AwesomeProject   //初始化项目
    react-native start   //重点，开启对应服务
    （可以用浏览器访问http://localhost:8081/index.android.bundle?platform=android看看是否可以看到打包后的脚本）

**安卓运行：**

保持packager开启，另外打开一个命令行窗口，然后在工程目录下运行

    react-native run-android
    

注意：
摇晃设备或按Menu键（Bluestacks模拟器按键盘上的菜单键，通常在右Ctrl的左边 或者左Windows键旁边），可以打开调试菜单，点击Dev Settings，选Debug server host for device，输入你的正在运行packager的那台电脑的局域网IP加:8081（同时要保证手机和电脑在同一网段，且没有防火墙阻拦），再按back键返回，再按Menu键，在调试菜单中选择Reload JS，就应该可以看到运行的结果了。
如果真实设备白屏但没有弹出任何报错，可以在安全中心里看看是不是应用的“悬浮窗”的权限被禁止了。

