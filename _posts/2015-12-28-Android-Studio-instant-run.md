---
layout:     post
title:      "Android Studio 2.0 的逆天功能Instant Run「转载」"
subtitle:   "主要介绍Instant Run的使用方法"
date:       2015-12-28
author:     "JackSunny"
header-img: "img/post-bg-android.jpg"
tags:
    - Android
    - Android Studio 2.0
    - Instant Run

---

作为一个Android开发者，很多的时候我们需要花大量的时间在bulid，运行到真机（虚拟机）上，对于ios上的Playground羡慕不已，这种情况将在Android Studio 2.0有了很大改善，使用instant run，在第一次运行之后，就可以快速的在真机中看见修改后的结果，不仅仅是UI可以直接显示，还包括代码逻辑。不用再苦苦等build了，节约生命呀！

## 首先要升级到Android Studio 2.0

---

目前Android Studio的2.0版本还在Canary Channel（金丝雀） 上面，所以想体验2.0的同学需要先把升级版本切换到Canary Channel 上面。

> Preferences -> System Settings ->Updates

![](http://segmentfault.com/img/bVq642)

可以切换升级版本

然后 *`check for updates`* 就可以升级了。（如果连接不上升级服务器，请墙一下）

## 升级android tools build

---

*`instant run`* 功能之后再*` android tools build`* 的 *`2.0.0`* 的版本才可以使用。

需要在 *`build.gradle`* 中指定

```java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0-alpha1'
    }
}
```

## 设置instant Run

---

> Preferences -> Build,Execution,Deployment -> Instant Run

![](http://segmentfault.com/img/bVq644)

有关于 Instant Run的设置

	1. 第一个设置是,是否开启Instant Run的，默认是开启。
	2. 第二个是,当代码变动的时候重启activity（亲测没有效果，不知道是不是还不是太完善）
	3. 第三个是，每次变动的时候都有个toast提示下

## 运行Instant Run

---

再没有运行项目的时候，我们的Run图标和以前是一样的。

![](http://segmentfault.com/img/bVq65a)

(话说这个Debug的图标好可爱)

我们写两个类，第一个类有一个按钮，点击后进入第二个activity。

![](http://7xpc6d.com1.z0.glb.clouddn.com/install%20run%20old.png)

准备把临时拜访换成别的字串比如“你好”，同时换掉左边的Icon。它是一个拥有自定义属性的自定义控件，布局代码片段为：

```java
<com.qianmi.shine.widget.CommonLeftIconRightButtonRelativeLayout
        android:id="@+id/ll_sudden_visit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:CLIRBRIconId="@drawable/icon_temp"
        app:CLIRBRTitleName="@string/sudden_visit"
        app:CLIRBRActionIconId="@drawable/btn_go_nor"
        />
```

首先我们需要先跑一下这个项目，然后先点击界面直到上述的界面为止停住不动，这个时候我们再修改上述代码（这一步是必须的，不然的Instant Run功能使用时会出现问题，导致重新运行）

运行了项目之后的图标是这样的：

![](http://segmentfault.com/img/bVq65b)

这个时候我们让模拟器保持在这个页面上，同时修改布局代码成：

```java
<com.qianmi.shine.widget.CommonLeftIconRightButtonRelativeLayout
        android:id="@+id/ll_sudden_visit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:CLIRBRIconId="@drawable/icon_resent"//修改1
        app:CLIRBRTitleName="你好"//修改2
        app:CLIRBRActionIconId="@drawable/btn_go_nor"
        />
```

然后点击带闪电的运行：

![](http://7xpc6d.com1.z0.glb.clouddn.com/splash.png)

可以看到界面快速的刷新成了：

![](http://7xpc6d.com1.z0.glb.clouddn.com/install%20run%20new.png)

## 最后说明

---

需要说明的是，我在使用过程中发现，改Instant Run仅仅适用于布局的修改。即我们可以把一次修改然后到运行看效果看作一个“周期”，在这个周期里面你仅仅修改了xml布局文件，或者说和逻辑代码不相关的文件，那么你点击运行的时候才会触发Instant Run，否则的话，Android Studio还是依然会重新编译运行。

其实想想也是合理的，比如若你修改了代码，而该代码恰好是当前界面的“逻辑前提”，那么你怎么仅仅刷当前界面就能得到正确结果呢？

对于到底目前Instant Run支持哪些形式的代码修改，官方有[一篇文章](https://sites.google.com/a/android.com/tools/tech-docs/instant-run)可供参考

另外google说优化了虚拟机部分，性能提高了50倍，是不是可以抛弃*`Genymotion了`*？

## 感谢

* [姜家志的文章](http://segmentfault.com/blog/jjz)
* [soaringEveryday](http://www.cnblogs.com/soaringEveryday/p/4991563.html)




