#一.Canvas简介#
Canvas我们可以称之为画布，能够在上面绘制各种东西，是安卓平台2D图形绘制的基础，非常强大。

**一般来说，比较基础的东西有两大特点:**

- **可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。**
- **比较难用：各种方法太过基础，想要完美的将这些操作组合起来有一定难度。**

#二.Canvas的常用操作速查表#
 <table>
        <tr>
            <th>操作类型</th>
            <th>相关API</th>
            <th>备注</th>
        </tr>
        <tr>
            <th>绘制颜色</th>
            <th>drawColor, drawRGB, drawARGB</th>
            <th>使用单一颜色填充整个画布</th>
        </tr>
        <tr>
            <th>绘制基本形状</th>
            <th>drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc</th>
            <th>依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧</th>
        </tr>
        <tr>
            <th>绘制图片</th>
            <th>drawBitmap, drawPicture</th>
            <th>绘制位图和图片</th>
        </tr>
 		<tr>
            <th>绘制文本</th>
            <th>drawText, drawPosText, drawTextOnPath</th>
            <th>依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字</th>
        </tr>
 		<tr>
            <th>绘制路径</th>
            <th>drawPath</th>
            <th>绘制路径，绘制贝塞尔曲线时也需要用到该函数</th>
        </tr>
 		<tr>
            <th>顶点操作</th>
            <th>drawVertices, drawBitmapMesh</th>
            <th>通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用</th>
        </tr>
 		<tr>
            <th>画布剪裁</th>
            <th>clipPath, clipRect</th>
            <th>设置画布的显示区域</th>
        </tr>
 		<tr>
            <th>画布快照</th>
            <th>save, restore, saveLayerXxx, restoreToCount, getSaveCount</th>
            <th>依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数</th>
        </tr>
 		<tr>
            <th>画布变换</th>
            <th>translate, scale, rotate, skew</th>
            <th>依次为 位移、缩放、 旋转、错切</th>
        </tr>
		<tr>
            <th>Matrix(矩阵)</th>
            <th>getMatrix, setMatrix, concat</th>
            <th>实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。</th>
        </tr>
 </table>

