#标签（建议使用小写字母）
- h1、h2...h6：标题（headline）
- p：段落（paragraph）
- a：链接

    <a href="http://www.w3school.com.cn">This is a link</a>

	**href属性创建指向另一个文档的链接**<br>
	**name属性创建文档内的书签**<br>
	**通过name属性创建书签后，可以通过href属性直接跳转到该书签位置**

- img：图像

    <img src="w3school.jpg" width="104" height="142" />
- br：定义换行,推荐使用<br/>
- hr:创建水平线（horizontal）,推荐使用<hr/>
- 文本格式化标签：<a href ="http://www.w3school.com.cn/html/html_formatting.asp">详细标签请看这里</a>
- q:短的引用，通常浏览器会用引号展示q标签的内容
   **<q>曾着卖糖君子哄，至今不信口甜人</q>**
- blockquote：长引用，浏览器会对blockquote标签的内容进行缩进处理
	<blockquote>我是长长长长长引用</blockquote>
- abbr:定义缩写或首字母缩略语

    <abbr title="狗吃剩下的东西">狗剩</abbr>
- dfn：可用abbr标签代替
- address：定义文档或文章的联系信息（作者/拥有者）
	<address>主角：路飞<br>兄弟：艾斯<br>小弟：乌索普、娜美、山治、索隆</address>
- cite：定义著作的标题，通常以斜体显示
	<p><cite>西游记</cite> 是中国玄幻小说鼻祖</p>
- bdo（bi-directional override）：定义双流向覆盖，覆盖当前文本方向
	<p><bdo>上海自来水来自海上,黄山落叶松叶落山黄</bdo></p>
- code：定义计算机代码文本
- kbd（keyboarding）：定义键盘输入文本
- samp：定义计算机代码示例
- var：定义变量
- pre：定义预格式化文本
- table:定义表格，其中可以用属性border定义边框宽度，没有此属性不显示边框
	- tr：定义表格中的行
	- td：定义每一行的各个单元格（table data）
- ul：定义无序列表（unordered list）
	- li：无序列表中的每个项目被此标签包围
		<ul>
			<li>孙悟空</li><li>猪八戒</li>
		</ul>
- ol：定义有序列表（ordered list）
	<ol>
		<li>沙和尚</li>
		<li>唐三藏</li>
	</ol>
- dl:自定义列表开始标签，自定义列表不仅仅是一列项目，而是项目及其注释的组合
	- dt：每个自定义列表项的开始标签
	- dd：每个自定义列表项的定义（注释）的开始标签
	<dl>
		<dt>孙悟空</dt>
		<dd>大闹天宫</dd>
		<dt>猪八戒</dt>
		<dd>调戏嫦娥</dd>
	</dl>
- div:块级元素,组合其他元素的容器，没有特定的含义，方便对包围元素添加属性进行格式化
	<div style = "font-family:arial;color:red;font-size:20px">
		<h1>赌神</h1>
		<p>从不拍正面照片</p>
	</div>
- span:内联元素，可作为文本的容器，没有特定的含义，方便对包围文本添加属性进行格式化
#属性
HTML 标签可以拥有属性。属性提供了有关 HTML 元素的更多的信息。属性总是以名称/值对的形式出现，比如：name="value"。属性总是在 HTML 元素的开始标签中规定

- style:提供了一种改变所有 HTML 元素的样式的通用方法。
	- 背景颜色：background-color

    <html>
		<body style="background-color:yellow">
			<h2 style="background-color:red">This is a heading</h2>
			<p style="background-color:green">This is a paragraph.</p>
		</body>
	</html>
	
	- 字体、颜色、尺寸

    <html>
		<body>
			<h1 style="font-family:verdana">A heading</h1>
			<p style="font-family:arial;color:red;font-size:20px;">A paragraph.</p>
		</body>
	</html>

	- 文本对齐样式

    <html>
		<body>
			<h1 style="text-align:center">This is a heading</h1>
			<p>The heading above is aligned to the center of this page.</p>
		</body>
	</html>
	