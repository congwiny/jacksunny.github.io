---
layout:     post
title:      "Shape自定义"
subtitle:   "很好的一篇关于Shape属性介绍，以及一些坑的填补方案"
date:       2016-2-19
author:     "JackSunny"
header-img: "img/post-bg-rwd.jpg"
tags:
    - shape
---

# 导读

本文主要介绍

> 1. 自定义shape 的 xml方式中的不同属性
> 2. 代码定义shape的用法

---

# 简介

`Android`中提供了各种类型的Drawable，也可以用XML定义各种Drawable。本文重点讲述如何用XML中的shape节点定义GradientDrawable。

将XML定义的drawable文件放在res/drawable目录下。

用XML文件定义GradientDrawable的语法如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```
该文件以`<shape\>`为根结点，其`shape`属性可取四种值：rectangle、oval、line或ring。以上语法格式中虽然列出了很多属性，但是并不是对于所有类型的shape都支持这些属性。下面分别对这四种shape进行讲解。

---

# rectangle

在res/drawable下面用XML文件定义了一个名为rectangle的GradientDrawable，其对应的shape值为rectangle，表明我们定义的drawable的形状是矩形。在layout文件中定义了一个TextView，其使用了上述drawable，如下所示：

```java
<TextView android:text="@string/hello_world"
        android:layout_width="200dp"
        android:layout_height="100dp"
        android:background="@drawable/rectangle" />
```

rectangle.xml定义如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
</shape>
```

--


#### 1.属性: `solid`
> 用来设置面的填充色的，对应的XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <solid android:color="#00ff00" />
</shape>
```
效果如下所示：

![](http://img.blog.csdn.net/20151230205832773)

--


#### 2.属性: `corners `
> 用于设置drawable四个拐角的半径，对应的XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <solid android:color="#00ff00" />
    <corners android:radius="20dp" />
</shape>
```

效果如下所示：

