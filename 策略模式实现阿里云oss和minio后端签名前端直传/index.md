# 使用策略模式实现阿里云OSS和MINIO后端签名前端直传


# 使用策略模式实现阿里云OSS和MINIO后端签名前端直传

## 需求

1. 无缝(通过配置，不改代码)切换外网模式和内网模式(阿里云和MINIO)
2. 文件前端直传(需要后端签名)，节约自己服务器的网络资源
3. 私有读写，如果需要向后端发起请求获取签名

## 实现

### UML图

![截屏2022-03-11 下午8.15.48](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-03-11%20%E4%B8%8B%E5%8D%888.15.48.png)

### 工厂模式实现(后端代码)

1. 配置

   ```yaml
   oss:
     # 使用oss的模式
     # 1. ALIYUN
     # 2. MINIO
     mode: ALIYUN
     # 上传目录，这里如果不想加，别的地方到处加
     dir: test/
     # 上传url过期时间 单位 s
     updateExpiryTime: 600
     # 查看过期时间 单位 s
     downloadExpiryTime: 600
     #阿里oss配置
     aliyun:
       endpoint: 
       keyid: 
       keysecret: 
       bucketname: 
     #minio配置  
     minio:
       endpoint: 
       keyid: 
       keysecret: 
       bucketName: 
   ```

2. 编写OSS接口，每一种oss模式都需要实现其所有方法

   ```java
   public interface OssInterface {
       /**
        * 文件上传
        * @param file
        * @param objectName
        * @return
        */
       String uploadFile(MultipartFile file, String objectName);
   
       /**
        * 判断文件是否存在
        * @param objectName
        * @return
        */
       boolean existFile(String objectName);
   
       /**
        * 获取上传签名
        * @return
        */
       OssRequestData getPolicy(String dir,String fileName);
   
       /**
        * 获取查看签名
        */
       String getExpiration(String objectName);
   }
   ```

