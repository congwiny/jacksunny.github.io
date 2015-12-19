---
layout:     post
title:      "Android圆弧进入及果冻效果「原创」"
subtitle:   "使用圆弧以及果冻效果达到动态进入新activity之目的"
date:       2015-12-19
author:     "JackSunny"
header-img: "img/post-bg-android.jpg"
tags:
    - Android
    - Volley
---


## 导语

---

> 之前发了一个[Drawable实现动画效果](/2015-12-18-Volley-primary)的帖子，有人找我要过相应圆弧绘制算法，仔细想想那个项目中有些地方还是值得分享出来的，趁现在项目不忙，就准备分享出来。

效果如下（即Activity-A 进入 到Activity-B时会出现的动画效果如上）：

![](http://img.blog.csdn.net/20151218150554522)

## 实现思路：

---

主要由两个动画组成，背景动画和按钮动画

1. 背景动画

	- 背景动画可分解为位移上移和圆弧的改变
	- 位移上移可以交给TranslateAnimation 来实现 ，更妙的是可以产生动画插值用来绘制椭圆 
	- 弧形改变可以用canvas来绘制，那么动态计算并绘制椭圆即可
	- 动态计算椭圆可以用TranslateAnimation产生的动画插值
	
	
2. 按钮动画

   - 根据图片可以看到是新的图片逐渐扩大，从一个点开始逐渐遮盖住了旧的图片，这一点可以用BitmapShader来实现
   - 其他就没了


## 会用到的技术点

---

1.[**TranslateAnimation的使用？**](http://blog.csdn.net/candicelijx/article/details/18088823)

2.[**Canvas 绘制圆弧？**](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1212/703.html)

```
    /**
     *
     * @param canvas 绘制椭圆
     */
    private void drawOval(Canvas canvas){
        // 起始 角度
        float startAngle = 0f;
        // 划过角度
        float sweepAngle = 180;
        final RectF rect = new RectF(0,0,400,100);
        Path path = new Path();
        path.addArc(rect,startAngle,sweepAngle);
    }
```

3.[**Canvas 叠加模式？**](http://www.cnblogs.com/DonkeyTomy/articles/3215137.html)

4.[**BitmapShader的使用？**](http://blog.csdn.net/aigestudio/article/details/41799811)

> Shader是着色器，BitmapShader是bitmap着色器，可以将一个Bitmap对象绘制到画布上.

5.[**阻尼振动函数如何实现？**](http://mobile.51cto.com/iphone-480600.htm)

> 阻尼振动曲线如下图

>![](http://s2.51cto.com/wyfs02/M02/6E/A7/wKioL1WCMCvzd5EGAAA_XuLQG3k707.jpg)

> 其实分析曲线后就能发现 其实就是指数函数（无限趋近于0）和 余弦曲线的结合而已。


## 实现过程

---

因为主要是背景动画和按钮动画，所以也拆成两部分

#### 背景动画的实现

##### *动画过程分析*

> 1. 位移动画可以不表，因为很简单，以一种速率 最多加一个速度叠加器即可
> 2. 椭圆可以看到先是移出时椭圆向下最大，然后慢慢缩小，最后在位移到 整体 1/2处椭圆为0，然后又向屏幕上方鼓出，慢慢增大，到 位移到最大位置时椭圆最大
> 3. 到最大位移位置后，圆弧以阻尼振动慢慢缩小，及至椭圆为0

##### *动画过程实现* 

**1. 创建位移动画**

同时创建出 0f - 1.0f 的时间插值

```
    /**
     *
     * @return 得到上移动画
     */
    private AnimationSet getAnimationSet(){
        // 之所以用 AnimationSet，是为了可能加入进来的其他动画效果
        final AnimationSet animationSet = new AnimationSet(true);
        animationSet.setDuration(1600);
        // TranslateAnimation.RELATIVE_TO_SELF 相对位置
        TranslateAnimation translateAnimation = new TranslateAnimation(
                TranslateAnimation.RELATIVE_TO_SELF,0,
                TranslateAnimation.RELATIVE_TO_SELF,0,
                TranslateAnimation.RELATIVE_TO_SELF,1.0f,
                TranslateAnimation.RELATIVE_TO_SELF,0){
            /**
             * 回调间隔在10 ms 左右
             * @param interpolatedTime 即为时间插值 0f - 1f，
             * @param t
             */
            @Override
            protected void applyTransformation(float interpolatedTime, Transformation t) {
                super.applyTransformation(interpolatedTime, t);
                // 半屏动画 设置插值
                mHalfScreenDrawable.setPercent(interpolatedTime);
                // 按钮动画 设置插值
                mRippleButtonDrawable.setPercent(interpolatedTime);
            }
        };
        animationSet.addAnimation(translateAnimation);
        return animationSet;
    }
```

**2. 创建绘制弧线**

向下的弧线可以理解为一块正方形和椭圆重叠，当绘制上/下半椭圆以及正确的混合模式即可得到一个正确的圆弧

```
// 图片混合模式为
mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.XOR));
```

![](http://img.blog.csdn.net/20151218181054959)

即 `当时间插值 为 0f - 0.5f的时候，控制椭圆所在矩形高度由大减小到0，同时绘制椭圆下半部`

  `当时间插值为 0.5f - 1.0f的时候，控制椭圆所在矩形高度由0增大到最大，同时绘制椭圆上半部`
  
  `当到1.0f时，根据阻尼振动函数 计算出递增插值下 包含椭圆 矩形的高度绘制半部椭圆即可`
  
  ```
    /**
     * 设置时间插值，并根据插值计算椭圆所在矩形高度，从而得到不同的椭圆弧度
     * @param value
     */
    public void setPercent(float value) {
        // 向下圆弧，绘制下半部椭圆
        if (value <= 0.5f) {
            mStartAngle = 0;
            final float top = mBottomRectF.top - mHeightTopRect * (0.5f - value);
            final float bottom = mBottomRectF.top + mHeightTopRect * (0.5f - value);
            mTopRectF.set(mTopRectF.left, top, mTopRectF.right, bottom);
        } else {
            // 向上圆弧，绘制上半部椭圆
            mStartAngle = 180;
            final float top = mBottomRectF.top - mHeightTopRect * (value * 2f - 1f) / 2f;
            final float bottom = mBottomRectF.top + mHeightTopRect * (value * 2f - 1f) / 2f;
            mTopRectF.set(mTopRectF.left, top, mTopRectF.right, bottom);
            if (value == 1.0f) {
                // 当到1.0f时，开始绘制阻尼振动
                startBackAnimation();
            }
        }
        invalidateSelf();
    }
  
  ```

```
    /**
     * // 当到1.0f时，开始绘制阻尼振动画
     */
    private void startBackAnimation() {
        isInBounceMode = true;
        handler.sendEmptyMessageDelayed(100, 5);
    }
```

```
    /**
     * TODO: 15/12/8 要修改为弱引用
     */
    private Handler handler = new Handler() {
        @Override
        public void dispatchMessage(Message msg) {
            super.dispatchMessage(msg);
            invalidateSelf();
            // 根据当前 阻尼函数计算得出的Y坐标计算得到包含椭圆矩形的上下Y坐标
            final float top = mBottomRectF.top - Math.abs(mBounceY) / 2;
            final float bottom = mBottomRectF.top + Math.abs(mBounceY) / 2;
            mTopRectF.set(mTopRectF.left, top, mTopRectF.right, bottom);

            // 根据当前 阻尼函数计算得出的Y坐标计算得到startAngel
            if (mBounceY > 0) {
                mStartAngle = 180;
            } else {
                mStartAngle = 0;
            }
        }
    };
```

```
    /**
     * 根据cursor插值，计算 阻尼 Y实时坐标
     * @param x
     * @return 阻尼振动函数 
     */
    private double getBounceY(long x) {
        if (x == 0)
            return 0;
        // 指数函数
        double bounceRate = Math.exp(-0.06 * x);
        if (bounceRate < 0.0001) {
            isInBounceMode = false;
        }
        // 增益函数和余弦曲线合并，就是阻尼函数曲线
        return bounceRate * Math.cos(0.3 * x);
    }
```

到这里，绘制背景动画关键代码已经完毕

#### 按钮动画的实现

##### *动画过程分析*
> 1. 可以看到是一张旧图片上，新的图片逐渐扩大，发现其实是两个动画的集合 ，一个是新的图片不停变大（保持圆形），第二个是新的图片圆心坐标有变化
> 2. 更深一步分析是新图片在 时间插值 0f-1f，半径从0-最大，圆心从 图片3/4处，变为最后1/2处

##### *动画过程实现* 

时间插值可以用之前创建出来的

如果你已经看过了并指导 BitmapShader是干什么的那么应该已经有了思路了吧？

```
    /**
     * 构造参数
     *
     * @param bitmap    新的图片
     * @param oldBitmap 旧图片
     */
    public RippleButtonDrawable(Bitmap bitmap, Bitmap oldBitmap) {
        mBitmapNew = bitmap;
        mBitmapOld = oldBitmap;
        // 设置图片着色器
        BitmapShader bitmapShader = new BitmapShader(bitmap, Shader.TileMode.CLAMP,
                Shader.TileMode.CLAMP);
        mPaintNew = new Paint();
        mPaintNew.setAntiAlias(true);
        mPaintNew.setShader(bitmapShader);
        mWidth = mBitmapNew.getWidth();
        mHeight = mBitmapNew.getHeight();
        mPaintOld = new Paint();
    }
```

```
    @Override
    public void draw(Canvas canvas) {
        // 绘制旧图片
        canvas.drawBitmap(mBitmapOld, 0, 0, mPaintOld);
        // 绘制圆形新图片
        canvas.drawCircle(
                getXCoordinateOfCenter(mPercent),
                getYCoordinateOfCenter(mPercent),
                getRadiusWithPercent(mPercent),
                mPaintNew);
    }
```

就这样就好了，是不是很简单？？？

`有更多的问题可以联系我`






















