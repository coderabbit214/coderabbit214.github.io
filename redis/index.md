# Redis


# Redis

## 一.基本指令

```
key * //查询当前数据库所有的键
exists <key> //判断某个键是否存在 1：true 2：false
type <key> //查看键的类型
del <key> //删除键
expire <key> <seconds> //为键设置过启德的时间 单位为秒
ttl <key> //查看还有多长时间过期 “-1”代表永不过期 “-2”代表已过期
dbsize //查看当前库的key的数量
Flushdb //清空当前库
Flushall //通杀全部库
```

## 二.Redis的五大数据类型

### 1.String(使用最多)

```
get <key> //查询对应的键值
set <key> <value> //添加键值对
append <key> <value> //给给定的key追加值
strlen <key> //获取值的长度
setnx <key> <value> //只有在key不存在时设置key的值

incr <key> //将key存储的数字值+1 只能对数字操作
decr <key> //将key存储的数字值-1 只能对数字操作
incrby <key> <步长> //将key存储的数字值+步长 只能对数字操作
decrby <key> <步长> //将key存储的数字值-步长 只能对数字操作

mset <key1> <value1> <key2> <value2> ... //同时设置多个键值对
mget <key1> <key2> <key3> ... //同时获取多尔衮value
msetnx <key1> <value1> <key2> <value2> ... //同时设置多个键值对;当且仅当所有给定的key都不存在时

getrange <key> <起始位置> <结束位置> //截取一段值；起始位置从0开始；包前也包后
setrange <key> <起始位置> <value> //从起始位置开始赋值覆盖

setex <key> <过期时间> <value> //设置键值的同时，设置过期时间

getset <key> <value> //以旧换新 设置了新值的同时获得旧值
```

### 2.List

> 单键多值
>
> 底层是双向链表 对两端的操作性高，通过索引操作中间节点性能较差

```
lpush/rpush <key> <value1> <value2>... //从左边/右边加入一个或多个值
lpop/rpop <key> //从左边/右边取出一个值

rpoplpush <key1> <key2> //从key1的右边取出一个放在key2的左边

lrange <key> <start> <stop> //按照索引下标获得元素（从左到右）负值从右到左
lindex <key> <index> //按照索引下标获得元素（从左到右）
llen <key> //获得列表长r度

linsert <key> after/before <value> <newvalue> //在value 后/前插入 newvalue 的值 如果有相同的值 只对第一个起作用
lrem <key> <n> <value> //删除n个value 正数从左往右删 负数从右往左 0代表删除所有
```

### 3.Set

> 与list功能类似 但是value不能重复
>
> String类型的无序集合
>
> 底层是value为空的hash表
>
> 删除查找的复杂度都是O(1)

```
sadd <key> <value1> <value2>... //加入一个或多个值
smembers <key> //取出该集合的所有值
sismember <key> <value> //判断集合<key>是否为含有该<value>值，有返回1，没有返回0
scard <key> //返回该集合的元素个数
srem <key> <value1> <value2> //删除集合中的某个元素
spop <key> //随机从元素中吐出一个值
srandmember <key> <n> //随机从元素中吐出N个值,不会删除

sinter <key1> <key2> //返回两个集合的交集
sunion <key1> <key2> //返回两个集合的并集
sdiff <key1> <key2> //返回两个集合的差集
```

### 4.Hash

> 是键值对集合
>
> 适合存储对象
>
> 类似java中的 Map<String,String>

```
hset <key> <field> <value> //为指定的key设定field和value
hget <key> <field> //取出指定的value
hmset <key> <field> <value> <field> <value>... //为指定的key设定一个或多个field和value

hexists <key> <field> //在key里面是否存在指定的field
hkeys <key> //获取hash表所有key
hvals <key> //获取hash表所有值
hgetall <key> //获取hash表所有的键值对
hincrby <key> <field> <increment> //增加某个field的值,只能是数字
hsetnx <key> <field> <value> //当不存在才创建该field
hlen <key> //获取hash表中的字段数量
hdel <key> <field1> <field2> //删除一个或多个hash表的字段
```

