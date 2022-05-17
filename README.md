# 自定义UI 

> 制作一个游动的锦鲤

先看效果图（注意那亿点点细节）：

![Fish](https://user-images.githubusercontent.com/30100887/168629698-ad7e1af5-edf6-4c01-9360-7785aa6b7995.gif)


## 1. 实现静态的效果

> 实现小鱼的绘制

​	Canvas 可以绘制正方形， 圆形，等， 实现如图绘制需要掌握贝塞尔曲线， 贝塞尔曲线可以绘制任何东西

静态效果：

![╬ó╨┼═╝╞¼_20220516203841 (2)](https://user-images.githubusercontent.com/30100887/168629941-123cb5ab-e716-4b25-8c67-10f8e5d12d66.jpg)


不规则得弧度， 需要贝塞尔曲线来画， 就好比鱼的身体部分。

1. 一阶贝塞尔曲线 （一个起点和一个结束点）

![image-20220516230628826](https://user-images.githubusercontent.com/30100887/168630004-fe216a7e-84f3-41e6-85e0-d9306ddcf38e.png)			

2. 二阶贝塞尔曲线（一个起点一个结束点和一个控制点）

![image-20220516230723915](https://user-images.githubusercontent.com/30100887/168630109-579f327c-26eb-458d-a85b-551779d0f2e6.png)

```java
mPath.quadTo(controlRight.x, controlRight.y,topRightPoint.x, topRightPoint.y);
```

3. 三阶贝塞尔曲线（一个起点一个结束点两个控制点）
![image-20220516230905993](https://user-images.githubusercontent.com/30100887/168630259-3ff71c1d-9391-4b29-a39d-080a28d81340.png)


> 计算鱼的宽高 
![╬ó╨┼═╝╞¼_20220516215419 (2)](https://user-images.githubusercontent.com/30100887/168630373-fc49e7a3-a387-4bc4-a7e7-3a2bc92a68c9.jpg)


> 绘制鱼鳍
![image-20220516231808945](https://user-images.githubusercontent.com/30100887/168630414-6abae73a-1f87-4fad-b15c-3dc901aeb6c1.png)


Drawable

- 一种可以再Canvas上进行绘制得抽象得概念
- 颜色， 图片都可以事一个Drawable
- Drawable可以通过XML进行定义， 或通过代码创建
- Android中Drawable是一个抽象类，每个具体得
- 实现简单， 比自定义View成本低
- 非图片类得Drawable所占空间少， 减少apk大小

## 2. 实现动态的效果

> 实现小鱼的原地摆动

改变尾巴的角度，就开始游动了，注意鱼尾和鱼身的拍动的角度是不一样的， 通过属性动画改变角度值。

```java
        // currentValue * 1.2=  360 * 整数  currentValue * 1.5= 360 * 整数
        // currentValue = 300 --- currentValue = 240
        // 300/4/5/3 = 5 和 240/4/5/3 = 4 的最小公倍速 ---》 5* 240 -- 4*300 == 1200
        ValueAnimator valueAnimator = ValueAnimator.ofFloat(0, 1200f);
        valueAnimator.setDuration(5 * 1000);// 周期
        valueAnimator.setRepeatMode(ValueAnimator.RESTART); // 设置循环模式， 先从-1到1，再从1到-1
        valueAnimator.setRepeatCount(ValueAnimator.INFINITE); // 循环次数，无限制
        valueAnimator.setInterpolator(new LinearInterpolator());
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                currentValue = (float) animation.getAnimatedValue();
                // 刷新
                invalidateSelf();
            }
        });
        valueAnimator.start();



        // Math.sin(Math.toRadians(currentValue)) --> -1到1 改变当前鱼的角度
        float fishAngle = (float) (fishMainAngle +
                Math.sin(Math.toRadians(currentValue * 1.2)) * 4)；
```



## 3. 交互（点击）

> 实现水波纹

手指按下的时侯，画一个圆， 从小到大， 逐渐消失

```java
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setDither(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(8);

        // 因为是容器，不会执行 onDraw，设置标志位，让onDraw执行
        setWillNotDraw(false);
        
        
            @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 获取手指按下的 x，y坐标，作为圆心的point
                touchX = event.getX();
                touchY = event.getY();

                // 水波纹的半径变化系数
                ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(this,
                        "ripple", 0, 1f).setDuration(1000);
                objectAnimator.start();

                makeTrail();
                break;
            default:
                break;
        }

        return super.onTouchEvent(event);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (touchX >= 0 && touchY >= 0) {
            mPaint.setAlpha(alpha);
            canvas.drawCircle(touchX, touchY, ripple * 150, mPaint);
        }
    }

	//  水波纹透明度 和 半径变化
    public void setRipple(float ripple) {
        alpha = (int) (100 * (1 - ripple));
        this.ripple = ripple;

        invalidate();
    }
```

> 小鱼的移动

小鱼实际上就是一张图片， 可以看到GIF中的小鱼黄色背景图片。

> 移动方式

1. 直接跳转到手指按下的位置

```java
/**
         * 直接跳转到手指按下的位置
         */
        ivFish.setTranslationX(touchX - ivFish.getLeft() - fishImageViewMiddle.x);
        ivFish.setTranslationY(touchY - ivFish.getTop() - fishImageViewMiddle.y);
```

2. 在路径上慢慢移动到手指按下的位置 -- 使用 ValueAnimator 方式实现，属性动画 -- ValueAnimator(值的改变，与属性无关),ObjectAnimator(属性动画), 属性要对应有属性的set方法。

```java
alueAnimator valueAnimator1 = ValueAnimator.ofFloat(ivFish.getTranslationX(),
                touchX - ivFish.getLeft() - fishImageViewMiddle.x);
        valueAnimator1.setDuration(2000);
        valueAnimator1.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
           public void onAnimationUpdate(ValueAnimator animation) {                float currentValue = (float) animation.getAnimatedValue();
                ivFish.setTranslationX(currentValue);
            }
        });
        valueAnimator1.start();

        // y方向的平移
        ValueAnimator valueAnimator2 = ValueAnimator.ofFloat(ivFish.getTranslationY(),
               touchY - ivFish.getTop() - fishImageViewMiddle.y);
        valueAnimator2.setDuration(2000);
        valueAnimator2.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float currentValue = (float) animation.getAnimatedValue();
                ivFish.setTranslationY(currentValue);
            }
       });
        valueAnimator2.start();
```

3 用贝塞尔曲线计算小鱼游动的路径

利用头部圆心，鱼身重心以及点击点坐标来唯一确定一个特征三角形

确定鱼身需要左转还是右转，知道三角形内较AOB的大小，就知道转动方向了。

![image-20220517160012815](https://user-images.githubusercontent.com/30100887/168768355-f0f75407-e8d7-4d0f-ab5b-17e9442ec75e.png)


通过三个点坐标，就可以计算夹角。AOB角度大小可以根据向量夹角计算,向量夹角计算公式：
$$
cosAOB = (OA*OB) / (|OA|*|OB|)
$$
其中OA*OB是向量的数量积，计算过程如下
$$
OA = （Ax-Ox, Ay-Oy）
$$

$$
OB =  (Bx-Ox, By-Oy)
$$

$$
OA*OB=(Ax-Ox)(Bx-Ox)+(Ay-Oy)*(By-Oy)
$$

```java
public static float includedAngle(PointF O, PointF A, PointF B) {
        // OA的长度
        float OALength = (float) Math.sqrt((A.x - O.x) * (A.x - O.x) + (A.y - O.y) * (A.y - O.y));
        // OB的长度
        float OBLength = (float) Math.sqrt((B.x - O.x) * (B.x - O.x) + (B.y - O.y) * (B.y - O.y));
        // OA*OB=(Ax-Ox)*(Bx-Ox)+(Ay-Oy)*(By-Oy)
        float AOB = (A.x - O.x) * (B.x - O.x) + (A.y - O.y) * (B.y - O.y);
        // cosAOB = (OA*OB)/(|OA|*|OB|)
        float cosAOB = AOB / (OALength * OBLength);

        // 角度 -- 反余弦
        float angleAOB = (float) Math.toDegrees(Math.acos(cosAOB));

        // 鱼是向左转弯，还是向右转弯
        // AB连线和 X轴的夹角 tan 值 - OB连线与X轴的夹角 tan值  --> 负数 说明点击在鱼的右侧，正数在左侧
        float direction = (A.y - B.y) / (A.x - B.x) - (O.y - B.y) / (O.x - B.x);

        // 点击在鱼头延长线上 -- angleAOB == 0 ---点击在鱼尾延长线上 angleAOB 180
        if(direction == 0){
            if (AOB >= 0) {
                return 0;
            } else
                return 180;
        } else {
            if (direction > 0) {
                return -angleAOB;
            } else {
                return angleAOB;
            }
        }
    }
```

> 鱼头转向 轨迹的切线

```java
objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animator) {
                float fraction = animator.getAnimatedFraction();
                pathMeasure.getPosTan(pathMeasure.getLength() * fraction, null, tan);
                // y轴与实际坐标相反，tan[1] 需要取反
                float angle = (float) Math.toDegrees(Math.atan2(-tan[1], tan[0]));
                fishDrawable.setFishMainAngle(angle);
            }
        });
```


