# guli-课程列表(day9)


# guli-课程列表(day9)

# 一.后端 Course

1. controller

   ```java
    //课程列表查询
       @PostMapping("{current}/{limit}")
       public R getCourseList(@PathVariable long current,
                              @PathVariable long limit,
                              @RequestBody(required = false) CourseQuery courseQuery) {
           //1.创建page对象
           Page<EduCourse> pageCourse = new Page<>(current,limit);
           //2.构建Wrapper
           QueryWrapper<EduCourse> queryWrapper = new QueryWrapper<>();
           //3.构建条件
           String title = courseQuery.getTitle();
           String status = courseQuery.getStatus();
           //4.添加条件
           if(!StringUtils.isEmpty(title)) {
               queryWrapper.like("title",title);
           }
           if(!StringUtils.isEmpty(status)) {
               queryWrapper.eq("status",status);
           }
           //5.根据时间排序
           queryWrapper.orderByDesc("gmt_create");
           //6.查询
           courseService.page(pageCourse, queryWrapper);
           long total = pageCourse.getTotal();//总记录数
           List<EduCourse> courses = pageCourse.getRecords();//每页教师数据集合
           return R.ok().data("total",total).data("rows",courses);
       }
    //删除课程
       @DeleteMapping("{courseId}")
       public R deleteCourseById(@PathVariable String courseId) {
           courseService.removeCourse(courseId);
           return R.ok();
       }
   ```

2. service

   ```java
       //课程描述注入
       @Autowired
       private EduCourseDescriptionService eduCourseDescriptionService;
       //注入小结和章节
       @Autowired
       private EduVideoService eduVideoService;
       @Autowired
       private EduChapterService eduChapterService; 
   //删除课程的方法
       @Override
       public void removeCourse(String courseId) {
           //1.删除小结
           eduVideoService.removeVideoByCourseId(courseId);
           //2.删除章节
           eduChapterService.removeChapterByCourseId(courseId);
           //3.删除描述
           eduCourseDescriptionService.removeById(courseId);
           //4.删除本身
           int result = baseMapper.deleteById(courseId);
           if (result == 0) {
               throw new GuliException(20001,"删除失败");
           }
       }
   ```

   - ```java
     //删除小结
         @Override
         public void removeVideoByCourseId(String courseId) {
             QueryWrapper<EduVideo> eduVideoQueryWrapper = new QueryWrapper<>();
             eduVideoQueryWrapper.eq("course_id",courseId);
             baseMapper.delete(eduVideoQueryWrapper);
         }
     ```

   - ```java
     @Override
         public void removeChapterByCourseId(String courseId) {
             QueryWrapper<EduChapter> eduChapterQueryWrapper = new QueryWrapper<>();
             eduChapterQueryWrapper.eq("course_id",courseId);
             baseMapper.delete(eduChapterQueryWrapper);
         }
     ```

# 二.前端 Course

1. api

   ```js
    //5.课程列表
     getCourseList(current, limit, courseQuery) {
       return request({
         url:`/eduservice/course/${current}/${limit}`,
         method:"post",
         data: courseQuery
       })
     },
     //6.删除课程
     deleteCourseById(courseId) {
       return request({
         url:"/eduservice/course/"+courseId,
         method:"delete"
       })
     }
   ```

2. view/list.vue

   ```html
   <template>
     <div class="app-container">
         课程列表
       <!--查询表单-->
       <el-form :inline="true" class="demo-form-inline">
         <el-form-item>
           <el-input v-model="courseQuery.title" placeholder="课程名称" />
         </el-form-item>
         
         <el-form-item>
           <el-select v-model="courseQuery.status" clearable placeholder="课程状态">
             <el-option :value="'Draft'" label="未发布" />
             <el-option :value="'Normal'" label="已发布" />
           </el-select>
         </el-form-item>
   
         <el-button type="primary" icon="el-icon-search" @click="getCourseListPage()">查询</el-button>
         <el-button type="default" @click="resetData()">清空</el-button>
       </el-form>
       <el-table :data="list" element-loading-text="数据加载中" border fit highlight-current-row>
         <el-table-column label="序号" width="70" align="center">
           <template slot-scope="scope">{{ (page - 1) * limit + scope.$index + 1 }}</template>
         </el-table-column>
         <el-table-column prop="title" label="课程名称" width="80" />
         <el-table-column label="课程状态" width="80">
           <template slot-scope="scope">{{ scope.row.status==='Normal'?'已发布':'未发布' }}</template>
         </el-table-column>
         <el-table-column prop="lessonNum" label="课时数" />
         <el-table-column prop="gmtCreate" label="添加时间" width="160" />
         <el-table-column prop="viewCount" label="浏览数量" width="60" />
         <el-table-column label="操作" width="200" align="center">
           <template slot-scope="scope">
             <router-link :to="'/course/updateTeacher/'+scope.row.id">
               <el-button type="primary" size="mini" icon="el-icon-edit">编辑课程的基本信息</el-button>
             </router-link>
             <router-link :to="'/teacher/updateTeacher/'+scope.row.id">
               <el-button type="primary" size="mini" icon="el-icon-edit">编辑课程的大纲</el-button>
             </router-link>
             <el-button
               type="danger"
               size="mini"
               icon="el-icon-delete"
               @click="removeCourseById(scope.row.id)"
             >删除课程信息</el-button>
           </template>
         </el-table-column>
       </el-table>
   
       <el-pagination
         :current-page="page"
         :page-size="limit"
         :total="total"
         style="padding: 30px 0; text-align: center;"
         layout="total, prev, pager, next, jumper"
         @current-change="getCourseListPage"
       /> 
     </div>
   </template>
   
   <script>
   import course from "@/api/edu/course";
   export default {
     data() {
       return {
         list: null, //查询之后接口返回集合
         page: 1, //当前页
         limit: 1, //每页记录数
         total: 0, //总记录数
         courseQuery: {}, //条件封装对象
       };
     },
     created() {
       //页面渲染之前执行，一般用于调用methods中定义的方法
       this.getCourseListPage();
     },
     methods: {
         //删除功能
         removeCourseById(courseId) {
             //删除功能
         this.$confirm("此操作将永久删除该讲师记录, 是否继续?", "提示", {
           confirmButtonText: "确定",
           cancelButtonText: "取消",
           type: "warning",
         }).then(() => {
           course
             .deleteCourseById(courseId)
             .then((response) => {
               //删除成功
               //提示信息
               this.$message({
                 type: "success",
                 message: "删除成功!",
               });
               //回到列表
               this.getCourseListPage();
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
       //页面渲染之后执行，一般用于创建具体方法
       //讲师列表方法
       getCourseListPage(page = 1) {
         this.page = page;
         course
           .getCourseList(this.page, this.limit, this.courseQuery)
           .then((response) => {
             //请求成功
             //response接口返回的数据
             this.list = response.data.rows;
             this.total = response.data.total;
           })
           .catch((error) => {
             console.log(error);
           });
       },
       resetData() {
         //清空的方法
         //表单数据清空
         this.courseQuery = {};
         //查询所有
         this.getCourseListPage();
       },
       
     },
   };
   </script>
   ```

   




