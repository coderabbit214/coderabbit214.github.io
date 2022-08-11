# guli-9.5-阿里云视频


# guli-9.5-阿里云视频

# 一.测试

## 1.阿里云控制台

https://help.aliyun.com/product/29932.html?spm=a2c4g.11186623.6.84.3b5873fboAolJQ

## 2.功能测试

1. 创建service_vod模块

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-core</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun.oss</groupId>
           <artifactId>aliyun-sdk-oss</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-vod</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-sdk-vod-upload</artifactId>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
       <dependency>
           <groupId>org.json</groupId>
           <artifactId>json</artifactId>
       </dependency>
       <dependency>
           <groupId>com.google.code.gson</groupId>
           <artifactId>gson</artifactId>
       </dependency>
   
       <dependency>
           <groupId>joda-time</groupId>
           <artifactId>joda-time</artifactId>
       </dependency>
   </dependencies>
   ```

2. 在test中创建InitObject类

   > 初始化操作

   ```java
   package com.guli.vodtest;
   
   import com.aliyun.oss.ClientException;
   import com.aliyuncs.DefaultAcsClient;
   import com.aliyuncs.profile.DefaultProfile;
   
   public class InitObject {
       public static DefaultAcsClient initVodClient(String accessKeyId, String accessKeySecret) throws ClientException {
           String regionId = "cn-shanghai";  // 点播服务接入区域
           DefaultProfile profile = DefaultProfile.getProfile(regionId, accessKeyId, accessKeySecret);
           DefaultAcsClient client = new DefaultAcsClient(profile);
           return client;
       }
   }
   
   ```

3. 根据视频id获取视频播放地址

   ```java
    //根据视频id获取视频播放地址
       public static void getPalyUrl() throws ClientException {
           //1.根据视频id获取视频播放地址
           //创建初始化对象
           DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient("accessKeyId","accessKeySecret");
           //创建获取视频地址request和response
           GetPlayInfoRequest request = new GetPlayInfoRequest();
           GetPlayInfoResponse response = new GetPlayInfoResponse();
           //向request对象里面设置视频id
           request.setVideoId("678bb315bf974ef0a23f2136ab155f13");
           //调用初始化对象里面的方法，传递request，获取数据
           response = defaultAcsClient.getAcsResponse(request);
   
           List<GetPlayInfoResponse.PlayInfo> playInfoList = response.getPlayInfoList();
           //播放地址
           for (GetPlayInfoResponse.PlayInfo playInfo:playInfoList) {
               System.out.println("播放地址："+playInfo.getPlayURL());
           }
           //Base信息
           System.out.println("视频名称："+response.getVideoBase().getTitle());
       }
   
   ```

4. 根据视频id获取视频凭证

   ```java
       //根据视频id获取视频凭证
       public static void getPlayAuth() throws ClientException {
           //根据视频id获取视频凭证
           //创建初始化对象
           DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient("accessKeyId", "accessKeySecret");
           //创建获取视频地址request和response
           GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();
           GetVideoPlayAuthResponse response = new GetVideoPlayAuthResponse();
           //向request对象里面设置视频id
           request.setVideoId("678bb315bf974ef0a23f2136ab155f13");
           //调用初始化对象里面的方法，传递request，获取视频凭证
           response = defaultAcsClient.getAcsResponse(request);
           //获取凭证
           System.out.println("视频凭证："+response.getPlayAuth());
       }
   ```

## 3.视频上传

1. 未开源jar包安装

   在jar包目录中进入cmd运行

   ```
   mvn install:install-file -DgroupId=com.aliyun -DartifactId=aliyun-sdk-vod-upload -Dversion=1.4.11 -Dpackaging=jar -Dfile=aliyun-java-vod-upload-1.4.11.jar
   ```

2. ```java
    public static void main(String[] args) throws ClientException {
           String accessKeyId = "accessKeyId";
           String accessKeySecret = "accessKeySecret";
           //上传后文件名称
           String title = "微博2333.mp4";
           //本地文件路径和名称
           String fileName = "E:\\视频存储\\pr储存\\19.12.10火炬女神\\微博.mp4";
           //本地上传视频
           UploadVideoRequest request = new UploadVideoRequest(accessKeyId, accessKeySecret, title, fileName);
           /* 可指定分片上传时每个分片的大小，默认为2M字节 */
           request.setPartSize(2 * 1024 * 1024L);
           /* 可指定分片上传时的并发线程数，默认为1，(注：该配置会占用服务器CPU资源，需根据服务器情况指定）*/
           request.setTaskNum(1);
           UploadVideoImpl uploader = new UploadVideoImpl();
           UploadVideoResponse response = uploader.uploadVideo(request);
           System.out.print("RequestId=" + response.getRequestId() + "\n");  //请求视频点播服务的请求ID
           if (response.isSuccess()) {
               System.out.print("VideoId=" + response.getVideoId() + "\n");
           } else {
               /* 如果设置回调URL无效，不影响视频上传，可以返回VideoId同时会返回错误码。其他情况上传失败时，VideoId为空，此时需要根据返回错误码分析具体错误原因 */
               System.out.print("VideoId=" + response.getVideoId() + "\n");
               System.out.print("ErrorCode=" + response.getCode() + "\n");
               System.out.print("ErrorMessage=" + response.getMessage() + "\n");
           }
       }
   ```

# 二.小节视频功能

## 1.后端

1. 引入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-core</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun.oss</groupId>
           <artifactId>aliyun-sdk-oss</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-vod</artifactId>
       </dependency>
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-sdk-vod-upload</artifactId>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
       <dependency>
           <groupId>org.json</groupId>
           <artifactId>json</artifactId>
       </dependency>
       <dependency>
           <groupId>com.google.code.gson</groupId>
           <artifactId>gson</artifactId>
       </dependency>
   
       <dependency>
           <groupId>joda-time</groupId>
           <artifactId>joda-time</artifactId>
       </dependency>
   </dependencies>
   ```

