---
layout: post
title: android styles
tags:
- android
- styles
categories: styles
description: android styles中的一些问题
---

在android studio xml中报如下错误：

 Rendering Problems NOTE: This project contains Java compilation errors, which can cause rendering failures for custom views. Fix compilation problems first.  The following classes could not be instantiated:
- android.support.v7.internal.app.WindowDecorActionBar (Open Class, Show Exception, Clear Cache)

解决办法：

 <!-- Base application theme. -->
    <style name="AppTheme" parent="Base.Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/main_title_bg</item>
        <item name="colorPrimaryDark">@color/main_title_bg</item>
        <item name="colorAccent">@color/main_color</item>
        <item name="android:windowContentOverlay">@null</item><!--actionbar的下阴影线消除-->
        <item name="android:textColorPrimary">@color/main_title</item>
    </style>  
需要使用Base.跟后面的theme