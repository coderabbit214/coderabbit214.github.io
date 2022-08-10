# guli(day5)


# guli(day5)

## 一.后端登录功能改造

1. 修改前端配置文件请求地址

   > config文件夹下dev.env.js

   ```js
   'use strict'
   const merge = require('webpack-merge')
   const prodEnv = require('./prod.env')
   
   module.exports = merge(prodEnv, {
     NODE_ENV: '"development"',
     // BASE_API: '"https://easy-mock.com/mock/5950a2419adc231f356a6636/vue-admin"',
     BASE_API: '"http://localhost:8081"',
   })
   ```

2. 进行登录调用两个方法，login登录操作方法，info用于登录之后获取用户信息

   - login 返回 token值
   - info 返回 roles name avatar

3. 后端开发接口

   ```java
   package com.guli.eduservice.controller;
   
   @RestController
   @RequestMapping("/eduservice/user")
   @CrossOrigin //解决跨域
   public class EduLoginController {
       //login
       @PostMapping("login")
       public R login() {
           return R.ok().data("token","admin");
       }
   
       //info
       @GetMapping("info")
       public R info() {
           return R.ok().data("roles","admin")
                   .data("name","admin")
                   .data("avatar","https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=137628589,3436980029&fm=26&gp=0.jpg");
   
       }
   }
   
   ```

4. 前端修改api文件夹login.js修改本地接口路径

   ```js
   import request from '@/utils/request'
   
   export function login(username, password) {
     return request({
       url: '/eduservice/user/login',
       method: 'post',
       data: {
         username,
         password
       }
     })
   }
   
   export function getInfo(token) {
     return request({
       url: '/eduservice/user/info',
       method: 'get',
       params: { token }
     })
   }
   ```

5. 最终测试 出现跨域问题

   > 通过一个地址去访问另外一个地址 三个地方任何一个出现不一样
   >
   > 1. 访问协议 http https
   > 2. ip地址
   > 3. 端口号 9825 8081

6. 跨域解决方式

   > 在后端接口controller添加注解@CrossOrigin 

   ```java
   package com.guli.eduservice.controller;
   
   @RestController
   @RequestMapping("/eduservice/user")
   @CrossOrigin //解决跨域
   public class EduLoginController {
       //login
       @PostMapping("login")
       public R login() {
           return R.ok().data("token","admin");
       }
   
       //info
       @GetMapping("info")
       public R info() {
           return R.ok().data("roles","admin")
                   .data("name","admin")
                   .data("avatar","https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=137628589,3436980029&fm=26&gp=0.jpg");
   
       }
   }
   
   ```

## 二.前端开发过程(思路)

### 1.路由

1. 添加路由

   > src/router/index.js

2. 添加路由 

   > 路由对应界面
   >
   >  component: () => import('@/views/edu/teacher/listTeacher'),

   ```js
   {
           path: 'listTeacher',
           name: '讲师列表',
           component: () => import('@/views/edu/teacher/listTeacher'),
           meta: { title: '讲师列表', icon: 'table' }
         },
   ```

3. 创建对应界面

### 2.接口

1. 在api文件夹创建js文件 定义接口地址，参数

   ```js
   import request from '@/utils/request'
   
   export default {
     //1.讲师列表（条件查询分页）
     getTeacherListPage(current, limit, teacherQuery) {
       return request({
         url: `/eduservice/teacher/pageTeacherCondition/${current}/${limit}`,
         method: "post",
         //teacherQuery条件对象，后端使用RequestBody获取数据
         //data表示把对象转换json进行传递到接口里面
         data: teacherQuery
       })
     },
     //2.删除功能
     removeTeacherById(id) {
       return request({
         url: `/eduservice/teacher/${id}`,
         method: "delete"
       })
     },
     //3.添加讲师
     addTeacher(teacher) {
       return request({
         url: `/eduservice/teacher/addTeacher`,
         method: "post",
         data: teacher
       })
     },
     //4.根据id查询
     getTeacherById(id) {
       return request({
         url: `/eduservice/teacher/selectTeacherById/${id}`,
         method: "get",
       })
     },
     //修改讲师
     updateTeacher(teacher) {
       return request({
         url: `/eduservice/teacher/updateTeacher`,
         method: "put",
         data: teacher
       })
     }
   }
   
   ```

