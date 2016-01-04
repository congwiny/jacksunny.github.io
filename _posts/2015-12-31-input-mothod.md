---
layout:     post
title:      "输入法相关姿势「汇总」(未完成...)"
subtitle:   "主要介绍和输入法有关的一些姿势"
date:       2015-12-31
author:     "JackSunny"
header-img: "img/post-bg-rwd.jpg"
tags:
    - Android
    - 输入法
    - SoftKeyBoard
    - InputMethod

---

# *导语*

本文你会看到

	1. 输入法源码分析
	2. 自定义一个数字键盘输入法
	3. 一个界面输入框调用自定义输入法和系统输入法
	4. 自定义汉字输入法
	5. 系统输入法的一些api（软键盘高度，输入法厂商等）


# 一.和输入法有关的一些常见问题

---

### 1.如何获取软键盘是否弹出状态 ？

当前获取软键盘状态有两种方式:

**A.监听界面view size变化**

> 假定当前界面根ViewGroup为rootView，软键盘隐藏时，它的高度为屏幕高度，当软键盘弹出时，会触发rootView 的 size 变化，从而可以根据rootView高度是否等于屏幕高度-100dp（）来判断可以隐藏或者显示屏幕底部虚拟按键导航栏，同样会引起size变化，为了杜绝这种情况，遂-100dp（）

>> 为什么要减去100dp？在有的手机上，例如`华为的某些机型上底部虚拟导航栏可以隐藏或者消失，同样会引起size变化`，为了杜绝这类情况，遂减去100dp（写到这里吐槽Google一百遍啊一百遍！！！），可见，这种方式也是有缺点的.

布局文件代码

```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/root_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <EditText
        android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="点击我显示输入法" />

</RelativeLayout>
```


```java
        mRootLayout = (RelativeLayout) findViewById(R.id.root_layout);
        mRootLayout.getViewTreeObserver().addOnGlobalLayoutListener(
                new ViewTreeObserver.OnGlobalLayoutListener() {
                    @Override
                    public void onGlobalLayout() {
                        // size变化会回调此方法，从而判断是否弹出软键盘
                    }
                });
```

