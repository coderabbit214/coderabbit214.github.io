# 前端-jQuery-DOM


# 前端-jQuery-DOM

## HTML

- text() - 设置或返回所选元素的文本内容
- html() - 设置或返回所选元素的内容（包括 HTML 标记）
- val() - 设置或返回表单字段的值

> 获取
>
> ​	直接使用
>
> 设置
>
> ​	1.括号里传递参数
>
> ​	2.使用回调函数function(i,origText) i:元素下标 origText:旧的值

```js
  $(document).ready(function(){
            // alert($("p").text())
            // alert($("div").html())
            $("button").click(function(){
                // $("input").val("dddddd")
                $("input").val(function(i,origText){
                    return "a"+origText+i;
                })
                // $("p").text("dsdsdsds")
                $("p").text(function(i,origText){
                    return "a"+origText+i;
                })
                // $("div").html("<h1>sdsd</h1>")

                $("div").attr("style","background-color: aquamarine")
            })
        })
```

- append() - 在被选元素的结尾插入内容
- prepend() - 在被选元素的开头插入内容
- after() - 在被选元素之后插入内容
- before() - 在被选元素之前插入内容

```js
function appendText()
{
var txt1="<p>Text.</p>";               // 以 HTML 创建新元素
var txt2=$("<p></p>").text("Text.");   // 以 jQuery 创建新元素
var txt3=document.createElement("p");  // 以 DOM 创建新元素
txt3.innerHTML="Text.";
$("p").append(txt1,txt2,txt3);         // 追加新元素
}
```

- remove() - 删除被选元素（及其子元素）
- empty() - 从被选元素中删除子元素

### 设置CSS

- addClass() - 向被选元素添加一个或多个类
- removeClass() - 从被选元素删除一个或多个类
- toggleClass() - 对被选元素进行添加/删除类的切换操作
- css() - 设置或返回样式属性

```js
$("p").css({"background-color":"yellow","font-size":"200%"});
```

### 尺寸

- width() 方法设置或返回元素的宽度（不包括内边距、边框或外边距）。

- height() 方法设置或返回元素的高度（不包括内边距、边框或外边距）。

- innerWidth() 方法返回元素的宽度（包括内边距）。
- innerHeight() 方法返回元素的高度（包括内边距）。

- outerWidth() 方法返回元素的宽度（包括内边距和边框）。
- outerHeight() 方法返回元素的高度（包括内边距和边框）。
- outerWidth(true) 方法返回元素的宽度（包括内边距、边框和外边距）。
- outerHeight(true) 方法返回元素的高度（包括内边距、边框和外边距）。

## 节点

### 祖先

- parent()  返回被选元素的直接父元素
- parents()  返回被选元素的所有祖先元素
- parentsUntil()  返回介于两个给定元素之间的所有祖先元素

### 后代

- children() 返回被选元素的所有子元素
- find() 返回被选元素的后代元素 参数“*”代表所有 可以写标签名 

### 同胞

- siblings() 返回被选元素的所有同胞元素
- next() 返回被选元素的下一个同胞元素
- nextAll() 返回被选元素后的所有同胞元素
- nextUntil() 返回介于两个给定参数之间的所有跟随的同胞元素 例$("h2").nextUntil("h6");
- prev() 向前
- prevAll()
- prevUntil()

### 过滤

* first() 方法返回被选元素的首个元素

* last() 最后一个

```
  //选取首个 <div> 元素内部的第一个 <p> 元素
  $(document).ready(function(){
    $("div p").first();
  });
```

- eq() 方法返回被选元素中带有指定索引号的元素

  ```
  $(document).ready(function(){
    $("p").eq(1);
  });
  ```

- filter() 规定一个标准

- not() 相反

  ```
  //返回带有类名 "intro" 的所有 <p> 元素
  $(document).ready(function(){
    $("p").filter(".intro");
  });
  ```

  