2. 在对应界面引入js文件 调用方法实现功能

## 三.讲师列表前端开发

1. 添加路由

   > src/router/index.js

   ```js
   {
       path: '/teacher',
       component: Layout,
       redirect: '/teacher/listTeacher',
       name: '讲师管理',
       meta: { title: '讲师管理', icon: 'example' },
       children: [
         {
           path: 'listTeacher',
           name: '讲师列表',
           component: () => import('@/views/edu/teacher/listTeacher'),
           meta: { title: '讲师列表', icon: 'table' }
         },
         {
           path: 'saveTeacher',
           name: '添加讲师',
           component: () => import('@/views/edu/teacher/saveTeacher'),
           meta: { title: '添加讲师', icon: 'tree' }
         }
       ]
     },
   ```

2. 创建路由对应界面

3. 定义访问的接口地址

   > src/api/edu/teacher.js

   ```js
   import request from '@/utils/request'
   
   export default {
     //1.讲师列表（条件查询分页）
     getTeacherListPage(current, limit, teacherQuery) {
       return request({
         url: `/eduservice/teacher/pageTeacherCondition/${current}/${limit}`,
         method: "post",
         //teacherQuery条件对象，后端使用RequestBody获取数据
         //data表示把对象转换json进行传递到接口里面
         data: teacherQuery
       })
     }
   }
   
   ```

4. 在listTeacher页面调用方法

5. 把数据进行显示

   ```html
   <template>
     <div class="app-container">
       <el-table :data="list" element-loading-text="数据加载中" border fit highlight-current-row>
         <el-table-column label="序号" width="70" align="center">
           <template slot-scope="scope">{{ (page - 1) * limit + scope.$index + 1 }}</template>
         </el-table-column>
         <el-table-column prop="name" label="名称" width="80" />
         <el-table-column label="头衔" width="80">
           <template slot-scope="scope">{{ scope.row.level===1?'高级讲师':'首席讲师' }}</template>
         </el-table-column>
         <el-table-column prop="intro" label="资历" />
         <el-table-column prop="gmtCreate" label="添加时间" width="160" />
         <el-table-column prop="sort" label="排序" width="60" />
         <el-table-column label="操作" width="200" align="center">
           <template slot-scope="scope">
             <router-link :to="'/teacher/updateTeacher/'+scope.row.id">
               <el-button type="primary" size="mini" icon="el-icon-edit">修改</el-button>
             </router-link>
             <el-button
               type="danger"
               size="mini"
               icon="el-icon-delete"
               @click="removeTeacherById(scope.row.id)"
             >删除</el-button>
           </template>
         </el-table-column>
       </el-table>
   </template>
   
   <script>
   import teacher from "@/api/edu/teacher";
   export default {
     data() {
       return {
         list: null, //查询之后接口返回集合
         page: 1, //当前页
         limit: 3, //每页记录数
         total: 0, //总记录数
         teacherQuery: {}, //条件封装对象
       };
     },
     created() {
       //页面渲染之前执行，一般用于调用methods中定义的方法
       this.getTeacherListPage();
     },
     methods: {
       //页面渲染之后执行，一般用于创建具体方法
       //讲师列表方法
       getTeacherListPage() {
         teacher
           .getTeacherListPage(this.page, this.limit, this.teacherQuery)
           .then((response) => {
             //请求成功
             //response接口返回的数据
             this.list = response.data.rows;
             this.total = response.data.total;
           })
           .catch((error) => {
             console.log(error);
           });
       } 
     },
   };
   </script>
   
   
   ```