2. 配置文件

   ```yml
   #服务端口
   server:
     port: 8083
   #服务名
   spring:
     application:
       name: service-vod
   #环境设置
     profiles:
       active: dev
     servlet:
       multipart:
         max-file-size: 1024MB
         max-request-size: 1024MB
   #阿里云vod
   aliyun:
     oss:
       file:
         keyid: keyid
         keysecret: keysecret
   
   
   ```

3. 创建启动类

   ```java
   package com.guli.vod;
   
   @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
   @ComponentScan(basePackages = {"com.guli"})
   public class VodApplication {
       public static void main(String[] args) {
           SpringApplication.run(VodApplication.class,args);
       }
   }
   
   ```

4. controller

   ```java
   package com.guli.vod.controller;
   
   import com.aliyuncs.DefaultAcsClient;
   import com.aliyuncs.exceptions.ClientException;
   import com.aliyuncs.vod.model.v20170321.DeleteVideoRequest;
   import com.guli.commonutils.R;
   import com.guli.servicebase.exception.GuliException;
   import com.guli.vod.service.VodService;
   import com.guli.vod.utils.ConstantVodUtils;
   import com.guli.vod.utils.InitVodCilent;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   import org.springframework.web.multipart.MultipartFile;
   
   @RestController
   @RequestMapping("eduvod/video")
   @CrossOrigin
   public class VodController {
       @Autowired
       private VodService vodService;
   
       //上传视频到阿里云的方法
       @PostMapping("uploadAlyiVideo")
       public R uploadAlyiVideo(MultipartFile file) {
           String videoId = vodService.uploadVideoAly(file);
           return R.ok().data("videoId",videoId);
       }
       //根据视频id删除视频
       @DeleteMapping("removeAlyVideo/{id}")
       public R removeAlyVideo(@PathVariable String id) {
   
           try {
               //初始化对象
               DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient(ConstantVodUtils.KEY_ID, ConstantVodUtils.KEY_SECRET);
               //创建删除视频request对象
               DeleteVideoRequest request = new DeleteVideoRequest();
               //向request中设置id
               request.setVideoIds(id);
               defaultAcsClient.getAcsResponse(request);
               return R.ok();
           } catch (ClientException e) {
               e.printStackTrace();
               throw new GuliException(20001,"删除失败");
           }
   
       }
   }
   
   ```

