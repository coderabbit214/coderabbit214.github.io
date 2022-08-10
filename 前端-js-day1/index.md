# 前端-js-(day1)


# 前端-js-(day1)

javaScript 包括：

1. ECMAScript
2. DOM
3. BOM

JS特点

1. 解释型语言
2. 类似C的语法结构
3. 动态语言
4. 基于原型的面向对象

## JS的 Hello World

// 控制浏览器弹出警告框
alert("这是我的第一行JS代码");
// 让计算机在页面中输出一个内容
document.write("Hello World");
//向控制台输出一个内容
console.log("11111111111");

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<!-- js代码需要编写在script标签中 -->
		<script type="text/javascript">
			// 控制浏览器弹出警告框
			alert("这是我的第一行JS代码");
			// 让计算机在页面中输出一个内容
			document.write("Hello World");
			//向控制台输出一个内容
			console.log("11111111111");
		</script>
	</head>
	<body>
		
	</body>
</html>

```

## JS的编写位置

1. 标签属性中

```html
<!-- 虽然可以写在标签的属性中，但是行为耦合，不方便维护，不推荐使用 -->
		<!-- 可以将JS代码编写到标签的onclick属性中 -->
		<button onclick="alert('点我')">点我</button>
		<!-- 可以将JS代码写在超链接的href属性中，点击时 会执行js代码 -->
		<a href="javascript:alert('.....')">.....</a>
		<a href="javascript:;">.....</a>
```

2. script标签中

```html
<!-- 可以将js代码写道script标签中 -->
		<script type="text/javascript">
			
		</script>
```

3. 外部文件

```html
 <!-- script标签一旦用于引入外部文件 ，就不能在编写代码，编写了也不生效 -->
		<script type="text/javascript" src="js/script.js">
			alert("内部");
		</script>
```

总体代码

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<!-- 可以将js代码编写到外部的js文件中，然后引入
		 可以在不同的文件中引入，也可以利用到浏览器的缓存机制
		 推荐使用
		 -->
		 <!-- script标签一旦用于引入外部文件 ，就不能在编写代码，编写了也不生效 -->
		<script type="text/javascript" src="js/script.js">
			alert("内部");
		</script>
		
		<!-- 可以将js代码写道script标签中 -->
		<script type="text/javascript">
			
		</script>
	</head>
	<body>
		
		<!-- 虽然可以写在标签的属性中，但是行为耦合，不方便维护，不推荐使用 -->
		<!-- 可以将JS代码编写到标签的onclick属性中 -->
		<button onclick="alert('点我')">点我</button>
		<!-- 可以将JS代码写在超链接的href属性中，点击时 会执行js代码 -->
		<a href="javascript:alert('.....')">.....</a>
		<a href="javascript:;">.....</a>
		
	</body>
</html>

```

## JS中的基本语法

1. 严格区分大小写
2. 每一条语句以分号结尾 如果不写分号 浏览器会自动添加（有时候会加错）
3. JS中会忽略多个空格和换行

## JS定义变量

关键字 var

## 标识符

1.  标识符：在JS中可以由我们自主命名的都可以称为标识符
2. 命名规则
   1. 可以有数字，字母，_，$，UTF-8中的所有字符
   2. 不能以数字开头
   3. 不能是ES中的关键字或保留字
   4. 驼峰命名法（规范）

## 数据类型

JS中一共有六种数据类型
				基本数据类型
					String 字符串
					Number	数值
					Boolean 布尔值
					Null 空值
					Undefined 未定义
				对象数据类型
					Object 对象

### String

String 使用单引号或双引号引起来（不能混用）
 引号不能嵌套

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script type="text/javascript">
			
		   // String 使用单引号或双引号引起来（不能混用）
		   // 引号不能嵌套
		   var str = "Hello";
		   // 字符串中可以使用\作为转义字符
		   /*
		   \n 换行
		   \t 制表符
		   \\ 表示一个\
		   */
		   var str1 = "hello\"ssss\"";
		    alert(str1);
		</script>
	</head>
	<body>
	</body>
</html>
```


