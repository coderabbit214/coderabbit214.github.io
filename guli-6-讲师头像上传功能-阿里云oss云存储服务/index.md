# guli-6-讲师头像上传功能-阿里云oss云存储服务


# guli-6-讲师头像上传功能-阿里云oss云存储服务

## 一.控制台使用

1. 打开阿里云控制台 进入oss
2. 创建bucket 生成 endpoint keyid keysecret bucketname

## 二.开发准备

1. 在service创建子模块 service_oss

2. 引入相关依赖

   ```properties
   <dependencies>
   <!--    阿里云oss依赖-->
       <dependency>
           <groupId>com.aliyun.oss</groupId>
           <artifactId>aliyun-sdk-oss</artifactId>
       </dependency>
   </dependencies>
   ```

3. 创建配置文件

   ```yml
   #端口
   server:
     port: 8082
   #服务名
   spring:
     application:
       name: service_oss
   #环境设置 dev,test,prod
     profiles:
       active: dev
   #阿里云oss
   #不同服务 地址不同
   aliyun:
     oss:
       file:
         endpoint: <yourEndpoint>
         keyid: <yourKeyid>
         keysecret: <yourKeysecret>
         #bucke可以在控制台创建 也可以使用java代码创建
         bucketname: <yourBucketname>
   ```

4. 创建启动类

   ```java
   package com.guli.oss;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
   import org.springframework.context.annotation.ComponentScan;
   
   @SpringBootApplication
   @ComponentScan(basePackages = "com.guli")
   public class OssApplication {
       public static void main(String[] args) {
           SpringApplication.run(OssApplication.class,args);
       }
   }
   
   ```

5. 启动 发现报错 

   错误 ：没有配置数据源

   解决方法：第一种：添加数据源配置

   ​					第二种：排除数据源配置自动装配

   > @SpringBootApplication(exclude =DataSourceAutoConfiguration.class)		

   ```java
   package com.guli.oss;
   @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
   @ComponentScan(basePackages = "com.guli")
   public class OssApplication {
       public static void main(String[] args) {
           SpringApplication.run(OssApplication.class,args);
       }
   }
   ```

## 三.上传文件接口实现

1. 创建常量，读取配置文件内容

   ```java
   package com.guli.oss.utils;
   
   import org.springframework.beans.factory.InitializingBean;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   
   //使用InitializingBean接口 当项目启动afterPropertiesSet方法自动执行
   @Component
   public class ConstantPropertiesUtils implements InitializingBean {
       //读取配置文件内容
       @Value("${aliyun.oss.file.endpoint}")
       private String endpoint;
       @Value("${aliyun.oss.file.keyid}")
       private String keyId;
       @Value("${aliyun.oss.file.keysecret}")
       private String keySecret;
       @Value("${aliyun.oss.file.bucketname}")
       private String bucketName;
   
       //定义公开静态常量
       public static String END_POINT;
       public static String KEY_ID;
       public static String KEY_SECRET;
       public static String BUCKET_NAME;
       @Override
       public void afterPropertiesSet() throws Exception {
           END_POINT = endpoint;
           KEY_ID = keyId;
           KEY_SECRET = keySecret;
           BUCKET_NAME = bucketName;
       }
   }
   ```

2. 创建controller,service

   controller

   ```java
   package com.guli.oss.controller;
   
   
   import com.guli.commonutils.R;
   import com.guli.oss.service.OssService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.CrossOrigin;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.multipart.MultipartFile;
   
   @RestController
   @RequestMapping("/eduoss/fileoss")
   @CrossOrigin
   public class OssController {
       @Autowired
       private OssService ossService;
       //上传头像方法
       @PostMapping("uploadOssFile")
       public R uploadOssFile(MultipartFile file) {
           //获取上传文件 MultipartFile
           String url = ossService.uploadFileAvatar(file);
           return R.ok().data("url",url);
       }
   }
   ```

   service

   ```java
   package com.guli.oss.service.impl;
   
   import com.aliyun.oss.OSS;
   import com.aliyun.oss.OSSClientBuilder;
   import com.guli.oss.service.OssService;
   import com.guli.oss.utils.ConstantPropertiesUtils;
   import org.springframework.stereotype.Service;
   import org.springframework.web.multipart.MultipartFile;
   import java.io.InputStream;
   import java.text.SimpleDateFormat;
   import java.util.Date;
   import java.util.UUID;
   
   @Service
   public class OssServiceImpl implements OssService {
       @Override
       public String uploadFileAvatar(MultipartFile file) {
           // Endpoint以杭州为例，其它Region请按实际情况填写。
           String endpoint = ConstantPropertiesUtils.END_POINT;
           // 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
           String accessKeyId = ConstantPropertiesUtils.KEY_ID;
           String accessKeySecret = ConstantPropertiesUtils.KEY_SECRET;
           String bucketName = ConstantPropertiesUtils.BUCKET_NAME;
           // 创建OSSClient实例。
           OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
   
   
           try {
               // 上传文件流。
               InputStream inputStream = file.getInputStream();
               //获取文件名称
               String fileName = file.getOriginalFilename();
               //第一个参数 Bucket名称
               //第二个参数 文件路径和文件名称
               //第三个参数 文件输入流
               ossClient.putObject(bucketName, fileName, inputStream);
               // 关闭OSSClient。
               ossClient.shutdown();
               //上传之后的文件路径返回
               //需要手动拼接
               return "https://"+bucketName+"."+endpoint+"/"+fileName;
   
           } catch (Exception e) {
               e.printStackTrace();
               return null;
           }
   
       }
   }
   
   ```

