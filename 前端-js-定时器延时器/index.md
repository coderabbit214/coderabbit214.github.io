# 前端-js-定时器延时器


# 前端-js-定时器延时器

## 定时调用

> setInterval()
>
> 定时调用
>
> 可以将一个函数，每隔一段时间执行一次
> 		  	参数：
> 		  		1.回调函数，该函数会每隔一段时间被调用一次
> 		  		2.每次调用间隔的时间，单位是毫秒
> 		  	返回值：
> 		  		返回一个Number类型的数据
> 		  		这个数字用来作为定时器的唯一标识
>
> clearInterval(timer) 关闭定时器

```javascript
var num = 1;
				
var timer = setInterval(function(){
					
	count.innerHTML = num++;
					
	if(num == 11){
		//关闭定时器
		clearInterval(timer);
	}
},1000);
```

## 延时调用

>  延时调用，
> 			 延时调用一个函数不马上执行，而是隔一段时间以后在执行，而且只会执行一次
> 			  延时调用和定时调用的区别，定时调用会执行多次，而延时调用只会执行一次

```javascript
var timer = setTimeout(function(){
				console.log(num++);
			},3000);
			
			//使用clearTimeout()来关闭一个延时调用
			clearTimeout(timer);
```


