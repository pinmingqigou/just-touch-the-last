**原文地址：[https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B01%5DCoordinateSystem.md](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B01%5DCoordinateSystem.md)**

# 一.屏幕坐标系和数学坐标系的区别 #
由于移动设备一般定义屏幕左上角为坐标原点，向右为x轴增大方向，向下为y轴增大方向， 所以在手机屏幕上的坐标系与数学中常见的坐标系是稍微有点差别的，详情如下：

（**PS：其中的∠a 是对应的，注意y轴方向！**）

![](https://camo.githubusercontent.com/55c557b7591ca661ebf2e6ad6cea0570592d6075/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a773166317179677a6676686f6a33303863306477676c722e6a7067)![](https://camo.githubusercontent.com/42523a55dc2c71f288166c04da524e350d9abfe4/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a7731663171796862717669686a333038633064776a72682e6a7067)

**实际屏幕上的默认坐标系如下：**

> PS：假设其中棕色部分为手机屏幕

![](https://camo.githubusercontent.com/a0c38e4f124e4b2b4a90e0767f24ee9e7b76f992/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a773166317179686a793768386a333038633064777133322e6a7067)

#二.View的坐标系#
**注意：View的坐标系统是相对于父控件而言的**

    getTop();       //获取子View左上角距父View顶部的距离
    getLeft();      //获取子View左上角距父View左侧的距离
    getBottom();    //获取子View右下角距父View顶部的距离
    getRight();     //获取子View右下角距父View左侧的距离

**如下图所示**

![](https://camo.githubusercontent.com/09df2f3f82180fd70ca62b084d906977215458f3/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326777316631717a7177766b6b626a33303863306477676d392e6a7067)

#三.MotionEvent中get和getRaw的区别#

    event.getX();       //触摸点相对于其所在组件坐标系的坐标
    event.getY();

    event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
    event.getRawY();

**如下图所示：**
> PS：其中相同颜色的内容是对应的，其中为了显示方便，蓝色箭头向左稍微偏移了一点.

![](https://camo.githubusercontent.com/e5f1fba429c2f848e8ab8a56401b1ea9853599d8/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a77316631723262646c7168626a333038633064777765772e6a7067)

#四.核心要点#
1. 在数学中常见的坐标系与屏幕默认坐标系的差别
2. View的坐标系是相对于父控件而言的
3. MotionEvent中的get和getRaw的区别

**参考文章：[http://blog.csdn.net/wangjinyu501/article/details/21827341](http://blog.csdn.net/wangjinyu501/article/details/21827341)**