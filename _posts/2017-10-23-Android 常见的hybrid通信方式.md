---
layout: post
title: Android 常见的 hybrid 通信方式
tags: [Android, hybrid, webview, 通信]
---

> hybrid通信，主要就是前端的js和Android端的通信方式，能够实现Web调用Native代码的方法主要有以下方法：

1. Schema：WebView 拦截页面跳转
2. JavascriptInterface
3. WebChromeClient.onJsPrompt()

> 而native代码通信js只能使用WebView.loadUrl(“javascript:function()”)，若需要Native与js进行双向通信，则可使用JSBridge，同时，还有一些开源框架如：safe-java-js-webview-bridge等

### JSInterface
> JSInterface 是Android原生提供的来进行 JS 和 Java 通信的方法，下面介绍它的具体方式。

首先先看一段html代码

```
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh-CN" dir="ltr">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <script type="text/javascript">
        function showToast(toast) {
            javascript:control.showToast(toast);
        }
        function log(msg){
            console.log(msg);
        }
    </script>

</head>

<body>
<input type="button" value="toast"
       onClick="showToast('Hello world')" />
</body>
</html>
```
很简单，一个button，点击这个button就执行js脚本中的showToast方法。

![image](https://raw.githubusercontent.com/rickyzhan/rickyzhan.github.io/master/assets/img-folder/jsinterface.png) 

而这个showToast方法做了什么呢？

```
function showToast(toast) {
    javascript:control.showToast(toast);
}
```
可以看到control.showToast，这个是什么我们等下再说，下面看java的代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="zjutkz.com.tranditionaljsdemo.MainActivity">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </WebView>

</LinearLayout>
public class MainActivity extends AppCompatActivity {

    private WebView webView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        webView = (WebView)findViewById(R.id.webView);

        WebSettings webSettings = webView.getSettings();

        webSettings.setJavaScriptEnabled(true);

        webView.addJavascriptInterface(new JsInterface(), "control");

        webView.loadUrl("file:///android_asset/interact.html");
    }

    public class JsInterface {

        @JavascriptInterface
        public void showToast(String toast) {
            Toast.makeText(MainActivity.this, toast, Toast.LENGTH_SHORT).show();
            log("show toast success");
        }

        public void log(final String msg){
            webView.post(new Runnable() {
                @Override
                public void run() {
                    webView.loadUrl("javascript: log(" + "'" + msg + "'" + ")");
                }
            });
        }
    }
}
```
首先界面很简单，一个WebView。在对应的activity中做的事也就几件，首先打开js通道
```
WebSettings webSettings = webView.getSettings();
webSettings.setJavaScriptEnabled(true);
```
然后通过WebView的addJavascriptInterface方法去注入一个自己写的interface

```
webView.addJavascriptInterface(new JsInterface(), "control");

public class JsInterface {

        @JavascriptInterface
        public void showToast(String toast) {
            Toast.makeText(MainActivity.this, toast, Toast.LENGTH_SHORT).show();
            log("show toast success");
        }

        public void log(final String msg){
            webView.post(new Runnable() {
                @Override
                public void run() {
                    webView.loadUrl("javascript: log(" + "'" + msg + "'" + ")");
                }
            });
        }
    }
```
可以看到这个interface我们给它取名叫control

最后loadUrl

```
webView.loadUrl("file:///android_asset/interact.html");
```
好了，让我们再看看js脚本中的那个showToast()方法

```
function showToast(toast) {
            javascript:control.showToast(toast);
        }
