# 前端-CSS标签与样式（day3）


# 前端-CSS标签与样式（day3）

## 标签

div：块级元素，换行

span： 内联元素，不换行



## CSS样式（有些样式存在浏览器兼容问题）

font-size:字体大小  px   em rem  cm  mm pt pc 

font-style:normal(默认)，  italic/oblique

font-weight: bold  bolder  normal  lighter  100-900

text-indent:缩进  2em

line-height: 100px

text-decoration:  underline/overline/text-though

color:英语单词red    十六进制（3位或6位）  rgbcolor : rgb(11,192,15)     rgba(1,27,182,0.5)  ;

其中a代表透明度  0-1之间的小数，0.3代表显示程度是30%

阴影text-shadow: 横坐标 纵坐标 发散程度

a:hover 悬浮

a:link 未访问

a:visited 已经访问过了

a:active 点击鼠标时

设置的顺序： link -> visitied > hover > active

一般建议： 只写 hover 即可



鼠标样式：cursor:pointer

https://www.w3school.com.cn/cssref/pr_class_cursor.asp



list-style-type: disc/none/square/decimal/circle



background-image:url(图片地址)

background:blue url(imgs/fuzhuang.png)  right  top no-repeat ;

repeat-x 

repeat-y

				background:url(imgs/bg.png)  no-repeat ;
				background-size:cover ;
	cover:铺满整个区域
	background-size: 50%  60%  ;
	表示 宽占整个网页的50%  ，高占整个网页的60%
	auto:默认，原图大小



边框的圆角：圆角边框

border-raduis:

```html
<html>

	<head>

		<meta charset="utf-8" /> 
		<title>圆角边框</title>
		<style type="text/css">
			div{
				/*box-shadow:横坐标 纵坐标 范围 颜色*/
				box-shadow:  10px 20px 10px gray ;
			
				width: 100px  ;
				height: 50px  ;
				border: 3px solid red ;
				/* border-radius: 10px 50px 80px 100px ;
				border-radius: 50px  ; 
				border-radius: 50px 100px  ;
				border-radius: 50px 100px  10px ;*/
				margin: 50px 0 ;
			}
			
			.divclass2
			{
				width: 50px  ;
				height: 100px  ;
				border: 3px solid red ;
			
			}
			
			
			
			div:nth-of-type(2){
				border-radius: 50px 50px 0 0 ;
			}
			
						
			div:nth-of-type(3){
				border-radius:  0 0 50px 50px ;
			}
			
			div:nth-of-type(4){
				border-radius:  0 50px 50px 0 ;
			}
			
			div:nth-of-type(5){
				border-radius:  50px 0 0 50px ;
			}
			
			#sx div
			{
				height: 50px ;
				width: 50px ;
				border: 3px blue solid ;
			}
			#sx div:nth-of-type(1){
				border-radius:50px 0 0 0 ;
			}
						
			#sx div:nth-of-type(2){
				border-radius: 0 50px 0 0 ;
			}
						
			#sx div:nth-of-type(3){
				border-radius: 0 0 50px  0 ;
			}
						
			#sx div:nth-of-type(4){
					border-radius: 0 0 0 50px  ;
			}
			
		</style>	
	</head>
	<body>  
			<div></div>
			<div></div>
			<div ></div>
			<div class="divclass2"></div>
			<div class="divclass2"></div>
			
			<div id="sx">
					<div></div>
					<div></div>
					<div></div>
					<div></div>
			
			</div>

	</body>
</html>


```

