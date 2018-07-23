#一.Canvas基本操作详解#
##1.绘制图片##
绘制有两种方法，drawPicture(矢量图) 和 drawBitmap(位图),接下来我们一一了解。

###(1)drawPicture###
**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**在AndroidMenifest文件中application节点下添android:hardwareAccelerated="false"以关闭整个应用的硬件加速。**

既然是drawPicture就要了解一下什么是Picture。 顾名思义，Picture的意思是图片。

**PS：你可以把Picture看作是一个录制Canvas操作的录像机。**

了解了Picture的概念之后，我们再了解一下Picture的相关方法。

<table>
	<tr>
		<th>
		相关方法
		</th>
		<th>
		简介
		</th>
	<tr>
	<tr>
		<th>
		public int getWidth ()
		</th>
		<th>
		获取宽度
		</th>
	<tr>
	<tr>
		<th>
		public int getHeight ()
		</th>
		<th>
		获取高度
		</th>
	<tr>
	<tr>
		<th>
		public Canvas beginRecording (int width, int height)
		</th>
		<th>
		开始录制 (返回一个Canvas，在Canvas中所有的绘制都会存储在Picture中)
		</th>
	<tr>
	<tr>
		<th>
		public void endRecording ()
		</th>
		<th>
		结束录制
		</th>
	<tr>
	<tr>
		<th>
		public void draw (Canvas canvas)
		</th>
		<th>
		将Picture中内容绘制到Canvas中
		</th>
	<tr>	
	<tr>
		<th>
		public static Picture createFromStream (InputStream stream)
		</th>
		<th>
		(已废弃)通过输入流创建一个Picture
		</th>
	<tr>
	<tr>
		<th>
		public void writeToStream (OutputStream stream)
		</th>
		<th>
		(已废弃)将Picture中内容写出到输出流中
		</th>
	<tr>
</table>

上面表格中基本上已经列出了Picture的所有方法，其中getWidth和getHeight没什么好说的，最后两个已经废弃也自然就不用关注了，排除了这些方法之后，只剩三个方法了，接下来我们就比较详细的了解一下：

**很明显，beginRecording 和 endRecording 是成对使用的，一个开始录制，一个是结束录制，两者之间的操作将会存储在Picture中。**

**使用示例：**

**准备工作:**

录制内容，即将一些Canvas操作用Picture存储起来，录制的内容是不会直接显示在屏幕上的，只是存储起来了而已。

    // 1.创建Picture
    private Picture mPicture = new Picture();
    
	---------------------------------------------------------------
    
    // 2.录制内容方法
    private void recording() {
        // 开始录制 (接收返回值Canvas)
        Canvas canvas = mPicture.beginRecording(500, 500);
        // 创建一个画笔
        Paint paint = new Paint();
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);

        // 在Canvas中具体操作
        // 位移
        canvas.translate(250,250);
        // 绘制一个圆
        canvas.drawCircle(0,0,100,paint);

        mPicture.endRecording();
    }
    
	---------------------------------------------------------------

    // 3.在使用前调用(我在构造函数中调用了)
      public Canvas3(Context context, AttributeSet attrs) {
        super(context, attrs);
        
        recording();    // 调用录制
    }

**具体使用:**

Picture虽然方法就那么几个，但是具体使用起来还是分很多情况的，由于录制的内容不会直接显示，就像存储的视频不点击播放不会自动播放一样，同样，想要将Picture中的内容显示出来就需要手动调用播放(绘制)，将Picture中的内容绘制出来可以有以下几种方法：

<table>
	<tr>
		<th>
		序号
		</th>
		<th>
		简介
		</th>
	</tr>
	<tr>
		<th>
		1
		</th>
		<th>
		使用Picture提供的draw方法绘制。
		</th>
	</tr>
	<tr>
		<th>
		2
		</th>
		<th>
		使用Canvas提供的drawPicture方法绘制。
		</th>
	</tr>
	<tr>
		<th>
		3
		</th>
		<th>
		将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。
		</th>
	</tr>
