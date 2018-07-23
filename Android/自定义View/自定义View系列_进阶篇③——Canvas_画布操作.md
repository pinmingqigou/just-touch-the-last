#一.Canvas的常用操作速查表#

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

#二.Canvas基本操作#
##1.画布操作##
**为什么要有画布操作？**

画布操作可以帮助我们用更加容易理解的方式制作图形。

例如： 从坐标原点为起点，绘制一个长度为20dp，与水平线夹角为30度的线段怎么做？

按照我们通常的想法(被常年训练出来的数学思维)，就是先使用三角函数计算出线段结束点的坐标，然后调用drawLine即可。

然而这是否是被固有思维禁锢了？

假设我们先绘制一个长度为20dp的水平线，然后将这条水平线旋转30度，则最终看起来效果是相同的，而且不用进行三角函数计算，这样是否更加简单了一点呢？

**合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果，这也是画布操作存在的原因。**

**PS: 所有的画布操作都只影响后续的绘制，对之前已经绘制过的内容没有影响。**

###A 位移(translate)###
translate是坐标系的移动，可以为图形绘制选择一个合适的坐标系。** 请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动，如下：**

![](https://camo.githubusercontent.com/91eab33ede633cc7100e3e042e6b916fe7fa0d30/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266317068343671616a33307530316863676d332e6a7067)

我们首先将坐标系移动一段距离绘制一个圆形，之后再移动一段距离绘制一个圆形，**两次移动是可叠加的。**

###B 缩放(scale)###

缩放提供了两个方法，如下：

    public void scale (float sx, float sy)

 	public final void scale (float sx, float sy, float px, float py)

这两个方法中前两个参数是相同的分别为x轴和y轴的缩放比例。而第二种方法比前一种多了两个参数，用来控制缩放中心位置的。

缩放比例(sx,sy)取值范围详解：

<table>
	<tr>
		<th>
			取值范围（n）
		</th>
		<th>
			说明
		</th>
	</tr>

	<tr>
		<th>
			[-∞, -1)
		</th>
		<th>
			先根据缩放中心放大n倍，再根据中心轴进行翻转
		</th>
	</tr>

	<tr>
		<th>
			-1
		</th>
		<th>
			根据缩放中心轴进行翻转
		</th>
	</tr>

	<tr>
		<th>
			(-1, 0)
		</th>
		<th>
			先根据缩放中心缩小到n，再根据中心轴进行翻转
		</th>
	</tr>

	<tr>
		<th>
			0
		</th>
		<th>
			不会显示，若sx为0，则宽度为0，不会显示，sy同理
		</th>
	</tr>

	<tr>
		<th>
			(0, 1)
		</th>
		<th>
			根据缩放中心缩小到n
		</th>
	</tr>

	<tr>
		<th>
			1
		</th>
		<th>
			没有变化
		</th>
	</tr>
	<tr>
		<th>
			(1, +∞)
		</th>
		<th>
			根据缩放中心放大n倍
		</th>
	</tr>
</table>

如果在缩放时稍微注意一下就会发现**缩放的中心默认为坐标原点,而缩放中心轴就是坐标轴**，如下：

		// 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f);                // 画布缩放

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

(为了更加直观，我添加了一个坐标系，可以比较明显的看出，缩放中心就是坐标原点)

![](https://camo.githubusercontent.com/22c4ddb8ff33c063017690570fa88b5cd09f6ed0/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a773166326631767068646a6a6a333075303168637439722e6a7067)

接下来我们使用第二种方法让缩放中心位置稍微改变一下，如下：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

(图中用箭头指示的就是缩放中心。)

![](https://camo.githubusercontent.com/f689a66c90b4c1fc9015b907e863932186b53e55/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a77316632663177376b7638646a333075303168637439732e6a7067)

前面两个示例缩放的数值都是正数，按照表格中的说明，当缩放比例为负数的时候会根据缩放中心轴进行翻转，下面我们就来实验一下：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);


        canvas.scale(-0.5f,-0.5f);          // 画布缩放

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/d1f1e49974434ccf789e7776e034a9f62963d20c/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266317837366f36716a333075303168633074752e6a7067)

> 为了效果明显，这次我不仅添加了坐标系而且对矩形中几个重要的点进行了标注，具有相同字母标注的点是一一对应的。

由于本次未对缩放中心进行偏移，所有默认的缩放中心就是坐标原点，中心轴就是x轴和y轴。

本次缩放可以看做是先根据缩放中心(坐标原点)缩放到原来的0.5倍，然后分别按照x轴和y轴进行翻转。

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);


        canvas.scale(-0.5f,-0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/fe018f338a3173eee4ad985957ba6b5882a73206/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266317874683470366a333075303168633075342e6a7067)

