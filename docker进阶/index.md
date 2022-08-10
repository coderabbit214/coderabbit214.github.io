# Docker进阶


# Docker进阶

## 概述

- 基于Go
- 应用
  - Web 应用的自动化打包和发布。
  - 自动化测试和持续集成、发布。
  - 在服务型环境中部署和调整数据库或其他的后台应用。
  - 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。
- **优势**
  - 快速
  - 响应式部署和扩展
  - 同一硬件上运行更多工作负载
- 虚拟化技术和容器化技术比较
  - 传统虚拟机，虚拟出硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件。
  - Docker容器内的应用直接运行在宿主机的内容，容器是没有自己的内核的，也没有虚拟硬件。
  - 每个容器都是相互隔离的，每个容器都有属于自己的文件系统，互不影响。

## 安装

> mac,win 直接官网下载安装
>
> 下面为centos7

1. 安装gcc

   ```shell
   yum -y install gcc  
   yum -y install gcc-c++  
   ```

2. 卸载旧版本

   ```
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   
   ```

3. 下载需要的安装包

   ```
   yum install -y yum-utils
   ```

4. 设置阿里云仓库（可以不设置，本人没有设置）

   ```
   yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo  #国外的地址
       
       # 设置阿里云的Docker镜像仓库
   yum-config-manager \
       --add-repo \
       https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  #国外的地址
   
   ```

5. 更新yum索引

   ```
   yum makecache fast
   ```

6. 安装docker

   ```
    yum install docker-ce docker-ce-cli containerd.io
   ```

## Docker常用命令

### 基础命令

```shell
docker version          #查看docker的版本信息
docker info             #查看docker的系统信息,包括镜像和容器的数量
docker 命令 --help       #帮助命令(可查看可选的参数)
docker COMMAND --help
#启动docker
systemctl start docker
#查看状态
systemctl status docker
```

### 镜像命令

- docker images 查看本地主机的所有镜像

  ```shell
  docker images #查看本地主机的所有镜像
  # 可选参数
  -a/--all 列出所有镜像
  -q/--quiet 只显示镜像的id
  ```

- docker search 搜索镜像

  ```shell
  # 举例：
  docker search mysql
  #搜索收藏数大于3000的镜像
  docker search mysql --filter=STARS=3000 
  ```

- docker pull 镜像名[:tag] 拉取镜像

  > 分层下载，联合文件系统

  ```shell
  docker pull mysql #如果不写tag默认就是latest
  docker pull mysql:5.7 #5.7版本
  ```

- docker rmi 删除镜像

  ```shell
  docker rmi -f  镜像id/REPOSITORY
  docker rmi -f  $(docker images -aq) #删除所有
  -f #强制
  ```

### 容器命令

- docker run [可选参数] 启动容器

  ```shell
  --name="名字"           #指定容器名字
  
  -d                     #后台方式运行
  -it                    #使用交互方式运行,进入容器查看内容
  
  -p                     #指定容器的端口
  (
  	-p ip:主机端口:容器端口 #配置主机端口映射到容器端口
  	-p 主机端口:容器端口
  	-p 容器端口
  )
  -P                     #随机指定端口(大写的P)
  ```

- 退出容器(使用-it方式启动)

  ```shell
  #停止并退出容器（后台方式运行则仅退出）
  exit
  #不停止容器退出
  #Ctrl+P+Q
  ```

- docker ps 查看容器

  ```shell
  docker ps # 默认列出正在运行的容器
  -a   # 列出所有容器的运行记录
  -n=? # 显示最近创建的n个容器
  -q   # 只显示容器的编号
  ```

- docker rm 删除容器

  ```shell
  docker rm 容器id                 #删除指定的容器,不能删除正在运行的容器,强制删除使用 rm -f
  docker rm -f $(docker ps -aq)   #删除所有的容器
  docker ps -a -q|xargs docker rm #删除所有的容器
  ```

- 启动停止

  ```shell
  docker start 容器id          #启动容器
  docker restart 容器id        #重启容器
  docker stop 容器id           #停止当前运行的容器
  docker kill 容器id           #强制停止当前容器
  ```

### 常用命令

- docker run -d 镜像名   后台启动

  ```shell
  docker run -d centos
  # 问题：docker ps 发现centos停止了
  # 常见问题：docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
  #nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
  ```

- docker logs 容器id     查看日志

  ```shell
  docker logs --help
  docker logs -tf 容器id
  -t 跟踪
  -f 带时间戳
  docker logs --tail number 容器id #num为要显示的日志条数
  ```

- docker top 容器id     查看容器中进程信息

  ```shell
  docker top dbaa4ae1699
  ```

- docker inspect 容器id   查看容器的元数据

  ```shell
  # 可以看到启动时带的参数，网卡信息等
  docker inspect dbaa4ae1699
  ```
  
