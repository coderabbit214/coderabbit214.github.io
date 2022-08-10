# guli-NUXT


# guli-NUXT

## 一.NUXT环境搭建

目录结构

- assets:一般放项目使用静态资源，比如css，js，img
- components:放项目使用相关组件
- layouts:定义网页布局方式
- pages:项目页面
- nuxt.config.js

nuxt页面加载过程

- default.vue

  ```
  头信息
  nuxt标签  <-  <ifrcam>引入    index.vue
  尾信息
  ```

nuxt 路由

- 第一种：固定路由

  ```html
   <router-link to="/course" tag="li" active-class="current">
                <a>课程</a>
              </router-link>
  ```

- 第二种：动态路由

  ```
  每次生成路由地址不同
  
  NUXT的动态路由是以下划线开头的vue文件，参数名为下划线后边的文件名
  ```

## 二.后端接口

1. 创建 service_cms 模块

2. 创建配置文件

   ```properties
   #数据库配置
   spring.datasource.url=jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.datasource.username=root
   spring.datasource.password=123456
   
   ##mybatis日志
   #mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
   #
   ##设置日志级别
   #logging.level.root=info
   
   #环境设置:dev,test,prod
   spring.profiles.active=dev
   
   #服务端口
   server.port=8081
   
   #服务名
   spring.application.name=service-edu
   
   #设置时间格式
   spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
   spring.jackson.time-zone=GMT+8
   
   #配置mapper xml文件的路径
   mybatis-plus.mapper-locations=classpath:com/guli/eduservice/mapper/xml/*.xml
   
   #nacos服务地址
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   
   #开启熔断机制
   feign.okhttp.enabled=true
   
   #设置hystrix超时时间
   hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=6000
   ```

3. 创建数据库表并生成相应代码

4. controller

   巨幕：

   ```java
   package com.guli.educms.controller;
   
   
   import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
   import com.guli.commonutils.R;
   import com.guli.educms.entity.CrmBanner;
   import com.guli.educms.service.CrmBannerService;
   import io.swagger.annotations.ApiOperation;
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   
   /**
    * <p>
    * 首页banner表 前端控制器
    * </p>
    *
    * @author testjava
    * @since 2020-09-19
    */
   @RestController
   @RequestMapping("/educms/banneradmin")
   @CrossOrigin
   public class BannerAdminController {
       @Autowired
       private CrmBannerService bannerService;
   
       //1 分页查询banner
       @GetMapping("pageBanner/{page}/{limit}")
       public R pageBanner(@PathVariable long page, @PathVariable long limit) {
           Page<CrmBanner> pageBanner = new Page<>(page,limit);
           bannerService.page(pageBanner,null);
           return R.ok().data("items",pageBanner.getRecords()).data("total",pageBanner.getTotal());
       }
   
       //2 添加banner
       @PostMapping("addBanner")
       public R addBanner(@RequestBody CrmBanner crmBanner) {
           bannerService.save(crmBanner);
           return R.ok();
       }
   
       @ApiOperation(value = "获取Banner")
       @GetMapping("get/{id}")
       public R get(@PathVariable String id) {
           CrmBanner banner = bannerService.getById(id);
           return R.ok().data("item", banner);
       }
   
       @ApiOperation(value = "修改Banner")
       @PutMapping("update")
       public R updateById(@RequestBody CrmBanner banner) {
           bannerService.updateById(banner);
           return R.ok();
       }
   
       @ApiOperation(value = "删除Banner")
       @DeleteMapping("remove/{id}")
       public R remove(@PathVariable String id) {
           bannerService.removeById(id);
           return R.ok();
       }
   }
   
   
   ```

   ```java
   package com.guli.educms.controller;
   
   
   import com.guli.commonutils.R;
   import com.guli.educms.entity.CrmBanner;
   import com.guli.educms.service.CrmBannerService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.CrossOrigin;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.List;
   
   /**
    * <p>
    * 首页banner表 前端控制器
    * </p>
    *
    * @author testjava
    * @since 2020-09-19
    */
   @RestController
   @RequestMapping("/educms/bannerfront")
   @CrossOrigin
   public class BannerFrontController {
       @Autowired
       private CrmBannerService bannerService;
   
       //查询所有banner
       @GetMapping("getAllBanner")
       public R getAllBanner() {
           List<CrmBanner> list = bannerService.selectAllBanner();
           return R.ok().data("list",list);
       }
   }
   
   
   ```

   老师，课程：

   ```java
   package com.guli.eduservice.controller.front;
   
   import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
   import com.guli.commonutils.R;
   import com.guli.eduservice.entity.EduChapter;
   import com.guli.eduservice.entity.EduCourse;
   import com.guli.eduservice.entity.EduTeacher;
   import com.guli.eduservice.service.EduCourseService;
   import com.guli.eduservice.service.EduTeacherService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.CrossOrigin;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.List;
   
   @RestController
   @RequestMapping("/eduservice/indexfront")
   @CrossOrigin
   public class IndexFrontController {
   
       @Autowired
       private EduCourseService eduCourseService;
   
       @Autowired
       private EduTeacherService eduTeacherService;
   
   
       @GetMapping("index")
       public R index() {
           //查询前八条课程
           QueryWrapper<EduCourse> wrapper = new QueryWrapper<>();
           wrapper.orderByDesc("id");
           wrapper.last("limit 8");
           List<EduCourse> edulist = eduCourseService.list(wrapper);
   
           //查询前四名讲师
           QueryWrapper<EduTeacher> wrapperTeacher = new QueryWrapper<>();
           wrapperTeacher.orderByDesc("id");
           wrapperTeacher.last("limit 4");
           List<EduTeacher> teacherlist = eduTeacherService.list(wrapperTeacher);
           return R.ok().data("eduList",edulist).data("teacherList",teacherlist);
       }
   }
   
   ```

