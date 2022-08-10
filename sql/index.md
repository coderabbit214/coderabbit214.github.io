# SQL


# SQL

## 创建数据库 : CREATE DATABASE 数据库名;

```sql
CREATE DATABASE teach;
```

## 进入数据库：USE 数据库名;

```sql
USE teach;
```

## 创建表：CREATE TABLE 表名;

​					属性名 属性类型 完整性约束,
​					属性名 属性类型 完整性约束,
​					属性名 属性类型 完整性约束	
​					);

>完整性约束：
>
>1. 主码约束: PRIMARY KEY 
>2. 参照完整性约束：FOREIGN KEY...REFERENCES...
>3. 唯一性约束: UNIQUE
>4. 非空值约束：NOT NULL
>5. 取值约束：CHECK
>
>属性类型：
>
>1. 整数
>   - bigint: 以8个字节来存储正负数, 范围：-2 63 到 2 63 -1 
>   - int: 以4个字节来存储正负数，范围：-2 31 到 2 31 -1 
>   - smallint: 以2个字节来存储正负数.，范围：-2 15 到 2 15 -1 
>   - tinyint: 是最小的整数类型,存储正整数，仅用1字节,范围:0至2 8 -1 
>   - bit: 值只能是0或1，当输入0以外的其他值时，系统均认为是1 常用来表示真假、男女等二值选择。
>2. 精确数值
>   - decimal:用来存储从-1038+1到1038 -1的固定精度和范围的数值型数据 • 必须指定范围和精度：decimal (p,q) 例：decimal (10,2) 
>   - numeric:和decimal相同
>3. 浮点
>   - float: 用8个字节来存储数据.最多可为53位. 范围为:-1.79E+308至1.79E+308. 
>   - real: 位数为24,用4个字节 数字范围:-3.04E+38至3.04E+38
>4. 字符串
>   - char: char(n)固定的长度为 n个字符的字符串, 不足的长度会用空格 补上. 
>   - varchar: varchar(n)可变的最长长度为n个字符的字符串，尾部的空 格会去掉.
>5. 时间日期
>   - date: 日期类型 • DATE 'yyyy-mm-dd' • Example: DATE '2004-09-30' 
>   - time:时间类型 • TIME 'hh:mm:ss' • Example: TIME '15:30:02.5‘ 
>   - datetime:日期时间类型

```sql
CREATE TABLE student(
sid CHAR(8) PRIMARY KEY,
sname VARCHAR(20) NOT NULL,
sgender CHAR(1),
sdept INT,
sbirth DATE
);
```

> 当多个属性为主键时

```sql
CREATE TABLE SC(
Sno CHAR(5) ,
Cno INT ,
Grade INT,
PRIMARY KEY (Sno, Cno));
```

## 修改基本表

ALTER TABLE <表名>

 [ ADD <新列名> <数据类型> | 完整性约束 ]

 [ DROP <列名>|<完整性约束名> ]

 [ MODIFY <列名> <数据类型> ]；

> - ADD子句：增加新列和新的完整性约束条件 
> - DROP子句：删除指定列或完整性约束条件 
> - MODIFY子句：用于修改列名和数据类型

### ADD

ALTER TABLE 表名 ADD 属性名 属性类型;

```sql
ALTER TABLE student ADD stime DATETIME;
```

### DROP

ALTER TABLE 表名 DROP 属性名;

ALTER TABLE Student DROP 主键约束的名字;

```sql
ALTER TABLE student DROP STIME;
```

### MODIFY

更改数据类型

```sql
ALTER TABLE student MODIFY sname VARCHAR(20);
```



### 删除基本表

DROP TABLE <表名>;

```sql
DROP TABLE student;
```

## 增删改查

### SELECT