## 四.接口完善

1. 多次上传文件名重复问题
2. 文件进行分类管理问题

解决代码

>
> ```java
>         //1.在文件名称中添加随机唯一值
>         String uuid = UUID.randomUUID().toString().replaceAll("-","");
>         fileName = uuid+fileName;
>         //2.把文件按照年月日进行分类
>         //获取当前日期
>         Date date = new Date();
>         SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd");
>         String time = format.format(date);
>         //拼接
>         fileName = time+"/"+fileName;
> ```
>

```java
package com.guli.oss.service.impl;

import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.guli.oss.service.OssService;
import com.guli.oss.utils.ConstantPropertiesUtils;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@Service
public class OssServiceImpl implements OssService {
    @Override
    public String uploadFileAvatar(MultipartFile file) {
        // Endpoint以杭州为例，其它Region请按实际情况填写。
        String endpoint = ConstantPropertiesUtils.END_POINT;
        // 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
        String accessKeyId = ConstantPropertiesUtils.KEY_ID;
        String accessKeySecret = ConstantPropertiesUtils.KEY_SECRET;
        String bucketName = ConstantPropertiesUtils.BUCKET_NAME;
        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);


        try {
            // 上传文件流。
            InputStream inputStream = file.getInputStream();
            //获取文件名称
            String fileName = file.getOriginalFilename();
            
            
            //1.在文件名称中添加随机唯一值
            String uuid = UUID.randomUUID().toString().replaceAll("-","");
            fileName = uuid+fileName;
            //2.把文件按照年月日进行分类
            //获取当前日期
            Date date = new Date();
            SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd");
            String time = format.format(date);
            //拼接
            fileName = time+"/"+fileName;
            
            
            //第一个参数 Bucket名称
            //第二个参数 文件路径和文件名称
            //第三个参数 文件输入流
            ossClient.putObject(bucketName, fileName, inputStream);
            // 关闭OSSClient。
            ossClient.shutdown();
            //上传之后的文件路径返回
            //需要手动拼接
            return "https://"+bucketName+"."+endpoint+"/"+fileName;

        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

    }
}

```

## 五.Nginx回顾

功能：

1. 请求转发

2. 负载均衡
3. 动静分离

下载地址：http://nginx.org/en/download.html

使用：cmd命令窗口

```
#在解压的目录下
启动：
nginx.exe

关闭：
nginx.exe -s stop
```

## 六.Nginx配置请求转发

找到配置文件：conf/nginx.conf

```
http {
	...
		server {
		#监听端口
        listen       9001;
        #主机
        server_name  localhost;
		#匹配路径 使用正则表达式
        location ~ /eduservice/ {
			proxy_pass http://localhost:8081;
		}
		location ~ /eduoss/ {
			proxy_pass http://localhost:8082;
		}
    }

	...
}
```

修改前端请求地址

config/dev.env.js

```js
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  // BASE_API: '"https://easy-mock.com/mock/5950a2419adc231f356a6636/vue-admin"',
  BASE_API: '"http://localhost:9001"',
})

```

## 七.上传头像前端整合

1. 组件准备

   > 把ImageCropper和PanThumb组件复制到src/components文件夹中

2. 引入

   ```js
   <script>
   import ImageCropper from '@/components/ImageCropper';
   import PanThumb from '@/components/PanThumb';
   export default {
   components: { ImageCropper, PanThumb },
   ...
   </script>
   ```

3. 页面使用组件

   > 注意修改接口地址
   >
   > :url="BASE_API+'/eduoss/fileoss/uploadOssFile'"

   ```html
   <!-- 讲师头像 -->
         <el-form-item label="讲师头像">
           <!-- 头衔缩略图 -->
           <pan-thumb :image="teacher.avatar" />
           <!-- 文件上传按钮 -->
           <el-button type="primary" icon="el-icon-upload" @click="imagecropperShow=true">更换头像</el-button>
           <!--v-show：是否显示上传组件
           :key：类似于id，如果一个页面多个图片上传控件，可以做区分
           :url：后台上传的url地址
           @close：关闭上传组件
           @crop-upload-success：上传成功后的回调-->
           <image-cropper
             v-show="imagecropperShow"
             :width="300"
             :height="300"
             :key="imagecropperKey"
             :url="BASE_API+'/eduoss/fileoss/uploadOssFile'"
             field="file"
             @close="close"
             @crop-upload-success="cropSuccess"
           />
         </el-form-item>
   .......
   data() {
       return {
         teacher: {
         },
         saveBtnDisabled: false, // 保存按钮是否禁用,
         imagecropperShow: false,//上传弹窗组件是否显示
         imagecropperKey: 0,//上传组件的key值
         BASE_API: process.env.BASE_API,//获取dev.env.js中的BASE_API
       };
     },
   ```

4. 其他方法实现

   ```js
   methods: {
       //关闭上传弹窗
       close() {
         this.imagecropperShow = false;
         //上传组件初始化
         this.imagecropperKey += 1;
       },
       //上传成功的方法
       cropSuccess(data) {
         this.imagecropperShow = false;
         this.teacher.avatar = data.url;
         this.imagecropperKey += 1;
       },
       ......
   ```

   

