# 前端-CSS-Grid网格布局


# 前端-CSS-Grid网格布局

## 容器属性

* grid-template-columns 列宽
  - grid-template-columns: 70% 30%;  百分比设置 
  - grid-template-columns: 1fr 2fr 1fr; 比例设置（推荐使用）
  - grid-template-columns: repeat(3,1fr); 使用函数进行比例设置
* grid-column-gap 列间距
* grid-row-gap 行间距
* grid-gap 统一设置列间距和行间距
* grid-auto-rows 行高
  - grid-auto-rows: 100px; 固定数值
  - grid-auto-rows: minmax(100px,auto); 通过函数设置最大最小值 防止内容超出
* justify-items 列对齐方式 默认值为stretch 拉伸
* align-items 行对齐方式 默认值为stretch 拉伸
* grid-template-areas 设置模板 与子组件属性grid-area搭配使用

## 子组件属性

- align-self 行对齐方式 默认值为stretch 拉伸
- justify-self 列对齐方式 默认值为stretch 拉伸
- grid-column 跨列
- grid-row 跨行
- grid-area 与grid-template-areas配合使用

## 基础布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CSS Grid2</title>
  <style>
    .wrapper {
      display: grid;
      /* 推荐 比例 */
      /* grid-template-columns: 1fr 2fr 1fr; */
      grid-template-columns: repeat(3,1fr);
      /* 间距 */
      grid-gap: 1rem;
      /* 行高 */
      grid-auto-rows: 100px;
      /* minmax函数 用来限制框的最大最小值 */
      grid-auto-rows: minmax(100px,auto);
    }

    .wrapper>div {
      background: #eee;
      padding: 1rem;
    }

    .wrapper>div:nth-child(odd) {
      background: #ddd;
    }
    .nested{
      display: grid;
      grid-template-columns: repeat(3,1fr);
      grid-gap: 1rem;
      grid-auto-rows: minmax(20px,auto);
    }
    .nested>div{
      border: 1px #333 solid;
      padding: 1rem;
    }
  </style>
</head>

<body>
  <div class="wrapper">
    <div>Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquid eum, quos unde soluta, sapiente accusamus
      nesciunt earum corrupti cumque explicabo ducimus labore similique? Aperiam totam incidunt reiciendis facilis
      blanditiis soluta!</div>
    <div>Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquid eum, quos unde soluta, sapiente accusamus
      nesciunt earum corrupti cumque explicabo ducimus labore similique? Aperiam totam incidunt reiciendis facilis
      blanditiis soluta!</div>
    <div class="nested">
      <div>111</div>
      <div>222</div>
      <div>333</div>
      <div>111</div>
      <div>222</div>
      <div>333</div>
      <div>111</div>
      <div>222</div>
      <div>333</div>
      <div>111</div>
      <div>222</div>
      <div>333</div>
    </div>
    <div>Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquid eum, quos unde soluta, sapiente accusamus
      nesciunt earum corrupti cumque explicabo ducimus labore similique? Aperiam totam incidunt reiciendis facilis
      blanditiis soluta!</div>
    <div>Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquid eum, quos unde soluta, sapiente accusamus
      nesciunt earum corrupti cumque explicabo ducimus labore similique? Aperiam totam incidunt reiciendis facilis
      blanditiis soluta!</div>
  </div>

</body>

</html>
```

## 跨行 跨列 具体使用

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CSS Grid2</title>
  <style>
    .wrapper {
      display: grid;
      grid-template-columns: 1fr 2fr 1fr;
      grid-auto-rows: minmax(100px,auto);
      grid-gap: 1rem;
      justify-items: stretch;
      align-items: stretch;
    }

    .wrapper>div {
      background: #eee;
      padding: 1rem;
    }

    .wrapper>div:nth-child(odd) {
      background: #ddd;
    }
    .box1{
      /* align-self: start; */
      /* 跨列 */
      grid-column: 1/3;
      grid-row: 1/3;
    }
    .box2{
      /* align-self: end; */
      grid-column: 3;
      grid-row: 1/3;
    }
    .box3{
      /* justify-self: end; */
      grid-column: 2/4;
      grid-row: 3;
    }
    .box4{
      /* justify-self: start; */
      grid-column: 1;
      grid-row: 2/4;
    }
 
  </style>
</head>

<body>
  <div class="wrapper">
    <div class="box1">
      111
    </div>
    <div class="box2">
      111
    </div>
    <div class="box3">
      111
    </div>
    <div class="box4">
      111
    </div>
  </div>

</body>

</html>
```

