# guli-7.5-添加课程


# guli-7.5-添加课程

## 一.使用代码生成器生成相关表代码

```java
strategy.setInclude("edu_course","edu_course_description","edu_chapter","edu_video");
```

## 二.创建vo类封装表单提交的数据

edu_course 和 edu_course_description

```java
package com.guli.eduservice.entity.vo;

import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.math.BigDecimal;

@Data
public class CourseInfoVo {
    private static final long serialVersionUID = 1L;
    @ApiModelProperty(value = "课程ID")
    private String id;
    @ApiModelProperty(value = "课程讲师ID")
    private String teacherId;
    @ApiModelProperty(value = "课程专业ID")
    private String subjectId;
    @ApiModelProperty(value = "课程标题")
    private String title;
    @ApiModelProperty(value = "课程销售价格，设置为0则可免费观看")
    private BigDecimal price;
    @ApiModelProperty(value = "总课时")
    private Integer lessonNum;
    @ApiModelProperty(value = "课程封面图片路径")
    private String cover;
    @ApiModelProperty(value = "课程简介")
    private String description;
}

```

## 三.controller 和 service

controller

```java
package com.guli.eduservice.controller;


import com.guli.commonutils.R;
import com.guli.eduservice.entity.vo.CourseInfoVo;
import com.guli.eduservice.service.EduCourseService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * <p>
 * 课程 前端控制器
 * </p>
 *
 * @author testjava
 * @since 2020-08-07
 */
@RestController
@RequestMapping("/eduservice/course")
@CrossOrigin
public class EduCourseController {
    @Autowired
    private EduCourseService courseService;

    //添加课程基本信息
    @PostMapping("addCourseInfo")
    public R addCourseInfo(@RequestBody CourseInfoVo courseInfoVo) {
        String id = courseService.addCourseInfo(courseInfoVo);
        return R.ok().data("courseId",id);
    }
}


```

service

```java
package com.guli.eduservice.service.impl;

import com.guli.eduservice.entity.EduCourse;
import com.guli.eduservice.entity.EduCourseDescription;
import com.guli.eduservice.entity.vo.CourseInfoVo;
import com.guli.eduservice.mapper.EduCourseMapper;
import com.guli.eduservice.service.EduCourseDescriptionService;
import com.guli.eduservice.service.EduCourseService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.guli.servicebase.exception.GuliException;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * <p>
 * 课程 服务实现类
 * </p>
 *
 * @author testjava
 * @since 2020-08-07
 */
@Service
public class EduCourseServiceImpl extends ServiceImpl<EduCourseMapper, EduCourse> implements EduCourseService {

    @Autowired
    private EduCourseDescriptionService eduCourseDescriptionService;

    @Override
    public String addCourseInfo(CourseInfoVo courseInfoVo) {
        //向课程表添加课程基本信息
        EduCourse eduCourse = new EduCourse();
        BeanUtils.copyProperties(courseInfoVo,eduCourse);
        int insert = baseMapper.insert(eduCourse);
        if (insert == 0) {
            throw new GuliException(20001,"添加课程信息失败");
        }
        //获取添加课程之后的id
        String cid = eduCourse.getId();
        //向课程简介表中添加信息
        EduCourseDescription eduCourseDescription = new EduCourseDescription();
        BeanUtils.copyProperties(courseInfoVo,eduCourseDescription);
        //设置id
        eduCourseDescription.setId(cid);
        eduCourseDescriptionService.save(eduCourseDescription);
        return cid;
    }
}

```

设置实体类id生成策略

```java
public class EduCourseDescription implements Serializable {

    @ApiModelProperty(value = "课程ID")
    @TableId(value = "id", type = IdType.INPUT)
    private String id;
```

## 四.前端

