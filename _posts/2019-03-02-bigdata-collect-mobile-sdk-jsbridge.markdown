---
layout: post
title:  "通过JsBridge关联WEBSDK与移动端SDK"
date:   2019-03-01 00:18:23 +0700
categories: [bigdata]
---

#### 概述
   大家在移动端应用中多少会看到一个原生的APP中打开H5页面，那么我们如何实现APP与H5页面的相应通讯，以及我们如何实现在APP中打开一个页面时，将页面的数据
上报到移动应用中，这是本文章接下聊的重点。

#### 移动开发中的WEBVIEW    
  Android WebView在Android平台上是一个特殊的View， 基于webkit引擎、展现web页面的控件，这个类可以被用来在你的app中仅仅显示一张在线的网页，还可以
用来开发浏览器。WebView内部实现是采用渲染引擎来展示view的内容，提供网页前进后退，网页放大，缩小，搜索。Android的Webview在低版本和高版本采用了不同的
webkit版本内核，4.4后直接使用了Chrome。现在很多APP都内置了Web网页，比如说很多电商平台，淘宝、京东、聚划算等等。WebView比较灵活，不需要升级客户端，
只需要修改网页代码即可。一些经常变化的页面可以用WebView这种方式去加载网页。例如中秋节跟国庆节打开的页面不一样，如果是用WebView显示的话，
只修改修改html页面就行，而不需要升级客户端。
##### 加载HTML的几种方式    
    
```
webView.loadUrl("http://www.viewc.org/test.html");//加载url  
webView.loadUrl("file:///android_asset/test.html");//加载asset文件夹下html  
//方式3：加载手机sdcard上的html页面
webView.loadUrl("content://com.viewc.webview/sdcard/test.html");  
//方式4 使用webview显示html代码
webView.loadDataWithBaseURL(null,"<html><head><title> 欢迎您 </title></head>" +
        "<body><h2>使用webview显示 html代码</h2></body></html>", "text/html" , "utf-8", null);
```
  
##### WEBVIEW与WEBChromeClient区别  
  WebViewClient主要帮助WebView处理各种通知、请求事件的，有以下常用方法：   
    - onPageFinished 页面请求完成   
    - onPageStarted 页面开始加载   
    - shouldOverrideUrlLoading 拦截url   
    - onReceivedError 访问错误时回调，例如访问网页时报错404，在这个方法回调的时候可以加载错误页面。  
    
  WebChromeClient主要辅助WebView处理Javascript的对话框、网站图标、网站title、加载进度等，有以下常用方法  
    - onJsAlert webview不支持js的alert弹窗，需要自己监听然后通过dialog弹窗   
    - onReceivedTitle 获取网页标题   
    - onReceivedIcon 获取网页icon   
    - onProgressChanged 加载进度回调  
  