6. 分页实现

   > 分页方法修改 需要修改当前页面数
   >
   > getTeacherListPage(page = 1) {
   >       this.page = page;
   >
   > ......

   ```html
   <template>
     <div class="app-container">
       <el-table :data="list" element-loading-text="数据加载中" border fit highlight-current-row>
         <el-table-column label="序号" width="70" align="center">
           <template slot-scope="scope">{{ (page - 1) * limit + scope.$index + 1 }}</template>
         </el-table-column>
         <el-table-column prop="name" label="名称" width="80" />
         <el-table-column label="头衔" width="80">
           <template slot-scope="scope">{{ scope.row.level===1?'高级讲师':'首席讲师' }}</template>
         </el-table-column>
         <el-table-column prop="intro" label="资历" />
         <el-table-column prop="gmtCreate" label="添加时间" width="160" />
         <el-table-column prop="sort" label="排序" width="60" />
         <el-table-column label="操作" width="200" align="center">
           <template slot-scope="scope">
             <router-link :to="'/teacher/updateTeacher/'+scope.row.id">
               <el-button type="primary" size="mini" icon="el-icon-edit">修改</el-button>
             </router-link>
             <el-button
               type="danger"
               size="mini"
               icon="el-icon-delete"
               @click="removeTeacherById(scope.row.id)"
             >删除</el-button>
           </template>
         </el-table-column>
       </el-table>
         
         
       <!-- 分页 -->
       <el-pagination
         :current-page="page"
         :page-size="limit"
         :total="total"
         style="padding: 30px 0; text-align: center;"
         layout="total, prev, pager, next, jumper"
         @current-change="getTeacherListPage"
       />
     </div>
   </template>
   
   <script>
   import teacher from "@/api/edu/teacher";
   export default {
     data() {
       return {
         list: null, //查询之后接口返回集合
         page: 1, //当前页
         limit: 3, //每页记录数
         total: 0, //总记录数
         teacherQuery: {}, //条件封装对象
       };
     },
     created() {
       //页面渲染之前执行，一般用于调用methods中定义的方法
       this.getTeacherListPage();
     },
     methods: {
       //页面渲染之后执行，一般用于创建具体方法
       //讲师列表方法
       getTeacherListPage(page = 1) {
         this.page = page;
         teacher
           .getTeacherListPage(this.page, this.limit, this.teacherQuery)
           .then((response) => {
             //请求成功
             //response接口返回的数据
             this.list = response.data.rows;
             this.total = response.data.total;
           })
           .catch((error) => {
             console.log(error);
           });
       }
     }
   };
   </script>
   
   
   ```

7. 条件查询实现

   > 界面使用elment-ui实现
   >
   > 使用 v-model 进行数据绑定

   ```html
   <!--查询表单-->
       <el-form :inline="true" class="demo-form-inline">
         <el-form-item>
           <el-input v-model="teacherQuery.name" placeholder="讲师名" />
         </el-form-item>
         <el-form-item>
           <el-select v-model="teacherQuery.level" clearable placeholder="讲师头衔">
             <el-option :value="1" label="高级讲师" />
             <el-option :value="2" label="首席讲师" />
           </el-select>
         </el-form-item>
         <el-form-item label="添加时间">
           <el-date-picker
             v-model="teacherQuery.begin"
             type="datetime"
             placeholder="选择开始时间"
             value-format="yyyy-MM-dd HH:mm:ss"
             default-time="00:00:00"
           />
         </el-form-item>
         <el-form-item>
           <el-date-picker
             v-model="teacherQuery.end"
             type="datetime"
             placeholder="选择截止时间"
             value-format="yyyy-MM-dd HH:mm:ss"
             default-time="00:00:00"
           />
         </el-form-item>
         <el-button type="primary" icon="el-icon-search" @click="getTeacherListPage()">查询</el-button>
         <el-button type="default" @click="resetData()">清空</el-button>
       </el-form>
   ```

   - 清空功能

     ```js
     resetData() {
           //清空的方法
           //表单数据清空
           this.teacherQuery = {};
           //查询所有
           this.getTeacherListPage();
         },
     ```

## 四.讲师删除功能

1. 每条记录后添加删除按钮

2. 按钮绑定删除事件 并传递参数：讲师的id

   ```html
   <el-button
               type="danger"
               size="mini"
               icon="el-icon-delete"
               @click="removeTeacherById(scope.row.id)"
             >删除</el-button>
   ```

3. 在src/api/edu/teacher.js定义删除接口地址

   ```js
   //2.删除功能
     removeTeacherById(id) {
       return request({
         url: `/eduservice/teacher/${id}`,
         method: "delete"
       })
     },
   ```

4. 页面调用 实现方法

   ```js
   removeTeacherById(id) {
         //删除功能
         this.$confirm("此操作将永久删除该讲师记录, 是否继续?", "提示", {
           confirmButtonText: "确定",
           cancelButtonText: "取消",
           type: "warning",
         }).then(() => {
           teacher
             .removeTeacherById(id)
             .then((response) => {
               //删除成功
               //提示信息
               this.$message({
                 type: "success",
                 message: "删除成功!",
               });
               //回到列表
               this.getTeacherListPage();
             })
             .catch((error) => {
               //提示信息
               this.$message({
                 type: "warning",
                 message: "删除失败!",
               });
             });
         });
       },
   ```

## 五.讲师添加功能

1. 创建路由

   ```js
   {
       path: '/teacher',
       component: Layout,
       redirect: '/teacher/listTeacher',
       name: '讲师管理',
       meta: { title: '讲师管理', icon: 'example' },
       children: [
         。。。
         {
           path: 'saveTeacher',
           name: '添加讲师',
           component: () => import('@/views/edu/teacher/saveTeacher'),
           meta: { title: '添加讲师', icon: 'tree' }
         }
         。。。
       ]
     },
   ```

2. 创建页面

3. 添加合适的elment-ui组件

   ```html
   <template>
     <div class="app-container">
       <el-form label-width="120px">
         <el-form-item label="讲师名称">
           <el-input v-model="teacher.name" />
         </el-form-item>
         <el-form-item label="讲师排序">
           <el-input-number v-model="teacher.sort" controls-position="right" min="0" />
         </el-form-item>
         <el-form-item label="讲师头衔">
           <el-select v-model="teacher.level" clearable placeholder="请选择">
             <!--数据类型一定要和取出的json中的一致，否则没法回填因此，这里value使用动态绑定的值，保证其数据类型是number-->
             <el-option :value="1" label="高级讲师" />
             <el-option :value="2" label="首席讲师" />
           </el-select>
         </el-form-item>
         <el-form-item label="讲师资历">
           <el-input v-model="teacher.career" />
         </el-form-item>
         <el-form-item label="讲师简介">
           <el-input v-model="teacher.intro" :rows="10" type="textarea" />
         </el-form-item>
         <!-- 讲师头像：TODO -->
         <el-form-item>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存</el-button>
         </el-form-item>
       </el-form>
     </div>
   </template>
   
   <script>
   import teacher from "@/api/edu/teacher";
   
   export default {
     data() {
       
     },
     created() {
       
     },
     methods: {
       
     },
   };
   </script>
   
   
   ```

4. api定义接口地址

   ```js
    //3.添加讲师
     addTeacher(teacher) {
       return request({
         url: `/eduservice/teacher/addTeacher`,
         method: "post",
         data: teacher
       })
     },
   ```

5. 在页面实现调用

   ```js
    methods: {
        saveOrUpdate() {  
           this.saveTeacher();
       },
       saveTeacher() {
         teacher
           .addTeacher(this.teacher)
           .then((response) => {
             //添加成功
             //提示信息
             this.$message({
               type: "success",
               message: "添加成功",
             });
             //回到讲师列表 路由跳转(重定向)
             this.$router.push({ path: "/teacher/listTeacher" });
           })
           .catch((error) => {
             //提示信息
             this.$message({
               type: "warning",
               message: "添加失败",
             });
           });
       },
     },
   
   ```

## 六.讲师修改

1. 每条记录后添加修改按钮

2. 通过路由跳转进入回显页面，添加路由

   > 重点 id的传递

   ```js
   {
       path: '/teacher',
       component: Layout,
       redirect: '/teacher/listTeacher',
       name: '讲师管理',
       meta: { title: '讲师管理', icon: 'example' },
       children: [
   		。。。
         {
           //接收id
           path: 'updateTeacher/:id',
           name: '编辑讲师',
           component: () => import('@/views/edu/teacher/saveTeacher'),
           meta: { title: '编辑讲师', icon: 'tree' },
           hidden:true
         }
       ]
     },
   ```

3. 修改前端页面修改按钮的路径 传递 id

   ```html
   <template slot-scope="scope">
   <router-link :to="'/teacher/updateTeacher/'+scope.row.id">
               <el-button type="primary" size="mini" icon="el-icon-edit">修改</el-button>
             </router-link>
       。。。
   ```

4. 定义api接口

   ```js
   //4.根据id查询
     getTeacherById(id) {
       return request({
         url: `/eduservice/teacher/selectTeacherById/${id}`,
         method: "get",
       })
     },
     //修改讲师
     updateTeacher(teacher) {
       return request({
         url: `/eduservice/teacher/updateTeacher`,
         method: "put",
         data: teacher
       })
     }
   ```

5. 在页面调用接口进行数据回显

   ```js
   //根据讲师id查询
       getTeacherById(id) {
         teacher
           .getTeacherById(id)
           .then((response) => {
             this.teacher = response.data.teacher;
           })
           .catch((error) => {});
       },
   ```

6. 判断何时调用数据回显

   ```js
   created() {
       //判断路径中是否有id值
         if (this.$route.params && this.$route.params.id) {
           const id = this.$route.params.id;
           this.getTeacherById(id);
         } 
     },
   
         
   ```

7. 修改实现

   1. 在页面调用修改的方法

      ```js
          //修改讲师
          updateTeacher() {
            teacher.updateTeacher(this.teacher).then((response) => {
              this.$message({
                type: "success",
                message: "修改成功!",
              });
              this.$router.push({ path: "/teacher/listTeacher" });
            });
          },
      ```

   2. 判断是修改 还是 增加

      ```js
       saveOrUpdate() {
            //判断是修改还是添加
            //判断路径中是否有id
            if (this.$route.params && this.$route.params.id) {
              //修改
              this.updateTeacher();
            } else {
              //添加
              this.saveTeacher();
            }
          },
      ```

## 七.路由切换问题

问题：第一次点修改 进行数据回显 然后点击添加讲师 进入表单页面：添加讲师页面还是显示修改的回显数据 正确效果应该是表单数据清空



解决方法：

> 进行表单数据清空(可能解决不了)

```js
created(){
//判断路径中是否有id值
      if (this.$route.params && this.$route.params.id) {
        const id = this.$route.params.id;
        this.getTeacherById(id);
      } else {
        //清空表单
        this.teacher = {};
      }
}
```

未能解决的原因：

​	多次路由跳转到同一页面，在页面中的created方法只会执行一次，后面再进行跳转时不会执行



最终解决

```js
created() {
    this.init();
  },
  watch: {
    //监听
    $route(to, from) {
      //路由发生变化 方法就会执行
      this.init();
    },
  },
  methods: {
    //初始化
    init() {
      //判断路径中是否有id值
      if (this.$route.params && this.$route.params.id) {
        const id = this.$route.params.id;
        this.getTeacherById(id);
      } else {
        //清空表单
        this.teacher = {};
      }
    },
}
```

使用 watch 方法进行监听 只要出现路由变化 就会执行init()方法