- 进入正在运行的容器

  ```shell
  # docker exec 进入容器后开启一个新的终端，可以在里面操作
  docker exec -it c703b5b1911f /bin/bash
  # docker attach 进入容器正在执行的终端，不会启动新的进程
  docker attach c703b5b1911f
  ```
  
- 拷贝操作

  ```shell
  #拷贝容器的文件到主机中
  docker cp 容器id:容器内路径  目的主机路径
  
  #拷贝宿主机的文件到容器中
  docker cp 目的主机路径 容器id:容器内路径
  ```
  
- 查看容器状态

  ```shell
  docker stats 容器id
  ```



## 镜像

> 镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需要的所有内容，包括代码，运行时（一个程序在运行或者在被执行的依赖）、库，环境变量和配置文件。

### 联合文件系统（分层）

> 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最終的
> 文件系统会包含所有底层的文件和目录

所有的 Docker 镜像都起始于一个基础镜像层，当进行修改或增加新的内容时，就会在当前镜像层之上，创建新的镜像层。
- 特点

  Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的项部！这一层就是我们通常说的容器层，容器之下的都叫镜像层！

### 创建镜像命令(本地)

```shell
# docker commit 
# -a 作者
# -m 提交信息
docker commit -a="coderabbit214" -m="es have ik" df6f751eddc4 elasticsearchik:6.5.4
```



## 容器数据卷

> 相当于数据同步，本机与容器中的文件相互同步
>
> -v 主机目录:容器目录

举例：

```shell
docker run -p 3306:3306 --name mysql -v /mydata/mysql/log:/var/log/mysql -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

<span style='color:red'>容器删除后，本地依然保存</span>

### 常用命令

- 创建数据卷

```
docker volume create my-vol
```

- 查看所有的数据卷

```
$ docker volume ls
local my-vol
```

- 查看指定数据卷的信息

```
$ docker volume inspect my-vol
```

- 删除数据卷 docker volume rm ...

```
$ docker volume rm my-vol
```

- 删除容器之时删除相关的卷

```
$ docker rm -v ...
```

### 具名挂载和匿名挂载

匿名挂载，具名挂载，指定路径挂载的命令区别如下：

-v 容器内路径 #匿名挂载（匿名挂载会在/var/lib/docker/volumes/目录下随机生成文件夹）

-v 卷名:容器内路径 #具名挂载

-v /宿主机路径:容器内路径 #指定路径挂载

### 读写

指定数据卷映射的相关参数：

ro —— readonly 只读。设置了只读则只能操作宿主机的路径，不能操作容器中的对应路径。

rw ----- readwrite 可读可写

举例

```shell
docker run -d -P --name nginx -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx -v juming-nginx:/etc/nginx:rw nginx
```

### 容器间数据同步实现

**--volumes-from**

过参数`--volumes-from`，设置容器2和容器1建立数据卷挂载关系

```shell
docker run -it --name 容器名称 --volumes-from 被同步数据容器 镜像id /bin/bash
```

<span style="color:red">容器间数据同步与本机和容器挂载卷相同，都是备份操作。</span>

## Dockerfile

> 用来构建docker镜像的脚本命令

### 初体验

新建文件 dockerfile01

```shell
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "----end----"
CMD /bin/bash
```

使用命令构建镜像

```shell
docker build --name 01 -f dockerfile01 -t coderabbit214/centos:1.0 .
```

启动自己构建的容器(容器内跟路径下出现volume01、volume02目录)

```shell
docker run -it 镜像id /bin/bash
```

### 指令

| 指令       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 指定基础镜像                                                 |
| MAINTAINER | 镜像是谁写的，姓名+邮箱                                      |
| RUN        | 镜像构建的时候需要运行的命令                                 |
| ADD        | 将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget |
| WORKDIR    | 镜像的工作目录                                               |
| VOLUME     | 挂载的目录                                                   |
| EXPOSE     | 保留端口配置                                                 |
| CMD        | 指定这个容器启动的时候要运行的命令（只有最后一个会生效）     |
| EMTRYPOINT | 指定这个容器启动的时候要运行的命令，可以追加命令             |
| ONBUILD    | 当构建一个被继承DockerFile，这个时候就会运行ONBUILD的指令，触发指令 |
| COPY       | 功能类似ADD，但是是不会自动解压文件，也不能访问网络资源      |
| ENV        | 构建的时候设置环境变量                                       |

### 实战（制作tomcat并发布）

1. 准备镜像文件tomcat压缩包

2. 编写dockerfile文件，文件名使用官方命名：Dockerfile ，build的时候会默认寻找当前目录下的文件，不需要使用-f参数指定

   ```shell
   FROM centos
   MAINTAINER coderabbit214<1157237955@qq.com>
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD apache-tomcat-9.0.56.tar.gz /usr/local/
   
   RUN yum -y install vim
   RUN yum install -y java-1.8.0-openjdk-devel.x86_64 
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.56
   ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.56
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.56/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.56/bin/logs/catalina.out
   ```

3. 使用该Dockerfile构建镜像

   ```shell
   docker build -t diytomcat:1.0 .
   ```

4. 启动生成的镜像，构建Tomcat容器

   ```
   docker run -d -p 8088:8080 --name diytomcat -v /home/dockerfile/tomcat/test:/usr/local/apache-tomcat-9.0.56/webapps/test diytomcat:1.0
   ```

5. 打开网页查看正确

6. 发布镜像到DockerHub

7. - 登陆：docker login -u 用户名
   - Push:docker push coderabbit214/diytomcat:1.0
     - 用户名➕镜像名➕版本



### 本地存储和加载

- docker commit

  ```
  docker commit -a "作者" -m "描述" 容器id xxx:latest(自定义新镜像名)
  ```

- docker save

  ```
  docker save 48b142d6d753 -o /Users/Mr_J/mytomcat.tar
  ```

- docker load

  ```
   docker load -i /Users/Mr_J/mytomcat.tar    
  ```

  

## docker网络

> docker network

查看所有的docker网络

```shell
docker network ls
```

Docker默认提供了四个网络模式，说明：

- bridge：容器默认的网络是桥接模式(自己搭建的网络默认也是使用桥接模式,启动容器默认也是使用桥接模式)。此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。
- none：不配置网络，容器有独立的Network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，配置IP等。
- host：容器和宿主机共享Network namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。
- container：创建的容器不会创建自己的网卡，配置自己的IP容器网络连通。容器和另外一个容器共享Network namespace（共享IP、端口范围）。

docker 创建容器默认使用桥接模式（在同一网段下，可以ping通）

举例：

```shell
# 创建容器
docker run --name tomcat2 -p 8081:8080 -d tomcat:9.0
docker run --name tomcat1 -p 8080:8080 -d tomcat:9.0

