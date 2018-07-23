#概述#
Tween Animation，翻译为补间动画，是View Animation两种中的一种，是Android开发中使用最普遍的动画，当然也是使用率最高的一种动画，因为它能够基本满足我们的动画需要，主要是通过对控件实现透明度(alpha)、尺寸(scale)、位置(translate)、旋转rotate)改变，通过集合(set)的方式，实现连续的动画效果。

#动画资源文件的位置#
![](http://7xs0af.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720170317152414.png)

#类型#
Tween Animation是View Animation其中之一，Tween Animation可以实现对控件实现以下四种变换:

- alpha：透明度渐变动画效果
- scale：尺寸变化动画效果
- translate：位置移动动画效果
- rotate：转移旋转动画效果

#使用方式#
- 在XML文件中定义一系列的动画标签，所有便签在set便签里面构成集合
- Java文件中代码实现

之所以可以这样，是因为四个标签都有对应的Java类：AlphaAnimation, RotateAnimation, ScaleAnimation, TranslateAnimation。

所在包：android.view.animation.*

都继承于父类：android.view.animation.Animation

#Animation类#
Tween Animation的所有效果都是继承于Animation类，Animation类作为一个抽象类，提供了动画基本属性的实现，需要由具体的子类来具体实现。

##直接子类##
- **AnimationSet：**代表着一组可以一起播放的动画
- **AlphaAnimation：**控制一个对象透明度的动画
- **RotateAnimation：**控制一个对象旋转的动画
- **ScaleAnimation：**控制一个对象尺寸的动画
- **TranslateAnimation：**控制一个对象位置的动画

##监听器##
- **Animation.AnimationListener：**动画监听器，主要包括监听动画执行中、执行结束、循环执行三个过程。

	    mAnimation.setAnimationListener(new Animation.AnimationListener() {
	    @Override
	    public void onAnimationStart(Animation animation) {
	            // 动画执行时
	        }
	
	        @Override
	        public void onAnimationEnd(Animation animation) {
	            // 动画结束时
	        }
	
	        @Override
	        public void onAnimationRepeat(Animation animation) {
	            // 动画循环时
	        }
	    });

##属性及其对应方法(所有子类都拥有)##
- **android:detachWallpaper：** 是否在壁纸上运行
	- **setDetachWallpaper(boolean detachWallpaper)**

- **android:duration：** 动画时长，单位毫秒
	- **setDuration(long)**

- **android:fillAfter：**设置为true，控件动画结束时将保持动画最后一帧（xml文件中，需要设置在set便签才生效）
	- **setFillAfter(boolean)**

- **android:fillBefore：**设置为true，控件动画结束时将保持动画开始第一帧（感觉很坑爹，设置true和false还有删除这个属性，效果都一样）
	- **setFillBefore(boolean)**

- **android:fillEnabled：** 效果和fillBefore一样（同样坑爹，经测试这个属性可有可无，求打脸）
	- **setFillEnabled(boolean)**

- **android:interpolator：**插值器。设置动画速率的变化(譬如加速、减速、匀速等)
	- **setInterpolator(Interpolator)**

- **android:repeatCount：**动画重复次数
	- **setRepeatCount(int)**

- **android:repeatMode：** 重复模式，有reverse(倒序)和restart(重复)两种，必须配合repeatCount一起使用
	- setRepeatMode(int)

- **android:startOffset：** 延迟一定毫秒之后才开始动画
	- **setStartOffset(long)**

- **android:zAdjustment：**表示被设置动画的内容在动画运行时在Z轴上的位置，有以下三个值
	- normal 默认值，保持内容在Z轴上的位置不变
	- top 保持在Z周最上层
	- bottom 保持在Z轴最下层
	- **setZAdjustment(int)**

##Interpolator##
Interpolator，又名插值器，主要是实现动画的速率变化。Interpolator是一个接口，抽象类BaseInterpolator实现Interpolator接口，BaseInterpolator的子类就是一系列Android提供的插值器

###用法###
- 在XML的标签下设置:android:interpolator=”@android:anim/accelerate_decelerate_interpolator”
- 在JAVA代码中使用：animation.setInterpolator(new AccelerateDecelerateInterpolator())

##Android 实现的所有插值器##
- **AccelerateDecelerateInterpolator：**开始和结束速度慢，中间部分加速
- **AccelerateInterpolator：**开始缓慢，然后加速
- **AnticipateInterpolator:**开始后退，然后前进
- **AnticipateOvershootInterpolator:**开始后退，然后前进，直到超出目标值，再后退至目标值
- **BounceInterpolator：**在结束时弹跳
- **CycleInterpolator：**在指定数量的周期内重复动画，速度变化遵循正弦规律
- **DecelerateInterpolator：**开始加速，结束缓慢
- **LinearInterpolator：**匀速
- **OvershootInterpolator：**前进，直到超出目标值，再后退至目标值
- **PathInterpolator：**根据路径变化改变速率

##AnimationSet##
AnimationSet是动画中非常重要的概念，继承于Animation类，它将很多独立的动画包裹到一个集合内，形成一个共同作用的动画效果。例如，我们会在res/anim/目录下的xml文件里面这样实现一个包含多种效果的动画(透明度、尺寸、位置、旋转)：

###用法示例###

    <set xmlns:android="http://schemas.android.com/apk/res/android"
	    android:fillAfter="true"
	    android:shareInterpolator="false">
	    <scale
	        android:duration="1500"
	        android:fromXScale="1.0"
	        android:fromYScale="1.0"
	        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:toXScale="1.4"
	        android:toYScale="0.6" />
	    <set
	        android:duration="1000"
	        android:interpolator="@android:anim/accelerate_interpolator"
	        android:startOffset="1500">
	        <scale
	            android:fromXScale="1.4"
	            android:fromYScale="0.6"
	            android:pivotX="50%"
	            android:pivotY="50%"
	            android:toXScale="0.0"
	            android:toYScale="0.0" />
	        <rotate
	            android:fromDegrees="0"
	            android:pivotX="50%"
	            android:pivotY="50%"
	            android:toDegrees="-45" />
	    </set>
	</set>

效果如下：

![](http://7xrxlv.com1.z0.glb.clouddn.com/ezgif.com-video-to-gif%20%282%29.gif)

当然，你可以使用Java代码来实现，逻辑和在XML中实现的是一样的，如下：

    public class MainActivity extends AppCompatActivity {
	    private ActivityMainBinding mainBinding;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        mainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
	
	        //参数为false，不共享Interpolator
	        AnimationSet set = new AnimationSet(false);
	        set.setFillAfter(true);
	
	        // Scale动画
	        ScaleAnimation scaleAnimation = new ScaleAnimation(1.0f, 1.4f, 1.0f, 0.6f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	        scaleAnimation.setInterpolator(new AccelerateDecelerateInterpolator());
	        scaleAnimation.setDuration(1500);
	        set.addAnimation(scaleAnimation);
	
	        // mSet集合
	        AnimationSet mSet = new AnimationSet(true);
	        mSet.setInterpolator(new AccelerateInterpolator());
	        mSet.setDuration(1000);
	        mSet.setStartOffset(1500);
	
	        // Scale动画
	        ScaleAnimation mScaleAnimation = new ScaleAnimation(1.4f, 0.0f, 0.6f, 0.0f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	        mSet.addAnimation(mScaleAnimation);
	        // Rotate动画
	        RotateAnimation mRotateAnimation = new RotateAnimation(0.0f, -45.0f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	        mSet.addAnimation(mRotateAnimation);
	
	        set.addAnimation(mSet);
	
	        mainBinding.image.startAnimation(set);
	    }
	}

###构造函数###
- **AnimationSet(Context context, AttributeSet attrs) ：**当需要加载AttributeSet时使用的构造函数
- **AnimationSet(boolean shareInterpolator)：**当从代码实现AnimationSet时，传入是否子动画是否共享插值器参数

###主要属性及其方法(父类以外的)###
- **void addAnimation (Animation a)：**把一个子动画加到Animation Set中去
- **long computeDurationHint ()：**获取最长时间的子动画的时间，单位毫秒
- **List getAnimations ()：**获取所有子动画对象的集合
- **long getStartTime ()：**获取动画什么时候应该播放，如果方法返回常量START_ON_FIRST_FRAME (-1)，表明动画未播放 
- **boolean getTransformation (long currentTime, 
Transformation t)：**获取动画执行到目前时的变换信息，存到Transformation中，主要用作调试用。例如我们需要当前Scale动画的变换信息：
    
	    Transformation outTransformation = new Transformation();
			if (scaleAnimation!=null){
			    scaleAnimation.getTransformation(AnimationUtils.currentAnimationTimeMillis(), outTransformation);
			    System.out.println("Matrix:" + outTransformation.getMatrix());
			    System.out.println("Alpha:" + outTransformation.getAlpha());
			    System.out.println("TransformationType:" + outTransformation.getTransformationType());
		}
- **void initialize (int width, 
int height, 
int parentWidth, 
int parentHeight)：**初始化动画的宽高和父动画的宽高
- **void reset ()：**重置动画的初始状态
- **void restrictDuration (long durationMillis)：** 限制动画不能超过durationMillis个毫秒
- **boolean willChangeBounds ()：**动画是否会改变view的尺寸边界。true为会
- **boolean willChangeTransformationMatrix ()：**动画是否会改变view的矩阵

##AlphaAnimation##
AlphaAnimation是继承于Animation的一个动画类型，它通过控制控件的透明度变化来实现动画效果，如我们常见的淡入淡出效果就是通过AlphaAnimaiton实现

不多说，我们来通过例子看一下AlphaAnimation的实现消失动画效果：

![](http://7xs0af.com1.z0.glb.clouddn.com/ezgif.com-video-to-gif%20%283%29.gif)

还是有两种写法，一种在XML里定义动画的样式，一种在Java代码里实现动画的样式

###用法示例###
**XML实现方式**

在res/anim目录下定义alpha.xml文件，在文件内写：

    <alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2500"
    android:fromAlpha="1.0"
    android:toAlpha="0.0" />

在Activity中加载xml文件，并绑定控件播放动画：

    AlphaAnimation animation = (AlphaAnimation) AnimationUtils.loadAnimation(getApplicationContext(), R.anim.alpha);
	image.startAnimation(animation);

-----------------

**Java代码实现方式**

    AlphaAnimation animation = new AlphaAnimation(1.0f, 0.0f);
	animation.setDuration(2500);
	mBinding.image.startAnimation(animation);

###构造函数###
- **AlphaAnimation(Context context, AttributeSet attrs)：**当需要加载AttributeSet时使用的构造函数
- **AlphaAnimation(float fromAlpha, float toAlpha)：**构造函数传入fromAlpha(开始时透明度)， toAlpha(结束时透明度)

###主要属性及其对应方法(父类以外的)###
- XML属性
	- **android:fromAlpha：**动画开始前控件的透明度，取值0.0-1.0，从透明到不透明
	- **android:toAlpha：**动画结束时控件的透明度，取值0.0-1.0，从透明到不透明

- **方法**
	- **boolean willChangeBounds ()：**动画是否会改变view的尺寸边界。true为会
	- **boolean willChangeTransformationMatrix ()：**动画是否会改变view的矩阵
	- **void applyTransformation (float interpolatedTime, Transformation t)：**AlphaAnimation的具体实现，只能在AlphaAnimation的子类中重写来实现自己需要的动画效果

##ScaleAnimation##
ScaleAnimation是继承于Animation的一个动画类型，它通过控制控件的尺寸大小来实现动画效果，如我们常见的放大，缩小效果就是通过ScaleAnimation实现

同样，先示范：

![](http://7xs0af.com1.z0.glb.clouddn.com/scale.gif)

还是有两种写法，一种在XML里定义动画的样式，一种在Java代码里实现动画的样式。

###用法示例###
**XML实现方式**

在res/anim目录下定义scale.xml文件，在文件内写：

    <set xmlns:android="http://schemas.android.com/apk/res/android"
	    android:fillAfter="true">
	    <scale
	        android:duration="1500"
	        android:fromXScale="1.0"
	        android:fromYScale="1.0"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:toXScale="2.0"
	        android:toYScale="2.0" />
	
	    <scale
	        android:duration="1500"
	        android:fromXScale="1.0"
	        android:fromYScale="1.0"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:startOffset="1500"
	        android:toXScale="0.0"
	        android:toYScale="0.0" />
	</set>

在Activity中加载xml文件，并绑定控件播放动画：

    Animation animation = AnimationUtils.loadAnimation(getApplicationContext(), R.anim.scale);
	image.startAnimation(animation);

---------------------

**java代码实现方式**

    AnimationSet set = new AnimationSet(false);
	set.setFillAfter(true);
	
	// 构造函数 fromX, toX, fromY, toY, pivotXType, pivotXValue, pivotYType, pivotYValue
	// pivotXType和pivotYType都有三种取值：Animation.ABSOLUTE, Animation.RELATIVE_TO_SELF, Animation.RELATIVE_TO_PARENT
	ScaleAnimation sa1 = new ScaleAnimation(1.0f, 2.0f, 1.0f, 2.0f, Animation.RELATIVE_TO_SELF,0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	sa1.setDuration(1500);
	
	ScaleAnimation sa2 = new ScaleAnimation(1.0f, 0.0f, 1.0f, 0.0f, Animation.RELATIVE_TO_SELF,0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	sa2.setDuration(1500);
	sa2.setStartOffset(1500);
	
	set.addAnimation(sa1);
	set.addAnimation(sa2);
	
	mBinding.image.startAnimation(set);

###构造函数###
- **ScaleAnimation(Context context, AttributeSet attrs)：**当需要加载AttributeSet时使用的构造函数
- **ScaleAnimation(float fromX, float toX, float fromY, float toY)：**X方向放大（缩小）前的相对比例，X方向放大（缩小）后的相对比例，Y方向放大（缩小）前的相对比例，Y方向放大（缩小）后的相对比例
- **ScaleAnimation(float fromX, float toX, float fromY, float toY, float pivotX, float pivotY)：** pivotX和pivotY是缩放动画围绕的中心点，当为数值时，表明是屏幕上的绝对坐标值，当为百分数（如50%）时，为相对自己本身控件的比例值，当为百分数加p（如50%p）时，为相对父布局的比例值
- **ScaleAnimation(float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)：**当我们需要输入pivotXValue和pivotYValue为百分数或者百分数加p类型的时候，我们用第三个构造函数显示是不行的，这时候需要传入pivotXType和pivotYType，它们有三种取值：Animation.ABSOLUTE, Animation.RELATIVE_TO_SELF, Animation.RELATIVE_TO_PARENT

###主要属性及其方法(父类以外的)###
- XML属性
	- **android:fromXScale：**动画开始前的X方向上的尺寸
	- **android:fromYScale：**动画开始前的Y方向上的尺寸
	- **android:toXScale：**动画结束时的X方向上的尺寸
	- **android:toYScale：**动画结束时的Y方向上的尺寸
	- **android:pivotX：**缩放动画围绕X方向的中心点，当为数值时，表明是屏幕上的绝对坐标值，当为百分数（如50%）时，为相对自己本身控件的比例值，当为百分数加p（如50%p）时，为相对父布局的比例值
	- **android:pivotY：**缩放动画围绕Y方向的中心点，当为数值时，表明是屏幕上的绝对坐标值，当为百分数（如50%）时，为相对自己本身控件的比例值，当为百分数加p（如50%p）时，为相对父布局的比例值

- 方法
	- **void initialize (int width, int height, int parentWidth, int parentHeight)：**初始化动画的宽高和父动画的宽高


##RotateAnimation##
RotateAnimation是继承于Animation的一个动画类型，它通过控制控件旋转来实现动画效果，如我们常见的转动效果就是通过RotateAnimation实现

先看一个小demo：

![](http://7xs0af.com1.z0.glb.clouddn.com/rotate.gif)

###实现方式###
还是有两种写法，一种在XML里定义动画的样式，一种在Java代码里实现动画的样式

**XML实现方式**

在res/anim目录下定义rotate.xml文件，在文件内写：

    <set xmlns:android="http://schemas.android.com/apk/res/android"
	    android:fillAfter="true">
	
	    <rotate
	        android:duration="1000"
	        android:fromDegrees="0"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:toDegrees="360" />
	
	    <rotate
	        android:duration="1500"
	        android:fromDegrees="0"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:startOffset="2000"
	        android:toDegrees="-1800" />
	</set>

在Activity中加载xml文件，并绑定控件播放动画：

    Animation animation = AnimationUtils.loadAnimation(getApplicationContext(), R.anim.rotate);
	image.startAnimation(animation);

**java代码实现**

    AnimationSet set = new AnimationSet(false);
	set.setFillAfter(true);
	
	RotateAnimation ra1 = new RotateAnimation(0f, 360f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	ra1.setDuration(1000);
	
	RotateAnimation ra2 = new RotateAnimation(0f, -1800f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
	ra2.setDuration(1500);
	ra2.setStartOffset(1000);
	
	set.addAnimation(ra1);
	set.addAnimation(ra2);
	
	mBinding.image.startAnimation(set);

###构造函数###
- **RotateAnimation(Context context, AttributeSet attrs)：**当需要加载AttributeSet时使用的构造函数
- **RotateAnimation(float fromDegrees, float toDegrees)：**从fromDegress角度转到toDegress角度(正数为顺时针，负数为逆时针)
- **RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY)：**加上旋转中心点，X方向中心点pivotX和Y方向中心点，取值仍为上面所介绍的三种
- **RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)：**加上中心点的取值类型

###主要属性及其方法(父类以外的)###
- XML属性
	- **android:fromDegrees：**动画开始前的角度
	- **android:toDegrees：**动画结束时的角度
	- **android:pivotX：**缩放动画围绕X方向的中心点，当为数值时，表明是屏幕上的绝对坐标值，当为百分数（如50%）时，为相对自己本身控件的比例值，当为百分数加p（如50%p）时，为相对父布局的比例值
	- **android:pivotY：**缩放动画围绕Y方向的中心点，当为数值时，表明是屏幕上的绝对坐标值，当为百分数（如50%）时，为相对自己本身控件的比例值，当为百分数加p（如50%p）时，为相对父布局的比例值

- 方法
	- **void initialize (int width, int height, int parentWidth, int parentHeight)：**初始化动画的宽高和父动画的宽高
	- **void applyTransformation (float interpolatedTime, Transformation t)：**RotateAnimation的具体实现，只能在RotateAnimation的子类中重写来实现自己需要的动画效果

##TranslateAnimation##
TranslateAnimation是继承于Animation的一个动画类型，它通过控制控件位置变化来实现动画效果，如我们常见的位移效果就是通过TranslateAnimation实现

TranslationAnimation相关的小Demo，围绕四周转动：

![](http://7xs0af.com1.z0.glb.clouddn.com/translate.gif)

###实现方式###
**XML实现方式**

在res/anim目录下定义translate.xml文件，在文件内写：

    <set xmlns:android="http://schemas.android.com/apk/res/android"
	    android:fillAfter="true">
	
	    <translate
	        android:duration="1000"
	        android:fromXDelta="0"
	        android:fromYDelta="0"
	        android:toXDelta="92%p"
	        android:toYDelta="0" />
	
	    <translate
	        android:duration="1000"
	        android:fromXDelta="0"
	        android:fromYDelta="0"
	        android:startOffset="1000"
	        android:toXDelta="0"
	        android:toYDelta="94%p" />
	    <translate
	        android:duration="1000"
	        android:fromXDelta="0"
	        android:fromYDelta="0"
	        android:startOffset="2000"
	        android:toXDelta="-92%p"
	        android:toYDelta="0" />
	
	    <translate
	        android:duration="1000"
	        android:fromXDelta="0"
	        android:fromYDelta="0"
	        android:startOffset="3000"
	        android:toXDelta="0"
	        android:toYDelta="-94%p" />
	</set>

在Activity中加载xml文件，并绑定控件播放动画：

    Animation animation = AnimationUtils.loadAnimation(getApplicationContext(), R.anim.translate);
	mBinding.image.startAnimation(animation);

**java代码实现方式**

同上

###构造函数###
- **TranslateAnimation(Context context, AttributeSet attrs)：**当需要加载AttributeSet时使用的构造函数
- **TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)：**fromXDelta（动画开始时的X方向上的位置），toXDelta（动画结束始时的X方向上的位置），fromYDelta（动画开始时的Y方向上的位置），toYDelta（动画结束始时的Y方向上的位置）
- **TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue, int fromYType, float fromYValue, int toYType, float toYValue)：** 加上类型参数

###主要属性及其方法(父类以外的)###
- 方法
	- **void initialize (int width, int height, int parentWidth, int parentHeight)：**初始化动画的宽高和父动画的宽高
	- **void applyTransformation (float interpolatedTime, Transformation t)：**TranslateAnimation的具体实现，只能在TranslateAnimation的子类中重写来实现自己需要的动画效果