**B.利用 [InputMethodManager](http://developer.android.com/intl/zh-cn/reference/android/view/inputmethod/InputMethodManager.html) 来获取**

> 这种方法需要提前说清楚的是，调用imm.showSoftInput方法从而得出软键盘状态，同时也会把软键盘强制弹出，这是个很让人纠结的问题，调用imm.hideSoftInputFromWindow也可以得到软键盘状态，但是会强制关闭软键盘（试过得到状态后强制软键盘回到之前状态，屏幕会闪动），所以，慎用（**`有解决办法请告知`**）

```ruby
		InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
        final EditText et = (EditText) findViewById(R.id.editText);
        IMMResult result = new IMMResult();
        imm.showSoftInput(et, 0,result);
        int res = result.getResult();
        if (res == InputMethodManager.RESULT_UNCHANGED_SHOWN){
            // 表示软键盘弹出
        }else{
            // 表示软键盘隐藏
        }
```

```
    private class IMMResult extends ResultReceiver {
        public int result = -1;
        public IMMResult() {
            super(null);
        }

        @Override
        public void onReceiveResult(int r, Bundle data) {
            result = r;
        }

        // poll result value for up to 500 milliseconds
        public int getResult() {
            try {
                int sleep = 0;
                while (result == -1 && sleep < 500) {
                    Thread.sleep(100);
                    sleep += 100;
                }
            } catch (InterruptedException e) {
                Log.e("IMMResult", e.getMessage());
            }
            return result;
        }
    }
```

### 2.如何获取软键盘高度 ？

获取软键盘高度是根据本文前面`一.1.A.获取软键盘是否弹出状态`中得到的，监听size变化，当软键盘弹出后size会改变，就可以拿到软键盘的高度,**`有其他方法请告知`**

```java
// 伪代码
// 屏幕高度
int heightOfScreen = a;
// 变化后的高度
int heightAfterChange = b;
// 键盘高度
int heightOfKeyBoard = heightOfScreen - heightAfterChange;
```

### 3.如何获取系统所有输入法厂商等 ？

> 可以获取系统所有注册输入法的包名等信息，本地维护一个关于厂商和包名的小型数据库即可得到厂商

```java
    InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
    // 得到所有输入法
    List<InputMethodInfo> methodList = imm.getInputMethodList();
    final int length = methodList.size();
    items = new String[length];
    for (int i = 0; i < length; i++) {
        InputMethodInfo inputMethodInfo = methodList.get(i);
        String temp = "input method:" +"\n"
                +"id                    -->"+inputMethodInfo.getId()+"\n"
                +",getPackageName       -->"+inputMethodInfo.getPackageName()+"\n" // 包名
                +",getServiceName       -->"+inputMethodInfo.getServiceName()+"\n"
                +",getSettingsActivity  -->"+inputMethodInfo.getSettingsActivity()+"\n"
                +",getIsDefaultResourceId  -->"+inputMethodInfo.getIsDefaultResourceId()+"\n"
                +",getSubtypeCount  -->"+inputMethodInfo.getSubtypeCount()+"\n"
                +",getComponent         -->"+inputMethodInfo.getComponent();
        Log.i(TAG,temp);
    }
```

### 4.如何获取系统当前输入法厂商等 ？

> 可以获取当前默认输入法的ID（对应于 上文所说 一.3 中 id属性 inputMethodInfo.getId()）

```java
	// 对应于 上文所说 一.3 中 id属性 inputMethodInfo.getId()
	// eg.搜狗：com.sohu.inputmethod.sogou/.SogouIME
    String DEFAULT_INPUT_METHOD = Settings.Secure.getString(
            getContentResolver(),
            Settings.Secure.DEFAULT_INPUT_METHOD
    );
    String INPUT_METHOD_SELECTOR_VISIBILITY = Settings.Secure.getString(
            getContentResolver(),
            Settings.Secure.INPUT_METHOD_SELECTOR_VISIBILITY
    );
    String ENABLED_INPUT_METHODS = Settings.Secure.getString(
            getContentResolver(),
            Settings.Secure.ENABLED_INPUT_METHODS
    );
    String SELECTED_INPUT_METHOD_SUBTYPE = Settings.Secure.getString(
            getContentResolver(),
            Settings.Secure.SELECTED_INPUT_METHOD_SUBTYPE
    );
    String currentInputInfo = "default:"
            +"DEFAULT_INPUT_METHOD:"+DEFAULT_INPUT_METHOD+"\n"
            +"ENABLED_INPUT_METHODS:"+ENABLED_INPUT_METHODS+"\n"
            + "SELECTED_INPUT_METHOD_SUBTYPE:"+SELECTED_INPUT_METHOD_SUBTYPE+"\n"
            +"INPUT_METHOD_SELECTOR_VISIBILITY:"+INPUT_METHOD_SELECTOR_VISIBILITY;
    Log.i(TAG,currentInputInfo);
```

# 二.官方输入法sample源码分析

*目的，通过分析官方源码来增加对输入法原理的理解*

> 从SDK 1.5版本以后，Android就开放它的IMF（Input Method Framework），让我们能够开发自己的输入法。而开发输入法最好的参考就是Android自带的Sample-SoftKeyboard，虽然这个例子仅包含英文和数字输入，但是它本身还算完整和清楚，对我们开始Android开发实战有很大帮助。
>> 下载的`sdk/sample/android-8/` 以及  `sdk/samples/android-23/legacy/` 下都有`SoftKeyboard`源码，我们用android-8中的

### 1.概述

从InputMethodServiceSample项目可以看出实现一个输入法至少需要CandidateView, LatinKeyboard, LatinKeyboardView,SoftKeyboard这四个文件：

* CandidateView负责显示软键盘上面的那个候选区域。
* LatinKeyboard负责解析并保存键盘布局，并提供选词算法，供程序运行当中使用。其中键盘布局是以XML文件存放在资源当中的。比如我们在汉字输入法下，按下b、a两个字母。LatinKeyboard就负责把这两个字母变成爸、把、巴等显示在CandidateView上。
* LatinKeyboardView负责显示，就是我们看到的按键。它与CandidateView合起来，组成了InputView，就是我们看到的软键盘。
* SoftKeyboard继承了InputMethodService，启动一个输入法，其实就是启动一个InputMethodService，当SoftKeyboard输入法被使用时，启动就会启动SoftKeyboard这个Service。


**待续**
---

# 三.自定义数字输入法

---



# 四.一个界面调用多种输入法

---

# 五.自定义中文输入法

---

# 五.感谢

* [Gavin's home](http://blog.csdn.net/deaboway/article/details/6246622)
* 
---