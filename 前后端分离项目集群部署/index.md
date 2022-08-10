# 前后端分离项目部署


# 前后端分离项目部署

### 环境准备

1. 三个centos7操作系统

   - 192.168.11.15（前端，mysql，redis）
   - 192.168.11.16（java服务）
   - 192.168.11.23（java服务）

2. Mysql

   > 开放访问权限

3. redis

   > 修改配置文件 开放访问
   >
   > bind 127.0.0.1 ::1 0.0.0.0
   >
   > protected-mode no

### 后端部署

1. 开放对应端口

   ```
   查询指定端口是否已开 
   firewall-cmd --query-port=8080/tcp
   添加指定需要开放的端口：
   firewall-cmd --add-port=8080/tcp --permanent
   重载入添加的端口：
   firewall-cmd --reload
   查询指定端口是否开启成功：
   firewall-cmd --query-port=8080/tcp
   ```

2. jar包部署

   ```
   java -jar xxx.jar
   //后台运行
   nohup java -jar xxx.jar
   ```

### 前端

1. 静态文件准备

   - 安装依赖

     ```
     npm install
     ```

   - 打包生成dist文件夹

     ```
     npm run build
     ```

2. nginx配置

   > - 静态文件夹地址
   > - location请求转发
   > - upstream配置

   ```
   http {
   ...
       upstream ruoyi{
           server 192.168.11.16:8080;
           server 192.168.11.23:8080;
       }
   
       server {
           listen       80;
           server_name  localhost;
   
   
           location / {
               root   /Users/mr_j/Desktop/dist;
               index  index.html index.htm;
           }
   
           location /prod-api/ {
               proxy_set_header Host $http_host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header REMOTE-HOST $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_pass http://ruoyi/;
   
           }
           ...
       }
       ...
   }
   ```

3. 启动