### 5.ZSet

>在Set的基础上加上评分

```
zadd <key> <score1> <value1> //将一个或多个元素及其评分加入;相同元素，不同分数，会将分数更新
zrange <key> <start> <stop> [WITHSCORES] // 指定输出索引范围内的成员;[WITHSCORES]可以携带分数
zrevrange <key> <start> <stop> //返回有序集中指定区间内的成员，通过索引，分数从高到底
zrangebyscore <key> <min> <max> //指定输出score区间内的成员
zincrby <key> <increment> <value> //为value元素的分数加上增量
zrem <key> <value> //移除有序集合中的一个或多个成员
zcard <key> //获取集合中的元素数量
zcount <key> <min> <max> //计算在有序集合中指定区间分数的成员数
zrank <key> <value> 返回有序集合指定成员的排名
```

#### 使用zset实现文章访问量的排行

```
redis2:0>zadd test 100000 java 80000 python 50000 php 120000 c++ 20000 js
"5"

redis2:0>zrevrange test 0 -1
 1)  "c++"
 2)  "java"
 3)  "python"
 4)  "php"
 5)  "js"

redis2:0>zrevrangebyscore test 120000 10redis2:0>0000
 1)  "c++"
 2)  "java"

```

## 三.Redis的相关配置

1. ip地址绑定
   - 默认 “bind 127.0.0.1”只接受本机访问
   - 使其他机器访问解决方法
     1. 注释“bind 127.0.0.1” 取消绑定id
     2. “protected-mode yes” yes改为no 关闭保护模式
2. tcp-backlog 
   - 默认“tcp-backlog 511”
   - 可以理解为请求到达后至进程处理前的队列
3. timeout
   - 一个空闲的客户端维持多长时间(秒)会关闭，0代表永不关闭，默认0
4. tcp-keepalive
   - 对客户端的心跳检测 官方推荐60s
5. daemonize
   - 是否为后台进程
6. pidfile
   - 存放pid文件的位置
7. loglevel
   - 默认 loglevel notice
   - 日志级别 四个等级
8. database
   - 设定库的数量 默认16
9. maxclient
   - 最大客户端链接数
10. maxmemory
    - 设置内存量
    - 超过后 会根据maxmemory-policy指定移除
11. maxmemory-policy
    - volatile-lru  使用LRU（最近最少使用）算法移除key，只对设置了过期时间的键
    - allkeys-lru 使用LRU算法移除key
    - volatile-random 在过期集合中移除key，只对设置了过期时间的键
    - allkeys-random 随机移除
    - volatile-ttl(即将过期) 移除ttl最小的key 
    - noeviction 不移除(头铁 等着报错)
12. maxmemory-samples
    - 设置样本大小
    - 3-7 一般3或5

## 四.连接java(SSM)

1. jar：

   - jedis-xxx.jar

2. ```java
   package com.jsh;
   
   import redis.clients.jedis.Jedis;
   
   public class Test {
       public static void main(String[] args) {
           Jedis jedis = new Jedis("127.0.0.1",6379);
           String result = jedis.ping();
           System.out.println(result);
   //        jedis.set("a","a");
           jedis.get("a");
           System.out.println("a");
           jedis.close();
       }
   }
   
   ```

   

### 手机短信验证例子

1. 前端

