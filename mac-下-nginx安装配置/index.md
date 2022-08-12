# mac下nginx安装配置


# mac 下 nginx安装配置

1. 安装

   ```
   brew install nginx
   ```

2. 相关命令

   ```
   # 启动
   sudo nginx
   
   # 停止
   sudo nginx -s stop
   
   # 重启
   sudo nginx -s reload
   ```

3. 修改配置

   ```
   vim /usr/local/etc/nginx/nginx.conf
   ```

4. 查看服务器（默认端口8080）

   http://localhost:8080/