1. 添加路由

   ```js
     //课程管理
     {
       path: '/course',
       component: Layout,
       redirect: '/course/list',
       name: '课程管理',
       meta: {
         title: '课程管理',
         icon: 'example'
       },
       children: [{
           path: 'list',
           name: '课程列表',
           component: () => import('@/views/edu/course/list'),
           meta: {
             title: '课程列表',
             icon: 'table'
           }
         },
         {
           path: 'save',
           name: '添加课程',
           component: () => import('@/views/edu/course/info'),
           meta: {
             title: '添加课程',
             icon: 'tree'
           }
         },
         {
           path: 'info/:id',
           name: 'EduCourseInfoEdit',
           component: () => import('@/views/edu/course/info'),
           meta: {
             title: '编辑课程基本信息',
             noCache: true
           },
           hidden: true
         },
         {
           path: 'chapter/:id',
           name: 'EduCourseChapterEdit',
           component: () => import('@/views/edu/course/chapter'),
           meta: {
             title: '编辑课程大纲',
             noCache: true
           },
           hidden: true
         },
         {
           path: 'publish/:id',
           name: 'EduCoursePublishEdit',
           component: () => import('@/views/edu/course/publish'),
           meta: {
             title: '发布课程',
             noCache: true
           },
           hidden: true
         }
       ]
     },
   ```

2. api/edu/course.js

   ```js
   import request from '@/utils/request'
   
   export default {
     //1.添加课程信息
     addCourseInfo(courseInfoVo) {
       return request({
         url:"/eduservice/course/addCourseInfo",
         method:"post",
         data: courseInfoVo
       })
     }
   }
   
   ```

3. 编写页面 实现接口调用

   ```html
   <template>
     <div class="app-container">
       <h2 style="text-align: center;">发布新课程</h2>
       <el-steps :active="1" process-status="wait" align-center style="marginbottom: 40px;">
         <el-step title="填写课程基本信息" />
         <el-step title="创建课程大纲" />
         <el-step title="提交审核" />
       </el-steps>
   
       <el-form label-width="120px">
         <el-form-item label="课程标题">
           <el-input v-model="courseInfo.title" placeholder=" 示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写" />
         </el-form-item>
   
         <!-- 所属分类 TODO -->
    
   
         <!-- 课程讲师 TODO -->
      
   
         <!-- 课程简介 TODO -->
   
         <el-form-item label="课程简介">
           <el-input v-model="courseInfo.description" placeholder=" " />
         </el-form-item>
   
         <!-- 课程封面 TODO -->
   
   
         <el-form-item label="课程价格">
           <el-input-number
             :min="0"
             v-model="courseInfo.price"
             controls-position="right"
             placeholder="免费课程请设置为0元"
           />元
         </el-form-item>
   
         <el-form-item>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存并下一步</el-button>
         </el-form-item>
       </el-form>
     </div>
   </template>
   <script>
   import course from "@/api/edu/course";
   import teacher from "@/api/edu/teacher";
   import subject from "@/api/edu/subject";
   export default {
     data() {
       return {
         saveBtnDisabled: false, // 保存按钮是否禁用
         courseInfo: {
           title: "",
           subjectId: "", //二级分类id
           subjectParentId: "", //一级分类id
           teacherId: "",
           lessonNum: 0,
           description: "",
           cover: "",
           price: 0,
         },
       };
     },
     created() {
     
     },
     methods: {
       //提交信息
       saveOrUpdate() {
         course.addCourseInfo(this.courseInfo).then((response) => {
           //提示信息
           this.$message({
             type: "success",
             message: "添加课程信息成功",
           });
           console.log(response.data);
           this.$router.push({
             path: "/course/chapter/" + response.data.courseId,
           });
         });
       },
     },
   };
   </script>
   
   ```

