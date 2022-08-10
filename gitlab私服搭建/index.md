# gitlab私服搭建


# gitlab私服搭建

## 前期准备

​     安装的时候,使用桥接模式.方便后面使用.    

​     硬件配置,内存8G,系统centos7,网络桥接模式

安装centos7,

​    

<img src="https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1622613355801.png" alt="1622613355801" style="zoom:200%;" />

![1622613403440](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1622613403440.png)

## 修改为静态IP地址,验证网络是否可以连通



## 安装gitlab

  软件名称:gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm



前期准备工作

```
 
1\安装SSH协议
yum install -y curl policycoreutils-python openssh-server

2\设置SSH服务开机自启动
systemctl enable sshd

3\启动SSH服务
systemctl start sshd

4\安装Postfix以发送通知邮件
yum -y install postfix

5\将postfix服务设置成开机自启动
systemctl enable postfix

6\启动postfix
systemctl start postfix

7\安装vim编辑器
yum install vim -y


```

安装gitlab

```
rpm -ivh gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm 
```

修改配置文件

```
#编辑配置文件
vim  /etc/gitlab/gitlab.rb
#修改访问URL
#格式：external_url 'http://ip:端口'
external_url 'http://ip:8000'

```

重置Gitlab

```
gitlab-ctl reconfigure
```

启动Gitlab

```
 gitlab-ctl restart
```

首次访问修改密码

​       访问：ip:8000/

​      密码设置为 7092890jiang

![1622620273365](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1622620273365.png)





![1622620376501](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1622620376501.png)



## 汉化

点击右上角 用户 -->settings-->Preferences -> Localization -> Language -> 简体中文



## 常见问题

502错误

如果出现502错误,优先增加内存.

1\ 端口被占用

   检查这个文件   /etc/gitlab/gitlab.rb

```
  external_url 'http://ip:8000'
```

   修改后,

​         gitlab-ctl reconfigure

​         gitlab-ctl restart

 2\ 内存不足

​    1) 增加虚拟机内存

​		![1622622603651](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/1622622603651.png)

      2) 开启交换分区swap

 



```
0-查看swap分区是否启动
	cat /proc/swaps

1- 新建一个文件夹 
  mkdir /data
  
  
2-创建一个4G大小的交换分区  bs*count=4294971392(4G)；

dd if=/dev/zero of=/data/swap bs=512 count=8388616

3-指定swap分区

mkswap /data/swap

4-查看内核参数vm.swappiness中的数值是否为0，如果为0则根据实际需要调整成60
查看： cat /proc/sys/vm/swappiness
设置： sysctl -w vm.swappiness=60
    
5-启动分区
swapon /data/swap

echo “/data/swap swap swap defaults 0 0” >> /etc/fstab

6-再次查看分区是否启动

cat /proc/swaps

7-重启gitlab

gitlab-ctl restart 

```

 


