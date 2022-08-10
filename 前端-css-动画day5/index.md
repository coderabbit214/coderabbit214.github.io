# 前端-CSS-动画(day5)


# 前端-CSS-动画(day5)

浏览器兼容问题 （部分浏览器 不支持）：

旧版本浏览器都不支持

过渡浏览器：需要加一些参数

			- IE9: -ms-
			- firefox: -moz-
			- chrome/safari:  -webkit-
			- Opera: -o- 

如果有@，则兼容性前缀增加到@的后面

@-webkit-keyframes x

新浏览器一般都支持



## 变形函数：transform:xx()

| 值                                                           | 描述                                    | 测试                                                         |
| :----------------------------------------------------------- | :-------------------------------------- | :----------------------------------------------------------- |
| none                                                         | 定义不进行转换。                        | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_rotate&p=22) |
| matrix(*n*,*n*,*n*,*n*,*n*,*n*)                              | 定义 2D 转换，使用六个值的矩阵。        | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_matrix) |
| matrix3d(*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*) | 定义 3D 转换，使用 16 个值的 4x4 矩阵。 |                                                              |
| translate(*x*,*y*)                                           | 定义 2D 转换。                          | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_translate) |
| translate3d(*x*,*y*,*z*)                                     | 定义 3D 转换。                          |                                                              |
| translateX(*x*)                                              | 定义转换，只是用 X 轴的值。             | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_translatex) |
| translateY(*y*)                                              | 定义转换，只是用 Y 轴的值。             | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_translatey) |
| translateZ(*z*)                                              | 定义 3D 转换，只是用 Z 轴的值。         |                                                              |
| scale(*x*,*y*)                                               | 定义 2D 缩放转换。                      | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_scale) |
| scale3d(*x*,*y*,*z*)                                         | 定义 3D 缩放转换。                      |                                                              |
| scaleX(*x*)                                                  | 通过设置 X 轴的值来定义缩放转换。       | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_scalex) |
| scaleY(*y*)                                                  | 通过设置 Y 轴的值来定义缩放转换。       | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_scaley) |
| scaleZ(*z*)                                                  | 通过设置 Z 轴的值来定义 3D 缩放转换。   |                                                              |
| rotate(*angle*)                                              | 定义 2D 旋转，在参数中规定角度。        | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_rotate) |
| rotate3d(*x*,*y*,*z*,*angle*)                                | 定义 3D 旋转。                          |                                                              |
| rotateX(*angle*)                                             | 定义沿着 X 轴的 3D 旋转。               | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_rotatex) |
| rotateY(*angle*)                                             | 定义沿着 Y 轴的 3D 旋转。               | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_rotatey) |
| rotateZ(*angle*)                                             | 定义沿着 Z 轴的 3D 旋转。               | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_rotatez) |
| skew(*x-angle*,*y-angle*)                                    | 定义沿着 X 和 Y 轴的 2D 倾斜转换。      | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_skew) |
| skewX(*angle*)                                               | 定义沿着 X 轴的 2D 倾斜转换。           | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_skewx) |
| skewY(*angle*)                                               | 定义沿着 Y 轴的 2D 倾斜转换。           | [测试](https://www.w3school.com.cn/tiy/c.asp?f=css_transform_skewy) |
| perspective(*n*)                                             | 为 3D 转换元素定义透视视图。            | 测试                                                         |



## 过渡：transition

```html
div
{
width:100px;
transition: width 2s;
-moz-transition: width 2s; /* Firefox 4 */
-webkit-transition: width 2s; /* Safari 和 Chrome */
-o-transition: width 2s; /* Opera */
}

div:hover
{
width:300px;
}
```

```
transition: property duration timing-function delay;
```

| 值                                                           | 描述                                |
| :----------------------------------------------------------- | :---------------------------------- |
| [transition-property](https://www.w3school.com.cn/cssref/pr_transition-property.asp) | 规定设置过渡效果的 CSS 属性的名称。 |
| [transition-duration](https://www.w3school.com.cn/cssref/pr_transition-duration.asp) | 规定完成过渡效果需要多少秒或毫秒。  |
| [transition-timing-function](https://www.w3school.com.cn/cssref/pr_transition-timing-function.asp) | 规定速度效果的速度曲线。            |
| [transition-delay](https://www.w3school.com.cn/cssref/pr_transition-delay.asp) | 定义过渡效果何时开始。              |

```html
<html>
	<head>
		<meta charset="UTF-8">
		<title>动画</title>
		<style>
			
			#mybox img
			{
				transition: all 2s ease-in-out ; 
				
			}
			
			#mybox img:hover
			{
				transform:rotate(360deg) scale(1.5)
			}
			
		</style>
	</head>
<body>




	<div id="mybox">
		<img src="imgs/1.jpg"  width="50px" height="50px"/>
	</div>
	
	
	
	
</body>
</html>
```

## animation: 动画

https://www.w3school.com.cn/css3/css3_animation.asp

| 属性                                                         | 描述                                                     | CSS  |
| :----------------------------------------------------------- | :------------------------------------------------------- | :--- |
| [@keyframes](https://www.w3school.com.cn/cssref/pr_keyframes.asp) | 规定动画。                                               | 3    |
| [animation](https://www.w3school.com.cn/cssref/pr_animation.asp) | 所有动画属性的简写属性，除了 animation-play-state 属性。 | 3    |
| [animation-name](https://www.w3school.com.cn/cssref/pr_animation-name.asp) | 规定 @keyframes 动画的名称。                             | 3    |
| [animation-duration](https://www.w3school.com.cn/cssref/pr_animation-duration.asp) | 规定动画完成一个周期所花费的秒或毫秒。默认是 0。         | 3    |
| [animation-timing-function](https://www.w3school.com.cn/cssref/pr_animation-timing-function.asp) | 规定动画的速度曲线。默认是 "ease"。                      | 3    |
| [animation-delay](https://www.w3school.com.cn/cssref/pr_animation-delay.asp) | 规定动画何时开始。默认是 0。                             | 3    |
| [animation-iteration-count](https://www.w3school.com.cn/cssref/pr_animation-iteration-count.asp) | 规定动画被播放的次数。默认是 1。                         | 3    |
| [animation-direction](https://www.w3school.com.cn/cssref/pr_animation-direction.asp) | 规定动画是否在下一周期逆向地播放。默认是 "normal"。      | 3    |
| [animation-play-state](https://www.w3school.com.cn/cssref/pr_animation-play-state.asp) | 规定动画是否正在运行或暂停。默认是 "running"。           | 3    |
| [animation-fill-mode](https://www.w3school.com.cn/cssref/pr_animation-fill-mode.asp) | 规定对象动画时间之外的状态。                             | 3    |

```html
<html>
	<head>
		<meta charset="UTF-8">
		<title>动画</title>
		<style>
				div{
					width:100px ;
					height:100px ;
					background:blue ; 
					
					animation: x 2s linear infinite ;
				}
				
				@keyframes x{
					0%{
						width:0px ;
						transform:translateX(100px);
					}
					
					25%{
						width:20px ;
						transform:translateX(200px);
					}
					
					50%{
						width:50px ;
						transform:translateX(300px);
					}
					75%{
						width:75px ;
						transform:translateX(400px);
					}
					
					
					100%{
						width:100px ;
						transform:translateX(500px);
					
					}
				
				}
			
		</style>
	</head>
<body>




	<div>
	
	</div>
	
	
	
	
</body>
</html>
```


