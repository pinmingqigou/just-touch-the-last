#简介
这门语言可用于 HTML 和 web，更可广泛用于服务器、PC、笔记本电脑、平板电脑和智能手机等设备

#概述
- JavaScript 是脚本语言
	- JavaScript 是一种轻量级的编程语言
	- JavaScript 是可插入 HTML 页面的编程代码
	- JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行
	- JavaScript 很容易学习

#基础语法
- 变量：使用var关键词来声明变量（**var声明的是局部变量，如果把值赋于未声明的变量，则该变量自动作为全局变量声明**）,当只是声明了变量但是未赋值时，其实默认赋值为undefined;如果已经定义了变量的值，再重新声明变量，该变量的值不会丢失；js的变量均为对象，当声明一个变量时，其实就创建了一个新对象
- 数据类型
	- js拥有动态类型：意味着相同的变量可用作不同的类型
	
	    	var x// x 为 undefined
	    	var x = 6;   // x 为数字
	    	var x = "Bill";  // x 为字符串
	
	- 字符串
	- 数字
	- 布尔
	- 数组
	- 对象：对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔

			var person={firstname:"Bill", lastname:"Gates", id:5566};
	
		**对象有两种寻址方式：**
			
			name=person.lastname;
			name=person["lastname"];

- 对象：js中的所有事物都是对象：字符串、数字、数据、日期等等。
	- 属性：对象的相关描述
	- 方法：对象执行的动作

- 函数：是由事件驱动的或者当它被调用时执行的可重复使用的代码块
	- 语法：函数就是包裹在花括号中的代码块，前面使用了关键词function

			function functionname(var1，var2...){
				这里是要执行的代码
			}