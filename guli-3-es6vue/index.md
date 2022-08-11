# guli-3-es6,vue


# guli-3-es6,vue

## 一.ES6

1. let声明变量

   ```js
   // var 声明的变量没有局部作用域
   // let 声明的变量 有局部作用域
   {
   var a = 0
   let b = 1
   }
   console.log(a) // 0
   console.log(b) // ReferenceError: b is not defined
   
   // var 可以声明多次
   // let 只能声明一次
   var m = 1
   var m = 2
   let n = 3
   let n = 4
   console.log(m) // 2
   console.log(n) // Identifier 'n' has already been declared
   ```

2. const声明常量（只读变量）

   ```js
   // 1、声明之后不允许改变
   const PI = "3.1415926"
   PI = 3 // TypeError: Assignment to constant variable.
   
   // 2、一但声明必须初始化，否则会报错
   const MY_AGE // SyntaxError: Missing initializer in const declaration
   ```

3. 解构赋值

   ```js
   //1、数组解构
   // 传统
   let a = 1, b = 2, c = 3
   console.log(a, b, c)
   // ES6
   let [x, y, z] = [1, 2, 3]
   console.log(x, y, z)
   
   //2、对象解构
   let user = {name: 'Helen', age: 18}
   // 传统
   let name1 = user.name
   let age1 = user.age
   console.log(name1, age1)
   // ES6
   let { name, age } = user//注意：结构的变量必须是user中的属性
   console.log(name, age)
   ```

4. 模板字符串

   ```js
   // 1、多行字符串
   let string1 = `Hey,
   can you stop angry now?`
   console.log(string1)
   // Hey,
   // can you stop angry now?
   
   // 2、字符串插入变量和表达式。变量名写在 ${} 中，${} 中可以放入 JavaScript 表达式。
   let name = "Mike"
   let age = 27
   let info = `My Name is ${name},I am ${age+1} years old next year.`
   console.log(info)
   // My Name is Mike,I am 28 years old next year.
   
   // 3、字符串中调用函数
   function f(){
    return "have fun!"
   }
   let string2 = `Game start,${f()}`
   console.log(string2); // Game start,have fun!
   
   ```

5. 声明对象简写

   ```js
   const age = 12
   const name = "Amy"
   // 传统
   const person1 = {age: age, name: name}
   console.log(person1)
   // ES6
   const person2 = {age, name}
   console.log(person2) //{age: 12, name: "Amy"}
   ```

6. 定义方法简写

   ```js
   // 传统
   const person1 = {
    sayHi:function(){
    console.log("Hi")
    }
   }
   person1.sayHi();//"Hi"
   // ES6
   const person2 = {
    sayHi(){
    console.log("Hi")
    }
   }
   person2.sayHi() //"Hi"
   
   ```

7. 对象拓展运算符

   ```js
   // 1、拷贝对象
   let person1 = {name: "Amy", age: 15}
   let someone = { ...person1 }
   console.log(someone) //{name: "Amy", age: 15}
   
   // 2、合并对象
   let age = {age: 15}
   let name = {name: "Amy"}
   let person2 = {...age, ...name}
   console.log(person2) //{age: 15, name: "Amy"}
   ```

8. 箭头函数

   > 箭头函数多用于匿名函数的定义

   ```js
   // 传统
   var f1 = function(a){
    return a
   }
   console.log(f1(1))
   // ES6
   var f2 = a => a
   console.log(f2(1))
   
   // 当箭头函数没有参数或者有多个参数，要用 () 括起来。
   // 当箭头函数函数体有多行语句，用 {} 包裹起来，表示代码块，
   // 当只有一行语句，并且需要返回结果时，可以省略 {} , 结果会自动返回。
   var f3 = (a,b) => {
    let result = a+b
    return result
   }
   console.log(f3(6,2)) // 8
   // 前面代码相当于：
   var f4 = (a,b) => a+b
   ```

## 二.vue

1. 局部组件

   - 定义

     ```js
     var app = new Vue({
      el: '#app',
      // 定义局部组件，这里可以定义多个局部组件
      components: {
      //组件的名字
      'Navbar': {
      //组件的内容
      template: '<ul><li>首页</li><li>学员管理</li></ul>'
      }
      }
     })
     ```

   - 使用

     ```html
     <div id="app">
      <Navbar></Navbar>
     </div>		
     ```
   
2. 全局组件

   - 定义 components/Navbar.js

     ```js
     // 定义全局组件
     Vue.component('Navbar', {
      template: '<ul><li>首页</li><li>学员管理</li><li>讲师管理</li></ul>'
     })
     ```

   - 使用

     ```html
     <div id="app">
      <Navbar></Navbar>
     </div>
     <script src="vue.min.js"></script>
     <script src="components/Navbar.js"></script>
     <script>
      var app = new Vue({
      el: '#app'
      })
     </script>
     ```

3. 生命周期

   ```js
   //数据渲染之前
   created() { // 第二个被执行的钩子方法
    console.log(this.message) //床前明月光
    this.show() //执行show方法
    // created执行时，data 和 methods 都已经被初始化好了！
    // 如果要调用 methods 中的方法，或者操作 data 中的数据，最早，只能在 created 中操作
   },
   //数据渲染之后
   mounted() { // 第四个被执行的钩子方法
    console.log(document.getElementById('h3').innerText) //床前明月光
    // 内存中的模板已经渲染到页面，用户已经可以看见内容
   },
   ```

4. 路由

   1. 引入js

      ```html
      <script src="vue.min.js"></script>
      <script src="vue-router.min.js"></script>
      ```

   2. 编写html

      ```html
      <div id="app">
       <h1>Hello App!</h1>
       <p>
       <!-- 使用 router-link 组件来导航. -->
       <!-- 通过传入 `to` 属性指定链接. -->
       <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
       <router-link to="/">首页</router-link>
       <router-link to="/student">会员管理</router-link>
       <router-link to="/teacher">讲师管理</router-link>
       </p>
       <!-- 路由出口 -->
       <!-- 路由匹配到的组件将渲染在这里 -->
       <router-view></router-view>
      </div>
      ```

   3. 编写js

      ```html
      <script>
       // 1. 定义（路由）组件。
       // 可以从其他文件 import 进来
       const Welcome = { template: '<div>欢迎</div>' }
       const Student = { template: '<div>student list</div>' }
       const Teacher = { template: '<div>teacher list</div>' }
       // 2. 定义路由
       // 每个路由应该映射一个组件。
       const routes = [
       { path: '/', redirect: '/welcome' }, //设置默认指向的路径
       { path: '/welcome', component: Welcome },
       { path: '/student', component: Student },
       { path: '/teacher', component: Teacher }
       ]
       // 3. 创建 router 实例，然后传 `routes` 配置
       const router = new VueRouter({
       routes // （缩写）相当于 routes: routes
       })
       // 4. 创建和挂载根实例。
       // 从而让整个应用都有路由功能
       const app = new Vue({
       el: '#app',
       router
       })
       // 现在，应用已经启动了！
      </script>
      ```

      

   





