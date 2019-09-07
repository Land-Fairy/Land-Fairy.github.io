---
title: servlet
date: 2019-09-06 20:30:12
categories: Java
---

> Tomcat + Servlet

<!-- more -->

## 1. Web

### 1.1 架构

- C/S
- B/S

### 1.2 资源分类

- 静态资源

> 所有用户访问的结果都相同，且可以直接被浏览器进行解析

> 比如:html, css, JavaScript  等

- 动态资源

> 每个用户访问相同资源后，得到的结果可能不一样

> 动态资源被访问后，需要先转换为静态资源，在返回给浏览器

> 比如: servlet/jsp,php,asp....

## 2. Tomcat

​	Web 服务器软件

### 2.1 安装

​	下载安装包 & 解压即可

### 2.2 启动

- 需要正确的配置了 JAVA_HOME 环境变量

```
bin/startup.bat 运行即可
```

### 2.3 访问

```
在浏览器 http://localhost:8080
```

### 2.4 修改端口号

```xml
<!-- A "Connector" represents an endpoint by which requests are received
         and responses are returned. Documentation at :
         Java HTTP Connector: /docs/config/http.html
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
    -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

### 2.5 关闭

```
方式一：(正常关闭)
	bin/shutdown.bat
方式二: (正常关闭)
	ctrl+c
方式三: (非正常关闭)
	点击启动窗口的×
```

### 2.6 项目部署

- 方式一: 直接将项目放到 webapps 目录下

```
方式一：
	将项目文件夹拷贝到 webapps 目录下，访问路径为 /文件夹名
方式二:
	将项目打成一个 war 包，然后将其放到 webapps 目录下，war包会被自动解压
```

- 方式二:配置 conf/server.xml文件

```
在<Host>标签体中配置
    <Context docBase="D:\hello" path="/hehe" />
    * docBase:项目存放的路径
    * path：虚拟目录
```

- 方式三: 在conf\Catalina\localhost创建任意名称的xml文件

```
文件内容:
	<Context docBase="D:\hello" />
	* 虚拟目录：xml文件的名称
```

- 方式四：将 Tomcat 继承到 IDEA中，并创建 JAVAEE项目，进行部署

### 2.7 JAVA 动态项目

- 项目目录结构

```
项目的根目录
	- WEB-INFO 目录
		- web.xml: web项目的核心配置文件
		- classes目录: 放置字节码文件的目录
		- lib目录: 放置依赖的 jar 包
```

## 3. Servlet(server applet)

### 3.1 概念

​	运行在 服务器端的小程序

> Servlet就是一个接口，定义了Java类被浏览器访问到(tomcat识别)的规则

> 对程序元来说，需要定义一个类，实现 Servlet 接口，复写接口中的方法。这样，浏览器访问的时候，tomcat 会创建该对象，并调用 复写的方法 进行执行

### 3.2 xml 配置方式

- 创建 JAVAEE web 项目
- 定义类，实现 servlet 接口

```java
public class ServletDemo1 implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init...");
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service...");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("destroy...");
    }
}

```

- 配置 web.xml 

```xml
	<servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>com.servlet.ServletDemo1</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>
```

- 配置 Tomcat，进行执行

#### 执行原理:

- server 接受到 浏览器请求，会解析 URL路径，获取访问的 Servlet 的资源路径
- 查找 web.xml，根据 url-pattern 与 Servlet 的资源路径比较，如果找到，根据 servlet-name 找到 servlet-class
- tomcat 将 servlet-class对应类加载进内存，创建其对象
- 由于对象实现了 Servlet 接口，因此可以直接调用其对应的方法

#### Servlet接口

- init()方法

> 只执行一次

- Servlet对象

> 默认情况下，在第一次被访问的时候，创建Servlet

> 由于 init()方法只执行一次，说明Servlet在内存中只存在一个对象，即其实单例的

> 在多个用户同时访问该接口时，使用的是同一个 Servlet，因此 尽量不要在 Servlet中定义成员变量，即使定义了，也不要对其值进行修改 （防止线程安全的问题)

> 可以在 servlet标签下进行配置

```xml
//负值表示第一次访问时创建
//为0或者正整数时，表示服务器启动后间隔多久创建
<servlet>
	 <load-on-startup>1</load-on-startup> 