> ```sql
> SELECT ALL|DISTINCT(默认ALL(不去重) DISTINCT(去重)) 属性名(查询目标列或者表达式)
> FROM 表名（从哪个表中查询）
> WHERE 分组前条件
> GROUP BY 属性名（根据属性值相同进行分组）
> HAVING 表达式（分组后条件）
> ORDER BY 属性名 ASC|DESC（根据属性名进行排序 默认升序(ASC) 降序(DESC)）
> ```

1. 查询全体学生的学号和姓名

   ```sql
   SELECT sno,sname FROM student; 
   ```

2. 查询全体学生的详细信息

   ```sql
   SELECT * FROM student; 
   ```

3. 查询全体学生的出生日期  NOW()取得当前时间

   ```sql
   SELECT 2020-sage FROM student; 
   ```

   > 查询出的值可以进行运算

   ```sql
   SELECT YEAR(NOW())-sage FROM student; 
   ```

   > NOW()取得当前时间

4. 对查询出的结果的列取别名 AS(可以省略)

   ```sql
   SELECT YEAR(NOW())-sage AS birthyear FROM student; 
   SELECT sno AS s,sname AS b FROM student; 
   ```

5. 去重 DISTINCT

   ```sql
   SELECT DISTINCT sno FROM sc; 
   ```

6. 查询结果 变小写 变大写

   ```sql
   SELECT LOWER(sdept) FROM student; 
   SELECT UPPER(sdept) FROM student; 
   ```

7.  where语句 查询所有年龄在20岁以下的学生姓名及其年龄

   ```sql
   SELECT sname,sage FROM student WHERE sage<20; 
   ```

8. 属性 BETWEEN 条件1 AND 条件2;

   ```sql
   --[例9] 查询年龄在20~23岁(包括20岁和23岁)之间的学生的姓名、系别和年龄
   SELECT sname,sdept,sage FROM student WHERE sage BETWEEN 20 AND 23;
   --[例10]  查询年龄不在20~23岁之间的学生姓名、系别和年龄
   SELECT sname,sdept,sage FROM student WHERE sage NOT BETWEEN 20 AND 23;
   ```

9. 属性名 IN('属性一','属性二','属性三')

   ```sql
   --[例11]  查询信息系(IS)、数学系(MA)和计算机科学系(CS)学生的姓名和性别
   SELECT sname,ssex FROM student WHERE sdept IN('CS','IS','MA');
   --[例12]查询既不是信息系、数学系，也不是计算机科学系的学生的姓名和性别
   SELECT sname,ssex FROM student WHERE sdept NOT IN('CS','IS','MA');
   ```

10. like 模糊查询 %表示若干字符 _表示一个字符

    > 避免原字符串中存在%或_的方法: 例十八 ，例十九

    ```sql
    --[例14]  查询所有姓刘学生的姓名、学号和性别
    SELECT sname,sno,ssex FROM student WHERE sname LIKE '刘%';
    --[例15]  查询所有不姓刘的学生姓名、学号和性别
    SELECT sname,sno,ssex FROM student WHERE sname NOT LIKE '刘%';
    --[例16]  查询姓"欧阳"且全名为三个汉字的学生的姓名
    SELECT sname FROM student WHERE sname LIKE '欧阳_';      
    --[例17]  查询名字中第2个字为"阳"字的学生的姓名和学号
    SELECT sname FROM student WHERE sname LIKE '_阳%'; 
    
    --[例18]  查询DB_Design课程的课程号和学分。
    SELECT cno,ccredit FROM course WHERE cname LIKE 'DB\_Design' ESCAPE'\\';       
    --[例19]  查询以"DB_"开头，且倒数第3个字符为 i的课程的详细情况
    SELECT * FROM course WHERE cname LIKE 'DB\_%i__' ESCAPE'\\';
    ```

11. IS NULL 判断为空 不可以使用 = NULL

    ```sql
    --[例20]  某些学生选修课程后没有参加考试，所以有选课记录，但没有考试成绩。查询缺少成绩的学生的学号和相应的课程号。
    SELECT sno,cno FROM sc WHERE grade IS NULL;
    -- [例21]  查所有有成绩的学生学号和课程号
    SELECT sno,cno FROM sc WHERE grade IS NOT NULL;  
    ```

