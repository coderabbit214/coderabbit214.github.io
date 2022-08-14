# Mysql权限


# Mysql权限

## 一.用户管理

1. 新建用户

   > 用户地址：
   >
   > - 'localhost':本地主机
   > - '%':任何主机
   > - '%.mysql.com':mysql.com域的所有主机
   > - 只有user没有@'<用户地址>':代表%

   ```sql
   create user '<用户名>'@'<用户地址>' identified by '<密码>';
   ```

2. 更改用户

   ```sql
   rename user '<旧用户名>'@'<旧用户地址>' to '<新用户名>'@'<新用户地址>';
   ```

3. 更改密码

   ```sql
   # root用户更改别的用户密码
   set password for '<用户名>'@'<用户地址>' = password('<新密码>');
   # 非root用户修改自身密码
   set password = password('<新密码>');
   # 使用 alter user
   alter user '<用户名>'@'<用户地址>' identified by '<新密码>';
   ```

4. 删除用户

   ```sql
   drop user '<用户名>'@'<用户地址>';
   ```

## 二.权限管理

1. 查询mysql有哪些权限

   ```sql
   show privileges ;
   ```

2. 授权(在用户创建好的前提下)

   > 权限:(常用)
   >
   > ​	all: 以下所有权限
   >
   > ​	select: 查询
   >
   > ​	insert: 插入
   >
   > ​	update: 修改
   >
   > ​	detete: 删除
   >
   > 
   >
   > on <数据库.表>: '*'代表所有
   >
   > 
   >
   > [with grant option]: 
   >
   > ​	代表此用户可以将授给此用户的权限给别的用户

   ```sql
   grant <权限列表> on <数据库.表> to '<用户名>'@'<用户地址>' [with grant option];
   ```

3. 查看权限

   ```sql
   # 查看当前用户的权限
   show grants;
   # 查看其他用户权限
   show grants for '<用户名>'@'<用户地址>';
   ```

4. 回收权限

   ```sql
   revoke <权限列表> on <数据库.表> from '<用户名>'@'<用户地址>'; 
   ```

## 三.日期函数

1. 获取当前时间

   ```sql
   select now();
   ```

2. 日期时间 Extract（选取） 函数

   ```sql
   SELECT TIME('2017-05-15 10:37:14');-- 获取时间：10:37:14
   SELECT YEAR('2017-05-15 10:37:14');-- 获取年份
   SELECT MONTH('2017-05-15 10:37:14');-- 获取月份
   SELECT DAY('2017-05-15 10:37:14');-- 获取日
   SELECT HOUR('2017-05-15 10:37:14');-- 获取时
   SELECT MINUTE('2017-05-15 10:37:14');-- 获取分
   SELECT SECOND('2017-05-15 10:37:14');-- 获取秒
   SELECT MICROSECOND('2017-05-15 10:37:14');-- 获取毫秒
   SELECT QUARTER('2017-05-15 10:37:14');-- 获取季度
   SELECT WEEK('2017-05-15 10:37:14');-- 20 (获取周)
   SELECT WEEK('2017-05-15 10:37:14', 7);-- ****** 测试此函数在MySQL5.6下无效
   SELECT WEEKOFYEAR('2017-05-15 10:37:14');-- 同week()
   SELECT DAYOFYEAR('2017-05-15 10:37:14');-- 135 (日期在年度中第几天)
   SELECT DAYOFMONTH('2017-05-15 10:37:14');-- 5 (日期在月度中第几天)
   SELECT DAYOFWEEK('2017-05-15 10:37:14');-- 2 (日期在周中第几天；周日为第一天)
   SELECT WEEKDAY('2017-05-15 10:37:14');-- 0
   SELECT WEEKDAY('2017-05-21 10:37:14');-- 6(与dayofweek()都表示日期在周的第几天，只是参考标准不同，weekday()周一为第0天，周日为第6天)
   SELECT YEARWEEK('2017-05-15 10:37:14');-- 201720(年和周)
   ```

3. 时间加减

   ```sql
   -- DATE_ADD(date,INTERVAL expr type) 从日期加上指定的时间间隔
   -- type参数可参考：http://www.w3school.com.cn/sql/func_date_sub.asp
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 YEAR);-- 表示：2018-05-15 10:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 QUARTER);-- 表示：2017-08-15 10:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 MONTH);-- 表示：2017-06-15 10:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 WEEK);-- 表示：2017-05-22 10:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 DAY);-- 表示：2017-05-16 10:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 HOUR);-- 表示：2017-05-15 11:37:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 MINUTE);-- 表示：2017-05-15 10:38:14
   SELECT DATE_ADD('2017-05-15 10:37:14',INTERVAL 1 SECOND);-- 表示：2017-05-15 10:37:15
    
   -- DATE_SUB(date,INTERVAL expr type) 从日期减去指定的时间间隔
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 YEAR);-- 表示：2016-05-15 10:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 QUARTER);-- 表示：2017-02-15 10:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 MONTH);-- 表示：2017-04-15 10:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 WEEK);-- 表示：2017-05-08 10:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 DAY);-- 表示：2017-05-14 10:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 HOUR);-- 表示：2017-05-15 09:37:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 MINUTE);-- 表示：2017-05-15 10:36:14
   SELECT DATE_SUB('2017-05-15 10:37:14',INTERVAL 1 SECOND);-- 表示：2017-05-15 10:37:13
    
   -- 经特殊日期测试，DATE_SUB(date,INTERVAL expr type)可放心使用
   SELECT DATE_SUB(CURDATE(),INTERVAL 1 DAY);-- 前一天：2017-05-11
   SELECT DATE_SUB(CURDATE(),INTERVAL -1 DAY);-- 后一天：2017-05-13
   SELECT DATE_SUB(CURDATE(),INTERVAL 1 MONTH);-- 一个月前日期：2017-04-12
   SELECT DATE_SUB(CURDATE(),INTERVAL -1 MONTH);-- 一个月后日期：2017-06-12
   SELECT DATE_SUB(CURDATE(),INTERVAL 1 YEAR);-- 一年前日期：2016-05-12
   SELECT DATE_SUB(CURDATE(),INTERVAL -1 YEAR);-- 一年后日期：20178-06-12
   -- MySQL date_sub() 日期时间函数 和 date_add() 用法一致，并且可以用INTERNAL -1 xxx的形式互换使用；
   -- 另外，MySQL 中还有两个函数 subdate(), subtime()，建议，用 date_sub() 来替代。
    
   -- MySQL 另类日期函数：period_add(P,N), period_diff(P1,P2)
   -- 函数参数“P” 的格式为“YYYYMM” 或者 “YYMM”，第二个参数“N” 表示增加或减去 N month（月）。
   -- MySQL period_add(P,N)：日期加/减去N月。
   SELECT PERIOD_ADD(201705,2), PERIOD_ADD(201705,-2);-- 201707  20170503
   -- period_diff(P1,P2)：日期 P1-P2，返回 N 个月。
   SELECT PERIOD_DIFF(201706, 201703);--
   -- datediff(date1,date2)：两个日期相减 date1 - date2，返回天数
   SELECT DATEDIFF('2017-06-05','2017-05-29');-- 7
   -- TIMEDIFF(time1,time2)：两个日期相减 time1 - time2，返回 TIME 差值
   SELECT TIMEDIFF('2017-06-05 19:28:37', '2017-06-05 17:00:00');-- 02:28:37
   ```

