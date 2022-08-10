# 前端-DOM查询


# 前端-DOM查询

window.onload(在整个页面加载之后执行)

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<script type="text/javascript">
			window.onload = function(){
				var btn = document.getElementById("btn");
				btn.onclick = function(){
					alert("ddd");
				};
			}
		</script>
	</head>
	<body>
		<button id="btn">sss</button>		
	</body>
</html>

```

## 获取元素节点对象

> 通过document对象调用
>
> 可以通过 innerHTML 属性获得元素内部的HTML代码
>
> ​	innerTest 忽略html标签
>
> 元素对象.属性名 可以读取到元素的属性值
>
> ​	读取class属性时 需要使用className

- getElementById() 通过id获取一个元素节点对象
- getElementsByTagName() 通过标签名获取一组元素节点对象
- getElementsByName()通过name属性获取一组元素节点对象

## 获取元素节点的子节点

> 通过具体的元素节点调用
>
> children 表示当前节点的所有子元素
>
> childNodes 表示当前节点的所有子节点(包括文本节点)
>
> firstElementChild 表示当前节点的第一个子元素
>
> firstChild 表示当前节点的第一个子节点
>
> lastChild 表示当前节点的最后子节点

- getElementsByTagName() 返回当前节点的指定标签名的后代节点

## 获取父节点和兄弟节点

> 通过具体的节点调用

- parentNode 表示当前节点的父节点
- previousElementSibling 表示当前节点的前一个兄弟元素
- previousSibling 表示当前节点的前一个兄弟节点
- nextElementSibling 表示当前节点的前一个兄弟元素
- nextSibling 表示当前节点的后一个兄弟节点

## 其他

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

   