```
这里的control就是我们的那个interface，调用了interface的showToast方法

```
@JavascriptInterface
public void showToast(String toast) {
    Toast.makeText(MainActivity.this, toast, Toast.LENGTH_SHORT).show();
    log("show toast success");
}
```
可以看到先显示一个toast，然后调用log()方法，log()方法里调用了js脚本的log()方法

```
function log(msg){
    console.log(msg);
}
```
js的log()方法做的事就是在控制台输出msg

> 这样我们就完成了js和java的互调，是不是很简单。但是大家想过这样有什么问题吗？如果你使用的是AndroidStudio，在你的webSettings.setJavaScriptEnabled(true);这句函数中，AndroidStudio会给你一个warning，这个提示的意思呢，就是如果你使用了这种方式去开启js通道，你就要小心XSS攻击了，具体的大家可以参考wooyun上的这篇文章。

> 虽然这个漏洞已经在Android 4.2上修复了，就是使用@JavascriptInterface这个注解。但是你得考虑兼容性啊，你不能保证，尤其在中国这样碎片化严重的地方，每个用户使用的都是4.2+的系统。所以基本上我们不会再利用Android系统为我们提供的addJavascriptInterface方法或者@JavascriptInterface注解来实现js和java的通信了。

### JSBridge

> JSBridge，顾名思义，就是和js沟通的桥梁

##### 原理：通过js的window.prompt方法将事先定义好的协议文本传输到java层，然后java层进行解析并调用相应的方法，最后通过callback将结果返回给js脚本

![image](https://raw.githubusercontent.com/rickyzhan/rickyzhan.github.io/master/assets/img-folder/webview.png) 

首先先说思路，有经验的同学可能都知道Android的WebView中有一个WebChromeClient类，这个类其实就是用来监听一些WebView中的事件的，我们发现其中有三个这样的方法:

```
@Override
public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
    return super.onJsPrompt(view, url, message, defaultValue, result);
}

@Override
public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    return super.onJsAlert(view, url, message, result);
}

@Override
public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    return super.onJsConfirm(view, url, message, result);
}
```
这三个方法其实就对应于js中的alert(警告框)，comfirm(确认框)和prompt(提示框)方法，那这三个方法有什么用呢？前面我们说了JSBridge的作用是提供一种js和java通信的框架，其实我们可以利用这三个方法去完成这样的事。比如我们可以在js脚本中调用alert方法，这样对应的就会走到WebChromeClient类的onJsAlert()方法中，我们就可以拿到其中的信息去解析，并且做java层的事情。那是不是这三个方法随便选一个就可以呢？其实不是的，因为我们知道，在js中，alert和confirm的使用概率还是很高的，特别是alert，所以我们最好不要使用这两个通道，以免出现不必要的问题;

好了，说到这里我们前期的准备工作也就做好了，其实就是通过重写WebView中WebChromeClient类的onJsPrompt()方法来进行js和java的通信。

首先大家都知道http是什么，其实我们的JSBridge也可以效仿一下http，定义一个自己的协议。比如规定sheme，path等等。下面来看一下一些的具体内容：

hybrid://JSBridge:1538351/method?{“message”:”msg”}

是不是和http协议有一点像，其实我们可以通过js脚本把这段协议文本传递到onPropmt()方法中并且进行解析。比如，sheme是hyrid://开头的就表示是一个hybrid方法，需要进行解析。后面的method表示方法名，message表示传递的参数等等。

- 在js脚本中把对应的方法名，参数等写成一个符合协议的uri，并且通过window.prompt方法发送给java层。 

- 在java层的onJsPrompt方法中接受到对应的message之后，通过JsCallJava类进行具体的解析。 

- 在JsCallJava类中，我们解析得到对应的方法名，参数等信息，并且在map中查找出对应的类的方法。

可能有人会觉得，直接在JsCallJava类中定义方法不就好了，这样还省的去写那么多的逻辑。

但是，如果把所有js脚本想要调用的方法都写在JsCallJava类中，会增加 JsCallJava 类的维护难度，做不到业务逻辑分离和解耦。

### UrlRouter

> UrlRouter，严格意义上，它并不算是js和java的通信，它只是一个通过url来让前端唤起native页面的框架。不过千万不要小看它的作用，如果协议定义的合理，它可以让前端，Android和iOS三端有一个高度的统一，十分方便。

思路：
其实吧，这个思路比JSBridge还要简单，就是我们通过自己实现的框架去拦截前端同学写的url，发现如果是符合我们UrlRouter的协议的话，就跳转到相应的页面。

至于怎么拦截呢？当然是通过WebViewClient类的shouldOverrideUrlLoading方法。

### safe-java-js-webview-bridge
> 这个开源项目用到的原理也是JSBridge，都是js调用prompt函数，传输一些参数，onJsPrompt方法拦截到prompt动作，然后解析数据，最后调用相应的Native方法。

如果方法正确执行，call方法就返回一个json字符串code=200，否则就传code=500，这个信息会通过prompt方法的返回值传给js，这样Html 5 代码就能知道有没有正确执行了。

参考：

[Android native和h5混合开发几种常见的hybrid通信方式](https://blog.csdn.net/jdsjlzx/article/details/51376739)

[Android Hybrid App通信](https://www.jianshu.com/p/1cf25c712040)

