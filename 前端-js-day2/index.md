# 前端-js-(day2)


# 前端-js-(day2)

## Null

> 表示空对象

## Undefined

> 声明变量但不给变量赋值

## 强制类型转换

转为String

> 1.toString()   不能转换null和undefind
>
> 2.String()

```javascript
var a = 123;
a = a.toString();
a = String(a);
```

转换为Number

> 1.Number()
>
> ​	字符串（数字）直接转换
>
> ​	如果有非数字内容 转换为NaN
>
> ​	true 1 false 0
>
> ​	null 0
>
> ​	undefind NaN
>
> 2.parseInt() 把字符串转换成整数
>
> ​	取出有效整数 只取第一个
>
> ​	parseFloat() 可以取得有效小数

```javascript
var a = "123";
a = Number(a);
```

## 对象

```javascript
//新建对象 
var obj = new Object();
//添加属性
obj.name = "sss";
//读取属性
console.log(obj.name);
```

检查对象中是否有属性

```javascript
cosole.log("name" in obj);
```

创建对象

```javascript
var obj2 = {
    name:"sss",
    age:14,
    sex:true
};
```

## 函数

```javascript
var fun = function(){
    
}
```

立即执行函数

```javascript
(function(a,b){
	console.log(a);
    console.log(b);
})(123,456);
```

对象中定义方法

```javascript
var obj2 = {
    name:"sss",
    age:14,
    sex:true,
    fun:function(){
        
    }
};
```

枚举对象中的属性，值

```javascript
for(var a in obj){
    console.log(a);
    console.log(obj[a]);
}
```


