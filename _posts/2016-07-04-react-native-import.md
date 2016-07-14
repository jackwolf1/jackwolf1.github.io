---
layout: post
title: 在Windows上将ReactNative集成到现有的Android项目
tags:
- react-native
- android
categories: react-native
description: 将ReactNative集成到现有的Android项目
---

React Natvie的官方文档的 Integrating with Existing Apps 已经很详细地教我们如何将React Natvie集成到现在的Android项目。我根据官方文档的步骤，在Windows上将React Native集成到现有的Android的过程记录下来。

集成到Android项目的要求：

一个现有的Android项目并搭建好环境。
首先，创建一个Android的项目（这里用来模拟现有的Andorid项目）
在现有的Android的build.gradle文件中增加React Natvie的依赖
点击 Maven Central 查看React Natvie的最新版本，这里的最新版本已经是0.18.0了。
在build.gradle文件中加入 
{% highlight null %}

`compile 'com.facebook.react:react-native:+'`  

{% endhighlight %}
在AndroidManifest.xml加入访问网络的权限
{% highlight null %}

    <uses-permission android:name="android.permission.INTERNET" />

{% endhighlight %}
将下面的代码复制到项目中（记得在AndroidManifest.xml注册该类）
{% highlight java linenos%}

    public class MyReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    
    public static void launch(Activity activity){
    Intent intent = new Intent(activity, MyReactActivity.class);
    activity.startActivity(intent);
    }
    
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
    mReactRootView = new ReactRootView(this);
    mReactInstanceManager = ReactInstanceManager.builder()
    .setApplication(getApplication())
    .setBundleAssetName("index.android.bundle")
    .setJSMainModuleName("index.android")
    .addPackage(new MainReactPackage())
    .setUseDeveloperSupport(BuildConfig.DEBUG)
    .setInitialLifecycleState(LifecycleState.RESUMED)
    .build();
    mReactRootView.startReactApplication(mReactInstanceManager, "MyAwesomeApp", null);
    
    setContentView(mReactRootView);
    }
    
    @Override
    protected void onResume() {
    super.onResume();
    
    if (mReactInstanceManager != null) {
    mReactInstanceManager.onResume(this, this);
    }
    }
    
    @Override
    protected void onPause() {
    super.onPause();
    
    if (mReactInstanceManager != null) {
    mReactInstanceManager.onPause();
    }
    }
    
    @Override
    public void invokeDefaultOnBackPressed() {
    super.onBackPressed();
    }
    
    
    @Override
    public void onBackPressed() {
    if (mReactInstanceManager != null) {
    mReactInstanceManager.onBackPressed();
    } else {
    super.onBackPressed();
    }
    }
    
    
    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
    mReactInstanceManager.showDevOptionsDialog();
    return true;
    }
    return super.onKeyUp(keyCode, event);
    }
    }

{% endhighlight %}
接下来，就要将JS增加到现有的项目。
在项目的根目录打开命令行(切换到根目录，按住Shift，右击就会出现‘在此处打开命令窗口’)
输入npm init，居然弹这些玩意出来（靠），一开始我以为出错了，后来是才知道是填东西的（生成package.json这个文件的）
机智的我在name那里输入了括号后面的文字，弹出下面的错误（不能够大写的），然后将括号的文字全部输入小写就可以了。
基本按括号后面的提示填就可以了，最后就在项目的根目录生成package.json这个文件。
最后提示Is this ok? 输入yes
然后，输入npm install --save react-native
接着，输入react-native start启动服务，发现报错了（package.json解析不了)。
打开package.json将test这行删掉，然后保存。
再运行react-native start，可以看到服务已启动，端口为8081。
在浏览器输入地址：http://localhost:8081/index.android.bundle?platform=android
如果可以打开，则证明服务无问题了。