4. 讲师下拉列表显示

   > 1. 编写方法调用api查询所有讲师
   > 2. 在created中初始化
   > 3. 显示到页面中

   ```html
   <template>
     <div class="app-container">
       <h2 style="text-align: center;">发布新课程</h2>
       <el-steps :active="1" process-status="wait" align-center style="marginbottom: 40px;">
         <el-step title="填写课程基本信息" />
         <el-step title="创建课程大纲" />
         <el-step title="提交审核" />
       </el-steps>
   
       <el-form label-width="120px">
         <el-form-item label="课程标题">
           <el-input v-model="courseInfo.title" placeholder=" 示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写" />
         </el-form-item>
   
         <!-- 所属分类 TODO -->
   
         <!-- 课程讲师 TODO -->
         <el-form-item label="课程讲师">
           <el-select v-model="courseInfo.teacherId" placeholder="请选择">
             <el-option
               v-for="teacher in teachers"
               :key="teacher.id"
               :label="teacher.name"
               :value="teacher.id"
             />
           </el-select>
         </el-form-item>
         <el-form-item label="总课时">
           <el-input-number
             :min="0"
             v-model="courseInfo.lessonNum"
             controls-position="right"
             placeholder="请填写课程的总课时数"
           />
         </el-form-item>
   
         <!-- 课程简介 TODO -->
   
         <el-form-item label="课程简介">
           <el-input v-model="courseInfo.description" placeholder=" " />
         </el-form-item>
   
         <!-- 课程封面 TODO -->
   
   
         <el-form-item label="课程价格">
           <el-input-number
             :min="0"
             v-model="courseInfo.price"
             controls-position="right"
             placeholder="免费课程请设置为0元"
           />元
         </el-form-item>
   
         <el-form-item>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存并下一步</el-button>
         </el-form-item>
       </el-form>
     </div>
   </template>
   <script>
   import course from "@/api/edu/course";
   import teacher from "@/api/edu/teacher";
   import subject from "@/api/edu/subject";
   export default {
     data() {
       return {
         saveBtnDisabled: false, // 保存按钮是否禁用
         courseInfo: {
           title: "",
           subjectId: "", //二级分类id
           subjectParentId: "", //一级分类id
           teacherId: "",
           lessonNum: 0,
           description: "",
           cover: "",
           price: 0,
         },
         teachers: [],
     },
     created() {
       this.findAll();
     },
     methods: {
       //查询所有讲师
       findAll() {
         teacher.findAll().then((response) => {
           this.teachers = response.data.items;
         });
       },
       //提交信息
       saveOrUpdate() {
         course.addCourseInfo(this.courseInfo).then((response) => {
           //提示信息
           this.$message({
             type: "success",
             message: "添加课程信息成功",
           });
           console.log(response.data);
           this.$router.push({
             path: "/course/chapter/" + response.data.courseId,
           });
         });
       },
     },
   };
   </script>
   
   ```

5. 课程分类二级联动

   > 1. 调用api查询所有一二级分类subject.getAllSubjectList
   >
   > 2. 将一级分类存储到  subjectNestedList: [],中 并显示到页面
   >
   > 3. 给一级分类添加方法 subjectLevelOneChanged 通过一级分类查询对应的二级分类 并显示
   >
   > 4. 有 BUG ：重新选择一级分类后 二级分类不清空
   >
   >    解决：手动清空一级分类的id

   ```html
   <template>
     <div class="app-container">
       <h2 style="text-align: center;">发布新课程</h2>
       <el-steps :active="1" process-status="wait" align-center style="marginbottom: 40px;">
         <el-step title="填写课程基本信息" />
         <el-step title="创建课程大纲" />
         <el-step title="提交审核" />
       </el-steps>
   
       <el-form label-width="120px">
         <el-form-item label="课程标题">
           <el-input v-model="courseInfo.title" placeholder=" 示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写" />
         </el-form-item>
   
         <!-- 所属分类 TODO -->
         <!-- 一级分类 -->
         <el-form-item label="课程类别">
           <el-select
             v-model="courseInfo.subjectParentId"
             placeholder="请选择"
             @change="subjectLevelOneChanged"
           >
             <el-option
               v-for="subject in subjectNestedList"
               :key="subject.id"
               :label="subject.title"
               :value="subject.id"
             />
           </el-select>
           <!-- 二级分类 -->
           <el-select v-model="courseInfo.subjectId" placeholder="二级分类">
             <el-option
               v-for="subject in subjectTwoList"
               :key="subject.id"
               :label="subject.title"
               :value="subject.id"
             />
           </el-select>
         </el-form-item>
   
         <!-- 课程讲师 TODO -->
         <el-form-item label="课程讲师">
           <el-select v-model="courseInfo.teacherId" placeholder="请选择">
             <el-option
               v-for="teacher in teachers"
               :key="teacher.id"
               :label="teacher.name"
               :value="teacher.id"
             />
           </el-select>
         </el-form-item>
         <el-form-item label="总课时">
           <el-input-number
             :min="0"
             v-model="courseInfo.lessonNum"
             controls-position="right"
             placeholder="请填写课程的总课时数"
           />
         </el-form-item>
   
         <!-- 课程简介 TODO -->
   
         <el-form-item label="课程简介">
           <el-input v-model="courseInfo.description" placeholder=" " />
         </el-form-item>
   
         <!-- 课程封面 TODO -->
   
         <el-form-item label="课程价格">
           <el-input-number
             :min="0"
             v-model="courseInfo.price"
             controls-position="right"
             placeholder="免费课程请设置为0元"
           />元
         </el-form-item>
   
         <el-form-item>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存并下一步</el-button>
         </el-form-item>
       </el-form>
     </div>
   </template>
   <script>
   import course from "@/api/edu/course";
   import teacher from "@/api/edu/teacher";
   import subject from "@/api/edu/subject";
   export default {
     data() {
       return {
         saveBtnDisabled: false, // 保存按钮是否禁用
         courseInfo: {
           title: "",
           subjectId: "", //二级分类id
           subjectParentId: "", //一级分类id
           teacherId: "",
           lessonNum: 0,
           description: "",
           cover: "",
           price: 0,
         },
         teachers: [],
         subjectNestedList: [],
         subjectTwoList: [],
   
       };
     },
     created() {
       this.findAll();
       this.getAllSubjectList();
     },
     methods: {
          //当点击某个一级分类显示对应的二级分类
       subjectLevelOneChanged(value) {
         //value就是一级分类id值
         //遍历所有的分类，包含一级和二级
         for (var i = 0; i < this.subjectNestedList.length; i++) {
           //每个一级分类
           var oneSubject = this.subjectNestedList[i];
           //判断：所有一级分类id 和 点击一级分类id是否一样
           if (value === oneSubject.id) {
             //从一级分类获取里面所有的二级分类
             this.subjectTwoList = oneSubject.twoSubjects;
             //把二级分类id值清空
             this.courseInfo.subjectId = "";
           }
         }
       },
       //查询所有一级分类
       getAllSubjectList() {
         subject.getAllSubjectList().then((response) => {
           this.subjectNestedList = response.data.list;
         });
       },
       //查询所有讲师
       findAll() {
         teacher.findAll().then((response) => {
           this.teachers = response.data.items;
         });
       },
       //提交信息
       saveOrUpdate() {
         course.addCourseInfo(this.courseInfo).then((response) => {
           //提示信息
           this.$message({
             type: "success",
             message: "添加课程信息成功",
           });
           console.log(response.data);
           this.$router.push({
             path: "/course/chapter/" + response.data.courseId,
           });
         });
       },
     },
   };
   </script>
   
   ```

