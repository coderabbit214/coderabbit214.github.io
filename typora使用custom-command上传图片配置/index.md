# Typora使用Custom Command上传图片配置


# Typora使用Custom Command上传图片配置

## 1.使用 node 安装 PicGo-Core

```
sudo npm install -g picgo
```

## 2.使用 picgo 命令安装 gitee-uploader 插件

```
picgo install gitee-uploader
```

## 3.配置 Typroa 上传服务设定

![截屏2021-04-29 上午11.20.06](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-04-29%20%E4%B8%8A%E5%8D%8811.20.06.png)

- 上传服务：Custom Command

- 命令：[your node path] [your picgo path] upload

  > 例：/usr/local/bin/node /usr/local/lib/node_modules/picgo/bin/picgo upload

## 4.配置文件

以阿里云为例

位置：/Users/\<UserName>/.picgo/config.json

```json
{
  "picBed": {
    "uploader": "aliyun",
    "current": "aliyun",
    "transformer": "path",
    "aliyun": {
      "accessKeyId": "",
      "accessKeySecret": "",
      "bucket": "", // 存储空间名
      "area": "", // 存储区域代号
      "path": "", // 自定义存储路径
       "customUrl": "", // 自定义域名，注意要加 http://或者 https://
       "options": "" // 针对图片的一些后缀处理参数 PicGo 2.2.0+ PicGo-Core 1.4.0+
      }
  },
  "picgoPlugins": {}
}
```

## 5.配置时间戳重命名文件

1. 安装插件

   ```
   picgo install super-prefix
   ```

   ![截屏2022-03-18 下午2.22.09](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/03/18/20220318-142212.png)

2. 配置/Users/\<UserName>/.picgo/config.json

   ```json
   {
     "picBed": {
       ···
       }
     },
     "picgoPlugins": {
       "picgo-plugin-super-prefix": true,
       "picgo-plugin-gitee-uploader": true
     },
     "picgo-plugin-super-prefix": {
       "prefixFormat": "YYYY/MM/DD/",
       "fileFormat": "YYYYMMDD-HHmmss"
     }
   }
   ```

   

