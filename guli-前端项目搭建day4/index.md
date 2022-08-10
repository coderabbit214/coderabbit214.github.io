# guli-前端项目搭建(day4)


# guli-前端项目搭建(day4)

## 一.axios

```js
var app = new Vue({
 el: '#app',
 data: {
 memberList: []//数组
 },
 created() { //调用
 this.getList()
 },
 methods: { //定义
 getList(id) {
 //vm = this
 axios.get('http://localhost:8081/admin/ucenter/member')
 .then(response => {
 console.log(response)
 this.memberList = response.data.data.items
 })
 .catch(error => {
 console.log(error)
 })
 }
 }
})
```

## 二.Node.js

## 三.npm

> 相当于前端的Maven

```
//项目初始化
npm init
#按照提示输入相关信息，如果是用默认值则直接回车即可。
#name: 项目名称
#version: 项目版本号
#description: 项目描述
#keywords: {Array}关键词，便于用户搜索到我们的项目
#最后会生成package.json文件，这个是包的配置文件，相当于maven的pom.xml

#如果想直接生成 package.json 文件，那么可以使用命令
npm init -y

//修改npm镜像
#经过下面的配置，以后所有的 npm install 都会经过淘宝的镜像地址下载
npm config set registry https://registry.npm.taobao.org
#查看npm配置信息
npm config list

//npm install命令的使用
npm install #根据package.json中的配置下载依赖，初始化项目

#使用 npm install 安装依赖包的最新版，
#模块安装的位置：项目目录\node_modules
#安装会自动在项目目录下添加 package-lock.json文件，这个文件帮助锁定安装包的版本
#同时package.json 文件中，依赖包会被添加到dependencies节点下，类似maven中的<dependencies>
npm install jquery

//只有package.json和package-lock.json文件 没有下载依赖时
#devDependencies节点：开发时的依赖包，项目打包到生产环境的时候不包含的依赖
#使用 -D参数将依赖添加到devDependencies节点
npm install --save-dev eslint
#或
npm install -D eslint
#全局安装
#Node.js全局安装的npm包和工具的位置：用户目录\AppData\Roaming\npm\node_modules
#一些命令行工具常使用全局安装的方式
npm install -g webpack
```

## 四.Babel

> Babel是一个广泛使用的转码器，可以将ES6代码转为ES5代码，从而在现有环境执行执行。 这意味着，你可以现在就用 ES6 编写程序，而不用担心现有环境是否支持。

1. 安装

   ```
   npm install --global babel-cli
   #查看是否安装成功
   babel --version
   ```

2. 使用

   1. 初始化项目

      ```
      npm init -y
      ```

   2. 创建文件 src/example.js

      ```js
      //下面是一段ES6代码：
      // 转码前
      // 定义数据
      let input = [1, 2, 3]
      // 将数组的每个元素 +1
      input = input.map(item => item + 1)
      console.log(input)
      ```

   3. 配置.babelrc

      > Babel的配置文件是.babelrc，存放在项目的根目录下，该文件用来设置转码规则和插件，基本格式如 下。

      ```json
      {
       "presets": [],
       "plugins": []
      }
      ```

      presets字段设定转码规则，将es2015规则加入 .babelrc：

      ```
      {
       "presets": ["es2015"],
       "plugins": []
      }
      ```

   4. 安装转码器(在项目中安装)

      ```
      npm install --save-dev babel-preset-es2015
      ```

   5. 转码

      ```
      # --out-file 或 -o 参数指定输出文件
      babel src/example.js --out-file dist1/compiled.js
      # 或者
      babel src/example.js -o dist1/compiled.js
      
      # --out-dir 或 -d 参数指定输出目录
      babel src --out-dir dist2
      # 或者
      babel src -d dist2
      ```

## 五.模块化

1. 导出模块

   ```js
   export default {
    getList() {
    console.log('获取数据列表2')
    },
    save() {
    console.log('保存数据2')
    }
    }
   ```

2. 导入模块

   ```js
   import user from "./userApi2.js"
   user.getList()
   user.save()
   ```

## 六.Webpack

> Webpack 是一个前端资源加载/打包工具

1. 安装

   ```
   npm install -g webpack webpack-cli
   
   //查看版本号
   webpack -v
   ```

2. 初始化项目

   1. 创建webpack文件夹

   2. 进入webpack目录，执行命令

      ```
      npm init -y
      ```

   3. 创建src文件夹

   4. src下创建common.js

      ```js
      exports.info = function (str) {
       document.write(str);
      }
      ```

   5. src下创建utils.js

      ```js
      exports.add = function (a, b) {
       return a + b;
      }
      ```

   6. src下创建main.js

      ```js
      const common = require('./common');
      const utils = require('./utils');
      common.info('Hello world!' + utils.add(100, 200));
      ```

3. JS打包

   1. webpack目录下创建配置文件webpack.config.js

      ```js
      const path = require("path"); //Node.js内置模块
      module.exports = {
       entry: './src/main.js', //配置入口文件
       output: {
       path: path.resolve(__dirname, './dist'), //输出路径，__dirname：当前文件所在路径
       filename: 'bundle.js' //输出文件
       }
      }
      ```

   2. 命令行执行编译命令

      ```
      webpack #有黄色警告
      webpack --mode=development #没有警告
      #执行后查看bundle.js 里面包含了上面两个js文件的内容并进行了代码压缩
      ```

4. CSS打包

   1. 安装style-loader和 css-loader

      > Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。 Loader 可以理解为是模块和资源的转换器。
      >
      > 首先我们需要安装相关Loader插件，css-loader 是将 css 装载到 javascript；style-loader 是让 javascript 认识css

      ```
      npm install --save-dev style-loader css-loader
      ```

   2. 修改webpack.config.js

      ```js
      const path = require("path"); //Node.js内置模块
      module.exports = {
       //...,
       output:{},
       module: {
       rules: [
       {
       test: /\.css$/, //打包规则应用到以css结尾的文件上
       use: ['style-loader', 'css-loader']
       } 
       ]
       }
      }
      ```

## 七.vue-element-admin

> vue-element-admin是基于element-ui 的一套后台管理系统集成方案。 
>
> 功能：https://panjiachen.github.io/vue-element-admin-site/zh/guide/#
>
> 功能 GitHub地址：https://github.com/PanJiaChen/vue-element-admin 
>
> 项目在线预览：https://panjiachen.gitee.io/vue-element-admin

安装

```js
# 解压压缩包
# 进入目录
cd vue-element-admin-master
# 安装依赖
npm install
# 启动。执行后，浏览器自动弹出并访问http://localhost:9527/
npm run dev
```

## 八.项目创建和基本配置

1. 项目目录

   ```
   .
   ├── build // 构建脚本
   ├── config // 全局配置
   ├── node_modules // 项目依赖模块
   ├── src //项目源代码
   ├── static // 静态资源
   └── package.jspon // 项目信息和依赖配置
   ```

   ```
   src
   ├── api // 各种接口
   ├── assets // 图片等资源
   ├── components // 各种公共组件，非公共组件在各自view下维护
   ├── icons //svg icon
   ├── router // 路由表
   ├── store // 存储
   ├── styles // 各种样式
   ├── utils // 公共工具，非公共工具，在各自view下维护
   ├── views // 各种layout
   ├── App.vue //***项目顶层组件***
   ├── main.js //***项目入口文件***
   └── permission.js //认证入口
   ```

2. 运行项目

   ```
   npm run dev
   ```

   