## 模板具体使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CSS Grid4</title>
  <style>
    .wrapper{
      display: grid;
      grid-template-areas: 
      'a a a'
      'b b c'
      'd e f';
      grid-gap: 1rem;
    }
    .box1{
      grid-area: a;
    }
    .box2{
      grid-area:b;
    }
    .box3{
      grid-area:c;
    }
    .box4{
      grid-area: d;
    }
    .box5{
      grid-area:e;
    }
    .box6{
      grid-area:f;
    }
    .wrapper>div {
      background: #eee;
      padding: 1rem;
    }

    .wrapper>div:nth-child(odd) {
      background: #ddd;
    }

    @media(max-width: 700px) {
      .wrapper{
        grid-template-areas: 
        'a''b''c''d''e''f';
      }
    }
  </style>
</head>
<body>
  <div class="wrapper">
    <div class="box1">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Ad laudantium explicabo impedit. Aliquid, aut doloremque illo eum ex deleniti unde ea non optio laboriosam voluptas ipsum voluptatibus illum, quaerat obcaecati inventore, temporibus vel earum quibusdam necessitatibus? Aperiam tempore ullam assumenda nulla facere! Architecto reiciendis nisi ex totam labore. Ad ipsam quae eligendi cupiditate maxime commodi reprehenderit similique exercitationem nam porro ducimus quos facere incidunt natus magni cum fugit culpa soluta, sed maiores doloribus voluptatum deserunt. Ex consequatur eius atque ipsa!
    </div>
    <div class="box2">Lorem ipsum dolor sit amet consectetur adipisicing elit. Saepe accusamus doloribus fuga dolores natus, sapiente vero veritatis aut facere itaque eos quam fugiat maxime aperiam possimus reiciendis? Voluptatibus, nostrum expedita!
    </div>
    <div class="box3">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Ad laudantium explicabo impedit. Aliquid, aut doloremque illo eum ex deleniti unde ea non optio laboriosam voluptas ipsum voluptatibus illum, quaerat obcaecati inventore, temporibus vel earum quibusdam necessitatibus? Aperiam tempore ullam assumenda nulla facere! Architecto reiciendis nisi ex totam labore. Ad ipsam quae eligendi cupiditate maxime commodi reprehenderit similique exercitationem nam porro ducimus quos facere incidunt natus magni cum fugit culpa soluta, sed maiores doloribus voluptatum deserunt. Ex consequatur eius atque ipsa!
    </div>
    <div class="box4">Lorem ipsum dolor sit amet consectetur adipisicing elit. Saepe accusamus doloribus fuga dolores natus, sapiente vero veritatis aut facere itaque eos quam fugiat maxime aperiam possimus reiciendis? Voluptatibus, nostrum expedita!
    </div>
    <div class="box5">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Ad laudantium explicabo impedit. Aliquid, aut doloremque illo eum ex deleniti unde ea non optio laboriosam voluptas ipsum voluptatibus illum, quaerat obcaecati inventore, temporibus vel earum quibusdam necessitatibus? Aperiam tempore ullam assumenda nulla facere! Architecto reiciendis nisi ex totam labore. Ad ipsam quae eligendi cupiditate maxime commodi reprehenderit similique exercitationem nam porro ducimus quos facere incidunt natus magni cum fugit culpa soluta, sed maiores doloribus voluptatum deserunt. Ex consequatur eius atque ipsa!
    </div>
    <div class="box6">Lorem ipsum dolor sit amet consectetur adipisicing elit. Saepe accusamus doloribus fuga dolores natus, sapiente vero veritatis aut facere itaque eos quam fugiat maxime aperiam possimus reiciendis? Voluptatibus, nostrum expedita!
    </div>
  
</div>
</body>
</html>
```