```java
		          <form class="navbar-form navbar-left" role="search" id="codeform">
				  <div class="form-group">
				    <input type="text" class="form-control" placeholder="填写手机号" name="phone_no">
				    <button type="button" class="btn btn-default" id="sendCode">发送验证码</button><br>
				    <font id="countdown" color="red" ></font>
				    <br>
				    <input type="text" class="form-control" placeholder="填写验证码" name="verify_code">
				    <button type="button" class="btn btn-default" id="verifyCode">确定</button>
				    <font id="result" color="green" ></font><font id="error" color="red" ></font>
				    </div>
				    </form>
<script type="text/javascript"> 
var t=120;//设定倒计时的时间 
var interval;
function refer(){  
    $("#countdown").text("请于"+t+"秒内填写验证码 "); // 显示倒计时 
    t--; // 计数器递减 
    if(t<=0){
    	clearInterval(interval);
    	$("#countdown").text("验证码已失效，请重新发送！ ");
    }
} 
$(function(){
	$("#sendCode").click( function () {       	   $.post("/Verify_code/CodeSenderServlet",$("#codeform").serialize(),function(data){
	    	 if(data=="true"){
	    		 t=120;
	    		 clearInterval(interval);
	    		 interval= setInterval("refer()",1000);//启动1秒定时  
	   		 }else if (data=="limit"){
	   			clearInterval(interval);
	   			$("#countdown").text("单日发送超过次数！ ")
	   		 }
		  });   
    });
	
	$("#verifyCode").click( function () {
$.post("/Verify_code/CodeVerifyServlet",$("#codeform").serialize(),function(data){
	    	 if(data=="true"){
	    		 $("#result").attr("color","green");
	    		 $("#result").text("验证成功");
	    		 clearInterval(interval);
	    		 $("#countdown").text("");
	   		 }else{
	    		 $("#result").attr("color","red");
	    		 $("#result").text("验证失败");
	   		 }
		  });   
    });		
});
</script>
```

2. controller

   - 模拟接收短信验证码并缓存

     ```java
     @WebServlet("/CodeSenderServlet")
     public class CodeSenderServlet extends HttpServlet {
     	private static final long serialVersionUID = 1L;
            
         /**
          * @see HttpServlet#HttpServlet()
          */
         public CodeSenderServlet() {
             super();
             // TODO Auto-generated constructor stub
         }
     
     	/**
     	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     	 */
     	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
     		
     		//获取手机号
     		String phone_no = request.getParameter("phone_no");
     		//模拟获取验证码
     		String code = getCode(6);
     		//拼接key
     		String codeKey = "Verify_code:" + phone_no + ":code";//Verify_code:12345:code
     		String countKey = "Verify_code:" + phone_no + ":count";
     		
     		Jedis jedis = new Jedis("127.0.0.1", 6379);
     		//判断发送验证码的次数
     		String count = jedis.get(countKey);
     		if(count == null) {
     			//代表第一次
     			jedis.setex(countKey, 24*60*60, "1");
     		}else if(Integer.parseInt(count) <= 2) {
     			jedis.incr(countKey);
     		}else if(Integer.parseInt(count) > 2) {
     			response.getWriter().print("limit");
     			jedis.close();
     			return ;
     		}
     		//向redis中进行存储，以手机号为键，以验证码为值
     		jedis.setex(codeKey, 120, code);
     		jedis.close();
     		response.getWriter().print(true);
     	}
     	
     	//模拟短信
     	private String getCode(int length) {
     		String code = "";
     		Random random = new Random();
     		for(int i = 0; i < length; i++) {
     			int rand = random.nextInt(10);
     			code += rand;
     		}
     		return code;
     	}
     
     }
     
     ```

   - 从缓存中取出返回到前端

     ```java
     package com.atguigu.servlet;
     
     @WebServlet("/CodeVerifyServlet")
     public class CodeVerifyServlet extends HttpServlet {
     	private static final long serialVersionUID = 1L;
            
         /**
          * @see HttpServlet#HttpServlet()
          */
         public CodeVerifyServlet() {
             super();
             // TODO Auto-generated constructor stub
         }
     
     	/**
     	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
     	 */
     	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
     	
     		//获取验证码和手机号
     		String phone_no = request.getParameter("phone_no");
     		String verify_code = request.getParameter("verify_code");
     		//拼接key
     		String codeKey = "Verify_code:" + phone_no + ":code";
     		//从redis中获取手机号所对应的验证码
     		Jedis jedis = new Jedis("127.0.0.1", 6379);
     		String code = jedis.get(codeKey);
     		if(code.equals(verify_code)) {
     			response.getWriter().print(true);
     		}
     		jedis.close();
     		
     	}
     
     }
     
     ```

