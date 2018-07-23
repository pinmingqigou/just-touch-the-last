#概述#
在Android系统中，官方给我们提供了两种类型的动画：属性动画(Property Animation) 和 视图动画(View Animation)，而视图动画又包含了两种类型：补间动画(Tween animation) 和 帧动画(Frame animation)。

Frame Animation，就是我们所说的帧动画，之所以要实现帧动画是因为它可以实现类似电影的动态效果，因为我们平时所拍的视频也是通过一张张照片插入每一帧，串联起来，从而实现连续播放的视觉效果，而这是Tween Animation无法实现，只能通过Frame Animation来实现。例如我们常见的App动态引导页，很多都是通过Frame Animation来实现的。

#实现实例#
- 首先在**res/drawable/**目录下，增添一个frame.xml文件，里面主要写每一帧播放哪一张照片。最外层标签必须是**animation-list**，子标签item对应每一帧，每一个item里面的drawable属性对应图片位置、duration对应一帧的时长。

	    <?xml version="1.0" encoding="utf-8"?>
		<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
		    android:oneshot="false">
		    <item
		        android:drawable="@drawable/g1"
		        android:duration="200" />
		    <item
		        android:drawable="@drawable/g2"
		        android:duration="200" />
		    <item
		        android:drawable="@drawable/g3"
		        android:duration="200" />
		    <item
		        android:drawable="@drawable/g4"
		        android:duration="200" />
		    <item
		        android:drawable="@drawable/g5"
		        android:duration="200" />
		    <item
		        android:drawable="@drawable/g6"
		        android:duration="200" />
		</animation-list>

- 在Activity中，实现如下代码

	    //将控件背景设置为我们的AnimationDrawable资源文件
		image.setBackgroundResource(R.drawable.frame);
		mBinding.play.setOnClickListener(new View.OnClickListener() {
		    @Override
		    public void onClick(View v) {
		        //拿到要编译成AnimationDrawable的背景
		        AnimationDrawable imageAnimation = (AnimationDrawable)image.getBackground();
		        //开始动画
		        imageAnimation.start();
		    }
		});


#AnimationDrawable类#
AnimationDrawable就是对应于我们自身定义的xml文件，在Java代码中将xml对象转为了AnimationDrawable之后，我们就可以通过它来获取xml文件里面的属性。

##继承关系##
继承：Object -> Drawable -> DrawableContainer -> AnimationDrawable

实现：Runnable, Animatable

##XML文件中可设置的属性##
- **android:drawable：** 用于该帧的图片
- **android:duration：** 每一帧的时长
- **android:oneshot：** true则只运行一次，false则重复动画
- **android:variablePadding：** 如果true，允许drawable文件的当前状态改变
- **android:visible：** 是否可见

##方法##
- **void addFrame (Drawable frame, int duration)：** 添加一帧到动画里面
- **int getDuration (int i)：**获取第i帧的时长
- **Drawable getFrame (int index)：** 获取第i帧的Drawbale
- **int getNumberOfFrames ()：**获取共有多少帧
- **void inflate (Resources r, XmlPullParser parser, AttributeSet attrs, Resources.Theme theme)：**从XML资源里面加载一个Drawable文件
- **boolean isOneShot ()：**判断是否单次播放
- **boolean isRunning ()：** 判断动画是否还在运行
- **Drawable mutate ()：**一个drawable如果使用了mutate()方法，那么对这个drawable属性（包括设置drawable的透明度）修改将不会共享
- **void setOneShot (boolean oneShot)：**设置动画播放一次或者循环
- **boolean setVisible (boolean visible, boolean restart)：**设置该AnimationDrawable是否可见
- **void start ()：**播放
- **void stop ()：**停止
- **void unscheduleSelf (Runnable what)：**让动画重新回到-1帧
