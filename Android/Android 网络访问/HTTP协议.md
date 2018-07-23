#目录#
![](http://upload-images.jianshu.io/upload_images/944365-6fde731a5fce94e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#计算机网络相关知识#
计算机网络体系结构分为五层，自上而下分别是应用、运输、网络、数据链路和物理层，如下图：

![](http://upload-images.jianshu.io/upload_images/944365-c9d9e61e63b966b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HTTP存在于最高层的应用层，简单介绍下应用层：

- 作用

	通过**应用层协议**定义应用进程间（运行的程序）的通信规则
	> 应用层协议主要有HTTP、SMTP、FTP协议等等

- 交互的数据单元称为报文
- 基本上是基于C/S方式

#HTTP介绍#
##定义##
即HyperText Transfer Protocol，超文本传输协议，属于应用层协议的一种
##作用##
规定了应用进程间通信（请求&响应）的准则
##特点##
- 无连接：HTTP本身是无连接的，即交换HTTP报文前不需要建立HTTP连接
- 无状态：HTTP协议是无状态的：数据传输过程中，并不保存任何历史信息和状态信息。无状态特性简化了服务器的设计，使服务器更容易支持大量并发的HTPP请求。
- 传输可靠性高：采用TCP作为运输层协议（面向连接、可靠传输），即交换报文时需要预先建立TCP连接
- 兼容性好：支持B/S模式(Browser/Serve)及C/S(Client/Server)模式；
- 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST
- 灵活：HTTP 允许传输任意类型的数据对象

##工作方式##
HTTP协议采用了请求/响应的工作方式，工作流程如图：
![](http://upload-images.jianshu.io/upload_images/944365-c7dbdbe6743d3abd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##HTTP报文详解##
HTTP的报文分为请求报文和响应报文

###HTTP请求报文###
**HTTP请求报文的组成**

![](http://upload-images.jianshu.io/upload_images/944365-167824523bd0692e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 请求行：用于声明”请求报文“、主机域名、资源路径和协议版本
- 请求头：说明客户端、服务器或报文的部分信息
- 请求体：用于存放需要发送给服务器的数据信息

####请求行####
- 组成

![](http://upload-images.jianshu.io/upload_images/944365-ee23b153f6d654c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 空格不能省

- 组成介绍

	**1 请求方法：即对请求对象的操作，请求方法有8种：**
	<table>
		<tr>
			<th>
				方法类别
			</th>
			<th>
				意义
			</th>
		</tr>
		<tr>
			<th>
				OPTION
			</th>
			<th>
				请求“选项”的信息
			</th>
		</tr>
		<tr>
			<th>
				HEAD
			</th>
			<th>
				请求读取”URL标志信息的首部“信息
			</th>
		</tr>
		<tr>
			<th>
				GET
			</th>
			<th>
				请求读取“URL标志的信息“的信息
			</th>
		</tr>
		<tr>
			<th>
				POST
			</th>
			<th>
				为服务器添加信息
			</th>
		</tr>
		<tr>
			<th>
				PUT
			</th>
			<th>
				为指定的URL下添加（存储）一个文档
			</th>
		</tr>
		<tr>
			<th>
				DELETE
			</th>
			<th>
				删除指定URL所标志的信息
			</th>
		</tr>
		<tr>
			<th>
				TRACE
			</th>
			<th>
				用于进行环回测试的请求报文
			</th>
		</tr>
		<tr>
			<th>
				CONNECT
			</th>
			<th>
				用于代理服务器
			</th>
		</tr>
	</table>

	**最常用的就是GET和POST方法**

	**2 请求路径**

	要了解请求地址，先来了解下URL概念：
	
	- 定义：Uniform Resoure Locator，统一资源定位符，是一种自愿位置的抽象唯一识别方法。
	- 作用：用于表示资源位置和访问这些资源的方法
	- 组成：

		<协议>：//<主机>：<端口>/<路径>
		
		
	> 协议：采用的应用层通信协议，比如在HTTP协议下的URL地址：
		HTTP：//<主机>：<端口>/<路径>

	> 主机：请求资源所在主机的域名

	> 端口和路径有时可以省略（HTTP默认端口号是80）

	从上面可以了解到，路径则是端口号后面符号”/“的部分，下面举例
	
	<table>
		<tr>
			<th>
				URL（统一资源定位符）
			</th>
			<th>
				PATH（路径）
			</th>
		</tr>
		<tr>
			<th>
				http://www.baidu.com/
			</th>
			<th>
				/
			</th>
		</tr>
		<tr>
			<th>
				http://www.weibo.com/2874748/home
			</th>
			<th>
				/2874748/home
			</th>
		</tr>
	</table>

	**3 协议版本**
	HTTP协议版本主要是1.0、1.1、2.0

- 请求行举例

	先假设：
	- URL地址为：http://www.tsinghua.edu.cn/chn/yxsz/index.htm
	- 请求报文采用GET方法
	- 请求报文采用HTTP1.1版本
	
	则请求行是：GET /chn/yxsz/index.htm HTTP/1.1

####请求头####
- 作用：说明客户端、服务器或报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式
- 举例：(URL地址：http://www.tsinghua.edu.cn/chn/yxsz/index.htm）

	Host：www.tsinghua.edu.cn (表示主机域名）

	User - Agent：Mozilla/5.0 (表示用户代理是使用Netscape浏览器）

####请求体####
- 作用：用于存放需要发送给服务器的数据信息
- 使用方式：目前来说，一共有三种

**A 数据交换格式**

请求体是可以是任意类型的，但服务器需要额外进行解析，如JSON

**B 键值对形式**

键与值之间用”=“连接，每个键值对间用&连接，且只能用ASCII字符，如Query String

    key1=value1&key2=value2

**C 分部分形式**

请求体被分为多个部分，应用场景是文件上传，比如邮件上传等等

- 每段以-- {boundary}开头
- 然后是该段的描述头
- 描述头之后空一行接内容
- 每段以-- {boundary}--结束

如下：
![](http://upload-images.jianshu.io/upload_images/944365-b52f89dab30dbad6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####请求报文实例####
结合上述说的请求行、请求头和请求体，现假设

- URL地址为：http://www.tsinghua.edu.cn/chn/yxsz/index.htm
- 请求报文采用GET方法
- 请求报文采用HTTP1.1版本
- 请求报文希望表明主机域名和用户代理是使用Netscape浏览器
- 请求体采用键值对形式

则请求报文如下：
![](http://upload-images.jianshu.io/upload_images/944365-7889517ed4ec54bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###HTTP响应报文###
**HTTP响应报文的组成**
![](http://upload-images.jianshu.io/upload_images/944365-524d4fbbb0b5242b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面可以看出，与请求报文相比，除了第一行（请求行VS状态行）以外，响应报文的其他结构与请求报文非常相似。其中，响应体是用于存放需要返回给客户端的数据信息的。

####状态行####
- 组成
![](http://upload-images.jianshu.io/upload_images/944365-834e3a1b316f265c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	> 其中，空格不能省

	**协议版本**

	HTTP协议版本主要是1.0、1.1、2.0

	**状态码**
	
	状态码分为5大类：

	<table>
		<tr>
			<th>
				类别
			</th>
		</tr>
		<tr>
			<th>
				含义
			</th>
		</tr>
		<tr>
			<th>
				1xx
			</th>
		</tr>
		<tr>
			<th>
				表示信息通知，如请求收到了或正在进行处理
			</th>
		</tr>
		<tr>
			<th>
				2xx
			</th>
		</tr>
		<tr>
			<th>
				表示成功，如接受或知道了
			</th>
		</tr>
		<tr>
			<th>
				3xx
			</th>
		</tr>
		<tr>
			<th>
				表示重定向，如要完成请求还必须采取进一步行动
			</th>
		</tr>
		<tr>
			<th>
				4xx
			</th>
		</tr>
		<tr>
			<th>
				客户的差错，如请求中有错误的语法或不能完成：404
			</th>
		</tr>
		<tr>
			<th>
				5xx
			</th>
		</tr>
		<tr>
			<th>
				表示服务器的差错，如服务器失效无法完成请求
			</th>
		</tr>
	</table>

	**状态信息**

	对状态码的简单解释
	> 具体详细的状态码信息可以看[http://tool.oschina.net/commons?type=5](http://tool.oschina.net/commons?type=5)

- 状态行举例
	- HTTP/1.1 202 Accepted(接受)
	- HTTP/1.1 301 Bad Request(永久性转移)
	- HTTP/1.1 404 Not Found(找不到)

####响应头####
- 作用：说明客户端、服务器或报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式

####响应体####
- 作用：用于存放需要返回给客户端的数据信息
- 使用方式：和请求体是一致的，同样分为：任意类型的数据交换格式、键值对形式和分部分形式，这里不作过多描述。