# FRP 内网穿透


# FRP 内网穿透

## 准备

- 一台拥有公网ip的服务器

## 服务器端安装配置FRPS

1. 下载文件 [github地址](https://github.com/fatedier/frp/releases)

   ![截屏2022-05-29 15.45.31](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/05/29/20220529-154658.png)

   ```
   wget https://github.com/fatedier/frp/releases/download/v0.43.0/frp_0.43.0_linux_amd64.tar.gz
   ```

2. 解压

   ```
   tar -zxvf frp_0.43.0_linux_amd64.tar.gz
   ```

3. 配置 frps.ini

   ```
   [common]
   # 端口
   bind_port = 7000
   # 密码token
   token = xxxxxxx
   ```

4. 创建**Systemctl**后台启动

   ```
   vim /lib/systemd/system/frps.service
   ```

   ```
   [Unit]
   Description=fraps service
   After=network.target syslog.target
   Wants=network.target
   
   [Service]
   Type=simple
   ExecStart=/root/frp/frps -c /root/frp/frps.ini # 根据实际情况修改
   
   [Install]
   WantedBy=multi-user.target
   ```

5. 启动服务设置随系统启动

   ```
   //设置随系统启动
   systemctl enable frps
   //启动
   systemctl start frps
   //查看启动情况:查看进程
   ps auxw
   ```

## 客户端安装配置FRPC

1. 下载对应系统的文件

2. 配置frpc.ini

   ```
   [common]
   //服务器ip
   server_addr = 123.57.48.176
   //服务器端口
   server_port = 7000
   //密码
   token = xxxxx
   
   //服务一
   [mysql]
   type = tcp
   local_ip = 127.0.0.1
   local_port = 3306
   //映射到服务器的端口
   remote_port = 9999
   //服务二 不能重名
   [mysql2]
   type = tcp
   local_ip = 127.0.0.1
   local_port = xxxx
   remote_port = 9999
   
   ```

3. 启动

   ```
   ./frpc -c frpc.ini
   ```

4. Mac 开机自启动

   1. 编写脚本

      ```
      sudo vim /Library/LaunchDaemons/frpc.plist
      ```

      ```
      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE plist PUBLIC -//Apple Computer//DTD PLIST 1.0//EN
      http://www.apple.com/DTDs/PropertyList-1.0.dtd >
      <plist version="1.0">
      <dict>
      <key>Label</key>
      <string>frpc</string>
      <key>ProgramArguments</key>
      <array>
      <string>/Users/mr_j/frpc/frpc</string>
      <string>-c</string>
      <string>/Users/mr_j/frpc/frpc.ini</string>
      </array>
      <key>KeepAlive</key>
      <true/>
      <key>RunAtLoad</key>
      <true/>
      </dict>
      </plist>
      ```

   2. 赋权并让生效

      ```
      sudo chown root /Library/LaunchDaemons/frpc.plist
      sudo launchctl load -w /Library/LaunchDaemons/frpc.plist
      ```

   3. 取消自启动

      ```
      sudo launchctl unload -w /Library/LaunchDaemons/frpc.plist
      ```

      

   


