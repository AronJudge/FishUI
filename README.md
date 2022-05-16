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

​	改变尾巴的角度，就开始游动了，注意鱼尾和鱼身的拍动的角度是不一样的



### 3. 交互（点击）

> 实现小鱼的点击游动
