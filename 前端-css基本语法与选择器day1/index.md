# 前端-CSS基本语法与选择器（day1）


#  前端-CSS基本语法与选择器（day1）

html： 内容  (文本框)

css:   样式 （给文本框设置 长度、宽度、颜色）

css优势：

- 内容与样式相分离
- css样式更加丰富
- 提高浏览器的加载速度、节约网络带宽，减少代码量
- 利于SEO优化

## 基本语法

选择器

{

​	样式名：样式值 ;

​	样式名：样式值 ;

​	样式名：样式值 ;

​	...

}

颜色：支持英文单词、#6个或3个十六进制

```html
<html>

	<head>

		<meta charset="utf-8" /> 

		<title>CSS</title>
		
		<style type="text/css">
				h1
				{
					color: red;  /*  #154245  */
					font-size : 20px ;
				}
		</style>
			
	</head>


	<body>  
			<h1>h1标签</h1>
			<h1>h1标签</h1>
			<h2>h2标签</h2>
			<h3>h3标签</h3>
			<h1>h1标签</h1>
			
	</body>
</html>


```

选择器：

- 标签选择器：直接编写标签名 h1  , p
- 类选择器：  .class值  ，可以用于多个元素
- id选择器：  #id值，只能用于一个元素

## 引入CSS样式的方式

- 行内样式
- 内部样式      head  style
- 外部样式

建议：在开发时 用内部样式；编写完毕后 改造成外部 ；

行内：不推荐， 赶时间、应急

行内：

```html
<h4 style="color:red;background-color:green;">  h4样式</h4>
```

内部

```html
	<head>
		<title>CSS</title>
		
		<style type="text/css">
				h1
				{
					color: red;  /*  #154245  */
					font-size : 20px ;
				}
				
		</style>
			
	</head>
```

外部：

先将样式独立保存在一个.css文件中，然后再在html文件中引入该.css文件

```html
<head>
		<!--外部样式：推荐做法link -->
		<link href="文件名.css" type="text/css" rel="stylesheet"/>
			
</head>
```

了解（不推荐）：

```htm
		
		<style type="text/css">
				<!-- 外部样式-->
				@import url("mycss.css") ;
		</style>
```



外部导入的两种方式（链接式,link； 导入式,@import）区别 ：

- 推荐link
- link属于xhtml规范； @import属于css2.1标准
- link将css预先加载到网页中，再进行编译显示
- @import：先显示网页结构，然后再加载CSS内容
- @import是css2.1独有的

> 提示： 注释：html <!-- -->，在css中注释 /*  */

CSS优先级问题： 

行内优先级 >外部 >内部

id选择器>类选择器 >标签选择器



## 复合选择器

提前：样式会继承，例如选择 ul，会给ul下面的li 起作用

## CSS2.1

后代选择器： 空格

交集选择器：连续书写 （没有继承性），要防止歧义

并集选择器：逗号 ,  



## CSS3 （类似jquery）

- 层次选择器

  - 后代选择器 ： 空格
  - 子选择器： >
  - 相邻同辈选择器（只对 后面的元素有效）: +
  - 通用同辈选择器（只对 后面的元素有效）: ~

- 结构伪类选择器

  ```html
  <html>
  
  	<head>
  
  		<meta charset="utf-8" /> 
  
  		<title>CSS</title>
  		
  		<style type="text/css">
  			/* 层次
  			ul>li
  			{
  				color : red ;
  			}
  			h4+h1
  			{
  				color:blue ;
  			}
  			h4~h1
  			{
  				color:blue ;
  			}
  			*/
  			/* 伪类选择器 */
              /* 每一个ul ol中 */
  			li:first-child
  			{
  				color :yellow ;
  			}
  			
  			li:last-child
  			{
  				color :red ;
  			}
  			
  			li:nth-child(3)
  			{
  				color: green ; 
  			}
  			/*全局*/
  			p:first-of-type{
  				background-color : purple ;
  			
  			}
  			
  			p:last-of-type{
  				background-color : pink ;
  			
  			}
  			
  			p:nth-of-type(2){
  				background-color : red ;
  			
  			}
  		</style>	
  	</head>
  	<body>  
  			<p>p1p1</p>
  			<p>p2p2</p>
  			<p>p3p3</p>
  			<p>p4p4</p>
  	
  	
  			<ul>
  				<li>aaa</li>
  				<li>bbb</li>
  				<li>ccc</li>
  				<li>ddd</li>
  			</ul>
  	
  			<h1>1111</h1>
  			<h1>1111</h1>
  			<h4>h4h4h4</h4>
  			<h1>AAAA111</h1>
  			<h1>AAAA222</h1>
  			<h1>1111</h1>
  			
  			
  			<h4>h4h4h4</h4>
  			<h1>BBBB11</h1>
  			<h1>BBBB22</h1>
  
  
  	
  	</body>
  </html>
  
  
  ```

  

- 属性选择器[]

  ```html
  <html>
  
  	<head>
  
  		<meta charset="utf-8" /> 
  
  		<title>CSS</title>
  		
  		<style type="text/css">
  			/* 层次
  			ul>li
  			{
  				color : red ;
  			}
  			h4+h1
  			{
  				color:blue ;
  			}
  			h4~h1
  			{
  				color:blue ;
  			}
  			*/
  			/* 伪类选择器 */
  			li:first-child
  			{
  				color :yellow ;
  			}
  			
  			li:last-child
  			{
  				color :red ;
  			}
  			
  			li:nth-child(3)
  			{
  				color: green ; 
  			}
  			
  			p:first-of-type{
  				background-color : purple ;
  			
  			}
  			
  			p:last-of-type{
  				background-color : pink ;
  			
  			}
  			
  			p:nth-of-type(2){
  				background-color : red ;
  			
  			}
  			/*选择有name属性，且标签是input的 
  			input[name]{
  				background-color: yellow ;
  			}*/
  			/*input，有name，且name的值是username
  			input[name="username"]{
  				background-color: yellow ;
  			}
  			*/
  			
  			/*name是以user开头的元素
  			input[name^=user]
  			{
  				background-color:green ;
  			}*/
  			/*以name结尾
  			input[name$=name]
  			{
  				background-color: purple;
  			}
  			*/
  			/*包含i的*/
  			input[name*=i]{
  				background-color: pink ; 
  			}
  		</style>	
  	</head>
  	<body>  
  	
  			<form>
  				id:<input type="text" name="uid" /><br/>
  				用户名:<input type="text" name="usernickname" /><br/>
  				真实名字:<input type="text" name="username" /><br/>
  				密码：<input type="password" name="pwd"/><br/>			
  			</form>
  	
  			<p>p1p1</p>
  			<p>p2p2</p>
  			<p>p3p3</p>
  			<p>p4p4</p>
  	
  	
  			<ul>
  				<li>aaa</li>
  				<li>bbb</li>
  				<li>ccc</li>
  				<li>ddd</li>
  			</ul>
  	
  			<h1>1111</h1>
  			<h1>1111</h1>
  			<h4>h4h4h4</h4>
  			<h1>AAAA111</h1>
  			<h1>AAAA222</h1>
  			<h1>1111</h1>
  			
  			
  			<h4>h4h4h4</h4>
  			<h1>BBBB11</h1>
  			<h1>BBBB22</h1>
  
  	</body>
  </html>
  
  
  ```
  
  

