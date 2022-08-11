# guli-8.5-章节,小节,发布功能


# guli-8.5-章节,小节,发布功能

# 一.章节，小节

1. 后端

   - controller

     > 章节

     ```java
     package com.guli.eduservice.controller;
     
     import com.guli.commonutils.R;
     import com.guli.eduservice.entity.EduChapter;
     import com.guli.eduservice.entity.chapter.ChapterVo;
     import com.guli.eduservice.service.EduChapterService;
     import com.guli.servicebase.exception.GuliException;
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
         //添加章节
         @PostMapping("addChapter")
         public R addChapter(@RequestBody EduChapter eduChapter) {
             eduChapterService.save(eduChapter);
             return R.ok();
         }
         //根据id查询
         @GetMapping("getChapterInfo/{chapterId}")
         public R getChapterInfo(@PathVariable String chapterId) {
             EduChapter eduChapter = eduChapterService.getById(chapterId);
             return R.ok().data("eduChapter",eduChapter);
         }
         //修改章节
         @PostMapping("updateChapterInfo")
         public R updateChapterInfo(@RequestBody EduChapter eduChapter) {
             eduChapterService.updateById(eduChapter);
             return R.ok();
         }
         //删除章节
         @DeleteMapping("{chapterId}")
         public R deleteChapter(@PathVariable String chapterId) {
             boolean flag = eduChapterService.deleteChapter(chapterId);
             if (flag) {
                 return R.ok();
             } else {
                 return R.error();
             }
     
         }
     }
     
     
     ```

     > 小节

     ```java
     package com.guli.eduservice.controller;
     
     
     import com.guli.commonutils.R;
     import com.guli.eduservice.entity.EduVideo;
     import com.guli.eduservice.service.EduVideoService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.*;
     
     /**
      * <p>
      * 课程视频 前端控制器
      * </p>
      *
      * @author testjava
      * @since 2020-08-07
      */
     @RestController
     @RequestMapping("/eduservice/video")
     @CrossOrigin //解决跨域
     public class EduVideoController {
         @Autowired
         private EduVideoService videoService;
     
         //添加小节
         @PostMapping("addVideo")
         public R addVideo(@RequestBody EduVideo eduVideo) {
             videoService.save(eduVideo);
             return R.ok();
         }
         //删除小节
         @DeleteMapping("deleteVideo/{id}")
         public R deleteVideo(@PathVariable String id) {
             videoService.removeById(id);
             return R.ok();
         }
         //根据id查询小节
         @GetMapping("getVideo/{id}")
         public R getVideo(@PathVariable String id) {
             EduVideo video = videoService.getById(id);
             return R.ok().data("video",video);
         }
         //修改小节
         @PostMapping("updateVideo")
         public R updateVideo(@RequestBody EduVideo eduVideo) {
             videoService.updateById(eduVideo);
             return R.ok();
         }
     }
     ```

   - service

     > 章节

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
     import com.guli.servicebase.exception.GuliException;
     import org.springframework.beans.BeanUtils;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Service;
     
     import java.util.ArrayList;
     import java.util.List;
     
     /**
      * <p>
      * 课程 服务实现类
      * </p>
      *
      * @author testjava
      * @since 2020-08-07
      */
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
     
         //删除章节的方法
         @Override
         public boolean deleteChapter(String chapterId) {
             //根据chapterId查询小节表，如果查询到数据，不进行删除
             QueryWrapper<EduVideo> eduVideoQueryWrapper = new QueryWrapper<>();
             eduVideoQueryWrapper.eq("chapter_id",chapterId);
             int videos = eduVideoService.count(eduVideoQueryWrapper);
             if (videos > 0) {//存在数据
                 throw new GuliException(20001,"无法删除");
             } else {
                 //没有小节可以进行删除
                 int result = baseMapper.deleteById(chapterId);
                 return result > 0;
             }
     
         }
     }
     
     ```

2. 前端

   - api

     > 章节

     ```js
     import request from '@/utils/request'
     export default {
       //1.根据课程id获取章节和小节数据列表
       getChapterById(courseId) {
         return request({
           url: `/eduservice/chapter/getChapterById/${courseId}`,
           method: 'get'
         })
       },
       //2.添加章节
       addChapter(eduChapter) {
         return request({
           url: `/eduservice/chapter/addChapter`,
           method: 'post',
           data:eduChapter
         })
       },
       //3.修改
       updateChapterInfo(eduChapter) {
         return request({
           url: `/eduservice/chapter/updateChapterInfo`,
           method: 'post',
           data:eduChapter
         })
       },
       //4.根据id查询
       getChapterInfo(courseId) {
         return request({
           url: `/eduservice/chapter/getChapterInfo/${courseId}`,
           method: 'get'
         })
       },
       //5.删除
       deleteChapter(courseId) {
         return request({
           url: `/eduservice/chapter/${courseId}`,
           method: 'delete'
         })
       },
     }
     
     ```

     > 小节

     ```js
     import request from '@/utils/request'
     export default {
     
         //添加小节
         addVideo(video) {
             return request({
                 url: '/eduservice/video/addVideo',
                 method: 'post',
                 data: video
               })
         },
         
         //删除小节
         deleteVideo(id) {
             return request({
                 url: '/eduservice/video/deleteVideo/'+id,
                 method: 'delete'
               })
         },
         //根据id查询小节
         getVideo(id) {
             return request({
                 url: '/eduservice/video/getVideo/'+id,
                 method: 'get'
               })
         },
         //修改小节
         updateVideo(video) {
             return request({
                 url: '/eduservice/video/updateVideo',
                 method: 'post',
                 data: video
               })
         },
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
         <el-button @click="openChapter()" type="text">添加章节</el-button>
         <!-- 添加和修改章节表单 -->
         <el-dialog :visible.sync="dialogChapterFormVisible" title="添加章节">
           <el-form :model="chapter" label-width="120px">
             <el-form-item label="章节标题">
               <el-input v-model="chapter.title" />
             </el-form-item>
             <el-form-item label="章节排序">
               <el-input-number v-model="chapter.sort" :min="0" controlsposition="right" />
             </el-form-item>
           </el-form>
           <div slot="footer" class="dialog-footer">
             <el-button @click="dialogChapterFormVisible = false">取 消</el-button>
             <el-button type="primary" @click="saveOrUpdate">确 定</el-button>
           </div>
         </el-dialog>
         <!-- 添加和修改课时表单 -->
         <el-dialog :visible.sync="dialogVideoFormVisible" title="添加课时">
           <el-form :model="video" label-width="120px">
             <el-form-item label="课时标题">
               <el-input v-model="video.title" />
             </el-form-item>
             <el-form-item label="课时排序">
               <el-input-number v-model="video.sort" :min="0" controls-position="right" />
             </el-form-item>
             <el-form-item label="是否免费">
               <el-radio-group v-model="video.free">
                 <el-radio :label="true">免费</el-radio>
                 <el-radio :label="false">默认</el-radio>
               </el-radio-group>
             </el-form-item>
             <el-form-item label="上传视频">
               <!-- TODO -->
             </el-form-item>
           </el-form>
           <div slot="footer" class="dialog-footer">
             <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
             <el-button  type="primary" @click="saveOrUpdateVideo">确 定</el-button>
           </div>
         </el-dialog>
         <!-- 章节 -->
         <ul class="chanpterList">
           <li v-for="chapter in chapterNestedList" :key="chapter.id">
             <p>
               {{ chapter.title }}
               <span class="acts">
                 <el-button type="text" @click="openVideo(chapter.id)">添加课时</el-button>
                 <el-button style type="text" @click="openEditChapter(chapter.id)">编辑</el-button>
                 <el-button type="text" @click="removeChapter(chapter.id)">删除</el-button>
               </span>
             </p>
             <!-- 视频 -->
             <ul class="chanpterList videoList">
               <li v-for="video in chapter.videoVos" :key="video.id">
                 <p>
                   {{ video.title }}
                   <span class="acts">
                     <el-button style type="text" @click="openEditVedio(video.id)">编辑</el-button>
                     <el-button type="text" @click="removeVideo(video.id)">删除</el-button>
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
     import video from "@/api/edu/video";
     export default {
       data() {
         return {
           saveBtnDisabled: false, // 保存按钮是否禁用
           courseId: "", // 所属课程章节嵌套课时列表
           chapterNestedList: [], //
           //封装章节数据
           chapter: {
             title: "",
             sort: 0,
           },
           video: {
             title: "",
             sort: 0,
             free: 0,
             videoSourceId: "",
           },
           dialogChapterFormVisible: false, //章节弹框
           dialogVideoFormVisible: false, //小节弹框
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
         //==============================小节操作====================================
         //修改时数据回显
         openEditVedio(id) {
           alert(0);
           //调用接口
           video.getVideo(id).then((response) => {
             alert(1);
             this.video = response.data.video;
           });
           //弹框
           this.dialogVideoFormVisible = true;
         },
         //修改小节
         updateEditVedio() {
            video.updateVideo(this.video).then((response) => {
             //关闭弹窗
             this.dialogVideoFormVisible = false;
             //提示信息
             this.$message({
               type: "success",
               message: "修改章节成功！",
             });
             this.video.id = null;
             //刷新页面
             this.getChapterById();
           });
         },
         //删除小节
         removeVideo(id) {
           this.$confirm("此操作将删除小节, 是否继续?", "提示", {
             confirmButtonText: "确定",
             cancelButtonText: "取消",
             type: "warning",
           }).then(() => {
             //点击确定，执行then方法
             //调用删除的方法
             video.deleteVideo(id).then((response) => {
               //删除成功
               //提示信息
               this.$message({
                 type: "success",
                 message: "删除小节成功!",
               });
               //刷新页面
               this.getChapterById();
             });
           }); //点击取消，执行catch方法
         },
         //添加小节弹框的方法
         openVideo(chapterId) {
           //弹框
           this.dialogVideoFormVisible = true;
           //设置章节id
           this.video.chapterId = chapterId;
     
           //表单数据清空
           this.video.title = '';
           this.video.sort = 0;
           this.video.free = 0;
           this.video.videoSourceId = '';
         },
         //添加小节
         addVideo() {
           //设置课程id
           this.video.courseId = this.courseId;
           video.addVideo(this.video).then((response) => {
             //关闭弹框
             this.dialogVideoFormVisible = false;
             //提示
             this.$message({
               type: "success",
               message: "添加小节成功!",
             });
             //刷新页面
             this.getChapterById();
           });
         },
         saveOrUpdateVideo() {
           if (this.video.id) {
             this.updateEditVedio();
           } else {
             this.addVideo();
           }
     
         },
     
         //==================章节操作=========================
         //删除章节
         removeChapter(chapterId) {
           this.$confirm("此操作将永久删除该章节记录, 是否继续?", "提示", {
             confirmButtonText: "确定",
             cancelButtonText: "取消",
             type: "warning",
           }).then(() => {
             chapter
               .deleteChapter(chapterId)
               .then((response) => {
                 //删除成功
                 //提示信息
                 this.$message({
                   type: "success",
                   message: "删除成功!",
                 });
                 //刷新页面
                 this.getChapterById();
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
         //修改章节弹框数据回显
         openEditChapter(chapterId) {
           //弹框
           this.dialogChapterFormVisible = true;
           //调用接口
           chapter.getChapterInfo(chapterId).then((response) => {
             this.chapter = response.data.eduChapter;
           });
         },
         //弹出添加章节页面
         openChapter() {
           //弹框
           this.dialogChapterFormVisible = true;
           //表单数据清空
           this.chapter.title = "";
           this.chapter.sort = 0;
         },
         //修改章节
         updateChapter() {
           chapter.updateChapterInfo(this.chapter).then((response) => {
             //关闭弹窗
             this.dialogChapterFormVisible = false;
             //提示信息
             this.$message({
               type: "success",
               message: "修改章节成功！",
             });
             //刷新页面
             this.getChapterById();
           });
         },
         //添加章节
         addChapter() {
           //设置课程id到chapter中
           this.chapter.courseId = this.courseId;
           chapter.addChapter(this.chapter).then((response) => {
             //关闭弹窗
             this.dialogChapterFormVisible = false;
             //提示信息
             this.$message({
               type: "success",
               message: "添加章节成功！",
             });
             //刷新页面
             this.getChapterById();
           });
         },
         saveOrUpdate() {
           if (this.chapter.id) {
             this.updateChapter();
           } else {
             this.addChapter();
           }
         },
         //根据id查询章节和小节
         getChapterById() {
           chapter.getChapterById(this.courseId).then((response) => {
             this.chapterNestedList = response.data.allChapter;
             console.log(this.chapterNestedList);
           });
         },
         previous() {
           console.log("previous");
           this.$router.push({ path: "/course/info/" + this.courseId });
         },
         next() {
           console.log("next");
           this.$router.push({ path: "/course/publish/" + this.courseId });
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

# 二.最终发布(xml重点,多表联查)

1. 后端 course

   - 编写接收数据的实体类

     ```java
     package com.guli.eduservice.entity.vo;
     
     import lombok.Data;
     
     @Data
     public class CoursePublishVo {
         private String id;
         private String title;
         private String cover;
         private Integer lessonNum;
         private String subjectLevelOne;
      private String subjectLevelTwo;
         private String teacherName;
      private String price;//只用于显示
     }
     
     ```
   
   - mapper
   
     ```java
     package com.guli.eduservice.mapper;
     
     import com.guli.eduservice.entity.EduCourse;
     import com.baomidou.mybatisplus.core.mapper.BaseMapper;
     import com.guli.eduservice.entity.vo.CoursePublishVo;
     
     public interface EduCourseMapper extends BaseMapper<EduCourse> {
         public CoursePublishVo getPublishCourseInfo(String courseId);
     }
     
     ```
   
- xml
  
  ```xml
     <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.guli.eduservice.mapper.EduCourseMapper">
         <!--sql语句：根据课程id查询课程确认信息-->
         <select id="getPublishCourseInfo" resultType="com.guli.eduservice.entity.vo.CoursePublishVo">
             SELECT ec.id,ec.title,ec.price,ec.lesson_num AS lessonNum,ec.cover,
                    et.name AS teacherName,
                    es1.title AS subjectLevelOne,
                    es2.title AS subjectLevelTwo
             FROM edu_course ec LEFT OUTER JOIN edu_course_description ecd ON ec.id=ecd.id
                                LEFT OUTER JOIN edu_teacher et ON ec.teacher_id=et.id
                                LEFT OUTER JOIN edu_subject es1 ON ec.subject_parent_id=es1.id
                        LEFT OUTER JOIN edu_subject es2 ON ec.subject_id=es2.id
             WHERE ec.id=#{courseId}
         </select>
     </mapper>
     
  ```
  
- 配置问题

  1. service/pom.xml

        ```xml
        <project>
        ...
        <!--解决xml文件加载问题-->
         <build>
                <resources>
                 <resource>
                        <directory>src/main/java</directory>
                        <includes>
                            <include>**/*.xml</include>
                        </includes>
                        <filtering>false</filtering>
                    </resource>
                </resources>
            </build>
        </project>
        ```

     2. src/main/resources/application.properties

        ```properties
        
        #配置mapper xml文件的路径
        mybatis-plus.mapper-locations=classpath:com/guli/eduservice/mapper/xml/*.xml
        ```

   - service

     ```java
         @Override
         public CoursePublishVo getPublishCourseInfo(String courseId) {
             CoursePublishVo coursePublishVo = baseMapper.getPublishCourseInfo(courseId);
             return coursePublishVo;
         }
     ```

   - controller

     ```java
         //根据课程id查询课程的全部信息
         @GetMapping("getPublishCourseInfo/{courseId}")
         public R getPublishCourseInfo(@PathVariable String courseId) {
             CoursePublishVo coursePublishVo = courseService.getPublishCourseInfo(courseId);
             return R.ok().data("coursePublishVo",coursePublishVo);
         }
         //课程最终发布
         //修改课程状态
         @GetMapping("publishCourse/{id}")
         public R publishCourse(@PathVariable String id) {
             EduCourse eduCourse = new EduCourse();
             eduCourse.setId(id);
             eduCourse.setStatus("Normal");
             courseService.updateById(eduCourse);
             return R.ok();
         }
     ```


2. 前端 course

   - api

     ```js
       //4.课程确认信息显示
       getPublishCourseInfo(courseId) {
         return request({
           url:"/eduservice/course/getPublishCourseInfo/"+courseId,
           method:"get"
         })
       },
       //4.课程发布
       publishCourse(courseId) {
         return request({
           url:"/eduservice/course/publishCourse/"+courseId,
           method:"get"
         })
       },
     ```

   - 页面

     ```html
     <template>
       <div class="app-container">
         <h2 style="text-align: center;">发布新课程</h2>
     
         <el-steps :active="3" process-status="wait" align-center style="margin-bottom: 40px;">
           <el-step title="填写课程基本信息" />
           <el-step title="创建课程大纲" />
           <el-step title="发布课程" />
         </el-steps>
     
         <div class="ccInfo">
           <img :src="coursePublish.cover" />
           <div class="main">
             <h2>{{ coursePublish.title }}</h2>
             <p class="gray">
               <span>共{{ coursePublish.lessonNum }}课时</span>
             </p>
             <p>
               <span>所属分类：{{ coursePublish.subjectLevelOne }} — {{ coursePublish.subjectLevelTwo }}</span>
             </p>
             <p>课程讲师：{{ coursePublish.teacherName }}</p>
             <h3 class="red">￥{{ coursePublish.price }}</h3>
           </div>
         </div>
     
         <div>
           <el-button @click="previous">返回修改</el-button>
           <el-button :disabled="saveBtnDisabled" type="primary" @click="publish">发布课程</el-button>
         </div>
       </div>
     </template>
     
     <script>
     import course from "@/api/edu/course";
     export default {
       data() {
         return {
           courseId: "",
           saveBtnDisabled: false, // 保存按钮是否禁用
           coursePublish: {},
         };
       },
       created() {
         //获取路由课程id值
         if (this.$route.params && this.$route.params.id) {
           this.courseId = this.$route.params.id;
           //调用接口中的方法
           this.getPublishCourseInfo();
         }
       },
       methods: {
         //查询
         getPublishCourseInfo() {
           course.getPublishCourseInfo(this.courseId).then((response) => {
             this.coursePublish = response.data.coursePublishVo;
             console.log(333);
           });
         },
         previous() {
           console.log("previous");
           this.$router.push({ path: "/course/chapter/" + this.courseId });
         },
         publish() {
           course.publishCourse(this.courseId).then((response) => {
             //提示
             this.$message({
               type: "success",
               message: "课程发布成功!",
             });
             //跳转课程列表页面
             this.$router.push({ path: "/course/list" });
           });
         },
       },
     };
     </script>
     <style scoped>
     .ccInfo {
       background: #f5f5f5;
       padding: 20px;
       overflow: hidden;
       border: 1px dashed #ddd;
       margin-bottom: 40px;
       position: relative;
     }
     .ccInfo img {
       background: #d6d6d6;
       width: 500px;
       height: 278px;
       display: block;
       float: left;
       border: none;
     }
     .ccInfo .main {
       margin-left: 520px;
     }
     
     .ccInfo .main h2 {
       font-size: 28px;
       margin-bottom: 30px;
       line-height: 1;
       font-weight: normal;
     }
     .ccInfo .main p {
       margin-bottom: 10px;
       word-wrap: break-word;
       line-height: 24px;
       max-height: 48px;
       overflow: hidden;
     }
     
     .ccInfo .main p {
       margin-bottom: 10px;
       word-wrap: break-word;
       line-height: 24px;
       max-height: 48px;
       overflow: hidden;
     }
     .ccInfo .main h3 {
       left: 540px;
       bottom: 20px;
       line-height: 1;
       font-size: 28px;
       color: #d32f24;
       font-weight: normal;
       position: absolute;
     }
     </style>
     ```

     

   

