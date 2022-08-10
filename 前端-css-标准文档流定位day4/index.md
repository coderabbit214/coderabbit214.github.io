# 前端-CSS-标准文档流，定位(day4)


# 前端-CSS-标准文档流，定位(day4)

## 标准文档流

## 组成

- 块级元素（block level）：自带换行 div ，可以设置宽和高

- 内联元素 (inline level) ：不换行  span, a  ，没有宽和高   （内敛元素不遵守盒子模型）

- inline-block：1.如果inline失效，可以尝试inline-block

  ​						 2.可以设置宽和高

  块级元素可以包含内敛，反之不行。

  正确：

  ```html
  			<div>
  				begin..
  				<span>span</span>
  				<span>span</span>
  				end...
  			
  			</div>
  ```

  错误：

  ```html
  			<span>
  				begin..
  				<div>divdiv</div>
  				<div>divdiv</div>
  				end...
  			</span>
  ```

  内敛元素和块级元素相互切换：

  display:  block |  inline   |none |inline-block ;

  

```html
<html>

	<head>

		<meta charset="utf-8" /> 

		<title>网站导航</title>
		
		<style type="text/css">
			*{
				margin: 0px ;
				padding: 0px ;
			}
			
			a{
				text-decoration: none ;
			}


			.nav-header
			{
				width: 100% ;
				height:85px ;
				background: rgba(0,0,0,0.7);
			}
			
			.head-contain
			{
				width: 700px ;
				height: 85px ;
				margin:0 auto ;
				text-align:center ;
			}
			
			.top-logo,.top-nav,.top-nav li,.top-right
			{
				display: inline-block ;
				vertical-align:top ;
				margin-top: 15px ;
			}
			
			.top-nav li
			{
				width: 90px ;
			}
			
			.top-nav li a
			{
				font-size: 17px ;
				color: #fff ;
				
			}
			
			.top-nav li a:hover
			{

				color: blue ;			
			}
			
			.top-right a
			{
				display:inline-block ;/*将内敛元素 转为 块级元素，使之遵循盒子模型*/
				font-size: 17px ;
				margin-top: 10px ;
				border-radius:30px ;
			}
			
			.top-right a:first-of-type
			{
				width:75px ;
				height: 35px ;
				border:1px orange solid ;
				line-height:35px ;
			}
			
			.top-right a:first-of-type:hover
			{
				color:red ;
				background: orange;
			}
			
		</style>	

	</head>
	<body> 
		<header class="nav-header">
			<div class="head-contain">
				<a href="#" class="top-logo"><img src="imgs/fd1.png" height="50px" width="50px"/></a>
				<nav class="top-nav">
					<ul>
						<li><a href="#">旅游天地</a></li>
						<li><a href="#">美食城</a></li>
						<li><a href="#">成长体系</a></li>
						<li><a href="#">当地特色</a></li>
						<li><a href="#">重点推荐</a></li>

					</ul>
				</nav>
				<div class="top-right">
					<a href="#">登录</a>
					<a href="#">开通会员</a>
				</div>
			</div>
		</header>
	</body>
</html>


```

## 浮动（float）

块级元素 -> 内敛元素 ：将不同行的元素放置到一行

浮动 ：将不同行的元素放置到一行

float: none | left | right



浮动（压缩）：压缩的是**自己**的空间

clear: 清除浮动（清除一半，还原了 块级元素的 换行特性，但没有还原空间）

clear:清除的是上一个元素（不是自己）

clear:left | right |both

```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>浮动</title>
	<style  type="text/css" >

		
		#father div
		{	
			border: 2px solid red ; 
			margin: 10px 0 ;
			background-color:yellow ; 
			
		}
		
		#father .mydiv1{
			float:left ; 
		}
		
		#father .mydiv2{
			clear: left ;
		}
		
		#father .mydiv3{
		
		}
		
		#father .mydiv3{
			
		}
	</style>
	</head>
	<body>
	  <div id="father">
	  
			
	  
		    <div class="mydiv1"><img src="imgs/fd1.png"  /></div>
			
		    <div class="mydiv2"><img src="imgs/fd2.png"  /></div>
		  
		    <div class="mydiv3"><img src="imgs/fd3.png"  /></div>
			<div class="mydiv4">
			这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...
			这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...这是浮动的演示案例...
			</div>
	  </div>
	</body>
</html>

```



浮动的元素：不在标准文档流之中 （网页无法识别原有的空间），脱离原有空间

```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>商品列表</title>
	<style  type="text/css" >
		*{
			margin: 0 ;
			padding: 0 ;
		}
		
		.shangpin ul  li
		{
			list-style: none ;
		}
		
		.shangpin{
			width: 720px ;
			/* background-color: yellow ; 	*/
			margin:10px auto ;
		
		}
		
		.shangpin h1
		{
			font-size:25px ; 
		}
		.shangpin h1 span
		{
			float:right ;
			margin-right:20px ;
			color: gray ; 
			font-size: 18px ; 
		}
		.shangpin ul li
		{
		
			float:left ;
			margin:20px ;
		}
		
		.shangpin ul li p
		{
			font-size: 15px;
			color: blue ;
		}
		
	
	</style>
	</head>
	<body>
	

		<div class="shangpin">
			<h1>商品列表<span>更多</span></h1>
			<ul>
				<li>
					<img src ="imgs/1.png" />
					<p>推荐商品 | 1024节爆款电脑</p>
				</li>		
				
				<li>
					<img src ="imgs/2.png" />
					<p>推荐商品 | 程序员节鼠标</p>
				</li>		
				
				<li>
					<img src ="imgs/1.png" />
					<p>推荐商品 | 程序猿节背包</p>
				</li>	
				
				<li>
					<img src ="imgs/1.png" />
					<p>推荐商品 | 程序节员笔记本</p>
				</li>

			</ul>
		</div>
	</body>
</html>

```



