---
layout: post
title: react-native-crosswalk-android
tags:
- react-native
- crosswalk
categories: react-native
description: 在react-native中使用crosswalk
---
在react-native中使用crosswalk

首先集成react-native，然后将webview替换成crosswalk。项目可以查看我的github。[https://github.com/jackwolf1/android-react-native-crosswalk](https://github.com/jackwolf1/android-react-native-crosswalk)
因为 crosswalk 包中包含的 javax* 会导致重复引用，需要在包中去掉

    unzip -j xwalk_core_library-${ver}.aar classes.jar
    zip -d classes.jar javax\*
    zip -r xwalk_core_library-${ver}.aar classes.jar
    rm -f classes.jar

将新生成的jar包放入libs中，并放入对应的jnilibs，crosswalk导入成功。
主activity如下：

    public class MainActivity extends ReactActivity {
    
    /**
     * Returns the name of the main component registered from JavaScript.
     * This is used to schedule rendering of the component.
     */
    @Override
    protected String getMainComponentName() {
    return "testProject";
    }
    
    /**
     * Returns whether dev mode should be enabled.
     * This enables e.g. the dev menu.
     */
    @Override
    protected boolean getUseDeveloperSupport() {
    return BuildConfig.DEBUG;
    }
    
    /**
     * A list of packages used by the app. If the app uses additional views
     * or modules besides the default ones, add more packages here.
     */
    @Override
    protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
    new MainReactPackage(),new AppReactPackage(this)
    );
    }
    }

AppReactPackage如下：

    public class AppReactPackage implements ReactPackage {
    Activity mActivity;
    
    public AppReactPackage(Activity activity) {
    mActivity = activity;
    }
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    
    return Collections.emptyList();
    }
    
    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
    return Collections.emptyList();
    }
    
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Arrays.<ViewManager>asList(
    new XWalkViewManager(mActivity));
    }
    }

