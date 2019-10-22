---
title: JDBC
date: 2019-09-04 23:02:25
categories: Java
---

> JDBC 的使用

<!-- more -->

### 1. 定义

> JDBC: 定义了一套操作所有关系型数据库的规则(接口)

> JDBC 只是定义了接口，并没有去实现，对应不同的 数据库，Orcale， Mysql，DB2 等，需要厂商自己实现之二写接口(又称为 驱动)

### 2. 快速使用

#### 2.1 导入驱动Jar包

1. 新建 lib 目录
2. 将 mysql-connector-java-5.1.37-bin.jar 放入 lib 目录
3. 选中jar包，右键 Add as library

#### 2.2 注册驱动

> 告诉程序应该使用哪一个数据库驱动 jar包

```java
/* 2. 注册驱动 */
// 内部使用反射的方式，创建了 一个对象，实现了 jdbc的接口
Class.forName("com.mysql.jdbc.Driver");
```

- 底层操作

> 使用 Class.forName 肯定是会使用 反射的方式 自动创建 该驱动对应的对象
>
> 驱动的注册是要调用 DriverManager.registerDriver 来完成的，那么 在该类创建的时候肯定封装了该方法，使得在 使用的反射的时候，可以自动的 调用 DriverManager.registerDriver 

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    // 使用静态代码块，底层调用 DriverManager.registerDriver 来完成
    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

##### 注意:

> 在mysql5之后的驱动 jar吧，内部自动定义了 java.sql.Driver 文件，内部包含了 驱动名称，自动可以完成。因此可以省略该步骤

#### 2.3 获取连接对象

```java
/* 3. 获取连接 
	参数: url => 指定连接的路径
			格式为: jdbc:mysql://ip地址(域名):端口号/数据库名称
			如果本机mysql，可简写为 jdbc:mysql:///数据库名称
		如果数据库出现乱码，可以指定参数 ？characterEncoding=utf8
*/
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db1", "root", "123456");
```

#### 2.4 创建 statement

> 用来执行 sql 的对象

```java
Statement statement = conn.createStatement();
```

#### 2.5 执行 statement

```java
String sql = "UPDATE balance set money=1000 where id=1";
int res = statement.executeUpdate(sql);
System.out.println(res);
```

其中:

```java
// 用来执行 select 语句
ResultSet executeQuery(String sql)  ：执行DQL（select)语句
// 其中 ResultSet 是 结果集对象，用来封装查询结果
```

```java
// 用来执行 DML语句 （insert， update， delete ）
// 也可以执行 DDL 语句 (create, alter, drop)
// 其中 int 返回值为 影响 的行数，改值用来判断 DML 语句是否成功，如果 > 0 则表示成功；否则失败
int executeUpdate(String sql) 
```

#### 2.6 关闭连接

```java
statement.close();
conn.close();
```

#### 更新数据demo

```java
public class Demo1 {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        /*1. 导入jar包*/

        /* 2. 注册驱动 */
        Class.forName("com.mysql.jdbc.Driver");

        /* 3. 获取连接 */
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db1", "root", "123456");

        /* 创建 statement */
        Statement statement = conn.createStatement();
        String sql = "UPDATE balance set money=1000 where id=1";
        /* 执行 Update */
        int res = statement.executeUpdate(sql);
        System.out.println(res);

        /* 关闭连接 */
        statement.close();
        conn.close();
    }
}
```

#### 插入数据demo

```java
public class Demo1 {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        /*1. 导入jar包*/

        /* 2. 注册驱动 */
        Class.forName("com.mysql.jdbc.Driver");

        /* 3. 获取连接 */
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db1", "root", "123456");

        String sql = "insert into balance (money) values (22222)";
        /* 创建 statement */
        Statement statement = conn.createStatement();

        /* 执行 Update */
        int res = statement.executeUpdate(sql);
        System.out.println(res);

        /* 关闭连接 */
        statement.close();
        conn.close();
    }
}
```

#### 查询Demo

#### 2.7 ResultSet

> 结果集对象，封装的是查询结果

- next()

> 游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true

> 初试时，游标在第一行前面，需要先执行一次 next()

- getXXX(参数)

