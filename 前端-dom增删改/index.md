# 前端-DOM增删改


# 前端-DOM增删改

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

   