##### 完整使用过程  
```aidl
  因为需要加载网页url，所以需要在AndroidManifest.xml中添加访问网络权限。  
  <uses-permission android:name="android.permission.INTERNET" />
  
  布局文件:activity_main.xml
  
  <?xml version="1.0" encoding="utf-8"?>
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
  
      <WebView
          android:id="@+id/webview"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>
  
      <ProgressBar
          android:id="@+id/progressbar"
          style="@android:style/Widget.ProgressBar.Horizontal"
          android:layout_width="match_parent"
          android:layout_height="3dip"
          android:max="100"
          android:progress="0"
          android:visibility="gone"/>
  </FrameLayout>
   
  外层FrameLayout，里面有WebView跟ProgressBar，WebView的宽高匹配父类，ProgressBar横向进度条，高度3dip，按照FrameLayout布局规则，ProgressBar会覆盖在WebView之上，默认是隐藏不显示。
  public class MainActivity extends AppCompatActivity {
      private WebView webView;
      private ProgressBar progressBar;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          progressBar= (ProgressBar)findViewById(R.id.progressbar);//进度条
          webView = (WebView) findViewById(R.id.webview);
          webView.loadUrl("http://139.196.35.30:8080/OkHttpTest/apppackage/test.html");//加载url
          webView.addJavascriptInterface(this,"android");//添加js监听 这样html就能调用客户端
          webView.setWebChromeClient(webChromeClient);
          webView.setWebViewClient(webViewClient);
          WebSettings webSettings=webView.getSettings();
          webSettings.setJavaScriptEnabled(true);//允许使用js
          /**
           * LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
           * LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
           * LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
           * LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。
           */
          webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);//不使用缓存，只从网络获取数据.
          //支持屏幕缩放
          webSettings.setSupportZoom(true);
          webSettings.setBuiltInZoomControls(true);
      }
  
      //WebViewClient主要帮助WebView处理各种通知、请求事件
      private WebViewClient webViewClient=new WebViewClient(){
          @Override
          public void onPageFinished(WebView view, String url) {//页面加载完成
              progressBar.setVisibility(View.GONE);
          }
  
          @Override
          public void onPageStarted(WebView view, String url, Bitmap favicon) {//页面开始加载
              progressBar.setVisibility(View.VISIBLE);
          }
  
          @Override
          public boolean shouldOverrideUrlLoading(WebView view, String url) {
              Log.i("ansen","拦截url:"+url);
              if(url.equals("http://www.google.com/")){
                  Toast.makeText(MainActivity.this,"国内不能访问google,拦截该url",Toast.LENGTH_LONG).show();
                  return true;//表示我已经处理过了
              }
              return super.shouldOverrideUrlLoading(view, url);
          }
  
      };
  
      //WebChromeClient主要辅助WebView处理Javascript的对话框、网站图标、网站title、加载进度等
      private WebChromeClient webChromeClient=new WebChromeClient(){
          //不支持js的alert弹窗，需要自己监听然后通过dialog弹窗
          @Override
          public boolean onJsAlert(WebView webView, String url, String message, JsResult result) {
              AlertDialog.Builder localBuilder = new AlertDialog.Builder(webView.getContext());
              localBuilder.setMessage(message).setPositiveButton("确定",null);
              localBuilder.setCancelable(false);
              localBuilder.create().show();
  
              //注意:
              //必须要这一句代码:result.confirm()表示:
              //处理结果为确定状态同时唤醒WebCore线程
              //否则不能继续点击按钮
              result.confirm();
              return true;
          }
  
          //获取网页标题
          @Override
          public void onReceivedTitle(WebView view, String title) {
              super.onReceivedTitle(view, title);
              Log.i("ansen","网页标题:"+title);
          }
  
          //加载进度回调
          @Override
          public void onProgressChanged(WebView view, int newProgress) {
              progressBar.setProgress(newProgress);
          }
      };
  
      @Override
      public boolean onKeyDown(int keyCode, KeyEvent event) {
          Log.i("ansen","是否有上一个页面:"+webView.canGoBack());
          if (webView.canGoBack() && keyCode == KeyEvent.KEYCODE_BACK){//点击返回按钮的时候判断有没有上一页
              webView.goBack(); // goBack()表示返回webView的上一页面
              return true;
          }
          return super.onKeyDown(keyCode,event);
      }
  
      /**
       * JS调用android的方法
       * @param str
       * @return
       */
      @JavascriptInterface //仍然必不可少
      public void  getClient(String str){
          Log.i("ansen","html调用客户端:"+str);
      }
  
      @Override
      protected void onDestroy() {
          super.onDestroy();
  
          //释放资源
          webView.destroy();
          webView=null;
      }
  }
  
  onCreate 查找控件，给webView设置加载url，添加js监听，监听的名称是”android”,设置webChromeClient跟webViewClient回调，通过getSettings方法获取WebSettings对象，设置允许加载js，设置缓存模式，支持缩放。
  webViewClient 重写了几个方法,onPageFinished页面加载完成隐藏进度条，onPageStarted页面开始加载显示进度条，shouldOverrideUrlLoading拦截url，如果请求url是打开google，不让他请求，因为google在国内不能访问，就算请求也请求不到还不如拦截掉，直接告诉用户不能访问。
  webChromeClient onJsAlert()因为WebView不支持alert弹窗，在这个方法中用AlertDialog去弹窗。onReceivedTitle获取网页标题。onProgressChanged页面加载进度，把加载进度给progressBar。
  onKeyDown 如果点击系统自带返回键&&webView有上一级页面，调用goBack返回。否则不处理。什么时候辉有上一级页面呢？就是你从首页跳转到了一个新页面,点击返回的时候会返回首页。如果本来就在首页点击返回的时候会退出app。
  getClient html页面的JS可以通过这个方法回调原生APP，这个方法有个注解@JavascriptInterface，这个是必须的，这个方法有个字符串参数，这个方法跟我们在onCreate中调用addJavascriptInterface传入的name一起使用的。例如html中想要回调这个方法可以这样写:javascript:android.getClient(“传一个字符串给客户端”);
  onDestroy activity销毁时释放webView资源。
```
  
  重点：给WEBVIEW设置固定的User-Agent  
```aidl

WebSettings settings = webview.getSettings();
        settings.setUserAgentString("Viewc-APP-3.4/VIEW-APP");//添加UA
        settings.setJavaScriptEnabled(true);
        //设置参数
        settings.setBuiltInZoomControls(true);
        settings.setAppCacheEnabled(true);// 设置缓存
        webview.setWebChromeClient(new WebChromeClient());
        webview.loadUrl(loadurl);
```    
  
#### H5与称动通讯的代理利器JSBridge   
  使用的是Native+H5的方式实现的。众所周知的是在Android中，Webview所实现的java与js的交互存在一些安全问题，并且这样的使用方式，没法让一套H5同时适
配Android和iOS两个平台，因此，就需要有一个中间组件来实现js与本地的代码的交互，也就是JsBridge。在Android平台我们选用了开源项目。
整个库的结构也比较简单：一个用来注入的js文件，一个自定义的Webview（包括webViewClient），以及作为载体的BridgeHandler。
  