6. 封面上传

   > 1. 调用/eduoss/fileoss/uploadOssFile存储图片（OSS）
   > 2. 把路径存到cover中
   > 3. 避免显示问题 给cover默认值

   ```html
   <template>
     <div class="app-container">
       <h2 style="text-align: center;">发布新课程</h2>
       <el-steps :active="1" process-status="wait" align-center style="marginbottom: 40px;">
         <el-step title="填写课程基本信息" />
         <el-step title="创建课程大纲" />
         <el-step title="提交审核" />
       </el-steps>
   
       <el-form label-width="120px">
         ....
   
         <!-- 课程封面 TODO -->
         <!-- 课程封面-->
         <el-form-item label="课程封面">
           <el-upload
             :show-file-list="false"
             :on-success="handleAvatarSuccess"
             :before-upload="beforeAvatarUpload"
             :action="BASE_API+'/eduoss/fileoss/uploadOssFile'"
             class="avatar-uploader"
           >
             <img :src="courseInfo.cover" />
           </el-upload>
         </el-form-item>
   
   
   
   .....
       </el-form>
     </div>
   </template>
   <script>
   import course from "@/api/edu/course";
   import teacher from "@/api/edu/teacher";
   import subject from "@/api/edu/subject";
   export default {
     data() {
       return {
         saveBtnDisabled: false, // 保存按钮是否禁用
         courseInfo: {
           title: "",
           subjectId: "", //二级分类id
           subjectParentId: "", //一级分类id
           teacherId: "",
           lessonNum: 0,
           description: "",
           cover: "/static/IMG_0882.png",
           price: 0,
         },
         teachers: [],
         subjectNestedList: [],
         subjectTwoList: [],
         BASE_API: process.env.BASE_API, //获取dev.env.js中的BASE_API
       };
     },
     created() {
     },
     methods: {
       //上传封面成功调用的方法
       handleAvatarSuccess(res) {
         this.courseInfo.cover = res.data.url;
       },
       //上传之前调用的方法
       beforeAvatarUpload(file) {
         const isJPG = file.type === "image/jpeg";
         const isLt2M = file.size / 1024 / 1024 < 2;
   
         if (!isJPG) {
           this.$message.error("上传头像图片只能是 JPG 格式!");
         }
         if (!isLt2M) {
           this.$message.error("上传头像图片大小不能超过 2MB!");
         }
         return isJPG && isLt2M;
       },
         .......
     },
   };
   </script>
   
   ```