## 五.Redis事务

- 编译出错会直接取消 例如：单词拼写错误
- 某一步如果在运行时出错 只会取消那一步的操作

```
multi //开启事务
exec //提交事务
discard //取消事务
```

演示

```
redis2:0>multi
"OK"

redis2:0>set a a
"QUEUED"

redis2:0>set b c
"QUEUED"

redis2:0>exec
 1)  "OK"
 2)  "OK"
```

#### 监视

```
watch <key> //监视key 如果发现被改变 事务取消
unwatch // 取消对所有key的监视 如果exec或discard执行了 就不需要再执行unwatch
```

#### 三特性

1. 单独的隔离操作：事务中的所有命令都会序列化，按顺序的执行。事务在等待执行的时候，不会被其他客户端发送来的米命令请求打断
2. 没有隔离级别的概念：队列中的所有命令没有提交exec之前都是不会被执行的
3. 不保证原子性：redis中如果一条命令执行失败，其后的命令仍然会被执行，没有回滚

## 六.持久化

### RDB

> 在指定的时间间隔内生成内存中整个数据集的持久化快照。快照文件默认被存储在当前文件夹中，名称为`dump.rdb`，可以通过dir和dbfilename参数来修改默认值。
>
> 恢复数据特别快，节省空间，存储的是数据
>
> Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何的IO操作的，这就确保了极高的性能。
>
> 缺点：最后一次持久化后数据可能丢失

#### 配置文件

```
# redis是基于内存的数据库，可以通过设置该值定期写入磁盘。 
# 注释掉“save”这一行配置项就可以让保存数据库功能失效 
# 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化） 
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化） 
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化） save 900 1 
save 300 10 
save 60 10000 

#当RDB持久化出现错误后，是否依然进行继续进行工作，yes：不能进行工作，no：可以继续进行工作，可以通过info中的rdb_last_bgsave_status了解RDB持久化是否有错误 
stop-writes-on-bgsave-error yes 

#使用压缩rdb文件，rdb文件压缩使用LZF压缩算法，yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间 推荐压缩
rdbcompression yes 

#是否校验rdb文件。从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置。 
rdbchecksum yes 

#rdb文件的名称 
dbfilename dump.rdb 

#数据目录，数据库的写入会在这个目录。
dir ./      默认在当前文件夹下
```

#### 触发条件

1. 通过配制文件中的save条件（可自己配置）

   ```shell
   save 900 1
   save 300 10
   save 60 10000
   ```

2. 手动通过save和bgsave命令

- save：save时只管保存，其他不管，全部阻塞
- bgsave：redis会在后台异步的进行快照操作，同时还可以响应客户端请求。可以通过lastsave命令获取最后一次成功执行快照的事件

#### 如何恢复

- 关闭Redis
- 把备份的文件拷贝到工作目录下
- 启动Redis,会自动加载

### AOF

> 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作补不可记录），只许追加文件但不可以改写文件，redis启动之初会读取改文件重新构建数据。保存的是appendonly.aof文件
>
> aof机制默认关闭，可以通过`appendonly = yes`参数开启aof机制，通过`appendfilename = myaoffile.aof`指定aof文件名称。

```shell
#aof持久化策略的配置
#no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
#always表示每次写入都执行fsync，以保证数据同步到磁盘。
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。
appendfsync everysec
```

对于触发aof重写机制也可以通过配置文件来进行设置：

```shell
# aof自动重写配置。当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候Redis能够调用bgrewriteaof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。
auto-aof-rewrite-percentage 100
# 设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写
auto-aof-rewrite-min-size 64mb
```

