# 前端-js-事件


# 前端-js-事件

监听鼠标移动

> onmousemove 
>
> event 传入参数
>
> event = event || window.event 解决兼容性问题
>
> clientX  clientY 相对于绑定元素
> pageX   pageY   相对于整个页面

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script>
			window.onload = function(){
				var box1 = document.getElementById("box1");
				var box2 = document.getElementById("box2");
				box1.onmousemove = function(event){
					event = event || window.event;
					var x = event.clientX;
					var y = event.clientY;
					box2.innerHTML = "x="+x+",y="+y;
				}
			}
		</script>
	</head>
	<body>
		<div style="width: 500px; height: 200px; border: black 1px solid;" id="box1">
			
		</div>
		<div style="width: 500px; height: 200px; border: black 1px solid;" id="box2">
			
		</div>
	</body>
</html>

```

## 事件的冒泡

> 事件默认会冒泡（一层一层往上传 默认开启）
>
> 关闭冒泡：
>
>  event = event || window.event;//兼容性问题
> 					event.cancelBubble = true;

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script type="text/javascript">
			window.onload = function(){
				var box1 = document.getElementById("box1");
				var box2 = document.getElementById("box2");
				var x = document.getElementById("x");
				x.onclick = function(event){
                    event = event || window.event;
					event.cancelBubble = true;
					alert("x");
				}
				box1.onclick = function(event){
					alert("box1");
				}
				box2.onclick = function(event){
					alert("box2");
				}
			}
		</script>
	</head>
	<body>
		<div id="box2" style="width: 500px;height: 500px; border: #7FFFD4 1px  solid;">
			<div id="box1" style="width: 50px;height: 50px; border: #7FFFD4 1px  solid;">
				<h2 id="x">rrr</h2>
			</div>
		</div>
	</body>
</html>

```

## 事件的委派

> 对冒泡原理的应用
>
> 解决了大量事件的问题

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script>
			window.onload= function(){
				var ul = document.getElementById("ul");
				ul.onclick = function(event){
					if(event.target.className=="link"){
						alert("a");
					}
				}
			}
		</script>
	</head>
	<body>
		<ul id="ul">
			<li><a href="javascript:;" class="link">aaaa</a></li>
			<li><a href="javascript:;" class="link">aaaa</a></li>
			<li><a href="javascript:;" class="link">aaaa</a></li>
		</ul>
	</body>
</html>

```

## 事件绑定

>方式一 
>
>只能绑定一个 this是绑定事件对象
> 	but0.onclick = function(){
> 	alert("1");
>
>方式二：可以绑定多个 除IE8   this是绑定事件对象
>addEventListener
>	参数：
>		1.事件字符串 不要on
>		2.回调函数
>		3.是否在捕获阶段触发事件，需要布尔值，一般传false（子->父）
>
>attachEvent 兼容ie8 不兼容火狐  this是window
>	参数
>		1.事件字符串 要on
>		2.回调函数

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script type="text/javascript">
			window.onload = function(){
				var but0 = document.getElementById("but0");
				// 方式一 只能绑定一个
				// but0.onclick = function(){
				// 	alert("1");
				// }
				/*
					方式二：可以绑定多个 除IE8
					addEventListener
					参数：
					1.事件字符串 不要on
					2.回调函数
					3.是否在捕获阶段触发事件，需要布尔值，一般传false
				*/
			   but0.addEventListener("click",function(){
				   alert("2");
			   },false)
			   but0.addEventListener("click",function(){
			   				   alert("2");
			   },false)
			   but0.addEventListener("click",function(){
			   				   alert("2");
			   },false)
			   /*
					attachEvent 兼容ie8 不兼容火狐
					参数
					1.事件字符串 要on
					2.回调函数
			   */
			  bnt0.attachEvent("onclick",function(){
				  alert("ie8")
			  })
			}
		</script>
	</head>
	<body>
		<button id="but0">按钮</button>
	</body>
</html>

```

bind() 解决兼容问题

```javascript
function bind(obj,eventStr,callback){
				  if(obj.addEventListener){
					  obj.addEventListener(eventStr,callback,false);
				  }else{
					  obj.attachEvent("on"+eventStr,function(){
						 // 在匿名函数中调回调函数
						  callback();
					  })
				  }
			  }
```

## 鼠标事件

1. onclick  点击
2. onmousemove 鼠标移动
3. onmousedown 鼠标点住
4. onmouseup 鼠标松开