12. 排序 ORDER BY 默认为升序 加上DESC变为降序

    ```sql
    --[例25]  查询选修了3号课程的学生的学号及其成绩，
    --查询结果按分数降序排列
    SELECT sno,grade FROM sc ORDER BY grade DESC;
    --[例26]  查询全体学生情况，查询结果按所在系的系号升序排列，同一系中的学生按年龄降序排列。
    SELECT * FROM student ORDER BY sdept,sage DESC;  
    ```

13. COUNT(属性名) 返回数量 COUNT(*)表示全部

    > 可以使用DISTINCT

    ```sql
    --[例27]  查询学生总人数。
    SELECT COUNT(*) FROM student;
    --[例28]  查询选修了课程的学生人数。
    SELECT COUNT(DISTINCT SNO) FROM sc;
    ```

14. 平均值AVG 最大值MAX 最小值MIN

    ```sql
    --[例29]  计算1号课程的学生平均成绩
    SELECT AVG(Grade) FROM sc WHERE cno=1;
    --[例30]  查询选修1号课程的学生最高分数
    SELECT MAX(Grade) FROM sc WHERE cno=1;
    ```

15. 分组 GROUP BY 属性名

    ```sql
    --[例31]  求各个课程号及相应的选课人数。
    SELECT cno,COUNT(sno) FROM sc GROUP BY cno;
    
    --[例32]  求各个课程号及相应的课程成绩在90分以上的学生人数
    SELECT cno,COUNT(*) FROM sc WHERE grade>=90 GROUP BY cno;
    ```

16. HAVING 对分组后的数据进行判断

    ```sql
    --[例33]  查询选修了3门以上课程的学生学号
    SELECT sno FROM sc GROUP BY sno HAVING COUNT(*)>=3;     
    
    --[例34]  查询有3门以上课程在90分以上的学生的学号
    --及90分以上的课程数
    SELECT sno,COUNT(*) FROM sc WHERE grade>=90 GROUP BY sno HAVING COUNT(*)>=3;          
    
    --[例35] 统计每门课程的最高分
    SELECT cno,grade FROM sc GROUP BY cno HAVING MAX(Grade);
    ```

17. 关联查询

    ```sql
    --[例32]  查询每个学生及其选修课程的情况。
    SELECT student.*,sc.* FROM student,sc;
    --[例33]  对[例32]用自然连接完成
    
    SELECT student.*,sc.* FROM student,sc WHERE student.sno=sc.sno;
    --例：查询计算机系（CS）学生的学号，姓名，所在系，
    --选修的课程号，课程名和成绩
    SELECT sc.sno,sname,sdept,sc.cno,cname,grade FROM student,sc,course WHERE student.sno=sc.sno AND course.cno=sc.cno AND sdept='CS';
    
    -- [例34]  查询每一门课的直接先修课的课程名
    SELECT a1.`Cname`,a2.`Cname` FROM course a1,course a2 WHERE a1.Cpno=a2.Cno;
    
    
    --[例35]  查询每一门课的间接先修课的课程号（即先修课的先修课）。
    SELECT a1.cno,a2.cpno FROM course a1,course a2 WHERE a1.Cpno=a2.Cno;     
    
    --[例36]  查询同时选修2号课程和3号课程学生的学号。
    SELECT c1.sno FROM sc c1,sc c2 WHERE c1.sno = c2.sno AND c1.cno!=c2.cno AND c1.cno=2 AND c2.cno =3; 
    ```

18. 左连接查询 left join  右连接 right join

    > 左连接，左表的记录将会全部表示出来，而右表只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

    ```sql
    --  例，查询全体学生信息及其选课信息
    
    SELECT student.*,sc.* FROM student LEFT JOIN sc ON student.sno=sc.sno;
    
    ```