</servlet>
```

- service()方法

> 每次访问Servlet时，Service方法都会被调用一次。

- destroy()方法

  > 只执行一次，在Servlet被销毁的时候进行执行

  	1. 只有服务器正常关闭时，才会执行destroy方法
  
   	2. destroy方法在Servlet被销毁之前执行，一般用于释放资源

### 3.3 注解方式

```
选择Servlet的版本3.0以上，可以不创建web.xml
```

- 创建JavaEE项目
- 实现 Servlet 接口
- 使用 注解  @WebServlet("/demo2")

```java
@WebServlet("/demo2")
public class ServletDemo2 implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {

    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

## 4. Servlet 实现类

### 4.1  HttpServlet

> 对http协议的一种封装，简化操作

- 使用

```
1. 定义类继承HttpServlet
2. 复写doGet/doPost方法
```

```java
@WebServlet("/demo4")
public class ServletDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("get...");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("post...");
    }
}
```

### 4.2  GenericServlet

> 将Servlet接口中其他的方法做了默认空实现，只将service()方法作为抽象

- 使用

```
1. 继承GenericServlet
2. 实现service()方法
```

```java
@WebServlet("/demo3")
public class ServletDemo3 extends GenericServlet {
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("do someting...");
    }
}
```

## 5. Request

### 5.1 继承体系

```
ServletRequest   ==> 接口
	HttpServletRequest ==> 子接口
	
```

### 5.2 获取请求消息数据

```java
// 请求URL
GET /day14/demo1?name=zhangsan HTTP/1.1
   
// 获取请求方式 ：GET
    String getMethod()  
// 获取虚拟目录：/day14
    String getContextPath()
// 获取Servlet路径: /demo1
    String getServletPath()
// 获取get方式请求参数：name=zhangsan
    String getQueryString()
// 获取请求URI：/day14/demo1 
    // URI 与 URL 的区别
    String getRequestURI()		/day14/demo1
    StringBuffer getRequestURL()  http://localhost/day14/demo1
//  获取协议及版本：HTTP/1.1
	String getProtocol()
// 获取客户机的IP地址：
    String getRemoteAddr()
```

### 5.3 获取请求头数据

```java
// 通过请求头的名称获取请求头的值
String getHeader(String name)
// 获取所有的请求头名称
Enumeration<String> getHeaderNames()
```

### 5.4 获取所有请求体数据

```java
// 1. 获取流
BufferedReader getReader() //获取字符输入流，只能操作字符数据
ServletInputStream getInputStream() //获取字节输入流，可以操作所有类型数据
// 2. 从流中获取数据
```

### 5.5 获取请求参数

```java
//根据参数名称获取参数值   
String getParameter(String name) 
//根据参数名称获取参数值的数组
String[] getParameterValues(String name)
//获取所有请求的参数名称
Enumeration<String> getParameterNames()
//获取所有参数的map集合
Map<String,String[]> getParameterMap()
```

### 5.6 中文乱码问题

- get 方式

> tomcat 8 已经将get方式乱码问题解决了

- post 方式

```java
// 在获取参数前，设置request的编码
request.setCharacterEncoding("utf-8");
```

### 5.7 请求转发

> 一种在  服务器内部  的资源跳转方式

#### 使用:

- 获取转发对象

```java
RequestDispatcher getRequestDispatcher(String path)
```

- 转发

```java
forward(ServletRequest request, ServletResponse response) 
```

#### 请求转发的特点:

- 浏览器地址栏路径不发生变化
- 只能转发到当前服务器内部资源中
- 即使多次转发，也是一次请求

#### 共享数据

- 域对象

> 一个有作用范围的对象，可以在范围内共享数据

- request 域

> 代表一次请求的范围，一般用于请求转发的多个资源中共享数据

- 方法

```java
1. void setAttribute(String name,Object obj):存储数据
2. Object getAttitude(String name):通过键获取值
3. void removeAttribute(String name):通过键移除键值对
```

### 用户登录Demo

- 导入jar包

```java
web/WEB-INFO/lib 目录下
1. commons-logging-1.2.jar
2. druid-1.0.9.jar
3. mysql-connector-java-5.1.37-bin.jar
4. spring-beans-5.0.0.RELEASE.jar
5. spring-core-5.0.0.RELEASE.jar
6. spring-jdbc-5.0.0.RELEASE.jar
7. spring-tx-5.0.0.RELEASE.jar
```

- 创建 User 表

```sql
	  CREATE DATABASE day1;
		USE day1;
		CREATE TABLE USER(

            id INT PRIMARY KEY AUTO_INCREMENT,
            userName VARCHAR(32) UNIQUE NOT NULL,
            passwd VARCHAR(32) NOT NULL
        );
```

- 创建 用户实体类

```java
package com.day.domain;

public class User {
    private String userName;
    private String passwd;

    public User() {
    }

    public User(String userName, String passwd) {
        this.userName = userName;
        this.passwd = passwd;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPasswd() {
        return passwd;
    }

    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "userName='" + userName + '\'' +
                ", passwd='" + passwd + '\'' +
                '}';
    }
}

```

- 创建 dao

```java
package com.day.dao;

import com.day.domain.User;
import com.day.utils.JdbcUtils;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;

public class UserDao {
    private JdbcTemplate tmplate = new JdbcTemplate(JdbcUtils.getDataSource());

    public User login(User loginUser) {
        try {
            String sql = "select * from user where userName=? and passwd=?";
            User user = tmplate.queryForObject(sql, new BeanPropertyRowMapper<User>(User.class),
                    loginUser.getUserName(), loginUser.getPasswd());
            return user;
        } catch (DataAccessException e) {
            e.printStackTrace();
            return null;
        }
    }
}

```

- 创建 JdbcUtils

