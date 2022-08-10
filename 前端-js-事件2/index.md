# 前端-js-事件2


# 前端-js-事件2

## 滚轮事件

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

## 键盘事件

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