> 获取数据
>
> 其中 XXX 代表 需要返回数据的类型，比如 int getInt();   String getString();
>
> 参数：
>
> ​	第一种： int 类型，代表查询列的编号，从1 开始，比如  getString(1); ==> 获取第一列的内容，该内容为 string类型
>
> ​	第二种:  string类型，表示需要查询列的名称，比如 getString("name"); ==> 获取 name 列的值

```java
public class Demo1 {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        /*1. 导入jar包*/

        /* 2. 注册驱动 */
        Class.forName("com.mysql.jdbc.Driver");

        /* 3. 获取连接 */
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db1", "root", "123456");

        String sql = "select * from balance";
        /* 创建 statement */
        Statement statement = conn.createStatement();

        /* 执行 Update */
        ResultSet resultSet = statement.executeQuery(sql);
        while(resultSet.next()) {
            System.out.println("id" + resultSet.getInt("id") + " money"+ resultSet.getInt(2));
        }
        /* 关闭连接 */
        statement.close();
        conn.close();
    }
}
```

#### 2.8 PreparedStatement

> 是 Statemtent 接口的子接口

- 与 Statement 比较

> Statement 每次执行SQL 语句，都先将 SQL 语句发送给 Mysql，mysql编译后，执行；
>
> 如果有多条类似语句，只是关键参数变化，仍然需要编译多次

> prepareStatement 首先将 SQL 发送给 数据库编译，后续会引用已变异的结果；
>
> 并且，可以解决SQL注入问题

- sql 注入问题

```
在拼接sql时，有一些sql的特殊关键字参与字符串的拼接。会造成安全性问题
	1. 输入用户随便，输入密码：a' or 'a' = 'a
	2. sql：select * from user where username = 'fhdsjkf' and password = 'a' or 'a' = 'a' 
```

- 使用

```java
String sql1 = "update balance set money= money-? where id=?";
preStmt1 = conn.prepareStatement(sql1);
/*
setXXX(index, value);
	其中: XXX 表示需要设置值的类型,比如 setInt setFloat 等等
	index 表示 设置 第几个 ? 的值， 计数从 1 开始
	value 表示 替换 ? 的值
*/
preStmt1.setInt(1, 100);
preStmt1.setInt(2, 1);
```

### 3. 抽取 JDBC 工具类

- 抽取目的

> 当前写法中，有很多重复内容，用来简化书写

- 分析
  1. 注册驱动抽取（由于只需要注册一次，可放入静态代码块执行)
  2. 获取连接对象抽取
  3. 释放资源抽取
  4. 使用配置文件来替代 获取连接中手动输入的 url + username + passwd

- 配置文件

> src目录下的 db.properties 文件

```
jdbc.Driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///db1
jdbc.user=root
jdbc.passwd=123456
```

- 实现

```java
package com.day.utils;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JdbcUtils {
    private static String driverName;
    private static String url;
    private static String user;
    private static String passwd;
    static {
        try {
            /* 读取配置文件, 并转化为输入流 */
            InputStream is = JdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");

            Properties pro = new Properties();
            pro.load(is);

            driverName = pro.getProperty("jdbc.Driver");
            url = pro.getProperty("jdbc.url");
            user = pro.getProperty("jdbc.user");
            passwd = pro.getProperty("jdbc.passwd");
		   // 注册驱动方法
            Class.forName(driverName);
        } catch (ClassNotFoundException | IOException e) {
            e.printStackTrace();
        }
    }
	// 获取连接方法
    public static Connection getConnection() {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(url, user, passwd);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }

    public static void close(Connection conn) {
        close(conn, null, null);
    }
	// 关闭资源方法
    public static void close(Connection conn, Statement st, ResultSet rt) {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (st != null) {
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (rt != null) {
            try {
                rt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}

```

- 使用

```java
public class Demo1 {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        /* 3. 获取连接 */
        Connection conn = JdbcUtils.getConnection();

        String sql = "select * from balance";
        /* 创建 statement */
        Statement statement = conn.createStatement();

        /* 执行 Update */
        ResultSet resultSet = statement.executeQuery(sql);
        while(resultSet.next()) {
            System.out.println("id" + resultSet.getInt("id") + " money"+ resultSet.getInt(2));
        }
        /* 关闭连接 */
        JdbcUtils.close(conn, statement, resultSet);
    }
}
```



### 4. JDBC 控制事务

#### 4.1 事务