> 添加了这么多的辅助内容，希望大家能够看懂。

本次对缩放中心点y轴坐标进行了偏移，故中心轴也向右偏移了。

**PS:和位移(translate)一样，缩放也是可以叠加的。**

       canvas.scale(0.5f,0.5f);
    
       canvas.scale(0.5f,0.1f);

调用两次缩放则 x轴实际缩放为0.5x0.5=0.25 y轴实际缩放为0.5x0.1=0.05

下面我们利用这一特性制作一个有趣的图形。

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(-400,-400,400,400);   // 矩形区域

        for (int i=0; i<=20; i++)
        {
            canvas.scale(0.9f,0.9f);
            canvas.drawRect(rect,mPaint);
        }

![](https://camo.githubusercontent.com/906107696fda7130f1c7821e58863e37509a738b/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a77316632663179666e3232786a333075303168637461392e6a7067)

###C 旋转(rotate)###
旋转提供了两种方法：

      public void rotate (float degrees)
      
      public final void rotate (float degrees, float px, float py)

和缩放一样，第二种方法多出来的两个参数依旧是控制旋转中心点的。

默认的旋转中心依旧是坐标原点：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/a66e51e38c7006008e7d2125ffd14b981efa1efa/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a77316632663179777333386e6a333075303168636d79382e6a7067)

改变旋转中心位置：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180,200,0);               // 旋转180度 <-- 旋转中心向右偏移200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/4055156947ee23090057590f1265e9fa2d6d4136/687474703a2f2f7777322e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266317a636d7762326a333075303168636d79392e6a7067)

好吧，旋转也是可叠加的

     canvas.rotate(180);
     canvas.rotate(20);

调用两次旋转，则实际的旋转角度为180+20=200度。

为了演示这一个效果，我做了一个不明觉厉的东西：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        canvas.drawCircle(0,0,400,mPaint);          // 绘制两个圆形
        canvas.drawCircle(0,0,380,mPaint);

        for (int i=0; i<=360; i+=10){               // 绘制圆形之间的连接线
            canvas.drawLine(0,380,0,400,mPaint);
            canvas.rotate(10);
        }

![](https://camo.githubusercontent.com/ef82f8339bb1f7f90bc559818758bfd968a29f96/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266317a736e6a30306a333075303168633735612e6a7067)

###D 错切(skew)###
skew这里翻译为错切，错切是特殊类型的线性变换。

错切只提供了一种方法：
    
      public void skew (float sx, float sy)

**参数含义：**

**float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值**

**float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值**

变换后:

    X = x + sx * y
    
    Y = y + sy * x

示例：

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 水平错切 <- 45度

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/d29b0d0ae0caa7ca682fbdb58157ecb114b4d72d/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266323068376932336a333075303168636467712e6a7067)

**如你所想，错切也是可叠加的，不过请注意，调用次序不同绘制结果也会不同**

        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 水平错切
        canvas.skew(0,1);                       // 垂直错切

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);

