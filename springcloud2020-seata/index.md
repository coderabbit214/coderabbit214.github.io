# springcloud2020 seata


# springcloud2020 seata

> Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。
>
> 多数据库事务
>
> 分布式事务处理过程的一ID+三组件模型：
>
> - Transaction ID XID 全局唯一的事务ID
> - 三组件概念
>   - TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
>   - TM (Transaction Manager) - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
>   - RM (Resource Manager) - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。



## 安装配置1.4

1. 下载https://github.com/seata/seata/releases

### 服务安装

1. registry.conf修改注册中心和配置中心为nacos

   ```properties
   registry {
     # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
     type = "nacos"
   
     nacos {
       application = "seata-server"
       serverAddr = "127.0.0.1:8848"
       group = "SEATA_GROUP"
       namespace = ""
       cluster = "default"
       username = "nacos"
       password = "nacos"
     }
   }
   
   
   config {
     # file、nacos 、apollo、zk、consul、etcd3
     type = "nacos"
   
     nacos {
       serverAddr = "127.0.0.1:8848"
       namespace = ""
       group = "SEATA_GROUP"
       username = "nacos"
       password = "nacos"
       dataId = "seataServer.properties"
     }
   }
   ```

2. file.conf修改

   ```properties
   store {
    ## store mode: file、db、redis
    mode = "db"
       ## database store property
        db {
        ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
        datasource = "druid"
        ## mysql/oracle/postgresql/h2/oceanbase etc.
        dbType = "mysql"
        driverClassName = "com.mysql.cj.jdbc.Driver"
        url = "jdbc:mysql://127.0.0.1:3307/seata?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=true"
        user = "root"
        password = "123456"
        minConn = 5
        maxConn = 100
        globalTable = "global_table"
        branchTable = "branch_table"
        lockTable = "lock_table"
        queryLimit = 100
        maxWait = 5000
        }
       }
   ```

3. 创建数据库seata

   ```sql
   
   CREATE TABLE IF NOT EXISTS `global_table`
   (
       `xid`                       VARCHAR(128) NOT NULL,
       `transaction_id`            BIGINT,
       `status`                    TINYINT      NOT NULL,
       `application_id`            VARCHAR(32),
       `transaction_service_group` VARCHAR(32),
       `transaction_name`          VARCHAR(128),
       `timeout`                   INT,
       `begin_time`                BIGINT,
       `application_data`          VARCHAR(2000),
       `gmt_create`                DATETIME,
       `gmt_modified`              DATETIME,
       PRIMARY KEY (`xid`),
       KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
       KEY `idx_transaction_id` (`transaction_id`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   -- the table to store BranchSession data
   CREATE TABLE IF NOT EXISTS `branch_table`
   (
       `branch_id`         BIGINT       NOT NULL,
       `xid`               VARCHAR(128) NOT NULL,
       `transaction_id`    BIGINT,
       `resource_group_id` VARCHAR(32),
       `resource_id`       VARCHAR(256),
       `branch_type`       VARCHAR(8),
       `status`            TINYINT,
       `client_id`         VARCHAR(64),
       `application_data`  VARCHAR(2000),
       `gmt_create`        DATETIME(6),
       `gmt_modified`      DATETIME(6),
       PRIMARY KEY (`branch_id`),
       KEY `idx_xid` (`xid`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   -- the table to store lock data
   CREATE TABLE IF NOT EXISTS `lock_table`
   (
       `row_key`        VARCHAR(128) NOT NULL,
       `xid`            VARCHAR(128),
       `transaction_id` BIGINT,
       `branch_id`      BIGINT       NOT NULL,
       `resource_id`    VARCHAR(256),
       `table_name`     VARCHAR(32),
       `pk`             VARCHAR(36),
       `gmt_create`     DATETIME,
       `gmt_modified`   DATETIME,
       PRIMARY KEY (`row_key`),
       KEY `idx_branch_id` (`branch_id`)
   ) ENGINE = InnoDB
     DEFAULT CHARSET = utf8;
   
   ```

4. 启动 nacos中查看服务是否注册成功

### nacos配置

1. 进入seata/conf目录下，创建一个nacos-config.sh文件：

   ```
   # touch nacos-config.sh
   ```

   将https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh的内容编辑进如nacos-config.sh

