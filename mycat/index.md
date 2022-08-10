# MyCat


# MyCat

## 本次使用虚拟机

- 172.16.140.8：mycat
- 172.16.140.9：master1
- 172.16.140.10：slave1
- 172.16.140.11：master2
- 172.16.140.12：slave2

## 作用

- 读写分离
- 数据分片：垂直拆分(分库)，水平拆分(分表)
- 多数据源

## 安装配置

[下载地址](http://dl.mycat.org.cn/)

解压

```
tar -zxvf software/Mycat-server-1.6.7.5-release-20200410174409-linux.tar.gz -C /usr/local/
```

![截屏2021-11-14 21.37.25](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-11-14%2021.37.25.png)

![截屏2021-11-14 21.38.18](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-11-14%2021.38.18.png)

### 配置

- server.xml

  ![截屏2021-11-14 21.44.00](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-11-14%2021.44.00.png)

- schema.xml![截屏2021-11-20 18.21.55](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-11-20%2018.21.55.png)

  

### 启动

```
/usr/local/mycat/bin/mycat console

启动MyCat： 
./mycat start 
查看启动状态： 
./mycat status 
停止： 
./mycat stop 
重启： 
./mycat restart 
```

## 连接

```
mysql -umyCat -p123456 -h 172.16.140.8 -P 8066
```

## 读写分离

### 一主一从

1. 主机配置（master1）

   修改配置文件：vim /etc/my.cnf

   ```sh
   #主服务器唯一ID
   server-id=1
   #启用二进制日志
   log-bin=mysql-bin
   # 设置不要复制的数据库(可设置多个) 
   binlog-ignore-db=mysql 
   binlog-ignore-db=information_schema 
   #设置需要复制的数据库 
   binlog-do-db=需要复制的主数据库名字 
   #设置logbin格式 
   binlog_format=MIXED
   #binlog日志文件 记得给mysql开启权限
   log-bin=/data/mysql/mysql-bin.log
   #binlog过期清理时间
   expire_logs_days=7
   #binlog每个日志文件大小
   max_binlog_size=100m
   #binlog缓存大小
   binlog_cache_size=4m
   #最大binlog缓存大小
   max_binlog_cache_size=512m
   ```

   > logbin格式
   >
   > - STATEMENT:直接复制语句
   >
   >   问题：例如now()这种函数会导致数据不同
   >
   > - ROW:直接修改结果
   >
   >   问题：如果修改语句一次性修改一万条，由于性能原因会导致同步慢，日志膨胀
   >
   > - MIXED：会自动选择前两种

2. 从机配置（slave1）

   ```shell
   修改配置文件:vim /etc/my.cnf 
   #从服务器唯一ID
   server-id=2
   #启用中继日志 
   relay-log=mysql-relay
   ```

3. 重启主机，从机mysql（防火墙要关闭）

4. 在主机上建立帐户并授权 slave

   ```sql
   GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'Mm_123456';
   
   #查询master的状态 
   show master status;
   #记录下File和Position的值 
   #执行完此步骤后不要再操作主服务器MySQL，防止主服务器状态值变化
   ```

5. 在从机上配置需要复制的主机

   ```sql
   #复制主机的命令
   CHANGE MASTER TO MASTER_HOST='主机的IP地址', MASTER_USER='slave',
   MASTER_PASSWORD='123123', MASTER_LOG_FILE='mysql-bin.具体数字',MASTER_LOG_POS=具体值;
   
   #启动从服务器复制功能 
   start slave; 
   #查看从服务器状态 
   show slave status;
   
   #下面两个参数都是Yes，则说明主从配置成功! 
   # Slave_IO_Running: Yes
   # Slave_SQL_Running: Yes
   ```

   > 如果配置错误检查Error
   >
   > 我遇到的错误 两个mysqluuid相同
   >
   > 解决
   >
   > 1. 清除主从
   >
   >    ```
   >    stop slave ;
   >    
   >    reset master ;
   >    ```
   >
   > 2. 删除其中一个的uuid，重启mysql
   >
   > 3. 重新配置主从

6. 测试

7. 测试mycat，发现读也在writeHost

8. 修改\<dataHost\>的balance属性，通过此属性配置读写分离的类型

   - balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上
   - balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从 模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡
   - balance="2"，所有读操作都随机的在 writeHost、readhost 上分发
   - balance="3"，所有读请求随机的分发到 readhost 执行，writerHost 不负担读压力

9. 测试读写分离

### 双主双从

#### 数据库配置

> 先清除原先一主一从的关系

1. 两个master配置

   - master1

     ```shell
     #主服务器唯一ID
     server-id=1
     #启用二进制日志
     log-bin=mysql-bin
     # 设置不要复制的数据库(可设置多个) 
     binlog-ignore-db=mysql 
     binlog-ignore-db=information_schema 
     #设置需要复制的数据库 
     binlog-do-db=需要复制的主数据库名字 
     #设置logbin格式 
     binlog_format=MIXED
     
     # 在作为从数据库的时候，有写入操作也要更新二进制日志文件
     log-slave-updates 
     #防止主键冲突
     #表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535 
     auto-increment-increment=2
     # 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
     auto-increment-offset=1
     ```

   - master2

     ```shell
     #主服务器唯一ID
     server-id=3
     #启用二进制日志
     log-bin=mysql-bin
     # 设置不要复制的数据库(可设置多个) 
     binlog-ignore-db=mysql 
     binlog-ignore-db=information_schema 
     #设置需要复制的数据库 
     binlog-do-db=需要复制的主数据库名字 
     #设置logbin格式 
     binlog_format=MIXED
     
     # 在作为从数据库的时候，有写入操作也要更新二进制日志文件
     log-slave-updates 
     #防止主键冲突
     #表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535 
     auto-increment-increment=2
     # 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
     auto-increment-offset=2
     ```

2. 双从机配置

   - slave1
   
     ```shell
     修改配置文件:vim /etc/my.cnf 
     #从服务器唯一ID
     server-id=2
     #启用中继日志 
     relay-log=mysql-relay
     ```
   
   - slave2
   
     ```shell
     修改配置文件:vim /etc/my.cnf 
     #从服务器唯一ID
     server-id=4
     #启用中继日志 
     relay-log=mysql-relay
     ```
   
3. 重启mysql
   
4. 两个master建立salve账户

   ```sql
   GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'Mm_123456';
   ```

5. 两个slave分别与两个master建立主从关系

6. 两个master互相建立主从关系

#### 关系图

![截屏2021-11-20 21.22.11](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-11-20%2021.22.11.png)

#### MyCat

- schema.xml

  > balance="1": 全部的readHost与stand by writeHost参与select语句的负载均衡。 
  >
  > writeType="0": 所有写操作发送到配置的第一个writeHost，第一个挂了切到还生存的第二个 
  >
  > writeType="1"，所有写操作都随机的发送到配置的 writeHost，1.5 以后废弃不推荐
  >
  > writeHost，重新启动后以切换后的为准，切换记录在配置文件中:dnindex.properties
  >
  >  switchType="1": 
  >
  > - 1 默认值，自动切换。 
  > - -1 表示不自动切换
  > - 2 基于 MySQL 主从同步的状态决定是否切换。

  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
  <mycat:schema xmlns:mycat="http://io.mycat/">
  
  	<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" randomDataNode="dn1" dataNode="dn1">
  	</schema>
  	<dataNode name="dn1" dataHost="host1" database="testMyCat" />
  	<!--  -->
  	<dataHost name="host1" maxCon="1000" minCon="10" balance="1"
  			  writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
  		<heartbeat>select user()</heartbeat>
  		<writeHost host="hostM1" url="jdbc:mysql://172.16.140.9:3306" user="root" password="Mm_123456">
  			<readHost host="hostS1" url="jdbc:mysql://172.16.140.10:3306" user="root" password="Mm_123456" />
  		</writeHost>
  
  		<writeHost host="hostM2" url="jdbc:mysql://172.16.140.11:3306" user="root" password="Mm_123456">
  			<readHost host="hostS2" url="jdbc:mysql://172.16.140.12:3306" user="root" password="Mm_123456" />
  		</writeHost>
  
  	</dataHost>
  </mycat:schema>
  ```

- 改完配置记得重启

#### 测试

- 读写分离
- 关闭(master1)写入功能正常
