> PS： Canvas常用方法在上面表格中已经全部列出了，当然还存在一些其他的方法未列出，具体可以参考官方文档[https://developer.android.com/reference/android/graphics/Canvas.html](https://developer.android.com/reference/android/graphics/Canvas.html)

#三.Canvas详解#
本篇内容主要讲解如何利用Canvas绘制基本图形。

**绘制颜色：**

    canvas.drawColor(Color.BLUE); //绘制蓝色

![](https://camo.githubusercontent.com/b2f869c619c0a1f905ab0ee305cf8e121422fee6/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237343234333777336a333075303168636a72712e6a7067)

**创建画笔：**

要想绘制内容，首先需要先创建一个画笔，如下：

     // 1.创建一个画笔
  	private Paint mPaint = new Paint();
  
  	// 2.初始化画笔
    private void initPaint() {
        mPaint.setColor(Color.BLACK);       //设置画笔颜色
        mPaint.setStyle(Paint.Style.FILL);  //设置画笔模式为填充
        mPaint.setStrokeWidth(10f);         //设置画笔宽度为10px
    }
  
  	// 3.在构造函数中初始化
    public SloopView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }

在创建完画笔之后，就可以在Canvas中绘制各种内容了。

**绘制点：**

可以绘制一个点，也可以绘制一组点，如下：

     canvas.drawPoint(200, 200, mPaint);     //在坐标(200,200)位置绘制一个点
        canvas.drawPoints(new float[]{          //绘制一组点，坐标位置由float数组指定
                500,500,
                500,600,
                500,700
        },mPaint);

![](https://camo.githubusercontent.com/8759eb34cb0afdeb325d95b23b4316c43d5e7225/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a77316632373433726b69666e6a3330753031686337346e2e6a7067)

**绘制直线：**

绘制直线需要两个点，初始点和结束点，同样绘制直线也可以绘制一条或者绘制一组：

    canvas.drawLine(300,300,500,600,mPaint);    // 在坐标(300,300)(500,600)之间绘制一条直线
        canvas.drawLines(new float[]{               // 绘制一组线 每四数字(两个点的坐标)确定一条线
                100,200,200,200,
                100,300,200,300
        },mPaint);

![](https://camo.githubusercontent.com/d6fc88d4fcbce77ab7afdd40767ec446e8c255a0/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a773166323734356b383379626a333075303168637133642e6a7067)

**绘制矩形：**

确定一个矩形最少需要四个数据，就是对角线的两个点的坐标值，这里一般采用左上角和右下角的两个点的坐标。

关于绘制矩形，Canvas提供了三种重载方法，第一种就是提供四个数值(矩形左上角和右下角两个点的坐标)来确定一个矩形进行绘制。 其余两种是先将矩形封装为Rect或RectF(实际上仍然是用两个坐标点来确定的矩形)，然后传递给Canvas绘制，如下：

    	// 第一种
        canvas.drawRect(100,100,800,400,mPaint);

        // 第二种
        Rect rect = new Rect(100,100,800,400);
        canvas.drawRect(rect,mPaint);

        // 第三种
        RectF rectF = new RectF(100,100,800,400);
        canvas.drawRect(rectF,mPaint);

以上三种方法所绘制出来的结果是完全一样的。

![](https://camo.githubusercontent.com/cd07eb77d3a183e08697dd0df57777f4762aec48/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237343738363932646a333075303168633379792e6a7067)

看到这里,相信很多观众会产生一个疑问，为什么会有Rect和RectF两种？两者有什么区别吗？

答案当然是存在区别的，两者最大的区别就是精度不同，Rect是int(整形)的，而RectF是float(单精度浮点型)的。除了精度不同，两种提供的方法也稍微存在差别，在这里我们暂时无需关注，想了解更多参见官方文档 Rect 和 RectF

**绘制圆角矩形：**

绘制圆角矩形也提供了两种重载方式，如下：

    	// 第一种
        RectF rectF = new RectF(100,100,800,400);
        canvas.drawRoundRect(rectF,30,30,mPaint);
        
        // 第二种
        canvas.drawRoundRect(100,100,800,400,30,30,mPaint);

上面两种方法绘制效果也是一样的，但鉴于第二种方法在API21的时候才添加上，所以我们一般使用的都是第一种。

![](https://camo.githubusercontent.com/1289c92b4e3c8bc24ca08da2827fae74c76268bf/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a77316632373437733363357a6a333075303168637133652e6a7067)

下面简单解析一下圆角矩形的几个必要的参数的意思。

很明显可以看出，第二种方法前四个参数和第一种方法的RectF作用是一样的，都是为了确定一个矩形，最后一个参数Paint是画笔，无需多说，与矩形相比，圆角矩形多出来了两个参数rx 和 ry，这两个参数是干什么的呢？

稍微分析一下，既然是圆角矩形，他的角肯定是圆弧(圆形的一部分)，我们一般用什么确定一个圆形呢？

答案是圆心 和 半径，其中圆心用于确定位置，而半径用于确定大小。

由于矩形位置已经确定，所以其边角位置也是确定的，那么确定位置的参数就可以省略，只需要用半径就能描述一个圆弧了。

但是，半径只需要一个参数，但这里怎么会有两个呢？

好吧，让你发现了，这里圆角矩形的角实际上不是一个正圆的圆弧，而是椭圆的圆弧，这里的两个参数实际上是椭圆的两个半径，他们看起来个如下图：

![](https://camo.githubusercontent.com/8adaa75b0812d66c85f318b46b09fd1c13ed8bd2/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a77316632373438666a7732626a333038633064776d78382e6a7067)

**红线标注的 rx 与 ry 就是两个半径，也就是相比绘制矩形多出来的那两个参数。**

我们了解到原理后，就可以为所欲为了，通过计算可知我们上次绘制的矩形宽度为700，高度为300，当你让 rx大于350(宽度的一半)， ry大于150(高度的一半) 时奇迹就出现了， 你会发现圆角矩形变成了一个椭圆， 他们画出来是这样的 ( 为了方便确认我更改了画笔颜色， 同时绘制出了矩形和圆角矩形 )：

    	// 矩形
        RectF rectF = new RectF(100,100,800,400);  
        
        // 绘制背景矩形
        mPaint.setColor(Color.GRAY);
        canvas.drawRect(rectF,mPaint);
        
        // 绘制圆角矩形
        mPaint.setColor(Color.BLUE);
        canvas.drawRoundRect(rectF,700,400,mPaint);

![](https://camo.githubusercontent.com/92871a109735a74aa2aefd55475e6eb134a4440f/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237343875677932706a333075303168637766312e6a7067)

其中灰色部分是我们所选定的矩形，而里面的圆角矩形则变成了一个椭圆，实际上在rx为宽度的一半，ry为高度的一半时，刚好是一个椭圆，通过上面我们分析的原理推算一下就能得到，而当rx大于宽度的一半，ry大于高度的一半时，实际上是无法计算出圆弧的，所以drawRoundRect对大于该数值的参数进行了限制(修正)，凡是大于一半的参数均按照一半来处理。

**绘制椭圆：**

相对于绘制圆角矩形，绘制椭圆就简单的多了，因为他只需要一个矩形作为参数:

    	// 第一种
        RectF rectF = new RectF(100,100,800,400);
        canvas.drawOval(rectF,mPaint);

        // 第二种
        canvas.drawOval(100,100,800,400,mPaint);

同样，以上两种方法效果完全一样，但一般使用第一种。

![](https://camo.githubusercontent.com/50d39a4865bcbed149103ae9dd31cd873d5d451c/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237346166786269796a333075303168637a6b732e6a7067)

绘制椭圆实际上就是绘制一个矩形的内切图形，原理如下，就不多说了：

![](https://camo.githubusercontent.com/3dd5f7bdb7466a3b725c9f8a1760deb3a596b4e2/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237346271316834726a333038633064776a726c2e6a7067)

> PS： 如果你传递进来的是一个长宽相等的矩形(即正方形)，那么绘制出来的实际上就是一个圆。

**绘制圆：**

绘制圆形也比较简单, 如下：

     canvas.drawCircle(500,500,400,mPaint);  // 绘制一个圆心坐标在(500,500)，半径为400 的圆。

绘制圆形有四个参数，前两个是圆心坐标，第三个是半径，最后一个是画笔。

![](https://camo.githubusercontent.com/fd3490a1259e28d3c39e3b6e12c9bf4e72312da6/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237346334316b6b6e6a333075303168636467662e6a7067)

**绘制圆弧：**

绘制圆弧就比较神奇一点了，为了理解这个比较神奇的东西，我们先看一下它需要的几个参数：

    // 第一种
	public void drawArc(@NonNull RectF oval, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint){}
    
	// 第二种
	public void drawArc(float left, float top, float right, float bottom, float startAngle,float sweepAngle, boolean useCenter, @NonNull Paint paint) {}

从上面可以看出，相比于绘制椭圆，绘制圆弧还多了三个参数：

    startAngle  // 开始角度
	sweepAngle  // 扫过角度
	useCenter   // 是否使用中心

通过字面意思我们基本能猜测出来前两个参数(startAngle， sweepAngel)的作用，就是确定角度的起始位置和扫过角度， 不过第三个参数是干嘛的？试一下就知道了,上代码：

      	RectF rectF = new RectF(100,100,800,400);
        // 绘制背景矩形
        mPaint.setColor(Color.GRAY);
        canvas.drawRect(rectF,mPaint);

        // 绘制圆弧
        mPaint.setColor(Color.BLUE);
        canvas.drawArc(rectF,0,90,false,mPaint);

        //-------------------------------------

        RectF rectF2 = new RectF(100,600,800,900);
        // 绘制背景矩形
        mPaint.setColor(Color.GRAY);
        canvas.drawRect(rectF2,mPaint);

        // 绘制圆弧
        mPaint.setColor(Color.BLUE);
        canvas.drawArc(rectF2,0,90,true,mPaint);

上述代码实际上是绘制了一个起始角度为0度，扫过90度的圆弧，两者的区别就是是否使用了中心点，结果如下：

![](https://camo.githubusercontent.com/6f04d0e69c95c1bc53d6324434a1cb7652e961f1/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237346431736d77656a33307530316863337a342e6a7067)

可以发现使用了中心点之后绘制出来类似于一个扇形，而不使用中心点则是圆弧起始点和结束点之间的连线加上圆弧围成的图形。这样中心点这个参数的作用就很明显了，不必多说想必大家试一下就明白了

     	RectF rectF = new RectF(100,100,600,600);
        // 绘制背景矩形
        mPaint.setColor(Color.GRAY);
        canvas.drawRect(rectF,mPaint);

        // 绘制圆弧
        mPaint.setColor(Color.BLUE);
        canvas.drawArc(rectF,0,90,false,mPaint);

        //-------------------------------------

        RectF rectF2 = new RectF(100,700,600,1200);
        // 绘制背景矩形
        mPaint.setColor(Color.GRAY);
        canvas.drawRect(rectF2,mPaint);

        // 绘制圆弧
        mPaint.setColor(Color.BLUE);
        canvas.drawArc(rectF2,0,90,true,mPaint);

![](https://camo.githubusercontent.com/f4ff154c89cbf1ba3123fb66dc41c10be088ad51/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a7731663237346533737572676a33307530316863337a342e6a7067)


#简要介绍Paint#
看了上面这么多，相信有一部分人会产生一个疑问，如果我想绘制一个圆，只要边不要里面的颜色怎么办？

很简单，绘制的**基本形状由Canvas确定**，但绘制出来的**颜色,具体效果则由Paint确定**。

如果你注意到了的话，在一开始我们设置画笔样式的时候是这样的：

     mPaint.setStyle(Paint.Style.FILL);  //设置画笔模式为填充

为了展示方便，容易看出效果，之前使用的模式一直为填充模式，实际上画笔有三种模式，如下：

    STROKE                //描边
	FILL                  //填充
	FILL_AND_STROKE       //描边加填充

为了区分三者效果我们做如下实验：

        Paint paint = new Paint();
        paint.setColor(Color.BLUE);
        paint.setStrokeWidth(40);     //为了实验效果明显，特地设置描边宽度非常大

        // 描边
        paint.setStyle(Paint.Style.STROKE);
        canvas.drawCircle(200,200,100,paint);

        // 填充
        paint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(200,500,100,paint);

        // 描边加填充
        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        canvas.drawCircle(200, 800, 100, paint);

一图胜千言，通过以上实验我们可以比较明显的看出三种模式的区别，如果只需要边缘不需要填充内容的话只需要设置模式为描边(STROKE)即可。

其实关于Paint的内容也是有不少的，这些只是冰山一角，在后续内容中会详细的讲解Paint

#小示例#
##简要介绍画布的操作##
画布操作详细内容会在下一篇文章中讲解, 不是本文重点，但以下示例中可能会用到，所以此处简要介绍一下。

<table>
	<tr>
		<th>相关操作</th>
		<th>简要介绍</th>
	</tr>
	<tr>
		<th>save</th>
		<th>保存当前画布状态</th>
	</tr>
	<tr>
		<th>restore</th>
		<th>回滚到上一次保存的状态</th>
	</tr>
	<tr>
		<th>translate</th>
		<th>相对于当前位置位移</th>
	</tr>
	<tr>
		<th>rotate</th>
		<th>旋转</th>
	</tr>
</table>

##制作一个饼状图##
在展示百分比数据的时候经常会用到饼状图，像这样：

![](https://camo.githubusercontent.com/9cdca1b8fcf82860cc0b0522d18cb3c19fb7128a/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a773166323734676d6e6c6b336a33303863306477676c712e6a7067)

##简单分析##
其实根据我们上面的知识已经能自己制作一个饼状图了。不过制作东西最重要的不是制作结果，而是制作思路。 相信我贴上代码大家一看就立刻明白了，非常简单的东西。不过嘛，咱们还是想了解一下制作思路：

先分析饼状图的构成，非常明显，饼状图就是一个又一个的扇形构成的，每个扇形都有不同的颜色，对应的有名字，数据和百分比。

经以上信息可以得出饼状图的最基本数据应包括：**名字 数据值 百分比 对应的角度 颜色。**

**用户关心的数据 ： 名字 数据值 百分比**
**需要程序计算的数据： 百分比 对应的角度**
**其中颜色这一项可以用户指定也可以用程序指定(我们这里采用程序指定)。**

##封装数据##

    public class PieData {
	    // 用户关心数据
	    private String name;        // 名字
	    private float value;        // 数值
	    private float percentage;   // 百分比
	    
	    // 非用户关心数据
	    private int color = 0;      // 颜色
	    private float angle = 0;    // 角度
	
	    public PieData(@NonNull String name, @NonNull float value) {
	        this.name = name;
	        this.value = value;
	    }
	}

> PS: 以上省略了get set方法

##自定义View：##
先按照自定义View流程梳理一遍(确定各个步骤应该做的事情)：

<table>
	<tr>
		<th>步骤</th>
		<th>关键字</th>
		<th>作用</th>
	</tr>
	<tr>
		<th>1</th>
		<th>构造函数</th>
		<th>初始化(初始化画笔Paint)</th>
	</tr>
	<tr>
		<th>2</th>
		<th>onMeasure</th>
		<th>测量View的大小(暂时不用关心)</th>
	</tr>
	<tr>
		<th>3</th>
		<th>onSizeChanged</th>
		<th>确定View大小(记录当前View的宽高)</th>
	</tr>
	<tr>
		<th>4</th>
		<th>onLayout</th>
		<th>确定子View布局(无子View，不关心)</th>
	</tr>
	<tr>
		<th>5</th>
		<th>onDraw</th>
		<th>实际绘制内容(绘制饼状图)</th>
	</tr>
	<tr>
		<th>6</th>
		<th>提供接口</th>
		<th>提供接口(提供设置数据的接口)</th>
	</tr>
</table>

代码如下：

    public class PieView extends View {
	    // 颜色表(注意: 此处定义颜色使用的是ARGB，带Alpha通道的)
	    private int[] mColors = {0xFFCCFF00, 0xFF6495ED, 0xFFE32636, 0xFF800000, 0xFF808000, 0xFFFF8C69, 0xFF808080,
	            0xFFE6B800, 0xFF7CFC00};
	    // 饼状图初始绘制角度
	    private float mStartAngle = 0;
	    // 数据
	    private ArrayList<PieData> mData;
	    // 宽高
	    private int mWidth, mHeight;
	    // 画笔
	    private Paint mPaint = new Paint();
	
	    public PieView(Context context) {
	        this(context, null);
	    }
	
	    public PieView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        mPaint.setStyle(Paint.Style.FILL);
	        mPaint.setAntiAlias(true);
	    }
	
	    @Override
	    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
	        super.onSizeChanged(w, h, oldw, oldh);
	        mWidth = w;
	        mHeight = h;
	    }
	
	    @Override
	    protected void onDraw(Canvas canvas) {
	        super.onDraw(canvas);
	        if (null == mData)
	            return;
	        float currentStartAngle = mStartAngle;                    // 当前起始角度
	        canvas.translate(mWidth / 2, mHeight / 2);                // 将画布坐标原点移动到中心位置
	        float r = (float) (Math.min(mWidth, mHeight) / 2 * 0.8);  // 饼状图半径
	        RectF rect = new RectF(-r, -r, r, r);                     // 饼状图绘制区域
	
	        for (int i = 0; i < mData.size(); i++) {
	            PieData pie = mData.get(i);
	            mPaint.setColor(pie.getColor());
	            canvas.drawArc(rect, currentStartAngle, pie.getAngle(), true, mPaint);
	            currentStartAngle += pie.getAngle();
	        }
	
	    }
	
	    // 设置起始角度
	    public void setStartAngle(int mStartAngle) {
	        this.mStartAngle = mStartAngle;
	        invalidate();   // 刷新
	    }
	
	    // 设置数据
	    public void setData(ArrayList<PieData> mData) {
	        this.mData = mData;
	        initData(mData);
	        invalidate();   // 刷新
	    }
	
	    // 初始化数据
	    private void initData(ArrayList<PieData> mData) {
	        if (null == mData || mData.size() == 0)   // 数据有问题 直接返回
	            return;
	
	        float sumValue = 0;
	        for (int i = 0; i < mData.size(); i++) {
	            PieData pie = mData.get(i);
	
	            sumValue += pie.getValue();       //计算数值和
	
	            int j = i % mColors.length;       //设置颜色
	            pie.setColor(mColors[j]);
	        }
	
	        float sumAngle = 0;
	        for (int i = 0; i < mData.size(); i++) {
	            PieData pie = mData.get(i);
	
	            float percentage = pie.getValue() / sumValue;   // 百分比
	            float angle = percentage * 360;                 // 对应的角度
	
	            pie.setPercentage(percentage);                  // 记录百分比
	            pie.setAngle(angle);                            // 记录角度大小
	            sumAngle += angle;
	
	            Log.i("angle", "" + pie.getAngle());
	        }
	    }
	}

> **PS: 在更改了数据需要重绘界面时要调用invalidate()这个函数重新绘制。**

##效果图##

![](https://camo.githubusercontent.com/505e5b1707a11bdf625e6c776af561dbef63bda7/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a773166323734677a3036766f6a33307530316863337a612e6a7067)

> **PS: 这个饼状图并没有添加百分比等数据，仅作为示例使用。**