##### 集成JsBridge   
```aidl
   repositories {  
        maven {url "https://jitpack.io"}  
    }  
    dependencies {  
        compile 'com.github.lzyzsd:jsbridge:1.0.4'
    }
```
  
##### 使用JSBridge  
```aidl
  webView.registerHandler("submitFromWeb", new BridgeHandler(){
      @Override
      public void handler(String data, CallBackFunction function){
          function.onCallBack("submitFrom web exe, response data from java");
      }
  }
```  
  
##### JS调用方式  
```aidl
    WebViewJavascriptBridge.send(
        data,
        function(responseData){
            //java中DefaultHandler所实现的方法中callback所定义的入参
        }
    )
```    
  
##### JSBridge 使用原理  
```aidl
WebViewJavascriptBridge.js　　　　　被注入到各个页面的js文件；提供初始化，注册Handler，调用Handler等方法。
WebViewJavascriptBridge.java　　　　bridge接口文件,定义了发送信息的方法，由BridgeWebView来实现。  
BridgeWebView.java　　　　　　　　　WebView的子类，提供了注册Handler，调用Handler等方法。  
BridgeWebViewClient.java　　　　 　　WebViewClient的子类，重写了ShouldOverrideUrlLoading，onPageFinish，onPageStart等方法。  
BridgeHandler.java　　　　 　　　　　作为Java与Js交互的载体。Java&Js通过Handler的名称来找到响应的Handler来操作。  
DefaultBridgeHandler.java　　　　　　BridgeHandler的子类，不做任何操作。仅为Java提供默认的接收数据的Handler  
CallBackFunction.java　　　　　　　　回调函数，Handler处理完成后，用来给Js发送消息。  
Message.java　　　　　　　　　　　　 消息对象，用来封装与js交互时的json数据，callid，responseid等。  
BridgeUtil.java　　　　　　　　　　　　工具类，提供从Url中提取数据，获取回调方法，注入js等方法  
```

##### JsBridge调用过程  
```aidl
1.Native初始化webview，注册Handler；加载页面完成后，将WebViewJavascriptBridge.js文件注入页面。查询消息队列是否有信息需要被接收。
2.H5页面初始化，注册Handler，查询消息队列是否有信息需要别接收。
3.用户操作，H5调用本地功能：Js将消息内容放在sendMessageQueue中，并设置iframe的src为yy://__QUEUE_MESSAGE__/
4.Webview设置的WebViewClient拦截到约定url，调用Webview的刷新消息队列的方法flushMessageQueue，此方法就是加载了一个url：javascript:WebViewJavascriptBridge._fetchQueue();,这也是Js中定义的方法，另外定义了一个回调；回调方法主要做了两件事：①判断Native是否为此返回数据保有响应回调操作，若有，则执行，若没有，则为判断callId，不为空时为这个callId初始化一个回调。②通过handlername判断是否为默认的Handler还是自定义的Handler，调用相应Handler的handler方法，入参为消息数据内容和第一步中定义的回调。【这段较为难消化，需要阅读代码来理解】
5.Js中_fetchQueue设置了iframe的src，内容为：yy://return/_fetchQueue/+第二步中放入sendMessageQueue中的消息内容。
6.WebViewClient拦截到url为yy://return/，调用WebView的handlerReturnData方法；通过url中定义的方法名，找到第四个步骤中定义的回调，并调用。回调方法走完后，删除此回调方法。
7.如果Js在调用Handler的时候设置了回调方法，也就是在第四步骤中的含有callId，就会调用queueMessage的方法，然后往下就是走Native给Js发送消息的步骤。
8.Ps: Native给Js发送消息的步骤跟上述从第三步骤到第七步骤完全相同，只不过Native和Js对象调换位置即可。
```
  
#### 定义中间通讯的协议   
  上面已经解释了如何实现WebView与H5之前的相互通讯，接下来，我们定义两者这间的通讯格式:  
  本质上通讯格式的定义无非以下几个要点：  
```aidl
1.H5页面需求知道当前页面的打开是不是在APP内  
    我们可以通过JS判断容器的User-Agent以实现判断当前页面是否在我们定的WebView中打开，如判断当前的User-Agent是不是VIEWC-APP  
2.移动端需要向WebView注入几个方法实现JS的调度将数据上报给移动端  
    根据WEBSDK我们抽像的方法凡非以下几个:
    1.1 页面浏览类方法  pageview
    1.2 点击类方法  clickEvent
    1.3 控件类曝光方法 showEvent
    1.4 其它自定义方法  customEvent  
每个方法通过JSON体的字符串信息进行数据传递  
```
  
  因此具体流程如下：  
```aidl
1.页面在WEBVIEW打开
2.页面JS判断我们当前是在ViewC 的WebView中打开  
3.调入APP Native注入有pageView等方法
4.将数据以JSON上报给JAVA代码  
5.整合移动端的信息  
6.Java代码进行格式化处理走Http上报给服务端

```
  