2. 然后到seata目录创建config.txt，注意跟nacos-config.sh不在同一个目录

   ```
   # touch config.txt
   ```

   将如下内容写入：

   ```properties
   service.vgroupMapping.my_test_tx_group=default
   service.default.grouplist=127.0.0.1:8091
   service.enableDegrade=false
   service.disableGlobalTransaction=false
   
   store.mode=db
   store.db.datasource=druid
   store.db.dbType=mysql
   store.db.driverClassName=com.mysql.cj.jdbc.Driver
   store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true
   store.db.user=username
   store.db.password=password
   store.db.minConn=5
   store.db.maxConn=30
   store.db.globalTable=global_table
   store.db.branchTable=branch_table
   store.db.queryLimit=100
   store.db.lockTable=lock_table
   store.db.maxWait=5000
   ```

3. 最后再执行导入：seata/conf

   ```
   # sh nacos-config.sh 127.0.0.1
   ```

4. 导入之后，可以在nacos看到配置信息

## 安装配置0.9

1. 下载https://github.com/seata/seata/releases

2. file.conf

   - service模块

     ```
     service {
         ##fsp_tx_group是自定义的
         vgroup_mapping.my.test.tx_group="fsp_tx_group" 
         default.grouplist = "127.0.0.1:8091"
         enableDegrade = false
         disable = false
         max.commitretry.timeout= "-1"
         max.ollbackretry.timeout= "-1"
     }
     ```

   - store模块

     ```
     ## transaction log store
     store {
     	## store mode: file, db
     	## 改成db
     	mode = "db"
     	
     	## file store
     	file {
     		dir = "sessionStore"
     		
     		# branch session size, if exceeded first try compress lockkey, still exceeded throws exceptions
     		max-branch-session-size = 16384
     		# globe session size, if exceeded throws exceptions
     		max-global-session-size = 512
     		# file buffer size, if exceeded allocate new buffer
     		file-write-buffer-cache-size = 16384
     		# when recover batch read size
     		session.reload.read_size= 100
     		# async, sync
     		flush-disk-mode = async
     	}
     
     	# database store
     	db {
     		## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
     		datasource = "dbcp"
     		## mysql/oracle/h2/oceanbase etc.
     		## 配置数据源
     		db-type = "mysql"
     		driver-class-name = "com.mysql.jdbc.Driver"
     		url = "jdbc:mysql://127.0.0.1:3306/seata"
     		user = "root"
     		password = "你自己密码"
     		min-conn= 1
     		max-conn = 3
     		global.table = "global_table"
     		branch.table = "branch_table"
     		lock-table = "lock_table"
     		query-limit = 100
     	}
     }
     ```

3. mysql新建库seata（sql文件在下载包里）

4. registry.conf

   ```
   registry {
     # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
     # 改用为nacos
     type = "nacos"
   
     nacos {
     	## 加端口号
       serverAddr = "localhost:8848"
       namespace = ""
       cluster = "default"
     }
     ...
   }
   ```

## 编码



![截屏2021-05-30 下午2.47.54](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2021-05-30%20%E4%B8%8B%E5%8D%882.47.54.png)

注意：file.conf

​			registry.conf

## 使用

> @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)

```java
package com.atguigu.springcloud.service.impl;

import com.atguigu.springcloud.dao.OrderDao;
import com.atguigu.springcloud.entity.Order;
import com.atguigu.springcloud.service.AccountService;
import com.atguigu.springcloud.service.OrderService;
import com.atguigu.springcloud.service.StorageService;
import io.seata.spring.annotation.GlobalTransactional;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Service
@Slf4j
public class OrderServiceImpl implements OrderService
{
    @Resource
    private OrderDao orderDao;
    @Resource
    private StorageService storageService;
    @Resource
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：下订单->扣库存->减余额->改状态
     */
    @Override
    @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
    public void create(Order order)
    {
        log.info("----->开始新建订单");
        //1 新建订单
        orderDao.create(order);

        //2 扣减库存
        log.info("----->订单微服务开始调用库存，做扣减Count");
        storageService.decrease(order.getProductId(),order.getCount());
        log.info("----->订单微服务开始调用库存，做扣减end");

        //3 扣减账户
        log.info("----->订单微服务开始调用账户，做扣减Money");
        accountService.decrease(order.getUserId(),order.getMoney());
        log.info("----->订单微服务开始调用账户，做扣减end");

        //4 修改订单状态，从零到1,1代表已经完成
        log.info("----->修改订单状态开始");
        orderDao.update(order.getUserId(),0);
        log.info("----->修改订单状态结束");

        log.info("----->下订单结束了，O(∩_∩)O哈哈~");

    }
}
```

​			



## 未解决问题

1.4版本配置出错