## 三.前端页面

1. 封装axios

   - 引入axios

   - 创建utils/request.js

     ```js
     import axios from 'axios'
     // 创建axios实例
     const service = axios.create({
       baseURL: 'http://localhost:9001', // api的base_url
       timeout: 20000 // 请求超时时间
     })
     export default service
     ```

2. api

   banner

   ```js
   import request from '@/utils/request'
   
   export default {
       //查询前两条banner数据
     getListBanner() {
       return request({
         url: '/educms/bannerfront/getAllBanner',
         method: 'get'
       })
     }
   }
   ```

   index

   ```js
   import request from '@/utils/request'
   
   export default {
       //查询热门课程和名师
     getIndexData() {
       return request({
         url: '/eduservice/indexfront/index',
         method: 'get'
       })
     }
   }
   ```

3. 页面

   - default

     ```html
     <template>
       <div class="in-wrap">
         <!-- 公共头引入 -->
         <header id="header">
           <section class="container">
             <h1 id="logo">
               <a href="#" title="谷粒学院">
                 <img src="~/assets/img/logo.png" width="100%" alt="谷粒学院">
               </a>
             </h1>
             <div class="h-r-nsl">
               <ul class="nav">
                 <router-link to="/" tag="li" active-class="current" exact>
                   <a>首页</a>
                 </router-link>
                 <router-link to="/course" tag="li" active-class="current">
                   <a>课程</a>
                 </router-link>
                 <router-link to="/teacher" tag="li" active-class="current">
                   <a>名师</a>
                 </router-link>
                 <router-link to="/article" tag="li" active-class="current">
                   <a>文章</a>
                 </router-link>
                 <router-link to="/qa" tag="li" active-class="current">
                   <a>问答</a>
                 </router-link>
               </ul>
               <!-- / nav -->
               <ul class="h-r-login">
                 <li id="no-login">
                   <a href="/sing_in" title="登录">
                     <em class="icon18 login-icon">&nbsp;</em>
                     <span class="vam ml5">登录</span>
                   </a>
                   |
                   <a href="/sign_up" title="注册">
                     <span class="vam ml5">注册</span>
                   </a>
                 </li>
                 <li class="mr10 undis" id="is-login-one">
                   <a href="#" title="消息" id="headerMsgCountId">
                     <em class="icon18 news-icon">&nbsp;</em>
                   </a>
                   <q class="red-point" style="display: none">&nbsp;</q>
                 </li>
                 <li class="h-r-user undis" id="is-login-two">
                   <a href="#" title>
                     <img
                       src="~/assets/img/avatar-boy.gif"
                       width="30"
                       height="30"
                       class="vam picImg"
                       alt
                     >
                     <span class="vam disIb" id="userName"></span>
                   </a>
                   <a href="javascript:void(0)" title="退出" onclick="exit();" class="ml5">退出</a>
                 </li>
                 <!-- /未登录显示第1 li；登录后显示第2，3 li -->
               </ul>
               <aside class="h-r-search">
                 <form action="#" method="post">
                   <label class="h-r-s-box">
                     <input type="text" placeholder="输入你想学的课程" name="queryCourse.courseName" value>
                     <button type="submit" class="s-btn">
                       <em class="icon18">&nbsp;</em>
                     </button>
                   </label>
                 </form>
               </aside>
             </div>
             <aside class="mw-nav-btn">
               <div class="mw-nav-icon"></div>
             </aside>
             <div class="clear"></div>
           </section>
         </header>
         <!-- /公共头引入 -->
           
         <nuxt/>
     
         <!-- 公共底引入 -->
         <footer id="footer">
           <section class="container">
             <div class>
               <h4 class="hLh30">
                 <span class="fsize18 f-fM c-999">友情链接</span>
               </h4>
               <ul class="of flink-list">
                 <li>
                   <a href="http://www.atguigu.com/" title="尚硅谷" target="_blank">尚硅谷</a>
                 </li>
               </ul>
               <div class="clear"></div>
             </div>
             <div class="b-foot">
               <section class="fl col-7">
                 <section class="mr20">
                   <section class="b-f-link">
                     <a href="#" title="关于我们" target="_blank">关于我们</a>|
                     <a href="#" title="联系我们" target="_blank">联系我们</a>|
                     <a href="#" title="帮助中心" target="_blank">帮助中心</a>|
                     <a href="#" title="资源下载" target="_blank">资源下载</a>|
                     <span>服务热线：010-56253825(北京) 0755-85293825(深圳)</span>
                     <span>Email：info@atguigu.com</span>
                   </section>
                   <section class="b-f-link mt10">
                     <span>©2018课程版权均归谷粒学院所有 京ICP备17055252号</span>
                   </section>
                 </section>
               </section>
               <aside class="fl col-3 tac mt15">
                 <section class="gf-tx">
                   <span>
                     <img src="~/assets/img/wx-icon.png" alt>
                   </span>
                 </section>
                 <section class="gf-tx">
                   <span>
                     <img src="~/assets/img/wb-icon.png" alt>
                   </span>
                 </section>
               </aside>
               <div class="clear"></div>
             </div>
           </section>
         </footer>
         <!-- /公共底引入 -->
       </div>
     </template>
     <script>
     import "~/assets/css/reset.css";
     import "~/assets/css/theme.css";
     import "~/assets/css/global.css";
     import "~/assets/css/web.css";
     
     export default {};
     </script>
     ```

   - pages/index

     ```html
     <template>
       
       <div>
         <!-- 幻灯片 开始 -->
       <div v-swiper:mySwiper="swiperOption">
           <div class="swiper-wrapper">
     
               <div v-for="banner in bannerList" :key="banner.id" class="swiper-slide" style="background: #040B1B;">
                   <a target="_blank" :href="banner.linkUrl">
                       <img :src="banner.imageUrl" :alt="banner.title">
                   </a>
               </div>
           </div>
           <div class="swiper-pagination swiper-pagination-white"></div>
           <div class="swiper-button-prev swiper-button-white" slot="button-prev"></div>
           <div class="swiper-button-next swiper-button-white" slot="button-next"></div>
       </div>
       <!-- 幻灯片 结束 -->
         
          <div id="aCoursesList">
           <!-- 网校课程 开始 -->
           <div>
             <section class="container">
               <header class="comm-title">
                 <h2 class="tac">
                   <span class="c-333">热门课程</span>
                 </h2>
               </header>
               <div>
                 <article class="comm-course-list">
                   <ul class="of" id="bna">
                     <li v-for="course in eduList" :key="course.id">
                       <div class="cc-l-wrap">
                         <section class="course-img">
                           <img
                             :src="course.cover"
                             class="img-responsive"
                             :alt="course.title"
                           >
                           <div class="cc-mask">
                             <a href="#" title="开始学习" class="comm-btn c-btn-1">开始学习</a>
                           </div>
                         </section>
                         <h3 class="hLh30 txtOf mt10">
                           <a href="#" :title="course.title" class="course-title fsize18 c-333">{{course.title}}</a>
                         </h3>
                         <section class="mt10 hLh20 of">
                           <span class="fr jgTag bg-green" v-if="Number(course.price) === 0">
                             <i class="c-fff fsize12 f-fA">免费</i>
                           </span>
                           <span class="fl jgAttr c-ccc f-fA">
                             <i class="c-999 f-fA">9634人学习</i>
                             |
                             <i class="c-999 f-fA">9634评论</i>
                           </span>
                         </section>
                       </div>
                     </li>
                    
                   </ul>
                   <div class="clear"></div>
                 </article>
                 <section class="tac pt20">
                   <a href="#" title="全部课程" class="comm-btn c-btn-2">全部课程</a>
                 </section>
               </div>
             </section>
           </div>
           <!-- /网校课程 结束 -->
           <!-- 网校名师 开始 -->
           <div>
             <section class="container">
               <header class="comm-title">
                 <h2 class="tac">
                   <span class="c-333">名师大咖</span>
                 </h2>
               </header>
               <div>
                 <article class="i-teacher-list">
                   <ul class="of">
                     <li v-for="teacher in teacherList" :key="teacher.id">
                       <section class="i-teach-wrap">
                         <div class="i-teach-pic">
                           <a href="/teacher/1" :title="teacher.name">
                             <img :alt="teacher.name" :src="teacher.avatar">
                           </a>
                         </div>
                         <div class="mt10 hLh30 txtOf tac">
                           <a href="/teacher/1" :title="teacher.name" class="fsize18 c-666">{{teacher.name}}</a>
                         </div>
                         <div class="hLh30 txtOf tac">
                           <span class="fsize14 c-999">{{teacher.career}}</span>
                         </div>
                         <div class="mt15 i-q-txt">
                           <p
                             class="c-999 f-fA"
                           >{{teacher.intro}}</p>
                         </div>
                       </section>
                     </li>
                     
                   </ul>
                   <div class="clear"></div>
                 </article>
                 <section class="tac pt20">
                   <a href="#" title="全部讲师" class="comm-btn c-btn-2">全部讲师</a>
                 </section>
               </div>
             </section>
           </div>
           <!-- /网校名师 结束 -->
         </div>
       </div>
     </template>
     
     <script>
     import banner from '@/api/banner'
     import index from '@/api/index'
     
     export default {
       data () {
         return {
     
           swiperOption: {
             //配置分页
             pagination: {
               el: '.swiper-pagination'//分页的dom节点
             },
             //配置导航
             navigation: {
               nextEl: '.swiper-button-next',//下一页dom节点
               prevEl: '.swiper-button-prev'//前一页dom节点
             }
           },
           //banner数组
           bannerList:[],
           eduList:[],
           teacherList:[]
         }
       },
       created() {
         //调用查询banner的方法
         this.getBannerList()
         //调用查询热门课程和名师的方法
         this.getHotCourseTeacher()
       },
       methods:{
         //查询热门课程和名师
         getHotCourseTeacher() {
           index.getIndexData()
             .then(response => {
               this.eduList = response.data.data.eduList
               this.teacherList = response.data.data.teacherList
             })
         },
         //查询banner数据
         getBannerList() {
           banner.getListBanner()
             .then(response => {
               this.bannerList = response.data.data.list
             })
         }
       }
     }
     </script>
     ```

     s