```shell
# 在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题。如果对延迟要求很高的应用，这个字段可以设置为yes，，设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,不会造成阻塞的问题（因为没有磁盘竞争），等rewrite完成后再写入，这个时候redis会丢失数据。Linux的默认fsync策略是30秒。可能丢失30秒数据。因此，如果应用系统无法忍受延迟，而可以容忍少量的数据丢失，则设置为yes。如果应用系统无法忍受数据丢失，则设置为no。
no-appendfsync-on-rewrite no
```

#### 如何恢复

##### 正常恢复

​    将文件放到dir指定的文件夹下，当redis启动的时候会自动加载数据，注意：`aof文件的优先级比dump大`。

##### 异常恢复

- 有些操作可以直接到appendonly.aof文件里去修改。

  eg：使用了flushall这个命令，此刻持久化文件中就会有这么一条命令记录，把它删掉就可以了

- 写坏的文件可以通过 `redis-check-aof --fix`进行修复

#### 优势

1. 根据不同的策略，可以实现每秒，每一次修改操作的同步持久化，就算在最恶劣的情况下只会丢失不会超过两秒数据。
2. 当文件太大时，会触发重写机制，确保文件不会太大。
3. 文件可以简单的读懂

#### 劣势

1. aof文件的大小太大，就算有重写机制，但重写所造成的阻塞问题是不可避免的
2. aof文件恢复速度慢。

### 总结

1. 如果你只希望你的数据在服务器运行的时候存在，可以不使用任何的持久化方式

2. 一般建议同时开启两种持久化方式。AOF进行数据的持久化，确保数据不会丢失太多，而RDB更适合用于备份数据库，留着一个做万一的手段。

3. 性能建议：

   因为RDB文件只用做后备用途，建议只在slave上持久化RDB文件，而且只要在15分钟备份一次就够了，只保留900 1这条规则。

   如果Enalbe AOF,好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价：1、带来了持续的IO；2、AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。

   如果不Enable AOF,仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时宕掉，会丢失10几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构。

## 七.复制(master/slaver)

> 就是我们常说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主
>
>  读写分离
>
>  容灾恢复
>
> 主服务器用来写
>
> 从服务器用来读
>
> 不管何时 从机都和主机数据相同
>
> 主机shutdown后，从机等待

### 配置(配置从服务器，不配置主服务器)

- 拷贝多个redis文件include
- 开启daemonize yes
- Pid文件名字pidfile
- 指定端口port
- Log文件名字
- Dump.rdb名字dbfilename
- Appendonly关掉或者换名字
- 如果要永久 在配置文件中 配置 slaveof 

[![t8PKDH.png](https://s1.ax1x.com/2020/06/01/t8PKDH.png)](https://imgchr.com/i/t8PKDH)

命令

```
info replication //打印主从相关信息
slaveof <ip> <port> //成为某个实例的从服务器
slaveof no one //将从机变为主机
```

### 薪火相传

含义:就是上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master,可以有效减轻master的写压力。

### 哨兵模式

反客为主的自动版，能够后台监控Master库是否故障，如果故障了根据投票数自动将slave库转换为主库。一组sentinel能同时监控多个Master。

使用步骤：

1. 在Master对应redis.conf同目录下新建sentinel.conf文件，名字绝对不能错；

2. 配置哨兵，在sentinel.conf文件中填入内容(可以配置多个)：

   ```shell
   #说明：最后一个数字1，表示主机挂掉后slave投票看让谁接替成为主机，得票数多少后成为主机。
   sentinel monitor 被监控数据库名字（自己起名字） ip port 1
   ```

3. 启动哨兵模式(路径按照自己的需求进行配置)：

   ```shell
   redis-sentinel  /myredis/sentinel.conf
   ```

注意：

1. 当master挂掉后，会通过选票进行选出下一个master。而且只有使用了sentinel.conf启动的才能开启选票
2. 当原来的master后来后，很不幸变成了slave。

## 八.Redis集群（以后再学）

> 实现了水平扩容，启动N个节点，将整个数据库储存分布在N个节点中，每个节点存储数据的1/N