19. 嵌套查询

    ```sql
    --  [例37]  查询与“刘晨”在同一个系学习的学生。
    --  ① 确定“刘晨”所在系名             
    SELECT Sdept FROM student WHERE sname = '刘晨';
    
    --  ② 查找所有在IS系学习的学生。    
    
    SELECT * FROM student WHERE sdept IN
    (SELECT Sdept FROM student WHERE sname = '刘晨');
    
    -- [例39]查询其他系中比IS系任一学生年龄小的学生姓名和年龄
    SELECT sname,sage FROM student WHERE sage<(
    SELECT MIN(sage) FROM student);
    
    -- [例39]  找出每个学生超过他选修课程平均成绩的课程号
        
    SELECT sno,cno FROM sc c1 WHERE grade>
    (SELECT AVG(Grade) FROM sc c2 WHERE c1.sno = c2.sno);
    
    -- [例40]查询其他系中比IS系所有学生年龄都大的学生姓名和年龄
    
    SELECT sname,sage FROM student WHERE sage> (
    SELECT MAX(sage) FROM student WHERE sdept = 'IS'
    );
    
    ```

20. NOT EXISTS/EXISTS

    >理解: NOT EXISTS 查询出所有结果后排除子句中的结果
    >
    >EXISTS查询出所有结果后选择子句中的结果

    ```sql
    -- 查询没有选择1号课程的学生名字
    SELECT sname FROM student s WHERE NOT EXISTS (
    SELECT * FROM sc WHERE sno=s.sno AND cno=1
    );
    ```


### INSERT

> INSERT INTO 表名(属性名1，属性名2......) 
>
> VALUES(属性值1，属性值二......);

```sql
/*[例1]将一个新学生记录(学号:95020;姓名:陈冬;
性别:男;所在系:IS;年龄:18岁)插入到Student表中
*/
INSERT INTO student VALUES('95020','陈冬','M',18,'IS');
-- [例2]  插入一条选课记录( '95020'，'1 ')
INSERT INTO sc(sno,cno) VALUES ('95020','1');
```

> 可以插入子查询的值

```sql
-- [例3] 对每一个系，求学生的平均年龄,
-- 并把结果存入数据表
-- 第一步：建表
   /*学生平均年龄*/
CREATE TABLE studentAvgAge(
	sdept VARCHAR(20),
	avgage SMALLINT
);
-- 第二步：插入数据
INSERT INTO studentAvgAge SELECT sdept,AVG(sage) FROM student GROUP BY sdept; 
```

### UPDATE

> UPDATE 表名
>
> SET 属性名=属性值
>
> WHERE 条件；

```sql
--[例4]  将学生95001的年龄改为22岁。
UPDATE student SET sage = 22 WHERE sno='95001';
-- 将学生95001的年龄改为21岁,性别改为女性。
UPDATE student SET sage = 21,ssex='F' WHERE sno='95001';
--[例5]  将所有学生的年龄增加1岁
UPDATE student SET sage=sage+1;
--[例6]  将信息系所有学生的年龄增加1岁      
UPDATE student SET sage=sage+1 WHERE sdept='CS';
--[例7]  将计算机科学系全体学生的成绩置零
UPDATE sc SET grade = 0 WHERE sno IN(SELECT sno FROM student WHERE sdept='IS');
```

### DELETE

> DELETE FROM 表名
>
> WHERE 条件
>
> > TRUNCATE删除所有数据保留表

```sql
--[例8]  删除学号为95019的学生记录
DELETE FROM student WHERE sno='95020';
--[例9]  删除2号课程的所有选课记录       
DELETE FROM sc WHERE cno='2';
--[例10]  删除所有的学生选课记录
-- 删除所有数据
TRUNCATE TABLE sc;
--[例11]  删除计算机科学系所有学生的选课记录。
DELETE FROM sc WHERE sno IN (SELECT sno FROM student WHERE sdept='IS');
```