3. 编写实现类

   - AliOssService

     ```java
     @Service
     public class AliOssService implements OssInterface {
     
         String endpoint = AliyunConstantPropertiesUtils.END_POINT;
         String accessKeyId = AliyunConstantPropertiesUtils.KEY_ID;
         String accessKeySecret = AliyunConstantPropertiesUtils.KEY_SECRET;
         String bucketName = AliyunConstantPropertiesUtils.BUCKET_NAME;
     
         /**
          * 文件上传
          *
          * @param file
          * @return
          */
         @Override
         public String uploadFile(MultipartFile file, String objectName) {
             OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
             try {
                 // 上传文件流。
                 InputStream inputStream = file.getInputStream();
                 //获取文件名称
                 //第一个参数 Bucket名称
                 //第二个参数 文件路径和文件名称
                 //第三个参数 文件输入流
                 ossClient.putObject(bucketName, objectName, inputStream);
                 // 关闭OSSClient。
                 ossClient.shutdown();
                 //上传之后的文件路径返回
                 //需要手动拼接
                 return "https://" + bucketName + "." + endpoint + "/" + objectName;
             } catch (Exception e) {
                 e.printStackTrace();
                 return null;
             }
         }
     
         /**
          * 判断文件是否存在
          */
         @Override
         public boolean existFile(String objectName) {
             // 创建OSSClient实例。
             OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
             boolean found = ossClient.doesObjectExist(bucketName, objectName);
             ossClient.shutdown();
             return found;
         }
     
         /**
          * 签名
          */
         @Override
         public OssRequestData getPolicy(String dir,String fileName){
             fileName = UUID.randomUUID()+"_"+fileName;
             String host = "https://" + bucketName + "." + endpoint;
             // 创建OSSClient实例。
             OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
             OssRequestData ossRequestData =null;
             try {
                 long expireEndTime = System.currentTimeMillis() + OssConstantPropertiesUtils.UPDATE_EXPIRY_TIME * 1000;
                 Date expiration = new Date(expireEndTime);
                 // PostObject请求最大可支持的文件大小为5 GB，即CONTENT_LENGTH_RANGE为5*1024*1024*1024。
                 PolicyConditions policyConds = new PolicyConditions();
                 policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);
                 policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir+fileName);
                 String postPolicy = ossClient.generatePostPolicy(expiration, policyConds);
                 byte[] binaryData = postPolicy.getBytes(StandardCharsets.UTF_8);
                 String encodedPolicy = BinaryUtil.toBase64String(binaryData);
                 String postSignature = ossClient.calculatePostSignature(postPolicy);
                 FormData formData = new FormData(accessKeyId,encodedPolicy,postSignature,dir,dir+fileName,fileName,String.valueOf(expireEndTime / 1000));
                 ossRequestData = new OssRequestData(host,"file",true,"post",formData);
             } catch (Exception e) {
                 System.out.println(e.getMessage());
             } finally {
                 ossClient.shutdown();
             }
             return ossRequestData;
         }
     
         /**
          * 获取签名授权访问
          * @param objectName
          * @return
          */
         @Override
         public String getExpiration(String objectName){
             OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
             Date expiration = new Date(System.currentTimeMillis() + OssConstantPropertiesUtils.DOWNLOAD_EXPIRY_TIME * 1000);
             URL url = ossClient.generatePresignedUrl(bucketName, objectName, expiration);
             ossClient.shutdown();
             return url.toString();
         }
     
     }
     ```

   - MinIOOssService

     ```java
     @Service
     public class MinIOOssService implements OssInterface {
     
         String endpoint = MinIOConstantPropertiesUtils.END_POINT;
         String accessKeyId = MinIOConstantPropertiesUtils.KEY_ID;
         String accessKeySecret = MinIOConstantPropertiesUtils.KEY_SECRET;
         String bucketName = MinIOConstantPropertiesUtils.BUCKET_NAME;
     
         /**
          * 文件上传
          *
          * @param file
          * @return
          */
         @Override
         public String uploadFile(MultipartFile file, String objectName) {
             MinioClient minioClient = MinioClient.builder()
                     .endpoint(endpoint)
                     .credentials(accessKeyId, accessKeySecret)
                     .build();
     
             InputStream in = null;
             try {
                 in = file.getInputStream();
                 minioClient.putObject(PutObjectArgs.builder().bucket(bucketName).object(objectName)
                         .stream(in, file.getSize(), -1).contentType(file.getContentType()).build());
             } catch (IOException e) {
                 e.printStackTrace();
                 throw new RuntimeException("文件异常");
             } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException | InvalidResponseException | NoSuchAlgorithmException | ServerException | XmlParserException e) {
                 e.printStackTrace();
                 throw new RuntimeException("上传异常");
             } finally {
                 try {
                     if (in != null) {
                         in.close();
                     }
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             return endpoint + "/" + objectName;
         }
     
         /**
          * 判断文件是否存在
          */
         @Override
         public boolean existFile(String objectName) {
             return false;
         }
     
         /**
          * 上传签名
          */
         @Override
         public OssRequestData getPolicy(String dir, String fileName) {
             fileName = UUID.randomUUID() + "_" + fileName;
             MinioClient minioClient = MinioClient.builder()
                     .endpoint(endpoint)
                     .credentials(accessKeyId, accessKeySecret)
                     .build();
             String product = null;
             OssRequestData ossRequestData =null;
             try {
                 product = minioClient.getPresignedObjectUrl(
                         GetPresignedObjectUrlArgs.builder()
                                 .method(Method.PUT)
                                 .bucket(bucketName)
                                 .object(dir + fileName)
                                 .expiry(OssConstantPropertiesUtils.UPDATE_EXPIRY_TIME)
                                 .build());
             } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException | InvalidResponseException | IOException | NoSuchAlgorithmException | XmlParserException | ServerException e) {
                 e.printStackTrace();
                 throw new RuntimeException("获取签名异常");
             }
             FormData formData = new FormData(accessKeyId,null,null,dir,dir+fileName,fileName,null);
             ossRequestData = new OssRequestData(product,"data",false,"put",formData);
             return ossRequestData;
         }
     
         /**
          * 查看签名
          */
         @Override
         public String getExpiration(String objectName) {
     
             MinioClient minioClient = MinioClient.builder()
                     .endpoint(endpoint)
                     .credentials(accessKeyId, accessKeySecret)
                     .build();
             String presignedObjectUrl = null;
             try {
                 presignedObjectUrl = minioClient.getPresignedObjectUrl(
                         GetPresignedObjectUrlArgs.builder()
                                 .method(Method.GET)
                                 .bucket(bucketName)
                                 .object(objectName)
                                 .expiry(OssConstantPropertiesUtils.DOWNLOAD_EXPIRY_TIME)
                                 .build()
                 );
             } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException | InvalidResponseException | IOException | NoSuchAlgorithmException | XmlParserException | ServerException e) {
                 e.printStackTrace();
                 throw new RuntimeException("获取签名异常");
             }
             return presignedObjectUrl;
         }
     }
     ```

4. 工厂实现

   ```java
   @Component
   public class OssFactory {
   
       @Value("${oss.mode}")
       private String endpoint;
   
       public OssInterface getOssService() {
           if ("ALIYUN".equals(endpoint)) {
               return new AliOssService();
           } else if ("MINIO".equals(endpoint)) {
               return new MinIOOssService();
           } else {
               throw new RuntimeException("没有这个选项，宝");
           }
       }
   }
   ```

5. Controller使用

   ```java
   @RestController
   @CrossOrigin
   public class OssController {
   
       @Autowired
       private OssFactory ossFactory;
   
       @Operation(summary = "上传文件")
       @PostMapping(value = "uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
       public String uploadFile(@RequestPart MultipartFile file){
           OssInterface ossService = ossFactory.getOssService();
           return ossService.uploadFile(file, OssConstantPropertiesUtils.DIR+UUID.randomUUID()+file.getOriginalFilename());
       }
   
       /**
        *
        * @return
        */
       @Operation(summary = "获取上传签名")
       @GetMapping(value = "getPolicy")
       public OssRequestData getPolicy(String fileName){
           OssInterface ossService = ossFactory.getOssService();
           return ossService.getPolicy(OssConstantPropertiesUtils.DIR,fileName);
       }
   
       /**
        * 可以添加redis对相同文件的请求进行缓存
        * @param objectName
        * @return
        */
       @Operation(summary = "获取查看签名")
       @GetMapping(value = "getExpiration")
       public String getExpiration(String objectName){
           OssInterface ossService = ossFactory.getOssService();
           return ossService.getExpiration(objectName);
       }
   }
   ```