7. 修改课程简介为富文本编辑器Tinymce

   1. 引入组件Tinymce

   2. 将脚本库复制到项目的static目录下

   3. 配置html变量

      > 在 guli-admin/build/webpack.dev.conf.js 中添加配置 使在html页面中可是使用这里定义的BASE_URL变量

      ```js
      new HtmlWebpackPlugin({
      ......,
      templateParameters: {
      BASE_URL: config.dev.assetsPublicPath + config.dev.assetsSubDirectory
       }
      })
      ```

   4. 引入js脚本

      > 在index.html 中引入js脚本

      ```html
      <script src=<%= BASE_URL %>/tinymce4.7.5/tinymce.min.js></script>
      <script src=<%= BASE_URL %>/tinymce4.7.5/langs/zh_CN.js></script>
      ```

   5. 在使用界面引入Tinymce

      ```js
      import Tinymce from '@/components/Tinymce'
      export default {
      components: { Tinymce },
      ......
      }
      ```

   6. 组件模板

      ```html
      <!-- 课程简介-->
      <el-form-item label="课程简介">
      <tinymce :height="300" v-model="courseInfo.description"/>
      </el-form-item>
      ```

   7. 组件样式

      ```html
      <style scoped>
      .tinymce-container {
      line-height: 29px;
      }
      </style>
      ```

   8. 图片的base64编码 

      Tinymce中的图片上传功能直接存储的是图片的base64编码，因此无需图片服务器


## 五.课程大纲功能

1. 后端

   - 创建两个实体类 章节和小节 章节中包括多个小节

     ChapterVo.java

     ```java
     package com.guli.eduservice.entity.chapter;
     
     import lombok.Data;
     
     import java.util.ArrayList;
     import java.util.List;
     
     @Data
     public class ChapterVo {
         private String id;
         private String title;
         //小结
         private List<VideoVo> videoVos = new ArrayList<>();
     }
     
     ```

     VideoVo.java

     ```java
     package com.guli.eduservice.entity.chapter;
     
     import lombok.Data;
     
     @Data
     public class VideoVo {
         private String id;
         private String title;
     }
     
     ```

   - 编写封装代码

     Controller

     ```java
     package com.guli.eduservice.controller;
     
     
     import com.guli.commonutils.R;
     import com.guli.eduservice.entity.chapter.ChapterVo;
     import com.guli.eduservice.service.EduChapterService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.*;
     
     import java.util.List;
     
     @RestController
     @RequestMapping("/eduservice/chapter")
     @CrossOrigin
     public class EduChapterController {
         //
         @Autowired
         private EduChapterService eduChapterService;
     
         //课程大纲列表，根据课程id进行查询
         @GetMapping("getChapterById/{courseId}")
         public R getChapterById(@PathVariable("courseId") String courseId){
             List<ChapterVo> chapterVos = eduChapterService.getChapterById(courseId);
             return R.ok().data("allChapter",chapterVos);
         }
     }
     ```

     service

     ```java
     package com.guli.eduservice.service.impl;
     
     import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
     import com.guli.eduservice.entity.EduChapter;
     import com.guli.eduservice.entity.EduVideo;
     import com.guli.eduservice.entity.chapter.ChapterVo;
     import com.guli.eduservice.entity.chapter.VideoVo;
     import com.guli.eduservice.mapper.EduChapterMapper;
     import com.guli.eduservice.service.EduChapterService;
     import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
     import com.guli.eduservice.service.EduVideoService;
     import org.springframework.beans.BeanUtils;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Service;
     
     import java.util.ArrayList;
     import java.util.List;
     
     @Service
     public class EduChapterServiceImpl extends ServiceImpl<EduChapterMapper, EduChapter> implements EduChapterService {
     
         @Autowired
         private EduVideoService eduVideoService;
         //课程大纲列表，根据id查询
         @Override
         public List<ChapterVo> getChapterById(String courseId) {
             //1.根据课程id查询课程中所有章节
             QueryWrapper<EduChapter> eduChapterQueryWrapper = new QueryWrapper<>();
             eduChapterQueryWrapper.eq("course_id",courseId);
             List<EduChapter> eduChapters = baseMapper.selectList(eduChapterQueryWrapper);
     
             //2.根据课程id查询课程中所有小节
             QueryWrapper<EduVideo> eduVideoQueryWrapper = new QueryWrapper<>();
             eduVideoQueryWrapper.eq("course_id",courseId);
             List<EduVideo> eduVideos = eduVideoService.list(eduVideoQueryWrapper);
             //3.遍历章节list进行封装
             List<ChapterVo> chapterVos = new ArrayList<>();
             for (EduChapter eduChapter:eduChapters) {
                 ChapterVo chapterVo = new ChapterVo();
                 BeanUtils.copyProperties(eduChapter,chapterVo);
     
                 //4.遍历小节list进行封装
                 List<VideoVo> videoVos = new ArrayList<>();
                 for (EduVideo eduVideo:eduVideos) {
                     //判断小节中的chapterid和章节中的id是否相同
                     if(eduVideo.getChapterId().equals(eduChapter.getId())) {
                         VideoVo videoVo = new VideoVo();
                         BeanUtils.copyProperties(eduVideo,videoVo);
                         videoVos.add(videoVo);
                     }
                 }
                 //把封装的小结放到章节中去
                 chapterVo.setVideoVos(videoVos);
                 chapterVos.add(chapterVo);
             }
     
             return chapterVos;
         }
     }
     
     ```