![](http://img.blog.csdn.net/20151230211531002)

> 只有当`shape`属性值为`rectangle`时，`<corners>`节点才会有用。`<corners>`节点的`radius`属性同时定义了四个角的半径，如果想让这四个角的半径不一样，可以分别设置`topLeftRadius`、`topRightRadius`、`bottomLeftRadius`和`bottomRightRadius`属性，当然，`raidus` 后，也可以单独对某个角的角度进行设置

--


#### 3.属性: `padding `
> 用于设置`drawable`的`padding`，可以分别设置`left`、`right`、`top`、`bottom`四个属性，其作用与直接对`TextView`设置的四个`paddingLeft`、`paddingRight`、`paddingTop`、`paddingBottom`属性等价。XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <solid android:color="#00ff00" />
    <corners android:radius="20dp" />
    <padding android:left="10dp"
        android:right="10dp"
        android:top="10dp"
        android:bottom="10dp" />
</shape>
```
效果如下所示：

![](http://img.blog.csdn.net/20151230220530879)

--


#### 4.属性: `size `
> 以用`<size>`节点的width和height属性设置drawable的尺寸。默认情况下，用`<shape>`定义的drawable会自动缩放到包含drawable的View尺寸范围。比如我们有如下的`ImageView`的src设置了`<shape>`定义的drawable，当我们设置了其`scaleType`值为`center`时，`<shape>`中定义的`size`尺寸就会限制drawable缩放。定义的ImageView如下所示：
> > 如果设定 textView的宽高自适应，设定size 宽高，也就等同于 设定 textView的宽高

```java
<ImageView
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:src="@drawable/rectangle"
    android:scaleType="center"
    />
```

对应的drawable定义如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <solid android:color="#00ff00" />
    <corners android:radius="20dp" />
    <size android:width="100dp"
        android:height="100dp" />
</shape>
```
效果如下所示：

![](http://img.blog.csdn.net/20151230222851866)

--



#### 5.属性: `stroke `
> 可用于设置drawable的轮廓线，通过`width`属性设置轮廓线的宽度，通过`color`属性设置轮廓线的颜色。默认情况下，`<stroke>`定义的是实线，除此之外，还可以设置`dashWidth`和`dashGap`属性，如果设置了这两个属性，那么就是虚线。其中，`dashWidth`定义了每个虚线段的长度，`dashGap`定义了两个虚线段之间的距离。对应的XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <solid android:color="#00ff00" />
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <stroke android:width="1dp"
        android:color="#0000ff"
        android:dashWidth="8dp"
        android:dashGap="2dp" />
</shape>
```
效果如下所示：

![](http://img.blog.csdn.net/20151230235143706)

--


#### 6.属性: `gradient `
> 上面我们说到，通过`<solid>`可以设置`drawable`的颜色，但是只是一种纯色，如果想让`drawable`产生渐变效果，可以使用`<gradient>`节点创建渐变色效果。`<gradient>`节点具有以下属性：`type`、`startColor`、`centerColor`、`endColor`、`angle`、`centerX`、`centerY`、`gradientRadius`。其中type有三种取值：`linear`、`radial`和`sweep`，默认值是`linear`。当type取不同的值时，`<gradient>`并不是支持全部属性，下面详细说明。

**6.1 属性: `linear `**

> 当`<gradient>`的type值为linear时，表示颜色是线性渐变的，此时支持startColor、centerColor、endColor、angle这四个属性，其他属性不支持。我们可以通过设置startColor和endColor指定渐变的起始颜色以及终止颜色，XML如下所示：


```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="linear"
        android:startColor="#ff0000"
        android:endColor="#0000ff" />
</shape>
```

我们将startColor设置为红色，endColor设置为蓝色，效果如下所示：

![](http://img.blog.csdn.net/20151231201111512)

我们还可以设置centerColor属性，指定中间色，XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" /&gt;
    <gradient android:type="linear"
        android:startColor="#ff0000"
        android:centerColor="#00ff00"
        android:endColor="#0000ff" />
</shape>
```

我们将中间色设置为绿色，效果如下所示：

![](http://img.blog.csdn.net/20151231201807871)

默认情况下，渐变是从左向右进行的，如果想调整渐变的方向可以设置`angle`属性，angle的默认值为`0`，对应着自左向右渐变,angle的单位是角度，angle的值必须是`45`的倍数，否则不会有渐变效果。我们可以更改angle值，XML如下所示:

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="linear"
        android:startColor="#ff0000"
        android:centerColor="#00ff00"
        android:endColor="#0000ff"
        android:angle="90" />
</shape>
```

我们将angle设置为90度，那么渐变方向就变成了从下上，效果如下所示：

![](http://img.blog.csdn.net/20151231203541693)

**6.2 属性: `radial `**

> 当`<gradient>`的type值为radial时，表示颜色从某点向周围辐射渐变，此时支持除angle之外的其他所有属性。我们必须通过设置gradientRadius属性以指定渐变的辐射半径，通过startColor指定起始颜色，XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="radial"
        android:gradientRadius="50dp"
        android:startColor="#ff0000" />
</shape>
```

我们将startColor设置为红色，效果如下所示：

![](http://img.blog.csdn.net/20151231215022176)

由上图我们可以发现，startColor（红色）从中心沿着圆的半径逐渐变淡。

在设置了startColor的基础上，我们还可以设置centerColor，XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle"&gt;
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="radial"
        android:gradientRadius="50dp"
        android:startColor="#ff0000"
        android:centerColor="#00ff00" />
</shape>
```

效果如下图所示：

![](http://img.blog.csdn.net/20151231215854184)

由上图可以看出，startColor（红色）从中心沿着圆的半径逐渐渐变到centerColor（绿色）。

除了设置startColor、centerColor，还可以设置endColor，XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="radial"
        android:gradientRadius="50dp"
        android:startColor="#ff0000"
        android:centerColor="#00ff00"
        android:endColor="#0000ff" />
</shape>
```

我们将centerColor设置为绿色，效果如下所示：

![](http://img.blog.csdn.net/20151231211248548)

由上图可以看出，startColor（红色）从中心沿着圆的半径逐渐渐变到centerColor（绿色），在指定的半径之外颜色用endColor（蓝色）填充。

其实，startColor、centerColor、endColor这三个属性可以任意组合，大家可以自己尝试一下各种组合的效果。

默认圆心的位置处于drawable的中心，我们可以通过centerX和centerY属性改变渐变圆心的位置，centerX和centerY的取值范围都是0到1，这两个属性的默认值都是0.5，drawable的左上角的centerX和centerY的值都是0，右下角的centerX和centerY的值都是1。我们改变centerX和centerY的值，XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
	android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="radial"
        android:gradientRadius="50dp"
        android:startColor="#ff0000"
        android:centerColor="#00ff00"
        android:endColor="#0000ff"
        android:centerX="0"
        android:centerY="0.5" />
</shape>
```

效果如下所示：

![](http://img.blog.csdn.net/20151231223514781)

**6.3 属性: `sweep `**

> 当<gradient>的type值为sweep时，表示颜色是围绕中心点360度顺时针旋转的，起点就是3点钟位置。

我们可以只设置startColor，XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="sweep"
        android:startColor="#ff0000" />
</shape>
```

我们将startColor设置为红色，效果如下所示：

![](http://img.blog.csdn.net/20151231230516251)

我们也可以只设置endColor设置为蓝色，效果如下所示：

![](http://img.blog.csdn.net/20151231231056081)

我们也可以只设置centerColor设置为绿色，效果如下所示：

![](http://img.blog.csdn.net/20151231231515991)

我们也可以同时设置startColor、centerColor、endColor分别设置为红、绿、蓝，效果如下所示：

![](http://img.blog.csdn.net/20151231232132309)

centerX和centerY的默认值都是0.5，表示中心点的默认位置就是drawable的中心，我们也可以更改centerX和centerY的值，从而更新中心点的位置，XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
android:shape="rectangle">
    <corners android:radius="20dp" />
    <padding android:left="20dp"
        android:top="20dp"
        android:right="20dp"
        android:bottom="20dp" />
    <gradient android:type="sweep"
        android:startColor="#ff0000"
        android:centerColor="#00ff00"
        android:endColor="#0000ff"
        android:centerX="0.25"
        android:centerY="0.5" />
</shape>
```

我们将centerX的值设置为0.25,效果如下所示：

![](http://img.blog.csdn.net/20151231233144174)

---

# oval

在res/drawable下面用XML文件定义了一个名为oval的GradientDrawable，其对应的shape值为oval，表示drawable的形状是椭圆，并将该drawable作为TextView的background。oval和rectangle的主要区别就是drawable的形状不同，大部分的节点属性的作用是相同的。

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
</shape>
```

--


#### 1.属性: `solid`
> 如同rectangle，我们可以通过solid指定drawable的颜色

如下图所示：

![](http://img.blog.csdn.net/20160101113928887)

--


#### 2.属性: `corners `
> oval不支持`<corners>`节点。

--


#### 3.属性: `padding `
> oval同样支持<padding>节点，将四个padding值设置为20dp，效果如下所示：

![](http://img.blog.csdn.net/20160101114641126)

--


#### 4.属性: `size`
> oval不支持`<size>`节点。


--


#### 5.属性: `stroke `
> 同rectangle一样，我们也可以为oval设定`<stroke>`，效果如下所示：

![](http://img.blog.csdn.net/20160101120347246)


--


#### 6.属性: `gradient `
> oval同样支持`<gradient>`节点实现渐变效果，基本效果等同于 rectangle ，组合的效果可以参见上述rectangle中的说明。


---

# line

> 在res/drawable下面用XML文件定义了一个名为line的GradientDrawable，其对应的shape值为`line`，并将该drawable作为TextView的background。当shape为`line`时，表示drawable的形状是线，该线会分割drawable。line只支持`<stroke>`和`<padding>`两个节点，不支持其他的节点。

#### 1.属性: `stroke`

> 各个属性等同于rectangle中stroke属性

<span style="color:red">但是需要注意的地方时，如果在4.0及以上的机器上运行显示的是一条实线！下面有三种方式可以解决这个问题</span>

	1.控件属性中 加入：android:layerType="software"
	2.AndroidManifest.xml中activity属性中加入：android:hardwareAccelerated="false"
	3.代码中针对控件：divider_under_pic.setLayerType(View.LAYER_TYPE_SOFTWARE,null);



XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line">
    <stroke android:color="#0000ff"
        android:width="1dp"
        android:dashWidth="8dp"
        android:dashGap="2dp" />
    <padding android:left="20dp"
        android:right="20dp"
        android:top="20dp"
        android:bottom="20dp" />
</shape>
```

效果如下所示：

![](http://img.blog.csdn.net/20160101130455719)


---

# ring

> 在res/drawable下面用XML文件定义了一个名为ring的GradientDrawable，其对应的shape值为ring，表示drawable的形状是圆环，并将该drawable作为TextView的background。所谓圆环就是大圆套小圆。当shape为ring时，shape有额外的四个属性可用：innerRadius、thickness、innerRadiusRatio、thicknessRatio。

需要特别注意的是，<span style="color:red">在API<=19的真机上使用shape为ring的drawable时，Android有个bug，会不显示drawable，解决办法是将shape设置useLevel属性为true</span>。


--


#### 1.属性: `innerRadius `, `thickness `
> 我们通过innerRadius指定小圆的半径，通过thickness指定大圆和小圆之间的宽度。XML如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:innerRadius="10dp"
    android:thickness="20dp"
    android:useLevel="false">
    <solid android:color="#00ff00" />
    <padding android:left="20dp"
        android:right="20dp"
        android:top="20dp"
        android:bottom="20dp" />
</shape>
```

效果如下所示 (会得到一个 宽高都为 `60dp`的圆环) ：

![](http://img.blog.csdn.net/20160101153625415)


--


#### 2.属性: `innerRadiusRatio `, `thicknessRatio `
> 我们还可以通过innerRadiusRatio指定小圆的半径，innerRadiusRatio的值是float类型，如果其值是9，表示小圆的半径等于TextView宽度的1/9。同样，也可以通过thicknessRatio指定大圆和小圆之间的宽度，其值类型也是float，如果值为8，则表示大小圆之间的宽度等于TextView的1/8。XML文件如下所示：

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:innerRadiusRatio="9"
    android:thicknessRatio="8"
    android:useLevel="false">
    <solid android:color="#00ff00" />
    <padding android:left="20dp"
        android:right="20dp"
        android:top="20dp"
        android:bottom="20dp" />
</shape>
```

效果如下所示：

![](http://img.blog.csdn.net/20160101154424134)



--


#### 3.属性: `stroke `
> 如果给ring设置了<stroke>，那么大小圆都会使用该设置的轮廓线，效果如下所示：

![](http://img.blog.csdn.net/20160101154825537)


--


#### 4.属性: `gradient `
> ring同样支持`<gradient>`节点实现渐变效果，基本效果等同于 rectangle ，组合的效果可以参见上述rectangle中的说明。



---

# 代码动态生成方式

> 仅列出部分属性

```java
mRippleChangedDrawable = new GradientDrawable();
// shape
mRippleChangedDrawable.setShape(GradientDrawable.OVAL);
// useLevel
mRippleChangedDrawable.setUseLevel(false);
// solid
mRippleChangedDrawable.setColor(Color.TRANSPARENT);
// stroke
mRippleChangedDrawable.setStroke(WIDTH_RIPPLE_WIDTH, color);
// size
mRippleChangedDrawable.setSize(100,100);
```
            

---

# 参考

[图文详解Andorid中用Shape定义GradientDrawable](http://www.androidchina.net/4203.html#rd?sukey=014c68f407f2d3e13756cff06b9baa28584128bb29f69add6d8f77c0bf439c9df6b75d47891d253e316fda1c5d523794)

[Android样式的开发:shape篇](http://keeganlee.me/post/android/20150830)

[Android share绘制虚线在手机上显示实线问题](http://wv1124.iteye.com/blog/2187204)




