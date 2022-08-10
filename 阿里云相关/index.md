# 阿里云相关


# 阿里云相关

## 短信服务

1. pom

   ```xml
   <dependencies>
   <!-- 阿里云短信依赖 -->
       <!-- json转换工具 -->
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
       </dependency>
       <!-- 阿里云短信依赖核心库 -->
       <dependency>
           <groupId>com.aliyun</groupId>
           <artifactId>aliyun-java-sdk-core</artifactId>
       </dependency>
   </dependencies>
   ```

2. controller

   ```java
   @RestController
   @RequestMapping("/edumsm/msm")
   @CrossOrigin
   public class MsmController {
   
       @Autowired
       private MsmService msmService;
   
       @Autowired
       private RedisTemplate<String,String> redisTemplate;
   
       //发送短信的方法
       @GetMapping("send/{phone}")
       public R sendMsm(@PathVariable("phone") String phone) {
           //从redis中获取验证码，如果获取到直接返回
           String code = redisTemplate.opsForValue().get(phone);
           if (!StringUtils.isEmpty(code)) {
               return R.ok();
           }
           //如果redis获取不到，进行阿里云发送
           //生成随机值，传递阿里云进行发送
           code = RandomUtil.getSixBitRandom();
           Map<String,Object> param = new HashMap<>();
           param.put("code",code);
           //调用service进行短信发送
           boolean isSend = msmService.send(param,phone);
           if (isSend) {
               //发送成功，把发送成功方法放在redis中
               //设置有效时间
               redisTemplate.opsForValue().set(phone,code,5, TimeUnit.MINUTES);
               return R.ok();
           } else {
               return R.error().message("发送短信错误");
           }
   
       }
   }
   
   
   ```

3. service

   ```java
   @Service
   public class MsmServiceImpl implements MsmService {
       //发送短信的方法
       @Override
       public boolean send(Map<String, Object> param, String phone) {
           if(StringUtils.isEmpty(phone)) return false;
         //修改阿里云参数 regionld accessKeyId secret
           DefaultProfile profile = DefaultProfile.getProfile("default","key","secret");
           IAcsClient client = new DefaultAcsClient(profile);
           //设置相关固定参数
           CommonRequest request = new CommonRequest();
           request.setMethod(MethodType.POST);
           request.setDomain("dysmsapi.aliyuncs.com");
           request.setVersion("2017-05-25");
           request.setAction("SendSms");
   
           //设置发送相关参数
           //手机号
           request.putQueryParameter("PhoneNumbers",phone);
           //签名名称 
           request.putQueryParameter("SignName","value");
           //模板code
           request.putQueryParameter("TemplateCode","value");
           //设置验证码 需要转换为JSON
           request.putQueryParameter("TemplateParam", JSONObject.toJSONString(param));
   
           //发送
           try {
               CommonResponse commonResponse = client.getCommonResponse(request);
               boolean success = commonResponse.getHttpResponse().isSuccess();
               return success;
   
           } catch (ClientException e) {
               e.printStackTrace();
               return false;
           }
       }
   }
   ```

## OSS

1. pom

   ```xml
   <dependencies>
   <!--    阿里云oss依赖-->
       <dependency>
           <groupId>com.aliyun.oss</groupId>
           <artifactId>aliyun-sdk-oss</artifactId>
           <version>3.5.0</version>
       </dependency>
   </dependencies>
   ```

2. 配置文件

   ```properties
   aliyun:
     oss:
       file:
         endpoint: <yourEndpoint>
         keyid: <yourKeyid>
         keysecret: <yourKeysecret>
         #bucke可以在控制台创建 也可以使用java代码创建
         bucketname: <yourBucketname>
   ```

3. 创建常量，读取配置文件内容

   ```java
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

4. controller

   ```java
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

5. service

   ```java
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

## 视频点播

1. pom

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

2. 配置

   ```yml
   #阿里云vod
   aliyun:
     oss:
       file:
         keyid: keyid
         keysecret: keysecret
   
   #文件大小设置
   spring:
     servlet:
       multipart:
         max-file-size: 1024MB
         max-request-size: 1024MB
   ```

3. 文件上传

   - controller

     ```java
         @Autowired
         private VodService vodService;
         
         //上传视频到阿里云的方法
         @PostMapping("uploadAlyiVideo")
         public R uploadAlyiVideo(MultipartFile file) {
             String videoId = vodService.uploadVideoAly(file);
             return R.ok().data("videoId",videoId);
         }
     ```

   - service

     ```java
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
     
               //获取视频id
                 return response.getVideoId();
             } catch (IOException e) {
                 e.printStackTrace();
                 return null;
             }
     
         }
     }
     ```

4. 删除

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

5. 获取凭证 播放

   - controller

     ```java
         //根据视频id获取视频凭证
         @GetMapping("getPlayAuth/{id}")
         public R getPlayAuth(@PathVariable String id) throws ClientException {
             //根据视频id获取视频凭证
             //创建初始化对象
             DefaultAcsClient defaultAcsClient = InitVodCilent.initVodClient(ConstantVodUtils.KEY_ID, ConstantVodUtils.KEY_SECRET);
             //创建获取视频地址request和response
             GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();
             GetVideoPlayAuthResponse response = new GetVideoPlayAuthResponse();
             //向request对象里面设置视频id
             request.setVideoId(id);
             //调用初始化对象里面的方法，传递request，获取视频凭证
             response = defaultAcsClient.getAcsResponse(request);
             //获取凭证
                 String playAuth = response.getPlayAuth();
                 return R.ok().data("playAuth",playAuth);
     
     
         }
     ```

   - api/vod.js

     ```js
     import request from '@/utils/request'
     export default {
       getPlayAuth(vid) {
         return request({
           url: `/eduvod/video/getPlayAuth/${vid}`,
           method: 'get'
         })
       }
     
     }
     ```

   - player/_vid.vue

     ```vue
     <template>
       <div>
     
         <!-- 阿里云视频播放器样式 -->
         <link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" >
         <!-- 阿里云视频播放器脚本 -->
         <script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js" />
     
         <!-- 定义播放器dom -->
         <div id="J_prismPlayer" class="prism-player" />
       </div>
     </template>
     <script>
     import vod from '@/api/vod'
     
     export default {
         layout: 'video',//应用video布局
         asyncData({ params, error }) {
            return vod.getPlayAuth(params.vid)
             .then(response => {
                 return { 
                     playAuth: response.data.data.playAuth,
                     vid: params.vid
                 }
             })
         },
         mounted() { //页面渲染之后  created
             new Aliplayer({
                 id: 'J_prismPlayer',
                 vid: this.vid, // 视频id
                 playauth: this.playAuth, // 播放凭证
                 encryptType: '1', // 如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
                 width: '100%',
                 height: '500px',
                 // 以下可选设置
                 // cover: 'http://guli.shop/photo/banner/1525939573202.jpg', // 封面
                 // qualitySort: 'asc', // 清晰度排序
     
                 // mediaType: 'video', // 返回音频还是视频
                 // autoplay: false, // 自动播放
                 // isLive: false, // 直播
                 // rePlay: false, // 循环播放
                 // preload: true,
                 // controlBarVisibility: 'hover', // 控制条的显示方式：鼠标悬停
                 // useH5Prism: true, // 播放器类型：html5
             }, function(player) {
                 console.log('播放器创建成功')
             })
         }
     
     }
     </script>
     
     
     ```

     

   

