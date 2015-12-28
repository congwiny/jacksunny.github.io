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

#Volley 初识

## 导语

> * 本文适合有Android开发经验或者想了解杰出网络开源框架的读者
> * 我不生产文章，我只是精华内容的搬运工，本文大部分内容为搜寻网络汇集而来


## 一. 认识 Volley

---

> 将采用 [` 5W-1H `](http://baike.baidu.com/link?url=CmW8mjUYCsw0yKQrkARrcCN6TXM81_gdLHS1GbSNkOuJ20XsoJhgopAk2S2kK59acGCZ3VHBv4E5eDxnK5nmma) 方式来认识 Volley

* ### WHAT

	Volley 是什么？


	> [*`Volley`*](http://developer.android.com/intl/zh-cn/training/volley/index.html)是 Google 推出的 Android 异步网络请求框架和图片加载框架。在 Google I/O 2013 大会上发布。

	>  		名字来源：a burst or emission of many things or a large amount at once

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

```java
public void cancelAll(final Object tag);
public void cancelAll(RequestFilter filter);
public void cancel();
```

###### 6. **利于扩展A**

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


```java
git clone https://github.com/mcxiaoke/android-volley.git
```

请注意，这个是Android Studio 版本。

### 原理初探

讲使用之前，稍微提一点Volley的原理，用以帮助 读者 更好理解稍后的例子

> Volley 中是以队列的方式 处理网络请求：来了一个新的网络请求，它会先检查Cache，如果没有，就会转移到网络，拿到结果后返回结果。

关于Volley的详细原理和架构之后会写文章阐述或者读者在网路上自行搜索。

---

### 使用Volley进行网络请求

我们以一个最简单的一个例子对 Volley Say HelloWorld 吧！

本例子中，我们将用 Volley 将用GET方式访问一个网址（可以看做接口），该接口会返回网页的所有信息。

根据之上的原理初探，我们需要一个队列，需要一个请求，然后处理返回

**1.创建队列 RequestQueue 对象**

```java
RequestQueue mQueue = Volley.newRequestQueue(context);
```
   
   > 注意这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的算法并发地发出这些请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。
   

**2.创建请求实例 StringRequest 对象(包含了对返回的处理回调)**

```java
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

```ruby
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

```java
mQueue.add(stringRequest); 
```

**4.在这之前，一定要加入网络访问权限，否则会失败的**

```java
<uses-permission android:name="android.permission.INTERNET" /> 
```

好了，是不是很简单，拿起这个例子运行一下吧，可以看到类似下图所示的log信息

![](http://7xpc6d.com1.z0.glb.clouddn.com/volly_response.png)

这证明HelloWorld已经成功了！

不过大家都知道，HTTP的请求类型通常有两种，GET和POST，刚才我们使用的明显是一个GET请求，那么如果想要发出一条POST请求应该怎么做呢？StringRequest中还提供了另外一种四个参数的构造函数，其中第一个参数就是指定请求类型的，我们可以使用如下方式进行指定：

```java
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener);
```

可是这只是指定了HTTP请求方式是POST，那么我们要提交给服务器的参数又该怎么设置呢？很遗憾，StringRequest中并没有提供设置POST参数的方法，但是当发出POST请求的时候，Volley会尝试调用StringRequest的父类——Request中的getParams()方法来获取POST参数，那么解决方法自然也就有了，我们只需要在StringRequest的匿名类中重写getParams()方法，在这里设置POST参数就可以了，代码如下所示：

```java
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {
	@Override
	protected Map<String, String> getParams() throws AuthFailureError {
		Map<String, String> map = new HashMap<String, String>();
		map.put("params1", "value1");
		map.put("params2", "value2");
		return map;
	}
};
```
关于简单的HelloWorld小程序就说到这里，上面介绍了简单的GET和POST方式发送数据，并在返回回调中用Log打印出来。而且Volley是开源的，只要你愿意，你可以自由地在里面添加和修改任何的方法，轻松就能定制出一个属于你自己的Volley版本。（关于Volley的修改部分稍后会放出文章链接）

---

HelloWorld例子中我们使用了StringRequest，当前Volley常用到的的Request类型如下

	1. StringRequest - 返回字符串数据;
	2. JsonObjectRequest - 返回JSONArray数据
	3. JsonArrayRequest - 返回JSONObject数据
	4. ImageRequest - 返回Bitmap类型数据

其实上述四种请求Request类型基本类似，他们都是`Request<T>`的子类，只不过封装了特别的返回类型而已。使用方式大同小异。

---

### 使用Volley进行图片请求

你可能会想：`“刚才不是已经返回String 或者 JsonObj类型的数据了么，拿到ImageUrl，用AsyncImageLoader不就行了？”`

没错,这样是可以，但是使用Volley有更简单的姿势~

让我们来用刚才提到的ImageRequest来做你之前用其他方式搞过N次的图片请求。首先回忆下用Volley的请求姿势：

使用Volley有三种使用网络图片的姿势

**1.ImageRequest加载图片**

> 前面我们已经学习过了StringRequest和JsonRequest的用法，并且总结出了它们的用法都是非常类似的，基本就是进行以下三步操作即可：
> 	
	1.创建一个RequestQueue对象。 
	2.创建一个Request对象。
	3.将Request对象添加到RequestQueue里面。

> 其中，StringRequest和JsonRequest都是继承自Request的，所以它们的用法才会如此类似。那么不用多说，今天我们要学习的ImageRequest，相信你从名字上就已经猜出来了，它也是继承自Request的，因此它的用法也是基本相同的,所以，我只贴出创建主要第二步，一三两步略过

```java
ImageRequest imageRequest = new ImageRequest(
		"http://developer.android.com/images/home/aw_dac.png",
		new Response.Listener<Bitmap>() {
			@Override
			public void onResponse(Bitmap response) {
				imageView.setImageBitmap(response);
			}
		}, 0, 0, Config.RGB_565, new Response.ErrorListener() {
			@Override
			public void onErrorResponse(VolleyError error) {
				imageView.setImageResource(R.drawable.default_image);
			}
		});
```

可以看到，ImageRequest的构造函数接收六个参数，第一个参数就是图片的URL地址，这个没什么需要解释的。第二个参数是图片请求成功的回调，这里我们把返回的Bitmap参数设置到ImageView中。第三第四个参数分别用于指定允许图片最大的宽度和高度，如果指定的网络图片的宽度或高度大于这里的最大值，则会对图片进行压缩，指定成0的话就表示不管图片有多大，都不会进行压缩。第五个参数用于指定图片的颜色属性，Bitmap.Config下的几个常量都可以在这里使用，其中ARGB_8888可以展示最好的颜色属性，每个图片像素占据4个字节的大小，而RGB_565则表示每个图片像素占据2个字节大小。第六个参数是图片请求失败的回调，这里我们当请求失败时在ImageView中显示一张默认图片。

程序ok后运行一下，就很快看到在ImageView中显示图片。如下图：

![](http://img.blog.csdn.net/20140413205455484?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.ImageLoader加载图片**

如果你觉得ImageRequest已经非常好用了，那我只能说你太容易满足了 ^_^。实际上，Volley在请求网络图片方面可以做到的还远远不止这些，而ImageLoader就是一个很好的例子。ImageLoader也可以用于加载网络上的图片，并且它的内部也是使用ImageRequest来实现的，不过ImageLoader明显要比ImageRequest更加高效，因为它不仅可以帮我们对图片进行缓存，还可以过滤掉重复的链接，避免重复发送请求。

> 由于ImageLoader已经不是继承自Request的了，所以它的用法也和我们之前学到的内容有所不同，总结起来大致可以分为以下四步：
> 
	1.创建一个RequestQueue对象。
	2.创建一个ImageLoader对象。
	3.获取一个ImageListener对象。
	4.调用ImageLoader的get()方法加载网络上的图片。

下面我们就来按照这个步骤，学习一下ImageLoader的用法吧。首先第一步的创建RequestQueue对象我们已经写过很多遍了，相信已经不用再重复介绍了，那么就从第二步开始学习吧，新建一个ImageLoader对象，代码如下所示：

```java
ImageLoader imageLoader = new ImageLoader(mQueue, new ImageCache() {
	@Override
	public void putBitmap(String url, Bitmap bitmap) {
	}

	@Override
	public Bitmap getBitmap(String url) {
		return null;
	}
});
```

可以看到，ImageLoader的构造函数接收两个参数，第一个参数就是RequestQueue对象，第二个参数是一个ImageCache对象，这里我们先new出一个空的ImageCache的实现即可。

接下来需要获取一个ImageListener对象，代码如下所示：

```java
ImageListener listener = ImageLoader.getImageListener(imageView,
		R.drawable.default_image, R.drawable.failed_image);
```

我们通过调用ImageLoader的getImageListener()方法能够获取到一个ImageListener对象，getImageListener()方法接收三个参数，第一个参数指定用于显示图片的ImageView控件，第二个参数指定加载图片的过程中显示的图片，第三个参数指定加载图片失败的情况下显示的图片。

最后，调用ImageLoader的get()方法来加载图片，代码如下所示：

```java
imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg", listener);
```

get()方法接收两个参数，第一个参数就是图片的URL地址，第二个参数则是刚刚获取到的ImageListener对象。当然，如果你想对图片的大小进行限制，也可以使用get()方法的重载，指定图片允许的最大宽度和高度，如下所示：

```java
imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg",
				listener, 200, 200);
```

现在运行一下程序并开始加载图片，你将看到ImageView中会先显示一张默认的图片，等到网络上的图片加载完成后，ImageView则会自动显示该图，效果如下图所示:

![](http://img.blog.csdn.net/20140413210233046)

虽然现在我们已经掌握了ImageLoader的用法，但是刚才介绍的ImageLoader的优点却还没有使用到。为什么呢？因为这里创建的ImageCache对象是一个空的实现，完全没能起到图片缓存的作用。其实写一个ImageCache也非常简单，但是如果想要写一个性能非常好的ImageCache，最好就要借助Android提供的LruCache功能了，如果你对LruCache还不了解，可以现在网络上自行检索下。下面直接给出代码：

```java
public class BitmapCache implements ImageCache {

	private LruCache<String, Bitmap> mCache;

	public BitmapCache() {
		int maxSize = 10 * 1024 * 1024;
		mCache = new LruCache<String, Bitmap>(maxSize) {
			@Override
			protected int sizeOf(String key, Bitmap bitmap) {
				return bitmap.getRowBytes() * bitmap.getHeight();
			}
		};
	}

	@Override
	public Bitmap getBitmap(String url) {
		return mCache.get(url);
	}

	@Override
	public void putBitmap(String url, Bitmap bitmap) {
		mCache.put(url, bitmap);
	}

}
```

可以看到，这里我们将缓存图片的大小设置为10M。接着修改创建ImageLoader实例的代码，第二个参数传入BitmapCache的实例，如下所示：

```java
ImageLoader imageLoader = new ImageLoader(mQueue, new BitmapCache());  
```

这样我们就把ImageLoader的功能优势充分利用起来了。

**3.NetworkImageView加载图片**

> 除了以上两种方式之外，Volley还提供了第三种方式来加载网络图片，即使用NetworkImageView。不同于以上两种方式，NetworkImageView是一个自定义控制，它是继承自ImageView的，具备ImageView控件的所有功能，并且在原生的基础之上加入了加载网络图片的功能。NetworkImageView控件的用法要比前两种方式更加简单，大致可以分为以下五步：
> 
	1.创建一个RequestQueue对象。
	2.创建一个ImageLoader对象。
	3.在布局文件中添加一个NetworkImageView控件。
	4.在代码中获取该控件的实例。
	5.设置要加载的图片地址。

其中，第一第二步和ImageLoader的用法是完全一样的，因此这里我们就从第三步开始学习了。首先修改布局文件中的代码，在里面加入NetworkImageView控件，如下所示：

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send Request" />
    <com.android.volley.toolbox.NetworkImageView 
        android:id="@+id/network_image_view"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center_horizontal"
        />
</LinearLayout>
```

接着在Activity获取到这个控件的实例，这就非常简单了，代码如下所示：

```java
networkImageView = (NetworkImageView) findViewById(R.id.network_image_view);
```

得到了NetworkImageView控件的实例之后，我们可以调用它的setDefaultImageResId()方法、setErrorImageResId()方法和setImageUrl()方法来分别设置加载中显示的图片，加载失败时显示的图片，以及目标图片的URL地址，如下所示：

```
networkImageView.setDefaultImageResId(R.drawable.default_image);
networkImageView.setErrorImageResId(R.drawable.failed_image);
networkImageView.setImageUrl("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg",
				imageLoader);
```

其中，setImageUrl()方法接收两个参数，第一个参数用于指定图片的URL地址，第二个参数则是前面创建好的ImageLoader对象。

好了，就是这么简单，现在重新运行一下程序，你将看到和使用ImageLoader来加载图片一模一样的效果，这里我就不再截图了。

这时有的朋友可能就会问了，使用ImageRequest和ImageLoader这两种方式来加载网络图片，都可以传入一个最大宽度和高度的参数来对图片进行压缩，而NetworkImageView中则完全没有提供设置最大宽度和高度的方法，那么`是不是使用NetworkImageView来加载的图片都不会进行压缩呢`？

`其实并不是这样的，NetworkImageView并不需要提供任何设置最大宽高的方法也能够对加载的图片进行压缩。这是由于NetworkImageView是一个控件，在加载图片的时候它会自动获取自身的宽高，然后对比网络图片的宽度，再决定是否需要对图片进行压缩。也就是说，压缩过程是在内部完全自动化的，并不需要我们关心，NetworkImageView会始终呈现给我们一张大小刚刚好的网络图片，不会多占用任何一点内存`.这也是NetworkImageView最简单好用的一点吧。

当然了，如果你不想对图片进行压缩的话，其实也很简单，只需要在布局文件中把NetworkImageView的layout_width和layout_height都设置成wrap_content就可以了，这样NetworkImageView就会将该图片的原始大小展示出来，不会进行任何压缩。

## 小结

---

本文主要介绍了volley是什么，并介绍了怎么用volley进行网络请求，图片获取，设置缓存等基础，并没有讲原理，之后的文章会放出。


### 感谢

--- 

* [为什么使用Volley框架](http://www.jeepshoe.org/628560891.htm)
* [郭霖的专栏](http://blog.csdn.net/guolin_blog/article/details/17482095)