</table>

以上几种方法主要区别：

<table>
	<tr>
		<th>
		主要区别
		</th>
		<th>
		分类
		</th>
		<th>
		简介
		</th>
	</tr>
	<tr>
		<th>
		是否对Canvas有影响
		</th>
		<th>
		1有影响
		2,3不影响
		</th>
		<th>
		此处指绘制完成后是否会影响Canvas的状态(Matrix clip等)
		</th>
	</tr>
	<tr>
		<th>
		可操作性强弱
		</th>
		<th>
		1可操作性较弱
		2,3可操作性较强
		</th>
		<th>
		此处的可操作性可以简单理解为对绘制结果可控程度。
		</th>
	</tr>
</table>

几种方法简介和主要区别基本就这么多了，接下来对于各种使用方法一一详细介绍：

**1.使用Picture提供的draw方法绘制:**

        // 将Picture中的内容绘制在Canvas上
        mPicture.draw(canvas);  

![](https://camo.githubusercontent.com/d25705e1e0a028445cbb978ee3cd98d31661a4ce/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b777a393935366c6a333075303168636467392e6a7067)

**PS：这种方法在比较低版本的系统上绘制后可能会影响Canvas状态，所以这种方法一般不会使用。**

**2.使用Canvas提供的drawPicture方法绘制**

drawPicture有三种方法：
    
    public void drawPicture (Picture picture)
    
    public void drawPicture (Picture picture, Rect dst)
    
    public void drawPicture (Picture picture, RectF dst)

和使用Picture的draw方法不同，Canvas的drawPicture不会影响Canvas状态。

**简单示例:**

    canvas.drawPicture(mPicture,new RectF(0,0,mPicture.getWidth(),200));

![](https://camo.githubusercontent.com/093d81b64b8aeab1e4f84298027c3ca657d05f81/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b777a73657161776a3330753031686337346f2e6a7067)

**PS:对照上一张图片，可以比较明显的看出，绘制的内容根据选区进行了缩放。**

**3.将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。**

        // 包装成为Drawable
        PictureDrawable drawable = new PictureDrawable(mPicture);
        // 设置绘制区域 -- 注意此处所绘制的实际内容不会缩放
        drawable.setBounds(0,0,250,mPicture.getHeight());
        // 绘制
        drawable.draw(canvas);

![](https://camo.githubusercontent.com/27a6541f2579cace72eba092e6dd953f173fa115/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b783062717577336a333075303168636467382e6a7067)

**PS:此处setBounds是设置在画布上的绘制区域，并非根据该区域进行缩放，也不是剪裁Picture，每次都从Picture的左上角开始绘制。**

> 注意：在使用Picture之前请关闭硬件加速，以免引起不必要的问题,如何关闭请参考这里：[https://github.com/GcsSloop/AndroidNote/issues/7](https://github.com/GcsSloop/AndroidNote/issues/7)

###(2)drawBitmap###
> 其实一开始知道要讲Bitmap我是拒绝的，为什么呢？因为Bitmap就是很多问题的根源啊有木有，Bitmap可能导致内存不足，内存泄露，ListView中的复用混乱等诸多问题。想完美的掌控Bitmap还真不是一件容易的事情。限于篇幅本文对于Bitmap不会过多的展开，只讲解一些常用的功能，关于Bitmap详细内容，以后开专题讲解

既然要绘制Bitmap，就要先获取一个Bitmap，那么如何获取呢？

**获取Bitmap方式:**
<table>
	<tr>
		<th>
		序号
		</th>
		<th>
		获取方式
		</th>
		<th>
		备注
		</th>
	</tr>
	<tr>
		<th>
		1
		</th>
		<th>
		通过Bitmap创建
		</th>
		<th>
		复制一个已有的Bitmap(新Bitmap状态和原有的一致) 或者 创建一个空白的Bitmap(内容可改变)
		</th>
	</tr>
	<tr>
		<th>
		2
		</th>
		<th>
		通过BitmapDrawable获取
		</th>
		<th>
		从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
		</th>
	</tr>
	<tr>
		<th>
		3
		</th>
		<th>
		通过BitmapFactory获取
		</th>
		<th>
		从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
		</th>
	</tr>
</table>

**通常来说，我们绘制Bitmap都是读取已有的图片转换为Bitmap绘制到Canvas上。**

很明显，第1种方式不能满足我们的要求，暂时排除。

第2种方式虽然也可满足我们的要求，但是我不推荐使用这种方式，至于为什么在后续详细讲解Drawable的时候会说明,暂时排除。

第3种方法我们会比较详细的说明一下如何从各个位置获取图片。

**通过BitmapFactory从不同位置获取Bitmap:**

**资源文件(drawable/mipmap/raw):**

        Bitmap bitmap = BitmapFactory.decodeResource(mContext.getResources(),R.raw.bitmap);

**资源文件(assets):**

        Bitmap bitmap=null;
        try {
            InputStream is = mContext.getAssets().open("bitmap.png");
            bitmap = BitmapFactory.decodeStream(is);
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

**内存卡文件:**

    Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/bitmap.png");

**网络文件:**

        // 此处省略了获取网络输入流的代码
        Bitmap bitmap = BitmapFactory.decodeStream(is);
        is.close();

既然已经获得到了Bitmap，那么就开始本文的重点了，将Bitmap绘制到画布上。

**绘制Bitmap：**

依照惯例先预览一下drawBitmap的常用方法：

    // 第一种
    public void drawBitmap (Bitmap bitmap, Matrix matrix, Paint paint)
    
    // 第二种
    public void drawBitmap (Bitmap bitmap, float left, float top, Paint paint)
    
    // 第三种
    public void drawBitmap (Bitmap bitmap, Rect src, Rect dst, Paint paint)
    public void drawBitmap (Bitmap bitmap, Rect src, RectF dst, Paint paint)

第一种方法中后两个参数(matrix, paint)是在绘制的时候对图片进行一些改变，如果只是需要将图片内容绘制出来只需要如下操作就可以了：

**PS:图片左上角位置默认为坐标原点。**

    canvas.drawBitmap(bitmap,new Matrix(),new Paint());

> 关于Matrix和Paint暂时略过吧，一展开又是啰啰嗦嗦一大段，反正挖坑已经是常态了，大家应该也习惯了

![](https://camo.githubusercontent.com/03f85adb8e1882b45faeab604e19d69f29be836e/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f3030355874646932677731663469786e6a726e38336a333075303168636162632e6a7067)

第二种方法就是在绘制时指定了图片左上角的坐标(距离坐标原点的距离)：

> 注意：此处指定的是与坐标原点的距离，并非是与屏幕顶部和左侧的距离, 虽然默认状态下两者是重合的，但是也请注意分别两者的不同。

    canvas.drawBitmap(bitmap,200,500,new Paint());

![](https://camo.githubusercontent.com/7f710e9f2779ac7c1e67fd3759395467e01637e3/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f3030355874646932677731663469786f75673278386a33307530316863676e342e6a7067)

第三种方法比较有意思，上面多了两个矩形区域(src,dst),这两个矩形选区是干什么用的？

<table>
	<tr>
		<th>
		名称
		</th>
		<th>
		作用
		</th>
	</tr>
	<tr>
		<th>
		Rect src
		</th>
		<th>
		指定绘制图片的区域
		</th>
	</tr>
	<tr>
		<th>
		Rect dst 或RectF dst
		</th>
		<th>
		指定图片在屏幕上显示(绘制)的区域
		</th>
	</tr>
</table>

示例：

        // 将画布坐标系移动到画布中央
        canvas.translate(mWidth/2,mHeight/2);

        // 指定图片绘制区域(左上角的四分之一)
        Rect src = new Rect(0,0,bitmap.getWidth()/2,bitmap.getHeight()/2);

        // 指定图片在屏幕上显示的区域
        Rect dst = new Rect(0,0,200,400);

        // 绘制图片
        canvas.drawBitmap(bitmap,src,dst,null);

![](https://camo.githubusercontent.com/0f5803a7643e15b76058daa0fc82180e2cac8480/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326777316634697871676b3872776a333075303168633735362e6a7067)

**详解：**

上面是以绘制该图为例，用src指定了图片绘制部分的区域，即下图中红色方框标注的区域。

![](https://camo.githubusercontent.com/0773f036e5da8ddfb22353bfbc705f367cc363af/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b783264617731716a3330356b30356b7133392e6a7067)

然后用dst指定了绘制在屏幕上的绘制，即下图中蓝色方框标注的区域，图片宽高会根据指定的区域自动进行缩放。

![](https://camo.githubusercontent.com/81a9c9814e02a34f9aaf4c4c2498a887d5be930b/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f3030355874646932677731663469787233736b636a6a33307530316863337a612e6a7067)

从上面可知，第三种方法可以绘制图片的一部分到画布上，这有什么用呢？

如果你看过某些游戏的资源文件，你可能会看到如下的图片(图片来自网络)：

![](https://camo.githubusercontent.com/036c9406aade77820b807c1da5f40d6a2e2f5558/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b78336c6b3175636a33306b6730346f6d7a372e6a7067)

用一张图片包含了大量的素材，在绘制的时候每次只截取一部分进行绘制，这样可以大大的减少素材数量，而且素材管理起来也很方便。

然而这和我们有什么关系呢？我们又不做游戏开发。

确实，我们不做游戏开发，但是在某些时候我们需要制作一些炫酷的效果，这些效果因为太复杂了用代码很难实现或者渲染效率不高。这时候很多人就会想起帧动画，将动画分解成一张一张的图片然后使用帧动画制作出来，这种实现方式的确比较简单，但是一个动画效果的图片有十几到几十张，一个应用里面来几个这样炫酷的动画效果就会导致资源文件出现一大堆，想找其中的某一张资源图片简直就是灾难啊有木有。但是把同一个动画效果的所有资源图片整理到一张图片上，会大大的减少资源文件数量，方便管理，妈妈再也不怕我找不到资源文件了，同时也节省了图片文件头、文件结束块以及调色板等占用的空间。

下面是利用drawBitmap第三种方法制作的一个简单示例：

资源文件如下：

![](https://raw.githubusercontent.com/GcsSloop/AndroidNote/master/CustomView/Advance/Res/Checkmark.png)

最终效果如下：

![](https://camo.githubusercontent.com/64a4f2aa1753ea1e5694cd3a741195a1e6ea9067/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b783637666b6b6f673330366730623433796e2e676966)

源码如下：
> PS:由于是示例代码，做的很粗糙，仅作为学习示例，不建议在任何实际项目中使用。

	package com.sloop.canvas;
	
	import android.content.Context;
	import android.graphics.Bitmap;
	import android.graphics.BitmapFactory;
	import android.graphics.Canvas;
	import android.graphics.Paint;
	import android.graphics.Rect;
	import android.os.Handler;
	import android.os.Message;
	import android.util.AttributeSet;
	import android.util.Log;
	import android.view.View;
	
	/**
	 * <ul type="disc">
	 * <li>Author: Sloop</li>
	 * <li>Version: v1.0.0</li>
	 * <li>Copyright: Copyright (c) 2015</li>
	 * <li>Date: 2016/2/5</li>
	 * <li><a href="http://www.sloop.icoc.cc"    target="_blank">作者网站</a>      </li>
	 * <li><a href="http://weibo.com/5459430586" target="_blank">作者微博</a>      </li>
	 * <li><a href="https://github.com/GcsSloop" target="_blank">作者GitHub</a>   </li>
	 * </ul>
	 */
	public class CheckView extends View {

	    private static final int ANIM_NULL = 0;         //动画状态-没有
	    private static final int ANIM_CHECK = 1;        //动画状态-开启
	    private static final int ANIM_UNCHECK = 2;      //动画状态-结束
	
	    private Context mContext;           // 上下文
	    private int mWidth, mHeight;        // 宽高
	    private Handler mHandler;           // handler
	
	    private Paint mPaint;
	    private Bitmap okBitmap;
	
	    private int animCurrentPage = -1;       // 当前页码
	    private int animMaxPage = 13;           // 总页数
	    private int animDuration = 500;         // 动画时长
	    private int animState = ANIM_NULL;      // 动画状态
	
	    private boolean isCheck = false;        // 是否只选中状态
	
	    public CheckView(Context context) {
	        super(context, null);
	
	    }
	
	    public CheckView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        init(context);
	    }
	
	    /**
	     * 初始化
	     * @param context
	     */
	    private void init(Context context) {
	        mContext = context;
	
	        mPaint = new Paint();
	        mPaint.setColor(0xffFF5317);
	        mPaint.setStyle(Paint.Style.FILL);
	        mPaint.setAntiAlias(true);
	
	        okBitmap = BitmapFactory.decodeResource(mContext.getResources(), R.drawable.checkres);
	
	        mHandler = new Handler() {
	            @Override
	            public void handleMessage(Message msg) {
	                super.handleMessage(msg);
	                if (animCurrentPage < animMaxPage && animCurrentPage >= 0) {
	                    invalidate();
	                    if (animState == ANIM_NULL)
	                        return;
	                    if (animState == ANIM_CHECK) {
	
	                        animCurrentPage++;
	                    } else if (animState == ANIM_UNCHECK) {
	                        animCurrentPage--;
	                    }
	
	                    this.sendEmptyMessageDelayed(0, animDuration / animMaxPage);
	                    Log.e("AAA", "Count=" + animCurrentPage);
	                } else {
	                    if (isCheck) {
	                        animCurrentPage = animMaxPage - 1;
	                    } else {
	                        animCurrentPage = -1;
	                    }
	                    invalidate();
	                    animState = ANIM_NULL;
	                }
	            }
	        };
	    }
	
	
	    /**
	     * View大小确定
	     * @param w
	     * @param h
	     * @param oldw
	     * @param oldh
	     */
	    @Override
	    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
	        super.onSizeChanged(w, h, oldw, oldh);
	        mWidth = w;
	        mHeight = h;
	    }
	
	    /**
	     * 绘制内容
	     * @param canvas
	     */
	    @Override
	    protected void onDraw(Canvas canvas) {
	        super.onDraw(canvas);
	
	        // 移动坐标系到画布中央
	        canvas.translate(mWidth / 2, mHeight / 2);
	
	        // 绘制背景圆形
	        canvas.drawCircle(0, 0, 240, mPaint);
	
	        // 得出图像边长
	        int sideLength = okBitmap.getHeight();
	
	        // 得到图像选区 和 实际绘制位置
	        Rect src = new Rect(sideLength * animCurrentPage, 0, sideLength * (animCurrentPage + 1), sideLength);
	        Rect dst = new Rect(-200, -200, 200, 200);
	
	        // 绘制
	        canvas.drawBitmap(okBitmap, src, dst, null);
	    }
	
	
	    /**
	     * 选择
	     */
	    public void check() {
	        if (animState != ANIM_NULL || isCheck)
	            return;
	        animState = ANIM_CHECK;
	        animCurrentPage = 0;
	        mHandler.sendEmptyMessageDelayed(0, animDuration / animMaxPage);
	        isCheck = true;
	    }
	
	    /**
	     * 取消选择
	     */
	    public void unCheck() {
	        if (animState != ANIM_NULL || (!isCheck))
	            return;
	        animState = ANIM_UNCHECK;
	        animCurrentPage = animMaxPage - 1;
	        mHandler.sendEmptyMessageDelayed(0, animDuration / animMaxPage);
	        isCheck = false;
	    }
	
	    /**
	     * 设置动画时长
	     * @param animDuration
	     */
	    public void setAnimDuration(int animDuration) {
	        if (animDuration <= 0)
	            return;
	        this.animDuration = animDuration;
	    }
	
	    /**
	     * 设置背景圆形颜色
	     * @param color
	     */
	    public void setBackgroundColor(int color){
	        mPaint.setColor(color);
	    }
	}

##2.绘制文字##

依旧预览一下相关常用方法：

    // 第一类
    public void drawText (String text, float x, float y, Paint paint)
    public void drawText (String text, int start, int end, float x, float y, Paint paint)
    public void drawText (CharSequence text, int start, int end, float x, float y, Paint paint)
    public void drawText (char[] text, int index, int count, float x, float y, Paint paint)
    
    // 第二类
    public void drawPosText (String text, float[] pos, Paint paint)
    public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)
    
    // 第三类
    public void drawTextOnPath (String text, Path path, float hOffset, float vOffset, Paint paint)
    public void drawTextOnPath (char[] text, int index, int count, Path path, float hOffset, float vOffset, Paint paint)

> PS 其中的CharSequence和String的区别可以到这里看看：[https://github.com/GcsSloop/AndroidNote/issues/16](https://github.com/GcsSloop/AndroidNote/issues/16) 

绘制文字部分大致可以分为三类：

第一类只能指定文本基线位置(基线x默认在字符串左侧，基线y默认在字符串下方)。

第二类可以分别指定每个文字的位置。

第三类是指定一个路径，根据路径绘制文字。

通过上面常用方法的参数也可看出，绘制文字也是需要画笔的，而且文字的大小,颜色,字体,对齐方式都是由画笔控制的。

不过嘛这里仅简单介绍几种常用方法(反正挖坑多了也不怕)，具体在讲解Paint时再详细讲解。

**Paint文本相关常用方法表**
<table>
	<tr>
		<th>
		标题
		</th>
		<th>
		相关方法
		</th>
		<th>
		备注
		</th>
	</tr>
	<tr>
		<th>
		色彩
		</th>
		<th>
		setColor setARGB setAlpha
		</th>
		<th>
		设置颜色，透明度
		</th>
	</tr>
	<tr>
		<th>
		大小
		</th>
		<th>
		setTextSize
		</th>
		<th>
		设置文本字体大小
		</th>
	</tr>
	<tr>
		<th>
		字体
		</th>
		<th>
		setTypeface
		</th>
		<th>
		设置或清除字体样式
		</th>
	</tr>
	<tr>
		<th>
		样式
		</th>
		<th>
		setStyle
		</th>
		<th>
		填充(FILL),描边(STROKE),填充加描边(FILL_AND_STROKE)
		</th>
	</tr>
	<tr>
		<th>
		对齐
		</th>
		<th>
		setTextAlign
		</th>
		<th>
		左对齐(LEFT),居中对齐(CENTER),右对齐(RIGHT)
		</th>
	</tr>
	<tr>
		<th>
		测量
		</th>
		<th>
		measureText
		</th>
		<th>
		测量文本大小(注意，请在设置完文本各项参数后调用)
		</th>
	</tr>
</table>

为了绘制文本，我们先创建一个文本画笔：

        Paint textPaint = new Paint();          // 创建画笔
        textPaint.setColor(Color.BLACK);        // 设置颜色
        textPaint.setStyle(Paint.Style.FILL);   // 设置样式
        textPaint.setTextSize(50);              // 设置字体大小

**第一类(drawText)**

第一类可以指定文本开始的位置，可以截取文本中部分内容进行绘制。

其中x，y两个参数是指定文本绘制两个基线,示例：

        // 文本(要绘制的内容)
        String str = "ABCDEFGHIJK";

        // 参数分别为 (文本 基线x 基线y 画笔)
        canvas.drawText(str,200,500,textPaint);

![](https://camo.githubusercontent.com/ec6e0dafb39ad15d4d359ada989dca8c583c3526/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f303035587464693267773166336e6a667369306c346a33306477306e757765792e6a7067)

> PS: 图中字符串下方的红线是基线y，基线x未在图中画出。

当然啦，除了能指定绘制文本的起始位置，还能只取出文本中的一部分内容进行绘制。

截取文本中的一部分，对于String和CharSequence来说只指定字符串下标start和end位置(注意：0<= start < end < str.length())

一般来说，**使用start和end指定的区间是前闭后开的，即包含start指定的下标，而不包含end指定的下标**，故[1,3)最后获取到的下标只有 下标1 和 下标2 的字符，就是"BC"。

示例：

        // 文本(要绘制的内容)
        String str = "ABCDEFGHIJK";
        // 参数分别为 (字符串 开始截取位置 结束截取位置 基线x 基线y 画笔)
        canvas.drawText(str,1,3,200,500,textPaint);

![](https://camo.githubusercontent.com/2e92c17e90073794227c276964879188d3a3c86a/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f303035587464693267773166336e6a6836363031386a33306477306e757133622e6a7067)

另外，对于字符数组char[]我们截取字符串使用起始位置(index)和长度(count)来确定。

同样，我们指定index为1，count为3，那么最终截取到的字符串是"BCD"。

其实就是从下标位置为1处向后数3位就是截取到的字符串，示例：

        // 字符数组(要绘制的内容)
        char[] chars = "ABCDEFGHIJK".toCharArray();
        
        // 参数为 (字符数组 起始坐标 截取长度 基线x 基线y 画笔)
        canvas.drawText(chars,1,3,200,500,textPaint);

![](https://camo.githubusercontent.com/2bebfb9404c8397e3bc117a4c859e5f7eadb7355/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f303035587464693267773166336e6a686e6c6462356a33306477306e7537346f2e6a7067)

**第二类(drawPosText)**

通过和第一类比较，我们可以发现，第二类中没有指定x，y坐标的参数，而是出现了这样一个参数float[] pos。

好吧，这个名为pos的浮点型数组就是指定坐标的，至于为啥要用数组嘛，因为这家伙野心比较大，想给每个字符都指定一个位置。

示例：

        String str = "SLOOP";

        canvas.drawPosText(str,new float[]{
                100,100,    // 第一个字符位置
                200,200,    // 第二个字符位置
                300,300,    // ...
                400,400,
                500,500
        },textPaint);

![](https://camo.githubusercontent.com/a343d11ce6accef7bebde0a52b90192f312ef727/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a773166326b7839666c3863386a33307530316863676c7a2e6a7067)

不过嘛，虽然虽然这个方法也比较容易理解，但是关于这个方法我个人是不推荐使用的，因为坑比较多，主要有一下几点：

<table>
	<tr>
		<th>
		序号
		</th>
		<th>
		反对理由
		</th>
	</tr>
	<tr>
		<th>
		1
		</th>
		<th>
		必须指定所有字符位置，否则直接crash掉，反人类设计
		</th>
	</tr>
	<tr>
		<th>
		2
		</th>
		<th>
		性能不佳，在大量使用的时候可能导致卡顿
		</th>
	</tr>
	<tr>
		<th>
		3
		</th>
		<th>
		不支持emoji等特殊字符，不支持字形组合与分解
		</th>
	</tr>
</table>

关于第二类的第二种方法：

    public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)

**第三类(drawTextOnPath)**

第三类要用到path这个大杀器，作为一个厉害角色怎么能这么轻易露脸呢，先保持一下神秘感，也就是说，下回再和大家见面喽。