5. service

   ```java
   package com.guli.vod.service.impl;
   
   import com.aliyun.vod.upload.impl.UploadVideoImpl;
   import com.aliyun.vod.upload.req.UploadStreamRequest;
   import com.aliyun.vod.upload.resp.UploadStreamResponse;
   import com.guli.vod.service.VodService;
   import com.guli.vod.utils.ConstantVodUtils;
   import org.springframework.stereotype.Service;
   import org.springframework.web.multipart.MultipartFile;
   
   import java.io.IOException;
   import java.io.InputStream;
   
   @Service
   public class VodServiceImpl implements VodService {
       @Override
       public String uploadVideoAly(MultipartFile file) {
   
           try {
               //fileName上传文件原始名称
               String fileName = file.getOriginalFilename();
               //title上传文件显示名称
               String title = fileName.substring(0,fileName.lastIndexOf("."));
               //inputStream上传文件输入流
               InputStream inputStream = file.getInputStream();
   
               UploadStreamRequest request = new UploadStreamRequest(ConstantVodUtils.KEY_ID, ConstantVodUtils.KEY_SECRET, title, fileName, inputStream);
               UploadVideoImpl uploader = new UploadVideoImpl();
               UploadStreamResponse response = uploader.uploadStream(request);
   
               return response.getVideoId();
           } catch (IOException e) {
               e.printStackTrace();
               return null;
           }
   
       }
   }
   
   ```

6. 文件大小设置 application

   ```yml
   spring:
     servlet:
       multipart:
         max-file-size: 1024MB
         max-request-size: 1024MB
   ```

## 2.Nginx

1. 配置8083端口规则

2. 设置文件上传大小设置

   ```
   http {
   	#文件大小配置
   	client_max_body_size 1204m;
   	
   	server {
           listen       9001;
           server_name  localhost;
   
           location ~ /eduservice/ {
   			proxy_pass http://localhost:8081;
   		}
   		location ~ /eduoss/ {
   			proxy_pass http://localhost:8082;
   		}
   		location ~ /eduvod/ {
   			proxy_pass http://localhost:8083;
   		}
       }
   }
   ```

## 3.前端

### 1.上传视频

1. 表单

   ```html
           <el-form-item label="上传视频">
             <el-upload
               :on-success="handleVodUploadSuccess"
               :on-remove="handleVodRemove"
               :before-remove="beforeVodRemove"
               :on-exceed="handleUploadExceed"
               :file-list="fileList"
               :action="BASE_API+'/eduvod/video/uploadAlyiVideo'"
               :limit="1"
               class="upload-demo"
             >
               <el-button size="small" type="primary">上传视频</el-button>
               <el-tooltip placement="right-end">
                 <div slot="content">
                   最大支持1G，
                   <br />支持3GP、ASF、AVI、DAT、DV、FLV、F4V、
                   <br />GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、
                   <br />MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、
                   <br />SWF、TS、VOB、WMV、WEBM 等视频格式上传
                 </div>
                 <i class="el-icon-question" />
               </el-tooltip>
             </el-upload>
           </el-form-item>
   ```

2. js

   ```js
       //上传视频成功的方法
    handleVodUploadSuccess(response,file,fileList) {
         this.video.videoSourceId = response.data.videoId;
         this.video.videoOriginalName = file.name;
       },
       //再次上传
       handleUploadExceed() {
         this.$message.warning("想要重新上传视频，请先删除已上传的视频");
       },
   ```

3. data

   ```js
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
           videoOriginalName:"",//视频名称
         },
         dialogChapterFormVisible: false, //章节弹框
         dialogVideoFormVisible: false, //小节弹框
         fileList: [],//上传文件列表
         BASE_API: process.env.BASE_API,//接口API地址
       };
     },
   ```

### 2.删除

1. api/video

   ```js
    //删除视频
       removeAlyVideo(id) {
           return request({
               url: '/eduvod/video/removeAlyVideo/'+id,
               method: 'delete',
             })
       }
   ```

2. 页面

   ```js
       //删除视频(点击确定)
       handleVodRemove() {
         //调用接口
         video.removeAlyVideo(this.video.videoSourceId).then(response=>{
            this.$message({
             type: "success",
             message: "删除成功！",
           });
           //清空视频
           this.fileList = [];
           //清空video中的视频id和视频名称
           this.videoSourceId = '';
           this.videoOriginalName = '';
         })
       },
       //删除视频之前(点击X)
       beforeVodRemove(file,fileList) {
         return this.$confirm(`确定移除 ${file.name}？`)
       },
   ```

   






