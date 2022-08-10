# 后端-MyBatis准备（JDBC回顾）


# 后端-MyBatis准备（JDBC回顾）

jdbc（Java DataBase Donnectivity）可以为多种关系型数据库

## JDBC

- 通过操作java程序 操作 JDBC DriverManager 操作 数据库驱动程序 操作 数据库
- JDBC API ：提供各种操作访问接口，Connection Statement PreparedStatement ResultSet
- JDBC DriverManager 管理不同的数据库驱动
- 各种数据库驱动：相应数据库厂商提供 作用：链接或者直接操作数据库

## JDBC API

### 作用

1. 建立连接
2. 发送sql语句
3. 返回处理结果

### DriverManager : 管理JDBC驱动

### Connection : 连接

### Statement (PreparedStatement) : 增删改 查

### CallableStatement : 调用数据库中的存储函数

### ResultSet : 返回的结果集

## JDBC访问数据库具体步骤

1. 导入数据库，加载具体驱动类
2. 与数据库建立连接
3. 发送sql，执行
4. 处理结果集

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Scanner;

public class JDBCDemo2 {
	private static final String URL = "jdbc:mysql://localhost:3306/testdata";
	private static final String USERNAME = "root";
	private static final String PWD = "123456";

	public static void update() {// 增删改
		Connection connection = null;
		Statement stmt = null;
		try {
			// a.导入驱动，加载具体的驱动类
			Class.forName("com.mysql.jdbc.Driver");// 加载具体的驱动类
			// b.与数据库建立连接
			connection = DriverManager.getConnection(URL, USERNAME, PWD);
			// c.发送sql，执行(增删改、查)
			stmt = connection.createStatement();
			//String sql = "insert into student values(1,'zs',23,'s1')";
//			String sql = "update student set STUNAME='ls' where stuno=1";
			String sql = "delete from student where stuno=1";
			// 执行SQL
			int count = stmt.executeUpdate(sql); // 返回值表示 增删改 几条数据
			// d.处理结果
			if (count > 0) {  
				System.out.println("操作成功！");
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} catch(Exception e) {
			e.printStackTrace();
		}
		finally {
			try {
				 if(stmt!=null) stmt.close();// 对象.方法
				 if(connection!=null)connection.close();
			}catch(SQLException e) {
				e.printStackTrace();
			}
		}
	}
	
	
	public static void query() {
		Connection connection = null;
		Statement stmt = null;
		ResultSet rs = null ; 
		try {
			// a.导入驱动，加载具体的驱动类
			Class.forName("com.mysql.jdbc.Driver");// 加载具体的驱动类
			// b.与数据库建立连接
			connection = DriverManager.getConnection(URL, USERNAME, PWD);
			// c.发送sql，执行(增删改、【查】)
			stmt = connection.createStatement();
//			String sql = "select stuno,stuname from student";
			Scanner input= new Scanner(System.in);
			System.out.println("请输入用户名：");
			String name = input.nextLine() ;
			System.out.println("请输入密码：");
			String pwd = input.nextLine() ;
			String sql = "select count(*) from login where uname='"+name+"' and upwd ='"+pwd+"' " ;
//			String sql = "select * from student where stuname like '%"+name+"%'";
			// 执行SQL(增删改executeUpdate()，查询executeQuery())
			rs = stmt.executeQuery(sql); // 返回值表示 增删改 几条数据
			// d.处理结果
			int count = -1;
			if(rs.next()) {
				count = rs.getInt(1) ;
			}
			if(count>0) {
				System.out.println("登陆成功！");
			}else {
				System.out.println("登陆失败！");
			}

		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} catch(Exception e) {
			e.printStackTrace();
		}
		finally {
			try {
				if(rs!=null) rs.close(); 
				 if(stmt!=null) stmt.close();// 对象.方法
				 if(connection!=null)connection.close();
			}catch(SQLException e) {
				e.printStackTrace();
			}
		}
	}
	
	
	public static void main(String[] args) {
//		update() ;
		query() ;
	}
}

```

## 数据库驱动

|  数据库   |                  具体驱动类                  |                          连接字符串                          |
| :-------: | :------------------------------------------: | :----------------------------------------------------------: |
|   mysql   |            com.mysql.jdbc.Driver             |            jdbc:mysql://localhost:3306/数据库名称            |
|  oracle   |           oracle.jdbc.OracleDriver           |         jdbc:oracle:thin:@localhost:1521:数据库名称          |
| sqlserver | com.microsoft.sqlserver.jdbc.SQLServerDriver | jdbc:microsoft.sqlserver:localhost:1433;databasename=数据库名称 |


