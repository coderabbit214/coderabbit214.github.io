# 前端-js-使用DOM操作CSS


# 前端-js-使用DOM操作CSS

## 内联样式

> 通过 元素.style.样式属性名 读取 修改
>
> 注意 ‘ - ’ 变为驼峰式命名

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<style>
			.box{
				width: 200px;
				height: 200px;
				background-color: aquamarine;
			}
		</style>
		<script>
			window.onload= function(){
				var box1 = document.getElementById("box1");
				var btn1 = document.getElementById("btn1");
				btn1.onclick = function(){
					box1.style.width = "300px";
					box1.style.height = "300px";
					box1.style.backgroundColor = "blue";
					alert(box1.style.backgroundColor);
				};
			};
		</script>
	</head>
	<body>
		<div class="box" id="box1"> 
			
		</div>
		<button id="btn1" >2222</button>
	</body>
</html>

```

## 读取当前样式

> 语法
>
> var obj = getComputedStyle(元素对象名,null);
>
> alert(obj.width);‘
>
> 该方法会返回一个对象 对象中封装了当前元素的样式
>
> 如果没有定义 会得到默认值

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<style>
			.box{
				width: 200px;
				height: 200px;
				background-color: aquamarine;
			}
		</style>
		<script>
			window.onload= function(){
				var box1 = document.getElementById("box1");
				var btn1 = document.getElementById("btn1");
				btn1.onclick = function(){
					//alert(box1.currentStyle.width); 
				 var obj = getComputedStyle(box1,null);
				  alert(obj.width);
				};
			};
		</script>
	</head>
	<body>
		<div class="box" id="box1"> 
			
		</div>
		<button id="btn1" >2222</button>
	</body>
</html>
```

## getStyle（自定义）解决兼容问题

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

## 其他

。。。。