  1. Druid连接方式
  2. 配置文件

  ```java
  // src 目录下 druid.properties
  driverClassName=com.mysql.jdbc.Driver
  url=jdbc:mysql://127.0.0.1:3306/db1
  username=root
  password=123456
  initialSize=5
  maxActive=10
  maxWait=3000
  ```

```java
package com.day.utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JdbcUtils {
    private static DataSource ds;
    static {
        try {
            /* 加载配置文件 */
            InputStream is = JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties");

            /* 创建配置文件对象 */
            Properties pro = new Properties();
            pro.load(is);
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static DataSource getDataSource() {
        return ds;
    }

    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    public static void close(Connection conn, Statement st, ResultSet rs) {
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

        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```

- Servlet

```java
package com.day.service;

import com.day.dao.UserDao;
import com.day.domain.User;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        /* 设置编码，防止乱码 */
        req.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf8");

        String userName = req.getParameter("userName");
        String passwd = req.getParameter("passwd");

        User loginUser = new User(userName, passwd);

        UserDao userDao = new UserDao();
        User user = userDao.login(loginUser);
        if (user == null) {
            resp.getWriter().write("登录失败");
        } else {
            req.getRequestDispatcher("suc.jsp").forward(req, resp);
        }
    }
}

```

- index.jsp

```jsp
<%-- Created by IntelliJ IDEA. --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>测试 Servlet</title>
  </head>
  <body>
    <form method="get" action="login" >
      用户名: <input type="text" name="userName">
      密码: <input type="text" name="passwd">
      提交: <input type="submit">
    </form>
  </body>
</html>
```

- suc.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: gao
  Date: 2019/9/6
  Time: 23:42
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录成功</title>
</head>
<body>
    欢迎登录!!!
</body>
</html>

```

## 6. Response

### 6.1 响应行

```java
setStatus(int sc) 
```

### 6.2 响应头

```java
setHeader(String name, String value) 
```

### 6.3 响应体

- 字符输出流

```java
PrintWriter getWriter()
```

- 字节输出流

```java
ServletOutputStream getOutputStream()
```

### 6.4 重定向

> 资源跳转的方式

- 实现

```java
//1. 设置状态码为302
response.setStatus(302);
//2.设置响应头location
response.setHeader("location","/day15/responseDemo2");

//简单的重定向方法
response.sendRedirect("/day15/responseDemo2");
```

- 特点

```
1. 地址栏发生变化
2. 重定向可以访问其他站点(服务器)的资源
3. 重定向是两次请求。不能使用request对象来共享数
```

## 7. ServletContext对象

### 7.1 概念

代表整个web应用，可以和程序的容器(服务器)来通信

### 7.2 获取

- 通过 request 对象

```java
request.getServletContext();
```

- 通过 HttpServlet 对象

```java
this.getServletContext();
```

### 7.3 功能

- 获取 MIME 类型

> 在互联网通信过程中定义的一种文件数据类型
>
> * 格式： 大类型/小类型   text/html		image/jpeg

```java
String getMimeType(String file) 
```

- 域对象=》共享数据

```java
1. setAttribute(String name,Object value)
2. getAttribute(String name)
3. removeAttribute(String name)
 * ServletContext对象范围：所有用户所有请求的数据
```

- 获取文件的真实路径

```java
String getRealPath(String path) 
```

```java
// 通过HttpServlet获取
ServletContext context = this.getServletContext();
// 获取文件的服务器路径
String b = context.getRealPath("/b.txt");//web目录下资源访问
System.out.println(b);

//WEB-INF目录下的资源访问
String c = context.getRealPath("/WEB-INF/c.txt");
System.out.println(c);

// /src目录下的资源访问
String a = context.getRealPath("/WEB-INF/classes/a.txt");
System.out.println(a);
```

## 8. 生成验证码Demo

- html

> 对 url 后拼接时间戳，是为了防止 静态url 缓存，导致 验证码图片不发生变化问题

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <script>
        /*
            分析：
                点击超链接或者图片，需要换一张
                1.给超链接和图片绑定单击事件
                2.重新设置图片的src属性值
         */
    window.onload = function(){
        //1.获取图片对象
        var img = document.getElementById("checkCode");
        //2.绑定单击事件
        img.onclick = function(){
            //加时间戳
            var date = new Date().getTime();

            img.src = "/checkCodeServlet?"+date;
        }
    }
    </script>
</head>
<body>
    <img id="checkCode" src="/checkCodeServlet" />
    <a id="change" href="">看不清换一张？</a>
</body>
</html>
```

- servlet

```java
@WebServlet("/checkCodeServlet")
public class CheckCodeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int width = 100;
        int height = 50;

        //1.创建一对象，在内存中图片(验证码图片对象)
        BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);

        //2.美化图片
        //2.1 填充背景色
        Graphics g = image.getGraphics();//画笔对象
        g.setColor(Color.PINK);//设置画笔颜色
        g.fillRect(0,0,width,height);

        //2.2画边框
        g.setColor(Color.BLUE);
        g.drawRect(0,0,width - 1,height - 1);

        String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghigklmnopqrstuvwxyz0123456789";
        //生成随机角标
        Random ran = new Random();

        for (int i = 1; i <= 4; i++) {
            int index = ran.nextInt(str.length());
            //获取字符
            char ch = str.charAt(index);//随机字符
            //2.3写验证码
            g.drawString(ch+"",width/5*i,height/2);
        }
        //2.4画干扰线
        g.setColor(Color.GREEN);

        //随机生成坐标点

        for (int i = 0; i < 10; i++) {
            int x1 = ran.nextInt(width);
            int x2 = ran.nextInt(width);

            int y1 = ran.nextInt(height);
            int y2 = ran.nextInt(height);
            g.drawLine(x1,y1,x2,y2);
        }
        //3.将图片输出到页面展示
        ImageIO.write(image,"jpg",response.getOutputStream());
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