2. 前端

   - api  chapter.js

     ```js
     import request from '@/utils/request'
     export default {
       getChapterById(courseId) {
         return request({
           url: `/eduservice/chapter/getChapterById/${courseId}`,
           method: 'get'
         })
       }
     }
     ```

   - 页面

     ```html
     <template>
       <div class="app-container">
         <h2 style="text-align: center;">发布新课程</h2>
         <el-steps :active="2" process-status="wait" align-center style="marginbottom: 40px;">
           <el-step title="填写课程基本信息" />
           <el-step title="创建课程大纲" />
           <el-step title="提交审核" />
         </el-steps>
         <el-button type="text">添加章节</el-button>
         <!-- 章节 -->
         <ul class="chanpterList">
           <li v-for="chapter in chapterNestedList" :key="chapter.id">
             <p>
               {{ chapter.title }}
               <span class="acts">
                 <el-button type="text">添加课时</el-button>
                 <el-button style type="text">编辑</el-button>
                 <el-button type="text">删除</el-button>
               </span>
             </p>
             <!-- 视频 -->
             <ul class="chanpterList videoList">
               <li v-for="video in chapter.videoVos" :key="video.id">
                 <p>
                   {{ video.title }}
                   <span class="acts">
                     <el-button type="text">编辑</el-button>
                     <el-button type="text">删除</el-button>
                   </span>
                 </p>
               </li>
             </ul>
           </li>
         </ul>
         <div>
           <el-button @click="previous">上一步</el-button>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
         </div>
       </div>
     </template>
     <script>
     import chapter from "@/api/edu/chapter";
     export default {
       data() {
         return {
           saveBtnDisabled: false, // 保存按钮是否禁用
           courseId: "", // 所属课程章节嵌套课时列表
           chapterNestedList: [], //
         };
       },
       created() {
         if (this.$route.params && this.$route.params.id) {
           this.courseId = this.$route.params.id;
           // 根据id获取课程基本信息
           this.getChapterById();
         }
       },
       methods: {
         getChapterById() {
           chapter.getChapterById(this.courseId).then((response) => {
             this.chapterNestedList = response.data.allChapter;
             console.log(this.chapterNestedList);
           });
         },
         previous() {
           console.log("previous");
           this.$router.push({ path: "/course/info/"+this.courseId });
         },
         next() {
           console.log("next");
           this.$router.push({ path: "/course/publish/"+this.courseId });
         },
       },
     };
     </script>
     <style scoped>
     .chanpterList {
       position: relative;
       list-style: none;
       margin: 0;
       padding: 0;
     }
     .chanpterList li {
       position: relative;
     }
     .chanpterList p {
       float: left;
       font-size: 20px;
       margin: 10px 0;
       padding: 10px;
       height: 70px;
       line-height: 50px;
       width: 100%;
       border: 1px solid #ddd;
     }
     .chanpterList .acts {
       float: right;
       font-size: 14px;
     }
     .videoList {
       padding-left: 50px;
     }
     .videoList p {
       float: left;
       font-size: 14px;
       margin: 10px 0;
       padding: 10px;
       height: 50px;
       line-height: 30px;
       width: 100%;
       border: 1px dotted #ddd;
     }
     </style>
     ```