# 查看ip（可以查看文件也可以安装ping命令）
docker exec -it tomcat1 cat /etc/hosts              
172.17.0.2	fca3c33ef5e3
docker exec -it tomcat2 cat /etc/hosts  
172.17.0.3	f49de8901a1e

# 使用ping（如果没有就先安装）命令查看是否可以连通
docker exec -it tomcat1 ping 172.17.0.3
# 可以连通
```



### 容器互联

#### --link

> `--link`的原理是在指定运行的容器上的/etc/hosts文件中添加容器名和ip地址的映射

举例：

```shell
# 创建容器 连接2
docker run --name tomcat3 --link tomcat2 -p 8082:8080 -d tomcat:9.0 
# 尝试ping
docker exec -it tomcat3 ping tomcat2  
```

可以ping通

![截屏2021-12-30 下午1.58.16](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2021-12-30%20%E4%B8%8B%E5%8D%881.58.16.png)

但是反过来容器Tomcat02通过容器名Tomcat03直接ping容器Tomcat03是不行的

原因：

查看tomcat3的/etc/hosts

有host配置，所以可以ping通

![截屏2021-12-30 下午2.00.46](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2021-12-30%20%E4%B8%8B%E5%8D%882.00.46.png)

查看tomcat2的/etc/hosts![截屏2021-12-30 下午2.02.08](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2021-12-30%20%E4%B8%8B%E5%8D%882.02.08.png)

没有host配置，所以不可以ping通



### 自定义网络（常用）

#### 命令

```shell
docker network --help
# 查看网络列表
docker network ls
# 查看网络详细信息
docker inspect <网络名称或者id>

# 创建网络
docker network create --help
--driver bridge   #指定bridge驱动程序来管理网络
--subnet 192.168.0.0/16 #指定网段的CIDR格式的子网
--gateway 192.168.0.1 	#指定主子网的IPv4或IPv6网关
```

#### 实例

```shell
# 创建网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

# 启动容器1
docker run --network mynet -d -P --name tomcat01 tomcat:8.0
# 启动容器2(后配置网络)
docker run -d -P --name tomcat02 tomcat:8.0
docker network connect mynet tomcat02 # 配置网络
# 查看网络
docker inspect mynet
```

网络截图

![截屏2022-01-06 下午1.20.05](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-06%20%E4%B8%8B%E5%8D%881.20.05.png)

#### Docker网络实战练习

##### Redis集群部署

通过以下脚本创建六个Redis 的配置信息：

```shell
for port in $(seq 1 6); \
do \
mkdir -p redis/node-${port}/conf
touch redis/node-${port}/conf/redis.conf
cat << EOF >redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 192.167.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done


