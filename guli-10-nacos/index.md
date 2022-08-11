# guli-10-Nacos


# guli-10-Nacos

> 1. 服务发现和服务健康监测 
> 2. 动态配置服务 
> 3. 动态DNS服务 
> 4. 服务及其元数据管理

## 一.安装与配置

1. [下载](https://github.com/alibaba/nacos/releases)

2. 启动

- Linux/Unix/Mac
   启动命令(standalone代表着单机模式运行，非集群模式)
   启动命令：sh startup.sh -m standalone

   

   Windows
   启动命令：cmd startup.cmd 或者双击startup.cmd运行文件。
   访问：http://localhost:8848/nacos
   用户名密码：nacos/nacos

## 二.服务注册

1. 配置pom依赖

   ```xml
   <!--服务注册-->
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   
   ```

2. 添加服务配置信息

   ```properties
   #nacos服务地址
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   
   #还需要配置
   #服务端口
   server.port=8081
   
   #服务名 不可以使用下划线
   spring.application.name=service-edu
   ```

3. 在启动类上添加注解

   ```
   @EnableDiscoveryClient
   ```

4. 启动客户端微服务，可以在Nacos服务列表中看到被注册的微服务

## 三.服务调用

1. 配置pom依赖

   ```xml
   <!--服务调用-->
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 调用端的启动类添加注解

   ```java
   @EnableFeignClients
   ```

3. 创建包和接口

   >创建client包 
   >
   >@FeignClient注解用于指定从哪个服务中调用功能 ，名称与被调用的服务名保持一致。 
   >
   >@GetMapping注解用于对被调用的微服务进行地址映射,路径要写全。 
   >
   >@PathVariable注解一定要指定参数名称，否则出错 
   >
   >@Component注解防止，在其他位置注入CodClient时idea报错
   >
   >

   ```java
   package com.guli.eduservice.client;
   
   @Component
   @FeignClient("service-vod")
   public interface VodClient {
       //定义调用方法的路径
       //根据视频id删除视频
       @DeleteMapping("/eduvod/video/removeAlyVideo/{id}")
       public R removeAlyVideo(@PathVariable("id") String id);
       //根据list删除多个视频
       @DeleteMapping("/eduvod/video/deleteBatch")
       public R deleteBatch(@RequestParam("videoIdList") List<String> videoIdList);
   }
   ```

## 四.业务

### 1.删除小结的同时删除视频

1. 前置：在vod/controller中

   ```java
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
   ```

2. 在服务中心中注册vod，edu服务

   - pom
   - application
   - 启动类注解

3. 在edu中进行服务调用配置

   - pom
   - 启动类注解

4. 在edu模块创建client包，创建接口，调用vod中的接口

   > @FeignClient注解用于指定从哪个服务中调用功能 ，名称与被调用的服务名保持一致。 
   >
   > @GetMapping注解用于对被调用的微服务进行地址映射,路径要写全。 
   >
   > @PathVariable注解一定要指定参数名称，否则出错 
   >
   > @Component注解防止，在其他位置注入CodClient时idea报错

   ```java
   package com.guli.eduservice.client;
   
   @Component
   @FeignClient("service-vod")
   public interface VodClient {
       //定义调用方法的路径
       //根据视频id删除视频
       @DeleteMapping("/eduvod/video/removeAlyVideo/{id}")
       public R removeAlyVideo(@PathVariable("id") String id);
   }
   
   ```

5. 在controller中调用

   ```java
       //删除小节
       @DeleteMapping("deleteVideo/{id}")
       public R deleteVideo(@PathVariable String id) {
           //删除阿里云视频
           EduVideo eduVideo = videoService.getById(id);
           String videoSourceId = eduVideo.getVideoSourceId();
           if (!StringUtils.isEmpty(videoSourceId)) {
               vodClient.removeAlyVideo(videoSourceId);
           }
           //删除数据库中内容
           videoService.removeById(id);
           return R.ok();
       }
   ```

### 2.删除课程的同时删除多个视频

1. vod/controller

   ```java
       //删除多个阿里云视频
       @DeleteMapping("deleteBatch")
       public R deleteBatch(@RequestParam("videoIdList") List<String> videoIdList) {
           vodService.removeMoreAlyVideo(videoIdList);
           return R.ok();
       }
   ```

2. service

   ```java
   @Override
       public void removeMoreAlyVideo(List<String> videoIdList) {
           try {
               //初始化对象
               DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient(ConstantVodUtils.KEY_ID, ConstantVodUtils.KEY_SECRET);
               //创建删除视频request对象
               DeleteVideoRequest request = new DeleteVideoRequest();
               //拆开列表中的数组1，2，3
               String join = StringUtils.join(videoIdList.toArray(), ",");
               //向request中设置id
               request.setVideoIds(join);
               defaultAcsClient.getAcsResponse(request);
           } catch (ClientException e) {
               e.printStackTrace();
               throw new GuliException(20001,"删除失败");
           }
       }
   ```

3. client

   ```java
   package com.guli.eduservice.client;
   @Component
   @FeignClient("service-vod")
   public interface VodClient {
       //定义调用方法的路径
       //根据视频id删除视频
       @DeleteMapping("/eduvod/video/removeAlyVideo/{id}")
       public R removeAlyVideo(@PathVariable("id") String id);
       //根据list删除多个视频
       @DeleteMapping("/eduvod/video/deleteBatch")
       public R deleteBatch(@RequestParam("videoIdList") List<String> videoIdList);
   }
   
   ```

4. 调用

   ```java
    //删除小结
       @Override
       @Transactional
       public void removeVideoByCourseId(String courseId) {
           //删除阿里云视频
           QueryWrapper<EduVideo> queryWrapper = new QueryWrapper<>();
           queryWrapper.eq("course_id",courseId);
           queryWrapper.select("video_sourse_id");
           List<EduVideo> eduVideos = baseMapper.selectList(queryWrapper);
           List<String> videoIds = new ArrayList<>();
           for(EduVideo eduVideo:eduVideos) {
               String videoSourceId = eduVideo.getVideoSourceId();
               if (!StringUtils.isEmpty(videoSourceId)) {
                   videoIds.add(videoSourceId);
               }
           }
           if (videoIds.size()>0) {
               vodClient.deleteBatch(videoIds);
           }
           //删除数据库中小结本身
           QueryWrapper<EduVideo> eduVideoQueryWrapper = new QueryWrapper<>();
           eduVideoQueryWrapper.eq("course_id",courseId);
           baseMapper.delete(eduVideoQueryWrapper);
       }
   ```

   