## 六.课程修改功能

1. 后端

   > 两个功能 
   >
   > 1.根据课程id查询课程的基本信息
   >
   > 2.根据课程id修改课程的基本信息

   - EduCourseController

     ```java
     //根据课程id查询课程的基本信息
         @GetMapping("getCourseInfo/{courseId}")
         public R getCourseInfo(@PathVariable("courseId") String courseId) {
             CourseInfoVo courseInfoVo = courseService.getCourseInfo(courseId);
             return R.ok().data("courseInfoVo",courseInfoVo);
         }
         //根据课程id修改课程的基本信息
         @PostMapping("updateCourseInfo")
         public R updateCourseInfo(@RequestBody CourseInfoVo courseInfoVo) {
             courseService.updateCourseInfo(courseInfoVo);
             return R.ok();
         }
     ```

   - service

     ```java
     //根据课程id查询课程信息
         @Override
         public CourseInfoVo getCourseInfo(String courseId) {
             //1.查询课程表
             EduCourse eduCourse = baseMapper.selectById(courseId);
             //2.查询描述表
             EduCourseDescription eduCourseDescription = eduCourseDescriptionService.getById(courseId);
             //3.封装
             CourseInfoVo courseInfoVo = new CourseInfoVo();
             BeanUtils.copyProperties(eduCourse,courseInfoVo);
             BeanUtils.copyProperties(eduCourseDescription,courseInfoVo);
             return courseInfoVo;
         }
     
         @Override
         public void updateCourseInfo(CourseInfoVo courseInfoVo) {
             //1.修改课程表
             EduCourse eduCourse = new EduCourse();
             BeanUtils.copyProperties(courseInfoVo,eduCourse);
             int i = baseMapper.updateById(eduCourse);
             if (i != 0) {
                 //2.修改简介表
                 EduCourseDescription eduCourseDescription = new EduCourseDescription();
                 BeanUtils.copyProperties(courseInfoVo,eduCourseDescription);
                 eduCourseDescriptionService.updateById(eduCourseDescription);
             } else {
                 throw new GuliException(20001,"引入失败");
             }
     
         }
     ```

2. 前端

   - api course.js

     ```js
      //2.根据课程id查询课程的基本信息
       getCourseInfo(courseId) {
         return request({
           url:"/eduservice/course/getCourseInfo/"+courseId,
           method:"get"
         })
       },
       //3.根据课程id修改课程的基本信息
       updateCourseInfo(courseInfoVo) {
         return request({
           url:"/eduservice/course/updateCourseInfo",
           method:"post",
           data: courseInfoVo
         })
       }
     ```

   - 修改 chapter页面，路径跳转

     ```js
     previous() {
           this.$router.push({ path: "/course/info/"+this.courseId });
         },
     ```

   - 在info页面进行数据回显

     ```js
     //获取路由id值
           if (this.$route.params && this.$route.params.id) {
             this.courseId = this.$route.params.id;
             //调用根据id查询课程的方法
             this.getInfo();
           } else {
             //初始化所有讲师
             this.findAll();
             //初始化一级分类
             this.getAllSubjectList();
           }
     ```

     getInfo 解决了二级分类数据不显示的问题

     ```js
     getInfo() {
           course.getCourseInfo(this.courseId).then((response) => {
             this.courseInfo = response.data.courseInfoVo;
             //1.查询出所有分类
             subject.getAllSubjectList().then((response) => {
               //2.获取所有一级分类
               this.subjectNestedList = response.data.list;
               //3.把所有一级分类数组进行遍历，比较当前courseInfo里面的一级分类id和所有一级分类id
               for (var i = 0; this.subjectNestedList.length; i++) {
                 //获取每个一级分类
                 var oneSubject = this.subjectNestedList[i];
                 //比较当前courseInfo里面的一级分类id和所有一级分类id
                 if (this.courseInfo.subjectParentId == oneSubject.id) {
                   //获取一级分类中的所有二级分类
                   this.subjectTwoList = oneSubject.twoSubjects;
                 }
               }
             });
             //初始化所有讲师
             this.findAll();
           });
         },
     ```

     