6. 请求签名时返回pojo

   ```java
   public class OssRequestData implements Serializable {
       /** 请求路径 */
       private String host;
       /** 文件参数key值 */
       private String fileFieldName;
       /** 参数是否使用FormData */
       private boolean withFormData;
       /** 请求方式 */
       private String method;
       /** formData参数 */
       private FormData formData;
   
       public OssRequestData(String host, String fileFieldName, boolean withFormData, String method, FormData formData) {
           this.host = host;
           this.fileFieldName = fileFieldName;
           this.withFormData = withFormData;
           this.method = method;
           this.formData = formData;
       }
   
       public String getHost() {
           return host;
       }
   
       public void setHost(String host) {
           this.host = host;
       }
   
       public String getFileFieldName() {
           return fileFieldName;
       }
   
       public void setFileFieldName(String fileFieldName) {
           this.fileFieldName = fileFieldName;
       }
   
       public boolean isWithFormData() {
           return withFormData;
       }
   
       public void setWithFormData(boolean withFormData) {
           this.withFormData = withFormData;
       }
   
       public String getMethod() {
           return method;
       }
   
       public void setMethod(String method) {
           this.method = method;
       }
   
       public FormData getFormData() {
           return formData;
       }
   
       public void setFormData(FormData formData) {
           this.formData = formData;
       }
   }
   
   public class FormData implements Serializable {
       private String OSSAccessKeyId;
       private String policy;
       private String signature;
       private String dir;
       private String key;
       private String fileName;
       private String expire;
   
       public FormData(String OSSAccessKeyId, String policy, String signature, String dir, String key, String fileName, String expire) {
           this.OSSAccessKeyId = OSSAccessKeyId;
           this.policy = policy;
           this.signature = signature;
           this.dir = dir;
           this.key = key;
           this.fileName = fileName;
           this.expire = expire;
       }
   
       public String getOSSAccessKeyId() {
           return OSSAccessKeyId;
       }
   
       public void setOSSAccessKeyId(String OSSAccessKeyId) {
           this.OSSAccessKeyId = OSSAccessKeyId;
       }
   
       public String getPolicy() {
           return policy;
       }
   
       public void setPolicy(String policy) {
           this.policy = policy;
       }
   
       public String getSignature() {
           return signature;
       }
   
       public void setSignature(String signature) {
           this.signature = signature;
       }
   
       public String getDir() {
           return dir;
       }
   
       public void setDir(String dir) {
           this.dir = dir;
       }
   
       public String getKey() {
           return key;
       }
   
       public void setKey(String key) {
           this.key = key;
       }
   
       public String getFileName() {
           return fileName;
       }
   
       public void setFileName(String fileName) {
           this.fileName = fileName;
       }
   
       public String getExpire() {
           return expire;
       }
   
       public void setExpire(String expire) {
           this.expire = expire;
       }
   }
   
   ```

### 前端适配

```html
<template>
  <div class="home" style="height: 100%">
    <el-upload
        :action="123"
        :data="dataObj"
        :http-request="upload"
        list-type="picture"
        :multiple="false"
        :file-list="fileList"
    >
      <el-button size="small" type="primary">点击上传</el-button>
    </el-upload>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Home',
  props: {
    value: String
  },
  data() {
    return {
      fileList: [],
    }
  },
  methods: {
    //向后端请求上传签名
    upload(item) {
      //请求参数
      let data;
      //文件
      let file = item.file;

      axios.get("http://127.0.0.1:8081/getPolicy?fileName=" + file.name).then(res => {
        res = res.data;
        //后端传递的请求参数
        let formData = res.formData;
        //文件oss上的根路径+文件名
        let key = formData.key;
        //判断是否需要formData
        if (res.withFormData) {
          data = new FormData();
          for (let param in formData) {
            data.append(param, formData[param])
          }
          data.append(res.fileFieldName, file);
        } else {
          data = file;
        }
        //存储
        axios({
          url: res.host,
          data: data,
          method: res.method,
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }).then(res => {
          //向后端存储（伪代码）：
          /**
           * save({
           *  ...
           *   url: this.dataObj.host + '/' + this.dataObj.key
           *  ...
           * })
           */
          //前端使用：
          axios.get("http://127.0.0.1:8081/getExpiration?objectName=" + key).then(res => {
            this.fileList.push({
              name: file.name,
              url: res.data
            })
          })
        })
      });

    },
  }
}
</script>
```

### 关于签名直传

#### 流程图

文件上传以及文件查看预览都是这个流程

![p139016](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/p139016.png)

## 项目地址

https://github.com/coderabbit214/parent-demo/tree/main/oss_demo


