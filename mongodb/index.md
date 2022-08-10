# Mongodb


# Mongodb

## 安装配置

1. [下载地址](https://www.mongodb.com/try/download/community)
2. 解压在/usr/local/目录下 MongoDB(这里用MongoDB代替文件夹名字)
3. 在MongoDB目录下新建data/db 和 log文件夹
4. 在终端中输出 "open -e .bash_profile"，打开bash_profile文件
5. 将安装目录的bin目录地址 "export PATH=${PATH}:/usr/local/mongoDB/bin" 添加到环境变量中，保存关闭
6. 在终端中输入"source .bash_profile"使配置立即生效
7. 在终端中输入 "mongod -version" 判断是否安装成功

## 启动

进入data和log所在目录命令行运行

> --dbpath  指定为刚才创建好的data目录 
>
> --logpath  指定log存放位置 
>
> --logappend  mongo在后台运行

```
sudo mongod --dbpath data/db --logpath log/mongod.log --logappend
```

## 连接

新打开一个命令行 输入mongo连接

## 关闭

新打开一个命令行  连接mongo

```
use admin;

db.shutdownServer();
```

## springboot集成

1. pom

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-mongodb</artifactId>
           </dependency>
   ```

2. properties

   ```properties
   spring.data.mongodb.uri=mongodb://127.0.0.1:27017/wolf2w
   logging.level.org.springframework.data.mongodb.core=debug
   ```

3. 继承Repository

   ```java
   public interface StrategyCommnetRepository extends MongoRepository<StrategyComment,String> {
   
   }
   ```

4. 使用

    

