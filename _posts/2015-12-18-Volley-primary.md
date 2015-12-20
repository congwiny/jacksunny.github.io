---
layout:     post
title:      "Volley入门及使用「汇总」"
subtitle:   "关于Volley入门知识及如何使用，扩展方法"
date:       2015-12-18
author:     "JackSunny"
header-img: "img/post-bg-android.jpg"
tags:
    - Android
    - Volley
---

# Volley 初识

## 导语

> * 本文适合有Android开发经验或者想了解杰出网络开源框架的读者
> * 我不生产文章，我只是精华内容的搬运工，本文大部分内容为搜寻网络汇集而来


## 一. 认识 Volley

---

> 将采用 [` 5W-1H `](http://baike.baidu.com/link?url=CmW8mjUYCsw0yKQrkARrcCN6TXM81_gdLHS1GbSNkOuJ20XsoJhgopAk2S2kK59acGCZ3VHBv4E5eDxnK5nmma) 方式来认识 Volley

* ### WHAT

	Volley 是什么？


	> [*`Volley`*](http://developer.android.com/intl/zh-cn/training/volley/index.html)是 Google 推出的 Android 异步网络请求框架和图片加载框架。在 Google I/O 2013 大会上发布。

	>     名字来源：a burst or emission of many things or a large amount at once

* ### WHY

	我们为什么使用它 ？

###### 1. **简化了Http请求的操作**
 
>    HttpURLConnection和HttpClient的用法还是稍微有些复杂的，如果不进行适当封装的话，很容易就会写出不少重复代码。

###### 2. **异步下载，结果可以直接在主线程进行操作**

> 在Android4.0以后，会发现，只要是写在主线程（就是Activity）中的HTTP请求，运行时都会报错，这是因为Android在4.0以后为了防止应用的ANR（aplication Not Response）异常.

> 而Volley则完全不用考虑这些问题。

###### 3. **JSON，图像等返回结果的处理**

> 在http请求之后一般都需要对数据进行预处理，尤其是对图片的预处理，而Volley对这些操作进行了基本的封装。在简单的使用情景下，基本够用了。但是如果需要对其进行一些复杂的操作，就需要增加自己的请求类。

> 可以直接将返回值设置给ImageView。

###### 4. **多级缓存**

> 我们在主线程中调用RequestQueue的add()方法来添加一条网络请求，这条请求会先被加入到缓存队列当中，如果发现可以找到相应的缓存结果就直接读取缓存并解析，然后回调给主线程。如果在缓存中没有找到结果，则将这条请求加入到网络请求队列中，然后处理发送HTTP请求，解析响应结果，写入缓存，并回调主线程。
	
###### 5. **多级别取消请求api**

> 某些情况下我们需要取消已经添加到队列的请求,volley对这一操作进行了很好的封装.

> 比如Activity被终止之后，如果继续使用其中的Context等，除了无辜的浪费CPU，电池，网络等资源，有可能还会导致程序crash，所以，我们需要处理这种一场情况。 

> 使用Volley的话，我们可以在Activity停止的时候，同时取消所有或部分未完成的网络请求。Volley里所有的请求结果会返回给主进程，如果在主进程里取消了某些请求，则这些请求将不会被返回给主线程。

``` java
public void cancelAll(final Object tag);
public void cancelAll(RequestFilter filter);
public void cancel();
```

###### 6. **利于扩展**

> 可以自定义请求request（即定义request数据类型）和返回response（即定义返回数据类型)。

###### 7. **系出名门**

> Google自己出品，品质有保证。

* ### WHERE&WHEN

	在哪里&什么时候使用它？
	
	> 发布时有一个很形象的配图
	![](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/network/volley/image/volley.png)

	从名字由来和配图中无数急促的火箭可以看出 Volley 的特点：特别适合数据量小，通信频繁的网络操作。（其实 应用中绝大多数的网络操作都属于这种类型）。
	
	我们平常用到的请求列表数据，更新信息等大部分（非大文件）都属于此等范畴。

* ### WHO

	谁来使用它？
	
	* 推荐小白及入门程序员略过，先学习使用Thread+handler或者AsyncTask等入门方法，在使用Volley
	* 推荐普通程序员来使用
	* 推荐文艺程序员可以理解原理修改后使用（因为volley也是有自己的缺点的，后提），适合自己的才是最好的

* ### HOW

	讲了这么多，想必对Volley是什么已经有了一个理解了，那么Volley怎么用呢？

## 二. 初级使用 Volley

---

### 代码下载

Volley是发布在Google Code上的，现在一般都是从GitHub上找官方的镜像代码，如 [Volley-GitHub地址](https://github.com/mcxiaoke/android-volley) (an unofficial mirror)。

```
git clone https://github.com/mcxiaoke/android-volley.git
```
请注意，这个是Android Studio 版本。

### 原理初探

讲使用之前，稍微提一点Volley的原理，用以帮助 读者 更好理解稍后的例子

Volley 中是以队列的方式 处理网络请求：来了一个新的网络请求，它会先检查Cache，如果没有，就会转移到网络，拿到结果后返回结果。

关于Volley的详细原理和架构之后会写文章阐述或者读者在网路上自行搜索。


### HelloWorld

我们以一个最简单的一个例子对 Volley Say HelloWorld 吧！

本例子中，我们将用 Volley 将用GET方式访问一个网址（可以看做接口），该接口会返回网页的所有信息。

根据之上的原理初探，我们需要一个队列，需要一个请求，然后处理返回

**1.创建队列 RequestQueue 对象**

```
   RequestQueue mQueue = Volley.newRequestQueue(context);
```
   
   > 注意这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的算法并发地发出这些请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。
   

**2.创建请求实例 StringRequest 对象(包含了对返回的处理回调)**

```
String url = "http://www.baidu.com";
StringRequest stringRequest = new StringRequest(url,  
      new Response.Listener<String>() {  
         @Override  
         public void onResponse(String response) {  
            Log.d("TAG", response);  
         }  
      }, new Response.ErrorListener() {  
         @Override  
         public void onErrorResponse(VolleyError error) {  
            Log.e("TAG", error.getMessage(), error);  
         }  
      });
```

> 可以看到，这里new出了一个StringRequest对象，StringRequest的构造函数需要传入三个参数，第一个参数就是目标服务器的URL地址，第二个参数是服务器响应成功的回调，第三个参数是服务器响应失败的回调。其中，目标服务器地址我们填写的是百度的首页，然后在响应成功的回调里打印出服务器返回的内容，在响应失败的回调里打印出失败的详细信息

**3.将请求实例对象 StringRequest 加入 RequestQueue 中，即可自动执行**

```
mQueue.add(stringRequest); 
```

**4.在这之前，一定要加入网络访问权限，否则会失败的**

```
<uses-permission android:name="android.permission.INTERNET" /> 
```

好了，是不是很简单，拿起这个例子运行一下吧，可以看到类似下图所示的log信息

![](http://7xpc6d.com1.z0.glb.clouddn.com/volly_response.png)

这证明已经成功了！


`未完待续`










### 参考以下文章
--- 
* [为什么使用Volley框架](http://www.jeepshoe.org/628560891.htm)
* [郭霖的专栏](http://blog.csdn.net/guolin_blog/article/details/17482095)

