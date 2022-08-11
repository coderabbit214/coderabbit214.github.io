# guli-6.5-EasyExcel读操作分类


# guli-6.5-EasyExcel读操作分类

## 一.读取Excel存储后端

1. 添加依赖 easyexcel

   ```properties
    <dependencies>
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>easyexcel</artifactId>
               <version>2.1.1</version>
           </dependency>
   </dependencies>
   ```

2. 使用代码生成器生成课程分类代码

3. 创建实体类和excel的对应关系

   ```java
   package com.guli.eduservice.entity.excel;
   
   
   @Data
   public class SubjectData {
       @ExcelProperty(value = "一级分类",index = 0)
       private String oneSubjectName;
       @ExcelProperty(value = "二级分类",index = 1)
       private String twoSubjectName;
   }
   
   ```

4. controller

   ```java
   package com.guli.eduservice.controller;
   
   @RestController
   @RequestMapping("/eduservice/subject")
   @CrossOrigin
   public class EduSubjectController {
       @Autowired
       private EduSubjectService subjectService;
   
       //添加课程分类
       //获取上传过来的文件，读取文件内容
       @PostMapping("addSubject")
       public R addSubject(MultipartFile file) {
           subjectService.addSubject(file,subjectService);
           return R.ok();
       }
   }
   
   
   ```

5. service

   ```java
   package com.guli.eduservice.service.impl;
   
   @Service
   public class EduSubjectServiceImpl extends ServiceImpl<EduSubjectMapper, EduSubject> implements EduSubjectService {
   
       //添加课程分类
       @Override
       public void addSubject(MultipartFile file,EduSubjectService eduSubjectService) {
           try {
               EasyExcel.read(file.getInputStream(), SubjectData.class,new SubjecExcelListener(eduSubjectService)).sheet().doRead();
           } catch (Exception e) {
               e.printStackTrace();
           }
   
       }
   
     
   }
   
   ```

6. 监听器

   > 在监听器中进行数据存储

   ```java
   package com.guli.eduservice.listener;
   
   public class SubjecExcelListener extends AnalysisEventListener<SubjectData> {
   
       //因为SubjecExcelListener不能交给spring进行管理，需要自己new，不能注入其他对象
       //不能实现数据库操作
       public EduSubjectService subjectService;
   
       public SubjecExcelListener() {
       }
   
       public SubjecExcelListener(EduSubjectService subjectService) {
           this.subjectService = subjectService;
       }
   
       //读取excel内容,一行一行读取
       @Override
       public void invoke(SubjectData data, AnalysisContext analysisContext) {
           if (data == null) {
               throw new GuliException(20001,"文件数据为空");
           }
           //判断一级分类是否重复
           EduSubject eduSubject = this.existOneSubject(data.getOneSubjectName(),subjectService);
           if (eduSubject == null) { //没有相同 进行添加
               eduSubject = new EduSubject();
               eduSubject.setParentId("0");
               eduSubject.setTitle(data.getOneSubjectName());
               subjectService.save(eduSubject);
           }
           //获取一级分类的id值
           String pid = eduSubject.getId();
           //判断二级分类是否重复
           EduSubject eduSubject2 = this.existTwoSubject(data.getTwoSubjectName(),subjectService,pid);
           if (eduSubject2 == null) { //没有相同 进行添加
               eduSubject2 = new EduSubject();
               eduSubject2.setParentId(pid);
               eduSubject2.setTitle(data.getTwoSubjectName());
               subjectService.save(eduSubject2);
           }
       }
       //判断一级分类不能重复添加
       private EduSubject existOneSubject(String name,EduSubjectService subjectService) {
           QueryWrapper<EduSubject> wrapper = new QueryWrapper<>();
           wrapper.eq("title",name);
           wrapper.eq("parent_id","0");
           EduSubject eduSubjectOne = subjectService.getOne(wrapper);
           return eduSubjectOne;
       }
       //判断二级分类不能重复添加
       private EduSubject existTwoSubject(String name,EduSubjectService subjectService,String pid) {
           QueryWrapper<EduSubject> wrapper = new QueryWrapper<>();
           wrapper.eq("title",name);
           wrapper.eq("parent_id",pid);
           EduSubject eduSubjectTwo = subjectService.getOne(wrapper);
           return eduSubjectTwo;
       }
       @Override
       public void doAfterAllAnalysed(AnalysisContext analysisContext) {
   
       }
   }
   
   ```

## 二.前端

1. 添加路由 src/router/index.js

   ```js
    //课程分类管理
     {
       path: '/subject',
       component: Layout,
       redirect: '/subject/list',
       name: '课程分类管理',
       meta: { title: '课程分类管理', icon: 'example' },
       children: [
         {
           path: 'list',
           name: '课程分类列表',
           component: () => import('@/views/edu/subject/list'),
           meta: { title: '课程分类列表', icon: 'table' }
         },
         {
           path: 'save',
           name: '添加课程分类',
           component: () => import('@/views/edu/subject/save'),
           meta: { title: '添加课程分类', icon: 'tree' }
         }
       ]
     },
   ```

