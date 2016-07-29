---
layout: post
title: Android分包MultiDex
tags:
- MultiDex
categories: MultiDex
description: Android分包MultiDex原理详解
---
用Gradle配置使用Multidex
Android 的 Gradle插件在 Android Build Tool 21.1开始就支持使用multidex了。

设置你的应用程序开发项目中使用multidex配置，要求你做出一些修改您的应用程序开发项目。：

修改Gradle的配置，支持multidex
修改你的manifest。让其支持multidexapplication类
修改Gradle的build如下：
{% highlight java %}  
android {
    compileSdkVersion 21
    buildToolsVersion "21.1.0"

    defaultConfig {
        ...
        minSdkVersion 14
        targetSdkVersion 21
        ...

        // Enabling multidex support.
        multiDexEnabled true
    }
    ...
}

dependencies {
  compile ‘com.android.support:multidex:1.0.0‘
}
{% endhighlight %}
Tips: 你可以在Gradle配置文件中的 multiDexEnabled 在 defaultConfig、 
buildType、productFlavor选项设置。

在manifest文件中，添加MultidexApplication Class的引用，如下所示：
{% highlight java %} 
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.multidex.myapplication">
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
</manifest>

{% endhighlight %}