display:   block  | inline   |none | inline-block;

float: left | right ;



## 溢出overflow

overflow: visible  | hidden  |scroll |auto;

scroll: 不论是否超出，都有滚动条

auto:如果超出，有滚动条；否则没有

```html

<html >
	<head>
	<meta charset=utf-8 />
	<title>overflow</title>
	<style  type="text/css" >

	body {

	}

	#content {
		width:400px;
		height:350px;
		border:1px red solid;
		overflow:auto ;
	}


	</style>
	</head>
	
	<body>

		
	
		<div id="content">
		  <img src="imgs/fd3.png"  />
		  <p >111111这是一些文是一些文是一些文是一
		  
		  
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  些文是一些文是一些文是一
		  
		  
		  些文是一些文是一些文是</p>
		  
		</div>
		
	</body>
</html>

```





## 定位

position: static(默认值，没有定位) | relative(相对定位) | absolute(绝对定位) |fixed（固定定位，个别版本浏览器不支持）



relative：相对于自身

注意位置：left是指往右偏移；top是下；其他同理



```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>浮动</title>
	<style  type="text/css" >
		div
		{
			border: 1px solid red ;
			margin:20px ;
		}
		
		#father div
		{
			height:150px ;
			background:lightgray ;
		}
		
		.mydiv1
		{
			position: relative ;
			right:-50px ;
			bottom:-50px ;
		
		}
		
	</style>
	</head>
	<body>
	
	
	
	  <div id="father">
	
	  
		    <div class="mydiv1">第一个div</div>
			
		    <div class="mydiv2">第一个di2</div>
		  
		    <div class="mydiv3">第一个div3</div>

	  </div>
	</body>
</html>

```

```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>浮动</title>
	<style  type="text/css" >
		#father{
			border: 1px solid blue ;
		}
		#father div
		{
			background-color: yellow  ;

		}
		
		#father
		{
			width: 300px ;
			height:300px ;
			margin:50px auto ;
		}
		
		#father div
		{
			width: 100px ;
			height:100px ;
		}
		
		.mydiv2,.mydiv4{
			position:relative ;
			left:200px ;
			top :-100px ;
		}

		.mydiv5
		{
			position: relative ;
			top: -300px ;
			right:-100px ;
		}

		
	</style>
	</head>
	<body>
	
	
	
	  <div id="father">
	
	  
		    <div class="mydiv1">第一个div1</div>
			
		    <div class="mydiv2">第一个di2</div>
		  
		    <div class="mydiv3">第一个div3</div>
			
		    <div class="mydiv4">第一个div4</div>
			
		    <div class="mydiv5">第一个div5</div>

	  </div>
	</body>
</html>

```



绝对定位：

position: absolute

是否存在 “已经定位的祖先元素”

- 存在 ：则参照 “已经定位的祖先元素”定位
- 不存在： 则参照浏览器定位

绝对定位：会脱离文档流（释放空间，会让网页无法识别）



```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>绝对定位（没有已经定位的祖先元素）</title>
	<style  type="text/css" >
		
		div
		{
			width:100px ;
			height:100px ;
			background: yellow ;
			
			position:absolute ;
			right :30px ;
			bottom: 30px ;
			 
		}
		
	</style>
	</head>
	<body>
	
	
	
	  <div >

		
	  </div>
	  
	  
	</body>
</html>

```



```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>绝对定位（有已经定位的祖先元素）</title>
	<style  type="text/css" >
		
		#father
		{
			width:600px ;
			height:600px ;
			border:1px solid blue ; 
			position: relative;
			top :30px ;
			left: 30px ;
	 
		}
		
		#father div
		{
			height:100px ;
			width: 200px ;

		}
		.mydiv1
		{
			background-color: red ;
		}
		
		.mydiv2
		{
			background-color: yellow ;
			position: absolute ;
			right: 100px ;
		}
		
		.mydiv3
		{
			background-color: blue ;
		}
		
	</style>
	</head>
	<body>
	
	
	
	  <div id="father">
			<div class="mydiv1">red</div>
			<div class="mydiv2">yellow</div>
			<div class="mydiv3">blue</div>
		
	  </div>
	  
	  
	</body>
</html>

```



position:fixed   参照浏览器定位

```html
<html>
	<head>
	<meta charset=utf-8/>
	<title>绝对定位（有已经定位的祖先元素）</title>
	<style  type="text/css" >
		

		
	
		.mydiv1
		{
			background-color: red ;
			height:200px ;
			width:200px ;
			
			position:fixed ;
			right:10px ;
			top:100px ;
		}
		
		.mydiv2
		{
			background-color: yellow ;
			height:80px ;
			width:100% ;
			position:fixed ;
			bottom:0 ;

		}
		

		
	</style>
	</head>
	<body>
	
	
	

			<div class="mydiv1">red</div>
			<a href="https://www.baidu.com"><div class="mydiv2">yellow</div></a>
		
		
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
		<p>xxxxxxxxxx</p>
	  
	  
	</body>
</html>

```



定位：

相对：relative  ，相对于自身定位

绝对：absolute，（如果有已定位的祖先：参照祖先 ；如果没有，参照浏览器）

fixed: 参照浏览器

## 透明度

- opacity:0.5

- filter:alpha(opacity=50)

> 浏览器兼容性问题，部分浏览器支持opacity，部分支持filter:alpha(opacity=50) .



