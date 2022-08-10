# 前端-js-DOM总结


# 前端-js-DOM总结

## 1.获取元素节点对象

- 方法：通过document对象调用

  - getElementById() 通过id获取一个元素节点对象
  - getElementsByTagName() 通过标签名获取一组元素节点对象
  - getElementsByName()通过name属性获取一组元素节点对象

- 属性：

  - innerHTML 属性获得元素内部的HTML代码
  - innerTest 忽略html标签
  - 元素对象.属性名 可以读取到元素的属性值 读取class属性时 需要使用className
  - children 表示当前节点的所有子元素
  - childNodes 表示当前节点的所有子节点(包括文本节点)
  - firstElementChild 表示当前节点的第一个子元素
  - firstChild 表示当前节点的第一个子节点
  - lastChild 表示当前节点的最后子节点
  - parentNode 表示当前节点的父节点
  - previousElementSibling 表示当前节点的前一个兄弟元素
  - previousSibling 表示当前节点的前一个兄弟节点
  - nextElementSibling 表示当前节点的前一个兄弟元素
  - nextSibling 表示当前节点的后一个兄弟节点

- 其他

  1. 获取body标签

     ```javascript
     var body = document.body;
     ```

  2. 获取HTML根标签

     ```javascript
     var html = document.documentElement;
     ```

  3. all 页面中所有元素

     ```javascript
     var all = document.all;
     ```

  4. 根据元素的class属性查询一组元素的节点对象

     ```javascript
     var box1 = document.getElementsByClassName("box1");
     ```

  5. 根据一个CSS选择器的字符串作为参数，查询一个元素节点对象

     只会返回第一个

     ```javascript
     var box1 = document.querySelector(".box1 div");
     ```

  6. 根据一个CSS选择器的字符串作为参数，查询所有元素节点对象 数组

     ```javascript
     var box1 = document.querySelectorAll(".box1 div");
     ```


## 2.修改元素节点对象

1. 创建一个标签

   ```javascript
   var li = document.createElement("li");
   ```

2. 创建文本

   ```javascript
   var text = document.createTextNode("文本");
   ```

3. 将文本加入到标签中 把一个节点加入到另一个节点当中

   ```javascript
   li.appendChild(text);
   ```

4. 在指定节点前加入新的子节点

   > insertBefore()
   >
   > 语法:父节点.insertBefore(新节点，旧节点)；

   ```javascript
   city.insertBefore(li,bj);
   ```

5. 替换节点

   > replaceChild()
   >
   > 使用指定子节点替换原有节点
   >
   > 语法:父节点.replaceChild(新节点，旧节点)；

   ```
   city.replaceChild(li,bj);
   ```

6. 删除节点

   > removeChild()
   >
   > 删除字节点
   >
   > 语法:父节点.removeChild(节点)；
   >
   > 子节点.parentNode.removeChild(子节点);

   ```
   city.removeChild(bj);
   ```


## 3.DOM操作CSS

1. 通过修改 元素.style.CSS属性名 = “属性值” 来修改元素的内联样式

2. 读取当前样式 

   ​		var obj = getComputedStyle(元素对象名,null);

   ​		obj.属性名 读取当前样式

   ​		IE8：alert(box1.currentStyle.width);读取当前样式

   解决兼容性问题：

   ​		

   ```javascript
   // obj元素对象 name属性名
   function getStyle(obj,name){
   					if(window.getComputedStyle){
   						return getComputedStyle(obj,null)[name];
   					}else{
   						return obj.currentStyle[name];
   					}
   				}
   ```


### 其他属性

```javascript
	/*
					获取可见高度和宽度 无法进行修改
					clientWidth
					clientHeight
					不带px 返回数字 可以直接进行计算
					包括内容区和内边距 不包括外边距	
				*/
				alert(box1.clientWidth);
				
				/*
					获取可见高度和宽度 无法进行修改
					offsetWidth
					offsetHeight
					不带px 返回数字 可以直接进行计算
					包括内容区和内边距和外边距	
				*/
				alert(box1.offsetWidth);
				
				/*
					offsetParent	
					定位父元素
					会获取到离当前元素最近的开启了定位的祖先元素（postion 不是默认值）
				*/
			   alert(box1.offsetParent);
				
				/*
					offsetLeft
					- 当前元素相对于其定位父元素的水平偏移量
					offsetTop
					- 当前元素相对于其定位父元素的垂直偏移量
				*/
				alert(box1.offsetLeft);
				
				/*
					scrollHeight
					scrollWidth
					- 获取元素在整个滚动区域的高度和宽度
					scrollLeft
					- 获取水平滚动条 滚动的距离
					scrollTop
					- 获取垂直滚动条 滚动的距离
				*/
			   disabled 可以设置一个元素是否禁用 true 禁用 false 不禁用
```

## 4.事件

### 事件的冒泡

> 事件默认会冒泡（一层一层往上传 默认开启）
>
> 关闭冒泡：
>
> event = event || window.event;//兼容性问题
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

### 事件的委派

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

### 事件绑定

>方式一 
>
>只能绑定一个 this是绑定事件对象
>	but0.onclick = function(){
>	alert("1");
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

### 鼠标事件 

> event 传入参数
>
> event = event || window.event 解决兼容性问题
>
> 获取鼠标位置
>
> clientX  clientY 相对于绑定元素
> pageX   pageY   相对于整个页面

- onclick  点击
- onmousemove 鼠标移动
- onmousedown 鼠标点住
- onmouseup 鼠标松开
- onmousemove 鼠标移动



### 滚轮事件

onmousewhell

(火狐不支持，火狐中使用DOMMouseScroll并且需要使用addEventListener)

向上 event.wheelDelta>0 可以获取滚轮方向(火狐不支持 使用event.detail<0)

```javascript
if(event.wheelDelta>0 || event.detail<0){
	alert("上");
}else{
	alert("下");
}
```

addEventListener 取消默认行为 event.preventDefault();

默认 return false;



### 键盘事件

一般绑定给可以聚焦的对象或者document

onkeydown

onkeyup



属性 event.

- keyCode 获取按键的编码
- altKey 判断alt是否被按下
- ctrlKey 判断ctrl是否被按下
- shiftKey 判断shift是否被按下

在文本框中输入 如果 加入 return false; 则无法输入

```javascript
//输入框中无法输入数字
input.onkeydown = function(event){
	event = event || window.event;
	if(event.keyCode>=48 && event.keyCode<=57){
		return false;
	}
}
```