![](https://camo.githubusercontent.com/bb6372a6f9ce637c362113437c409b78c5e073da/687474703a2f2f7777332e73696e61696d672e636e2f6c617267652f30303558746469326a7731663266323077307266666a33307530316863676d382e6a7067)

###E 快照(save)和回滚(restore)###
**Q: 为什存在快照与回滚**

**A：画布的操作是不可逆的，而且很多画布操作会影响后续的步骤，例如第一个例子，两个圆形都是在坐标原点绘制的，而因为坐标系的移动绘制出来的实际位置不同。所以会对画布的一些状态进行保存和回滚。
与之相关的API:**

<table>
	<tr>
		<th>
		相关API
		</th>
		<th>
		简介
		</th>
	</tr>

	<tr>
		<th>
		save
		</th>
		<th>
		把当前的画布的状态进行保存，然后放入特定的栈中
		</th>
	</tr>

	<tr>
		<th>
		saveLayerXxx
		</th>
		<th>
		新建一个图层，并放入特定的栈中
		</th>
	</tr>

	<tr>
		<th>
		restore
		</th>
		<th>
		把栈中最顶层的画布状态取出来，并按照这个状态恢复当前的画布
		</th>
	</tr>

	<tr>
		<th>
		restoreToCount
		</th>
		<th>
		弹出指定位置及其以上所有的状态，并按照指定位置的状态进行恢复
		</th>
	</tr>

	<tr>
		<th>
		getSaveCount
		</th>
		<th>
		获取栈中内容的数量(即保存次数)
		</th>
	</tr>
</table>

下面对其中的一些概念和方法进行分析：

**状态栈：**

其实这个栈我也不知道叫什么名字，暂时叫做状态栈吧，它看起来像下面这样：

![](https://camo.githubusercontent.com/2658da04cbb54e51269897d63d35f44e524085da/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a773166326766716d753273676a333038633064776d78772e6a7067)

这个栈可以存储画布状态和图层状态。

**Q：什么是画布和图层？**

**A：实际上我们看到的画布是由多个图层构成的，如下图(图片来自网络)：**

![](https://camo.githubusercontent.com/b37d4315958821445ad9b889b5eb4db325512de4/687474703a2f2f7777312e73696e61696d672e636e2f6c617267652f30303558746469326a77316632676672633666646b6a33303863306477676c722e6a7067)

**实际上我们之前讲解的绘制操作和画布操作都是在默认图层上进行的。
在通常情况下，使用默认图层就可满足需求，但是如果需要绘制比较复杂的内容，如地图(地图可以有多个地图层叠加而成，比如：政区层，道路层，兴趣点层)等，则分图层绘制比较好一些。
你可以把这些图层看做是一层一层的玻璃板，你在每层的玻璃板上绘制内容，然后把这些玻璃板叠在一起看就是最终效果。**

**save 方法**

      // 保存全部状态
      public int save ()
      
      // 根据saveFlags参数保存一部分状态
      public int save (int saveFlags)

<table>
	<tr>
		<th>
		数据类型
		</th>
		<th>
		名称（saveFlags）
		</th>
		<th>
		简介
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		ALL_SAVE_FLAG
		</th>
		<th>
		默认，保存全部状态
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		CLIP_SAVE_FLAG
		</th>
		<th>
		保存剪辑区
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		CLIP_TO_LAYER_SAVE_FLAG
		</th>
		<th>
		剪裁区作为图层保存
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		FULL_COLOR_LAYER_SAVE_FLAG
		</th>
		<th>
		保存图层的全部色彩通道
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		HAS_ALPHA_LAYER_SAVE_FLAG
		</th>
		<th>
		保存图层的alpha(不透明度)通道
		</th>
	</tr>

	<tr>
		<th>
		int
		</th>
		<th>
		MATRIX_SAVE_FLAG
		</th>
		<th>
		保存Matrix信息(translate, rotate, scale, skew)
		</th>
	</tr>
</table>

可以看到第二种方法比第一种多了一个saveFlags参数，使用这个参数可以只保存一部分状态，更加灵活，这个saveFlags参数具体可参考上面表格中的内容。

每调用一次save方法，都会在栈顶添加一条状态信息，以上面状态栈图片为例，再调用一次save则会在第5次上面载添加一条状态。

**saveLayerXxx 方法**

saveLayerXxx有比较多的方法：

    // 无图层alpha(不透明度)通道
    public int saveLayer (RectF bounds, Paint paint)
    public int saveLayer (RectF bounds, Paint paint, int saveFlags)
    public int saveLayer (float left, float top, float right, float bottom, Paint paint)
    public int saveLayer (float left, float top, float right, float bottom, Paint paint, int saveFlags)
    
    // 有图层alpha(不透明度)通道
    public int saveLayerAlpha (RectF bounds, int alpha)
    public int saveLayerAlpha (RectF bounds, int alpha, int saveFlags)
    public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha)
    public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha, int saveFlags)

**注意：saveLayerXxx方法会让你花费更多的时间去渲染图像(图层多了相互之间叠加会导致计算量成倍增长)，使用前请谨慎，如果可能，尽量避免使用。**

使用saveLayerXxx方法，也会将图层状态也放入状态栈中，同样使用restore方法进行恢复。

这个暂时不过多讲述，如果以后用到详细讲解。(因为这里面东西也有不少啊QAQ)

**restore 方法**

状态回滚，就是从栈顶取出一个状态然后根据内容进行恢复。

同样以上面状态栈图片为例，调用一次restore方法则将状态栈中第5次取出，根据里面保存的状态进行状态恢复。

**restoreToCount**

弹出指定位置以及以上所有状态，并根据指定位置状态进行恢复。

以上面状态栈图片为例，如果调用restoreToCount(2) 则会弹出 2 3 4 5 的状态，并根据第2次保存的状态进行恢复。

**getSaveCount**

获取保存的次数，即状态栈中保存状态的数量，以上面状态栈图片为例，使用该函数的返回值为5。

不过请注意，该函数的最小返回值为1，即使弹出了所有的状态，返回值依旧为1，代表默认状态。

**常用格式**
虽然关于状态的保存和回滚啰嗦了不少，不过大多数情况下只需要记住下面的步骤就可以了：

       save();  //保存状态
    
       ...  //具体操作
    
       restore();   //回滚到之前的状态
    
这种方式也是最简单和最容易理解的使用方法。