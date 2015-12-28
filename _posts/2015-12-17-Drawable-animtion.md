---
layout:     post
title:      "Drawable根据时间插值自定义动画「原创」"
subtitle:   "关于Drawable的不常用之用法介绍"
date:       2015-12-17
author:     "JackSunny"
header-img: "img/post-bg-ios9-web.jpg"
tags:
    - Android
    - Drawable
    - Animation
    - Canvas
---

##本文会带来什么？

* xml如何自定义圆角 shapeDrawable？环形？三角形？
* drawable有多少种？用法是什么？
* 绘制圆弧用法？原理

**上面的统统没有！只说将一个关于Drawable不常用到的用法**

---

### 我们要实现的效果如下图

![](httP://img.blog.csdn.net/20151217105800205)


> *第一个想法你想用什么来实现？在view 的 onDraw中实现？这也许是很多人的第一个想法，我也是*
> *但想起之前在github上翻看某大神源码时看到用自定义Drawable + 动画器插值实现了一个太阳冉冉升起的动画，就想采用类似的做法来做*


### 具体思路如下：

1. gif图片中的弧形动画以及阻尼动画，说白了也就是canvas绘制圆弧等的组合，即根据时间变化计算绘制相应图形即可
2. Drawable中有draw(Canvas canvas)方法，在此方法中可通过canvas进行绘制
3. View实现TranslationAnimation，可实现整体的上移或者下移，同时创建时间插值
4. 把View动画创建的时间插值动态传递给Drawable，Drawable根据时间插值，动态计算并绘制图形

### 好处如下：

1. 使用Drawable而不是在View中来绘制，不会影响到之前View中onDraw的代码和逻辑
2. Drawable一般当做背景，与view耦合很小
3. Animation产生插值，可设置不同加速器 产生不同效果

**可见，我们所有需要的前置条件都已经满足了，现在开始 实现**

***

# 以下为部分代码


### 1.创建TranslationAnimation产生时间插值

```java
final AnimationSet as = new AnimationSet(true);
as.setDuration(600);
TranslateAnimation translateAnimation = new TranslateAnimation(
        ScaleAnimation.RELATIVE_TO_SELF,0,
        ScaleAnimation.RELATIVE_TO_SELF,0,
        ScaleAnimation.RELATIVE_TO_SELF,1.0f,
        ScaleAnimation.RELATIVE_TO_SELF,0){
    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        super.applyTransformation(interpolatedTime, t);
        myDrawable.setPercent(interpolatedTime, true);
    }
};
as.addAnimation(translateAnimation);
return as;
```
		
        
### 2.自定义Drawable及讲解

```java
public class MyAnimationDrawable3 extends Drawable implements Drawable.Callback {
	private Paint mPaint;
	private int mWidth;
	private int mHeight;
	public MyAnimationDrawable3(int width,int height) {
    	mPaint = new Paint();
    	mWidth = width;
    	mHeight = height;
	}

	@Override
	public void draw(Canvas canvas) {
    	// 主要绘制代码
	//  canvas.drawRoundRect(rectF, 30, 30, mPaint);
	}

	@Override
	public int getIntrinsicWidth() {
    	return mWidth;
	}

	@Override
	public int getIntrinsicHeight() {
    	return mHeight;
	}

	@Override
	public void setAlpha(int alpha) {
    	mPaint.setAlpha(alpha);
	}

	@Override
	public void setColorFilter(ColorFilter cf) {
   	 	mPaint.setColorFilter(cf);
	}

	@Override
	public int getOpacity() {
    	return PixelFormat.TRANSLUCENT;
	}

	@Override
	public void invalidateDrawable(Drawable who) {
    	final Callback callback = getCallback();
    	if (callback != null) {
        	callback.invalidateDrawable(this);
    	}
	}

	@Override
	public void scheduleDrawable(Drawable who, Runnable what, long when) {
    	final Callback callback = getCallback();
    	if (callback != null) {
        	callback.scheduleDrawable(this, what, when);
    	}
	}

	@Override
	public void unscheduleDrawable(Drawable who, Runnable what) {
    	final Callback callback = getCallback();
    	if (callback != null) {
        	callback.unscheduleDrawable(this, what);
    	}
	}
}
```
    
     
> 其中draw中负责具体绘制，getIntrinsicWidth,getIntrinsicHeight为指定宽高，setAlpha,setColorFilter等都很简单，最后，该类实现Drawable.CallBack，实现最后三个方法，作用是“把你实现的该接口注册到动画Drawable上可以实现动态Drawable的调度和执行” 

### 3.传递时间插值进入Drawable

```java
 /**
 * @param percent 时间插值
 */
public void setPrecent(float percent){
    if (percent > 0.5f){
        //逻辑代码，计算弧度等
    }else{
        //逻辑代码，计算弧度等
    }
    // 刷新 draw 图形
    invalidateSelf();
}
```

> 传递进去时间插值后，即可根据时间插值计算相关弧度，在draw中进行绘制即可完成动态图中的效果（绘制逻辑 就不说了，大概为插值0-1绘制向上半圆弧，到顶点根据阻尼振动函数绘制半径变化的圆弧）       
        
        
        
        
        
        
        
    