2. 添加对应前端页面

   ```html
   <template>
     <div class="app-container">
       <el-form label-width="120px">
           
         <el-form-item label="信息描述">
           <el-tag type="info">excel模版</el-tag>
           <el-tag>
             <i class="el-icon-download" />
             <a :href="'/static/模板.xlsx'">点击下载模版</a>
           </el-tag>
         </el-form-item>
           
         <el-form-item label="选择Excel">
           <el-upload
             ref="upload"
             :auto-upload="false"
             :on-success="fileUploadSuccess"
             :on-error="fileUploadError"
             :disabled="importBtnDisabled"
             :limit="1"
             :action="BASE_API+'/eduservice/subject/addSubject'"
             name="file"
             accept="application/vnd.ms-excel"
           >
             <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
             <el-button
               :loading="loading"
               style="margin-left: 10px;"
               size="small"
               type="success"
               @click="submitUpload"
             >上传到服务器</el-button>
           </el-upload>
         </el-form-item>
       </el-form>
     </div>
   </template>
   
   <script>
   export default {
     data() {
       return {
         BASE_API: process.env.BASE_API, // 接口API地址
         importBtnDisabled: false, // 按钮是否禁用,
         loading: false,
       };
     },
     created() {},
     methods: {
       //点击上传文件到接口
       submitUpload() {
         this.importBtnDisabled = true;
         this.loading = true;
         //js:documet.getElementById("upload").submit();
         this.$refs.upload.submit();
       },
       //上传成功
       fileUploadSuccess() {
           this.loading = false;
           this.$message({
             type: "success",
             message: "添加成功",
           });
           //跳转回列表页面
           this.$router.push({path:"/subject/list"});
       },
       //上传失败
       fileUploadError() {
         this.loading = false;
         this.$message({
           type: "error",
           message: "导入失败",
         });
       },
     },
   };
   </script>
   ```

## 三.列表分级显示

1. 前端路由 页面

   > 重点： data2: [
   >       ],
   >       defaultProps: {
   >         children: "twoSubjects",
   >         label: "title",
   >       },
   >
   > 参数设置
   >
   > 注意数据格式

   ```html
   <template>
     <div class="app-container">
       <el-input v-model="filterText" placeholder="Filter keyword" style="margin-bottom:30px;" />
   
       <el-tree
         ref="tree2"
         :data="data2"
         :props="defaultProps"
         :filter-node-method="filterNode"
         class="filter-tree"
         default-expand-all
       />
     </div>
   </template>
   
   <script>
   import subject from '@/api/edu/subject';
   export default {
     data() {
       return {
         filterText: "",
         data2: [
         ],
         defaultProps: {
           children: "twoSubjects",
           label: "title",
         },
       };
     },
     watch: {
       filterText(val) {
         this.$refs.tree2.filter(val);
       },
     },
     created(){
         this.getAllSubjectList();
     },
   
     methods: {
       filterNode(value, data) {
         if (!value) return true;
         return data.title.toLowerCase().indexOf(value) !== -1;
       },
       getAllSubjectList(){
           subject.getAllSubjectList().then(response => {
               this.data2 = response.data.list;
           }).catch((error) => {
             console.log(error);
           });
       }
     },
   };
   </script>
   
   ```

2. 前端api    src/api/edu/subject.js

   ```js
   import request from '@/utils/request'
   
   export default {
     //查询课程分类
     getAllSubjectList() {
       return request({
         url:"/eduservice/subject/getAllSubject",
         method:"get"
       })
     }
   }
   
   ```

3. 后端创建controller

   ```java
    //课程结构的列表功能
       @GetMapping("getAllSubject")
       public R getAllSubject(){
           List<OneSubject> list = subjectService.getAllOneTwoSubject();
           return R.ok().data("list",list);
       }
   ```

4. 实体类

   - OneSubject

     ```java
     package com.guli.eduservice.entity.subject;
     
     import lombok.Data;
     
     import java.util.ArrayList;
     import java.util.List;
     
     @Data
     public class OneSubject {
         private String id;
         private String title;
         private List<TwoSubject> twoSubjects = new ArrayList<>();
     }
     
     ```

   - TwoSubject

     ```java
     package com.guli.eduservice.entity.subject;
     
     import lombok.Data;
     
     @Data
     public class TwoSubject {
         private String id;
         private String title;
     }
     
     ```

     

5. service

   重点 主要在于数据的格式

   ```java
    @Override
       public List<OneSubject> getAllOneTwoSubject() {
           //1.查询出所有一级分类
           QueryWrapper<EduSubject> queryWrapperOne = new QueryWrapper<>();
           queryWrapperOne.eq("parent_id",0);
           //查询 或者使用 this.list(queryWrapperOne);
           List<EduSubject> oneEduSubjects = baseMapper.selectList(queryWrapperOne);
           //2.查询出所有二级分类
           QueryWrapper<EduSubject> queryWrapperTwo = new QueryWrapper<>();
           queryWrapperTwo.ne("parent_id",0);
           List<EduSubject> twoEduSubjects = baseMapper.selectList(queryWrapperTwo);
           //创建list集合 用于存储最终封装的数据
           List<OneSubject> finalSubject = new ArrayList<>();
           //3.封装一级分类
           for (EduSubject eduSubject:oneEduSubjects) {
               OneSubject oneSubject = new OneSubject();
   //            oneSubject.setId(eduSubject.getId());
   //            oneSubject.setTitle(eduSubject.getTitle());
   
               BeanUtils.copyProperties(eduSubject,oneSubject);
               //4.封装二级分类
               List<TwoSubject> twoSubjects = new ArrayList<>();
               for (EduSubject eduSubject2:twoEduSubjects) {
                   TwoSubject twoSubject = new TwoSubject();
                   if (eduSubject2.getParentId().equals(eduSubject.getId())) {
                       BeanUtils.copyProperties(eduSubject2,twoSubject);
                       twoSubjects.add(twoSubject);
                   }
               }
               oneSubject.setTwoSubjects(twoSubjects);
               finalSubject.add(oneSubject);
           }
   
           return finalSubject;
       }
   ```

   