> 一个包含多个步骤的业务操作。如果这个业务操作被事务管理，则这多个步骤要么同时成功，要么同时失败。

> JDBC 中 使用 Connection 对象来管理事务

#### 4.2 API

- 开启事务

```java
conn.setAutoCommit(false)
```

- 提交事务

```java
conn.commit()
```

- 回滚事务

```
conn.rollback()
```

#### Demo

> 在Exception 的 catch 中，进行 事务的回滚

```java
public class Demo1 {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement preStmt1 = null;
        PreparedStatement preStmt2 = null;
        try {
            /* 获取连接 */
            conn = JdbcUtils.getConnection();
            /* 开启事务*/
            conn.setAutoCommit(false);

            String sql1 = "update balance set money= money-? where id=?";
            String sql2 = "update balance set money= money+? where id=?";

            /* 创建 statement */
            preStmt1 = conn.prepareStatement(sql1);
            preStmt1.setInt(1, 100);
            preStmt1.setInt(2, 1);
            preStmt2 = conn.prepareStatement(sql2);
            preStmt2.setInt(1, 100);
            preStmt2.setInt(2, 2);

            /* 执行 Update */
            int i1 = preStmt1.executeUpdate();
            int i2 = preStmt2.executeUpdate();
            System.out.println(i1);
            System.out.println(i2);
            /* 正常提交事务 */
            conn.commit();
        } catch (Exception e) {
            /* 撤销提交 */
            try {
                if (conn != null) {
                    conn.rollback();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            /* 关闭连接 */
            JdbcUtils.close(conn, preStmt1, null);
            JdbcUtils.close(null, preStmt2, null);
        }
    }
}
```

### 5. 数据库连接池

#### 概念

> 其实就是一个容器(集合)，存放数据库连接的容器

> 当系统初始化好后，容器被创建，容器中会申请一些连接对象，当用户来访问数据库时，从容器中获取连接对象，用户访问完之后，会将连接对象归还给容器

#### 好处

- 节约资源
- 访问更加高效

#### 标准接口(DataSource)

javax.sql 包下, 一般都有数据库厂商来进行实现

- 获取连接

```java
getConnection()
```

- 归还连接

> 如果连接对象Connection是从连接池中获取的，那么调用Connection.close()方法，则不会再关闭连接了。而是归还连接

```
conn.close()
```

### 6. C3P0(数据库连接池)

#### 6.1 导入jar包

```java
c3p0-0.9.5.2.jar
mchange-commons-java-0.2.12.jar
mysql-connector-java-5.1.37-bin.jar 
```

#### 6.2 定义配置文件

- 名称(固定)

```
c3p0.properties 或者 c3p0-config.xml
```

- 存放目录

```
src目录之下
```

- 配置文件内容

```xml
<c3p0-config>
  <!-- 使用默认的配置读取连接池对象 -->
  <default-config>
  	<!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db1</property>
    <property name="user">root</property>
    <property name="password">123456</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">10</property>
    <property name="checkoutTimeout">3000</property>
  </default-config>

  <named-config name="otherc3p0"> 
    <!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/day25</property>
    <property name="user">root</property>
    <property name="password">root</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">8</property>
    <property name="checkoutTimeout">1000</property>
  </named-config>
</c3p0-config>
```



#### 6.3 创建连接池对象

```java
DataSource ds = new ComboPooledDataSource();
```

#### 6.4 从连接池获取连接

```java
Connection conn = ds.getConnection();
```

#### 6.5 使用c3p0的 JdbcUtils

```java
public class JdbcUtils {
    private static DataSource ds;
    static {
        ds = new ComboPooledDataSource();
    }

    public static Connection getConnection() {
        Connection conn = null;
        try {
            conn = ds.getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }

    public static void close(Connection conn) {
        close(conn, null, null);
    }

    public static void close(Connection conn, Statement st, ResultSet rt) {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (st != null) {
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (rt != null) {
            try {
                rt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }


```

### 7. Druid(数据库连接池)

#### 7.1 导入jar包

```java
druid-1.0.9.jar
mysql-connector-java-5.1.37-bin.jar 
```

#### 7.2 定义配置文件

1. 配置文件是 properties 文件
2. 名称和位置任意，可以随意放置

#### 7.3 获取连接池对象

```java
DataSource ds = DruidDataSourceFactory.createDataSource(pro);
```

#### 7.4 获取连接

