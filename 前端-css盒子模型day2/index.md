# 前端-CSS盒子模型（day2）


# 前端-CSS盒子模型（day2）

作用/意义： 决定不同类型元素之间的位置关系（平面位置、空间位置）

空间位置的 覆盖关系： 从下往上： 背景色<背景图片<元素内容



## 边框：border

边框颜色：border-color ;

border-color : red ;  代表上下左右四个边框 全是红色

border-color : red  yellow;  上下：red   左右:yellow

border-color : red  yellow blue;   上    "左右"   下

border-color : red  yellow blue green; 上 右  下 左    （顺时针）



border-top-color :blue ;

border-bottom-color :yellow;

border-left-color :red;

border-right-color :green;



边框粗细 border -width ; 

				border-width: medium ;
				
				border-width: thin  ;
				border-width: thick  ;
				border-width: 10px  ;


				border-width: thick thin 10px 100px;
				border-top-width: 100px ;


边框样式border-style ;

				border-style : solid double dotted dashed;   
				
				border-top-style : solid;
边框可以缩写：

border: solid  1px  red ;



缩写：

border - top :





## 外边距：margin

					/*margin-top:100px;
					margin-bottom:100px;
					margin:100px 0 100px 1 ;


​					margin:10px  0 ;
​					

					margin-top: 100px;

## 内边距：padding

					padding-top:30px;
					padding-bottom:10px;
					padding-right:0px;
					padding-left:130px;



### 盒子模型案例1：

```html
<html>

	<head>

		<meta charset="utf-8" /> 

		<title>CSS</title>
		
		<style type="text/css">
			li
			{
				list-style: none ;

			}
			
			body,ul,li,h1
			{
				margin:0px ;
				padding:0px ;
			}
			
			#games_list ul li a{
				text-decoration: none ;
				color: #000 ;
			}
			
			/*a:hover  表示鼠标悬浮的样式*/
			#games_list ul li a:hover{
				color:lightgray ;
			}
			

			#games_list
			{
				/*solid:实线
					gray:颜色 灰色
					1px：边框促销
				*/
				
				border:solid gray 1px;
				width:124px;
				/*内边距*/
				padding-bottom: 10px ;
				background-color: #c0d4ef  ;
				/*外边距*/
				margin:10px auto ;

			}
		
			.title{
				background:url(imgs/title_icon.png)  5px 8px   no-repeat	;
				padding-left : 20px ;
				color: gray ;
				line-height:30px;
				font-size: 15px ;
			}
			
			#games_list>ul
			{
				padding-left:10px ;
				padding-right:15px ;
			}
			
			#games_list>ul>li
			{
				background: url(imgs/right.png) right -5px no-repeat ;
				border-top:solid 1px blue ;
			}
		</style>	
	</head>
	<body>  
			<div id="games_list">
				<h1 class="title">游戏列表</h1>
				<ul>
						<li><a href="#">超级玛丽</a></li>
						<li><a href="#">贪吃蛇</a></li>
						<li><a href="#">红色警戒</a></li>
						<li><a href="#">魂斗罗</a></li>
						<li><a href="#">蜘蛛纸牌</a></li>
				</ul>
			</div>
	
		

	
	</body>
</html>


```

### 案例二：

```html
<html>

	<head>

		<meta charset="utf-8" /> 

		<title>盒子模型-商品列表</title>
		
		<style type="text/css">
			body,div,h1,dl,dt,dd{
				margin: 0px;
				padding: 0px ;
			}
			
			#production{
				background-color: #fffff2 ;
				width: 195px ;
			}
			
			#production>h1
			{
					font-size: 25px ;
					font-weight: bold ;
					line-height: 40px ;
					text-indent: 1em ;
			}
			
			#production>dl>dt{
				padding-left: 25px ;
				line-height: 30px ;
				font-size: 15px ;
			}
			
			#production>dl>dd{
				padding-left: 25px ;
				line-height: 25px ;
				font-size: 13px ;
				border-bottom: 1px dashed lightgray	 ;
			}
			
			#production .dianzi
			{
				background: url(imgs/dianzi.png) 4px 5px no-repeat
			
			}
			
			#production .fuzhuang
			{
				background: url(imgs/fuzhuang.png) 4px 5px no-repeat
				
			}
			
			
		</style>	
	</head>
	<body>  

	
	
		<div id="production">
			<h1>全部分类</h1>
			<dl>
				<dt class="dianzi">电子设备</dt>
				<dd>	
					手机/电脑/笔记本/鼠标<br/>
					手机/电脑/笔记本/鼠标<br/>
					手机/电脑/笔记本/鼠标<br/>
					手机/电脑/笔记本/鼠标<br/>
					手机/电脑/笔记本/鼠标<br/>
				</dd>
			</dl>

			<dl>
				<dt class="fuzhuang">服装</dt>
				<dd>	
					男装/女装/春装/秋装<br/>
					男装/女装/春装/秋装<br/>
					男装/女装/春装/秋装<br/>
					男装/女装/春装/秋装<br/>
					男装/女装/春装/秋装<br/>
				</dd>
			</dl>
		</div>
	</body>
</html>


```

### 案例：图片列表

```html
<html>

	<head>

		<meta charset="utf-8" /> 

		<title>盒子模型-商品列表</title>
		
		<style type="text/css">
			div,h1,ul,li,a
			{
				/*为了防止各个浏览器的内边距、外边距不兼容情况，一般在开发时 第一步将边距全设置为0  */
				margin: 0px ; 
				padding:0px ;
			}
			
			ul,li
			{
				list-style-type: none ;
			}
			
			#production
			{
				/* background-color: yellow ; */
				width: 318px ;
			}
			
			#production>h1{
				margin-left: 5px ;
				background-color: #24221e ;
				font-family: SimHei, Georgia, 'New York', Times, TimesNR, 'New Century Schoolbook',
     serif;
				font-weight: 700;
				font-size: 24px;
				color: #FFF ;
				line-height: 32px;
				padding-left: 18px;
			}
			
			ul li a img
			{
				border: 5px solid #FFF;
			}
			
			ul li a:hover img
			{
				border: 5px solid rgba(255,144,0, 0.3);
			}
			
			
		</style>	
	</head>
	<body>  
	
		<div id="production">
			<h1>1F 电子设备</h1>
			<ul>
				<li> 
					<a href="#"> 
							<img src="imgs/1.png" title="xxx" alt="替换文字"> </img>   
					</a>  
				</li>
				
				<li> 
					<a href="#"> 
							<img src="imgs/2.png" title="xxx" alt="替换文字"> </img>   
					</a>  
				</li>
				
				<li> 
					<a href="#"> 
							<img src="imgs/3.png" title="xxx" alt="替换文字"> </img>   
					</a>  
				</li>
				

				
			</ul>
			
			
		</div>
	</body>
</html>


```