XWalkViewManager如下：

    public class XWalkViewManager extends SimpleViewManager<XWalkView> {
    
    private static final String REACT_CLASS = "XWalkView";
    
    private static final String HTML_ENCODING = "UTF-8";
    private static final String HTML_MIME_TYPE = "text/html; charset=utf-8";
    
    private static final String HTTP_METHOD_POST = "POST";
    
    public static final int COMMAND_GO_BACK = 1;
    public static final int COMMAND_GO_FORWARD = 2;
    public static final int COMMAND_RELOAD = 3;
    
    // Use `webView.loadUrl("about:blank")` to reliably reset the view
    // state and release page resources (including any running JavaScript).
    private static final String BLANK_URL = "about:blank";
    
    private XWalkViewConfig mWebViewConfig;
    
    private Activity mActivity;
    
    private static class ResourceClient extends XWalkResourceClient {
    
    private boolean mLastLoadFailed = false;
    private int progressInPercent=0;
    
    public ResourceClient(XWalkView view) {
    super(view);
    
    }
    
    @Override
    public void onReceivedResponseHeaders(XWalkView view, XWalkWebResourceRequest request, XWalkWebResourceResponse response) {
    super.onReceivedResponseHeaders(view, request, response);
    }
    
    @Override
    public void onLoadFinished(XWalkView webView, String url) {
    super.onLoadFinished(webView, url);
    if (!mLastLoadFailed) {
    RXWalkView reactWebView = (RXWalkView) webView;
    reactWebView.callInjectedJavaScript();
    emitFinishEvent(webView, url);
    }
    }
    
    @Override
    public void onLoadStarted(XWalkView webView, String url) {
    super.onLoadStarted(webView, url);
    mLastLoadFailed = false;
    
    dispatchEvent(
    webView,
    new TopLoadingStartEvent(
    webView.getId(),
    SystemClock.nanoTime(),
    createWebViewEvent(webView, url)));
    }
    
    @Override
    public void onReceivedLoadError(XWalkView webView, int errorCode, String description, String failingUrl) {
    super.onReceivedLoadError(webView, errorCode, description, failingUrl);
    mLastLoadFailed = true;
    
    // In case of an error JS side expect to get a finish event first, and then get an error event
    // Android WebView does it in the opposite way, so we need to simulate that behavior
    emitFinishEvent(webView, failingUrl);
    
    WritableMap eventData = createWebViewEvent(webView, failingUrl);
    eventData.putDouble("code", errorCode);
    eventData.putString("description", description);
    
    dispatchEvent(
    webView,
    new TopLoadingErrorEvent(webView.getId(), SystemClock.nanoTime(), eventData));
    }
    
    @Override
    public void doUpdateVisitedHistory(XWalkView webView, String url, boolean isReload) {
    super.doUpdateVisitedHistory(webView, url, isReload);
    dispatchEvent(
    webView,
    new TopLoadingStartEvent(
    webView.getId(),
    SystemClock.nanoTime(),
    createWebViewEvent(webView, url)));
    }
    
    private void emitFinishEvent(XWalkView webView, String url) {
    dispatchEvent(
    webView,
    new TopLoadingFinishEvent(
    webView.getId(),
    SystemClock.nanoTime(),
    createWebViewEvent(webView, url)));
    }
    
    private static void dispatchEvent(XWalkView webView, Event event) {
    ReactContext reactContext = (ReactContext) ((RXWalkView)webView).getReactContext();
    EventDispatcher eventDispatcher =
    reactContext.getNativeModule(UIManagerModule.class).getEventDispatcher();
    eventDispatcher.dispatchEvent(event);
    }
    
    
    @Override
    public void onProgressChanged(XWalkView view, int progressInPercent) {
    super.onProgressChanged(view, progressInPercent);
    this.progressInPercent = progressInPercent;
    }
    
    private WritableMap createWebViewEvent(XWalkView webView, String url) {
    WritableMap event = Arguments.createMap();
    event.putDouble("target", webView.getId());
    // Don't use webView.getUrl() here, the URL isn't updated to the new value yet in callbacks
    // like onPageFinished
    event.putString("url", url);
    event.putBoolean("loading", !mLastLoadFailed && this.progressInPercent != 100);
    event.putString("title", webView.getTitle());
    event.putBoolean("canGoBack", webView.getNavigationHistory().canGoBack());
    event.putBoolean("canGoForward", webView.getNavigationHistory().canGoForward());
    return event;
    }
    }
    
    /**
     * Subclass of {@link WebView} that implements {@link LifecycleEventListener} interface in order
     * to call {@link WebView#destroy} on activty destroy event and also to clear the client
     */
    private static class RXWalkView extends XWalkView implements LifecycleEventListener {
    private @Nullable String injectedJS;
    private ReactContext reactContext;
    /**
     * WebView must be created with an context of the current activity
     *
     * Activity Context is required for creation of dialogs internally by WebView
     * Reactive Native needed for access to ReactNative internal system functionality
     */
    public RXWalkView(Context context) {
    super(context);
    }
    
    public ReactContext getReactContext() {
    return reactContext;
    }
    
    public void setReactContext(ReactContext reactContext) {
    this.reactContext = reactContext;
    }
    
    @Override
    public void onHostResume() {
    // do nothing
    }
    
    @Override
    public void onHostPause() {
    // do nothing
    }
    
    @Override
    public void onHostDestroy() {
    cleanupCallbacksAndDestroy();
    }
    
    public void setInjectedJavaScript(@Nullable String js) {
    injectedJS = js;
    }
    
    public void callInjectedJavaScript() {
    if (
    injectedJS != null &&
    !TextUtils.isEmpty(injectedJS)) {
    super.load(null,"javascript:(function() {\n" + injectedJS + ";\n})();");
    }
    }
    
    private void cleanupCallbacksAndDestroy() {
    super.setResourceClient(null);
    super.onDestroy();
    }
    }
    
    public XWalkViewManager(Activity activity) {
    mActivity = activity;
    mWebViewConfig = new XWalkViewConfig() {
    public void configWebView(XWalkView webView) {
    }
    };
    }
    
    public XWalkViewManager(XWalkViewConfig webViewConfig) {
    mWebViewConfig = webViewConfig;
    }
    
    @Override
    public String getName() {
    return REACT_CLASS;
    }
    
    @Override
    protected XWalkView createViewInstance(ThemedReactContext reactContext) {
    RXWalkView webView = new RXWalkView(mActivity);
    webView.setReactContext(reactContext);
    webView.setResourceClient(new ResourceClient(webView));
    reactContext.addLifecycleEventListener(webView);
    mWebViewConfig.configWebView(webView);
    
    if (ReactBuildConfig.DEBUG && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WebView.setWebContentsDebuggingEnabled(true);
    }
    
    return webView;
    }
    
    @ReactProp(name = "url")
    public void setUrl(XWalkView view,@android.support.annotation.Nullable String url) {
    view.load(url,null);
    }
    
    @ReactProp(name = "javaScriptEnabled")
    public void setJavaScriptEnabled(XWalkView view,@android.support.annotation.Nullable boolean enabled) {
    
    try {
    Method ___getBridge = XWalkView.class.getDeclaredMethod("getBridge");
    
    ___getBridge.setAccessible(true);
    
    XWalkViewBridge xWalkViewBridge = null;
    
    xWalkViewBridge = (XWalkViewBridge) ___getBridge.invoke(view);
    
    xWalkViewBridge.getSettings().setJavaScriptEnabled(enabled);
    
    } catch (InvocationTargetException e) {
    e.printStackTrace();
    } catch (NoSuchMethodException e) {
    e.printStackTrace();
    } catch (IllegalAccessException e) {
    e.printStackTrace();
    }
    
    }
    
    @ReactProp(name = "scalesPageToFit")
    public void setScalesPageToFit(XWalkView view, boolean enabled) {
    view.getSettings().setUseWideViewPort(!enabled);
    }
    
    @ReactProp(name = "domStorageEnabled")
    public void setDomStorageEnabled(XWalkView view, boolean enabled) {
    try {
    Method ___getBridge = XWalkView.class.getDeclaredMethod("getBridge");
    
    ___getBridge.setAccessible(true);
    
    XWalkViewBridge xWalkViewBridge = null;
    
    xWalkViewBridge = (XWalkViewBridge) ___getBridge.invoke(view);
    
    xWalkViewBridge.getSettings().setDomStorageEnabled(enabled);
    
    } catch (InvocationTargetException e) {
    e.printStackTrace();
    } catch (NoSuchMethodException e) {
    e.printStackTrace();
    } catch (IllegalAccessException e) {
    e.printStackTrace();
    }
    
    }
    
    
    @ReactProp(name = "userAgent")
    public void setUserAgent(XWalkView view, @Nullable String userAgent) {
    view.getSettings().setUserAgentString(userAgent);
    
    }
    
    @ReactProp(name = "mediaPlaybackRequiresUserAction")
    public void setMediaPlaybackRequiresUserAction(XWalkView view, boolean requires) {
    try {
    Method ___getBridge = XWalkView.class.getDeclaredMethod("getBridge");
    
    ___getBridge.setAccessible(true);
    
    XWalkViewBridge xWalkViewBridge = null;
    
    xWalkViewBridge = (XWalkViewBridge) ___getBridge.invoke(view);
    
    xWalkViewBridge.getSettings().setMediaPlaybackRequiresUserGesture(requires);
    
    } catch (InvocationTargetException e) {
    e.printStackTrace();
    } catch (NoSuchMethodException e) {
    e.printStackTrace();
    } catch (IllegalAccessException e) {
    e.printStackTrace();
    }
    }
    
    @ReactProp(name = "injectedJavaScript")
    public void setInjectedJavaScript(XWalkView view, @Nullable String injectedJavaScript) {
    ((RXWalkView) view).setInjectedJavaScript(injectedJavaScript);
    }
    
    @ReactProp(name = "source")
    public void setSource(XWalkView view, @Nullable ReadableMap source) {
    if (source != null) {
    if (source.hasKey("html")) {
    String html = source.getString("html");
    if (source.hasKey("baseUrl")) {
    view.load(
    source.getString("baseUrl"), html);
    } else {
    view.load(null,html);
    }
    return;
    }
    if (source.hasKey("uri")) {
    String url = source.getString("uri");
    if (source.hasKey("method")) {
    String method = source.getString("method");
    if (method.equals(HTTP_METHOD_POST)) {
    byte[] postData = null;
    if (source.hasKey("body")) {
    String body = source.getString("body");
    try {
    postData = body.getBytes("UTF-8");
    } catch (UnsupportedEncodingException e) {
    postData = body.getBytes();
    }
    }
    if (postData == null) {
    postData = new byte[0];
    }
    //view.postUrl(url, postData);
    return;
    }
    }
    HashMap<String, String> headerMap = new HashMap<>();
    if (source.hasKey("headers")) {
    ReadableMap headers = source.getMap("headers");
    ReadableMapKeySetIterator iter = headers.keySetIterator();
    while (iter.hasNextKey()) {
    String key = iter.nextKey();
    headerMap.put(key, headers.getString(key));
    }
    }
    //view.loadUrl(url, headerMap);
    view.load(url,null);
    return;
    }
    }
    view.load(null,BLANK_URL);
    }
    
    @Override
    protected void addEventEmitters(ThemedReactContext reactContext, XWalkView view) {
    // Do not register default touch emitter and let WebView implementation handle touches
    view.setResourceClient(new ResourceClient(view));
    }
    
    @Override
    public @Nullable Map<String, Integer> getCommandsMap() {
    return MapBuilder.of(
    "goBack", COMMAND_GO_BACK,
    "goForward", COMMAND_GO_FORWARD,
    "reload", COMMAND_RELOAD);
    }
    
    @Override
    public void receiveCommand(XWalkView view, int commandId, @Nullable ReadableArray args) {
    switch (commandId) {
    case COMMAND_GO_BACK:
    view.getNavigationHistory().navigate(XWalkNavigationHistory.Direction.BACKWARD, 1);
    break;
    case COMMAND_GO_FORWARD:
    view.getNavigationHistory().navigate(XWalkNavigationHistory.Direction.FORWARD, 1);
    break;
    case COMMAND_RELOAD:
    view.reload(XWalkView.RELOAD_NORMAL);
    break;
    }
    }
    
    @Override
    public void onDropViewInstance(XWalkView webView) {
    super.onDropViewInstance(webView);
    ((ThemedReactContext) ((RXWalkView)webView).getReactContext()).removeLifecycleEventListener((RXWalkView) webView);
    ((RXWalkView) webView).cleanupCallbacksAndDestroy();
    }
    }