```java
 Connection conn = ds.getConnection();
```

#### 7.5 使用 druid 的 JdbcUtils

```java
public class JdbcUtils {
    private static DataSource ds;
    static {
        try {
            Properties pro = new Properties();
            InputStream is = JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties");
            pro.load(is);
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    public static void close(Connection conn) {
        close(conn, null, null);
    }

    public static void close(Connection conn, Statement st, ResultSet rt) {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (st != null) {
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (rt != null) {
            try {
                rt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```

### 8. Spring JDBC

> Spring框架对JDBC的简单封装。提供了一个JDBCTemplate对象简化JDBC的开发

#### 8.1 导入jar包

```java
commons-logging-1.2.jar
spring-beans-5.0.0.RELEASE.jar
spring-core-5.0.0.RELEASE.jar
spring-jdbc-5.0.0.RELEASE.jar
spring-tx-5.0.0.RELEASE.jar
mysql-connector-java-5.1.37-bin.jar
```

#### 8.2 创建 JdbcTemplate 对象

> 需要传入DataSource

```java
JdbcTemplate tmplate = new JdbcTemplate(ds);
```

#### 8.3 插入数据

> 使用 update 方法，同时可以传入 ？ 站位的参数

```java
/* 插入数据 */
    public static void Insert() {
        String sql = "insert into balance (money) values (?)";
        int res = tmplate.update(sql, 222333);
        System.out.println(res);
    }
```

#### 8.4 删除数据

```java
public static void Delte(){
        String sql = "delete from balance where money=?";
        int res = tmplate.update(sql,222333);
        System.out.println(res);
    }
```

#### 8.5 查询一条记录

- queryForMap
  1. 查询的是一条记录
  2. 返回时 key, value 的 map，key为列名，value为对应的值

```java
/* 查询一条记录 */
    public static void QueryOne(){
        String sql = "select * from balance where id=?";
        /* 返回一条记录，所有 列组成一个 map */
        Map<String, Object> b = tmplate.queryForMap(sql, 1);
        System.out.println(b); // 打印结果 {id=1, money=600.0
    }
```

#### 8.6 查询多条记录(结果为 list<map>)

```java
/* 查询多条，封装为 list */
    public static void QueryAll() {
        String sql = "select * from balance";
        /* 返回多条，每条是一个map*/
        List<Map<String, Object>> maps = tmplate.queryForList(sql);
        for(Map<String, Object> m :maps) {
            System.out.println(m);
        }
    }
```

#### 8.7 查询多条记录(手动封装为对象)

- query

> 查询结果，将结果封装为JavaBean对象

- Balance 对象

```java
package com.day.domain;

public class Balance {
    private int ID;
    private int money;

    public int getID() {
        return ID;
    }

    public void setID(int ID) {
        this.ID = ID;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Balance{" +
                "ID=" + ID +
                ", money=" + money +
                '}';
    }
}

```

- RowMapper

```java
class BalanceMapper implements RowMapper<Balance> {
    @Override
    public Balance mapRow(ResultSet resultSet, int i) throws SQLException {
        Balance b = new Balance();
        int id = resultSet.getInt("id");
        int money = resultSet.getInt("money");

        b.setID(id);
        b.setMoney(money);
        return b;
    }
}
```

- 实现

```java
/* 查询多条，并将结果封装为 list<Balance> */
    public static void QueryAllMapper() {
        String sql = "select * from balance";
        BalanceMapper bMapper = new BalanceMapper();
        // 使用 自己实现的额 RowMapper
        List<Balance> res = tmplate.query(sql, bMapper);
        for (Balance b : res) {
            System.out.println(b);
        }
    }
```

#### 8.8 查询多条记录(使用现有RowMapper)

```java
/* 查询多条，并封装结果*/
    public static void QueryAllBean() {
        String sql = "select * from balance";
        List<Balance> res = tmplate.query(sql, new BeanPropertyRowMapper<Balance>(Balance.class));
        for (Balance b : res) {
            System.out.println(b);
        }
    }
```

#### 8.9 查询总的条数

- queryForObject
  1. 查询单条，单个字段

```java
/* 查询总的条数 */
    public static void QueryCount() {
        String sql = "select count(1) from balance";
        Integer count = tmplate.queryForObject(sql, int.class);
        System.out.println(count);
    }
```