```

创建六个容器

```shell
#第1个Redis容器
for port in $(seq 1 6); \
do \
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
    -v /Users/Mr_J/redis/node-${port}/data:/data \
    -v /Users/Mr_J/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
    -d --net redis --ip 192.167.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
done


```

创建分片集群

```shell
redis-cli --cluster create 192.167.0.11:6379 192.167.0.12:6379 192.167.0.13:6379 192.167.0.14:6379 192.167.0.15:6379 192.167.0.16:6379 --cluster-replicas 1
```

查看集群信息

![截屏2022-01-06 下午1.57.19](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-06%20%E4%B8%8B%E5%8D%881.57.19.png)

![截屏2022-01-06 下午1.58.01](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-06%20%E4%B8%8B%E5%8D%881.58.01.png)

##### SpringBoot项目打包镜像

1. 本地测试无误，打jar包

2. Dockerfile

   ```shell
   FROM java:8
   COPY actuator-demo-0.0.1-SNAPSHOT.jar /app.jar
   CMD ["--server.port=8080"]
   EXPOSE 8080
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

3. 启动

   ```shell
   docker run -d -p 8080:8080 demo
   ```

4. 测试



## Docker Compose

> 容器编排
>
> 定义和运行多个容器

命令

```shell
docker-compose build #构建（重新构建）项目
docker-compose up #启动
docker-compose down #停止
```

### 三步骤

1. Dockerfile保证我们的项目在任何地方都可以运行。
2. docker-compose.yml文件编写
3. docker-compose up启动项目

### 安装

查看官网

[Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)

### 体验

> [Get started with Docker Compose | Docker Documentation](https://docs.docker.com/compose/gettingstarted/)

1. 创建项目目录

   ```shell
   mkdir composetest
   cd composetest
   ```

2. 准备项目

   - app.py

     ```python
     import time
     
     import redis
     from flask import Flask
     
     app = Flask(__name__)
     cache = redis.Redis(host='redis', port=6379)
     
     def get_hit_count():
         retries = 5
         while True:
             try:
                 return cache.incr('hits')
             except redis.exceptions.ConnectionError as exc:
                 if retries == 0:
                     raise exc
                 retries -= 1
                 time.sleep(0.5)
     
     @app.route('/')
     def hello():
         count = get_hit_count()
         return 'Hello World! I have been seen {} times.\n'.format(count)
     ```

   - requirements.txt

     ```
     flask
     redis
     ```

3. 编写Dockerfile

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM python:3.7-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run"]
   ```

4. 编写docker-compose.yml

   ```yaml
   version: "3.9"
   services:
     web:
       build: .
       ports:
         - "5000:5000"
     redis:
       image: "redis:alpine"
   ```

5. 运行

   ```shell
   docker-compose up
   ```

   ![截屏2022-01-16 下午5.07.38](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%885.07.38.png)
   
#### 注意

- docker生成两个镜像，启动了两个容器

- 网络规则：项目中的内容在同一个网络下

  ![截屏2022-01-16 下午5.15.15](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%885.15.15.png)

  ![截屏2022-01-16 下午5.14.52](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%885.14.52.png)

### 详解docker-compose.yml

- 版本对应[Compose file | Docker Documentation](https://docs.docker.com/compose/compose-file/)
- 第三部分命令查询[Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/)

```yaml
# 第一部分：版本
version: "3.9"
# 第二部分：服务
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
    
# 第三部分：其他
	# 网络
	# 卷
	# 。。。
```

#### 注意的命令

- depends_on (注意依赖关系)

  ```yaml
  version: "3.9"
  services:
    web:
      build: .
      depends_on:
        - db
        - redis
    redis:
      image: redis
    db:
      image: postgres
  ```

### 开源项目

[Quickstart: Compose and WordPress | Docker Documentation](https://docs.docker.com/samples/wordpress/)

- docker-compose.yml

```yaml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

- 启动命令

```shell
docker-compose up -d
```

![截屏2022-01-16 下午5.43.10](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%885.43.10.png)

![截屏2022-01-16 下午5.46.54](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%885.46.54.png)

### 微服务项目部署

1. 编写项目

   ![截屏2022-01-16 下午6.01.10](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/%E6%88%AA%E5%B1%8F2022-01-16%20%E4%B8%8B%E5%8D%886.01.10.png)

2. 编写Dockerfile

   ```dockerfile
   FROM java:8
   
   COPY demo-0.0.1-SNAPSHOT.jar /app.jar
   
   CMD ["--server.port=8080"]
   
   EXPOSE 8080
   
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

3. 编写docker-compose.yml

   ```yaml
   version: '3.9'
   
   services:
     demo:
       build: .
       depends_on:
         - redis
       ports:
         - "8080:8080"
     redis:
       image: "library/redis:alpine"
   ```

4. 启动


