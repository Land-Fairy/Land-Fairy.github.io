---
title: Cookie&Session
date: 2019-09-07 16:44:08
categories: Java
---

> Session & Cookie

<!-- more -->

## 1. 会话

### 1.1 概念

> 一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止

> 一次会话中包含多次请求和响应, 那么就需要在这一次会话的多个请求之间，进行共享数据

### 1.2 方式

- 客户端会话技术

> Cookie

- 服务器端会话技术

> Session

## 2. Cookie

> 客户端会话技术，将数据保存到客户端

### 2.1 使用

#### 创建Cookie &绑定数据

```
new Cookie(String name, String value) 
```

#### 发送Cookie对象

```
// 可以创建多个cookie对象，调用多个 addCookie进行发送多次
response.addCookie(Cookie cookie) 
```

#### 获取Cookie & 拿取数据

```java
Cookie[]  request.getCookies()
```

### 2.2 实现

> 基于响应头set-cookie和请求头cookie实现

### 2.3  保存时间问题

```java
默认情况下，当浏览器关闭后，Cookie数据被销毁
/* 持久化存储
	正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效
	负数：默认值
	零：删除cookie信息
*/
setMaxAge(int seconds)
```

### 2.4 保存中文问题

```
 1. 在tomcat 8 之前 cookie中不能直接存储中文数据。需要将中文通过 URL 编码
 2. 在tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析
```

### 2.5 cookie 共享问题

```
1. 默认情况下，对于多个web项目，cookie是不能共享的；
2. 如果需要共享，可以设置 path为 "/"
	setPath(String path):设置cookie的获取范围。默认情况下，设置当前的虚拟目录
3. 如果需要在 多个 tomcat服务器之间，共享 cookie
	setDomain(String path):如果设置一级域名相同，那么多个服务器之间cookie可以共享
	如: setDomain(".baidu.com"),那么tieba.baidu.com和news.baidu.com中cookie可以共享
```

### 2.6 cookie 的特点

```
1. cookie存储数据在客户端浏览器
2. 浏览器对于单个cookie 的大小有限制(4kb) 以及 对同一个域名下的总cookie数量也有限制(20个)
==>
1. cookie一般用于存出少量的不太敏感的数据
```

### 判断是否第一次登录Demo

- 要求

```java
在服务器中的Servlet判断是否有一个名为lastTime的cookie
 1. 有：不是第一次访问
     1. 响应数据：欢迎回来，您上次访问时间为:2018年6月10日11:50:20
     2. 写回Cookie：lastTime=2018年6月10日11:50:01
 2. 没有：是第一次访问
     1. 响应数据：您好，欢迎您首次访问
     2. 写回Cookie：lastTime=2018年6月10日11:50:01
```

- servlet

> 1. 注意判断 cookies 是否为 null，防止 空指针错误
> 2. 数据 urlEncode 编解码

```java
@WebServlet("/cookieTest")
public class CookieTest extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //设置响应的消息体的数据格式以及编码
        response.setContentType("text/html;charset=utf-8");

        //1.获取所有Cookie
        Cookie[] cookies = request.getCookies();
        boolean flag = false;//没有cookie为lastTime
        //2.遍历cookie数组
        if(cookies != null && cookies.length > 0){
            for (Cookie cookie : cookies) {
                //3.获取cookie的名称
                String name = cookie.getName();
                //4.判断名称是否是：lastTime
                if("lastTime".equals(name)){
                    //有该Cookie，不是第一次访问
                    flag = true;//有lastTime的cookie

                    //设置Cookie的value
                    //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
                    Date date  = new Date();
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                    String str_date = sdf.format(date);
                    System.out.println("编码前："+str_date);
                    //URL编码
                    str_date = URLEncoder.encode(str_date,"utf-8");
                    System.out.println("编码后："+str_date);
                    cookie.setValue(str_date);
                    //设置cookie的存活时间
                    cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
                    response.addCookie(cookie);

                    //响应数据
                    //获取Cookie的value，时间
                    String value = cookie.getValue();
                    System.out.println("解码前："+value);
                    //URL解码：
                    value = URLDecoder.decode(value,"utf-8");
                    System.out.println("解码后："+value);
                    response.getWriter().write("<h1>欢迎回来，您上次访问时间为:"+value+"</h1>");
                    break;
                }
            }
        }

        if(cookies == null || cookies.length == 0 || flag == false){
            //没有，第一次访问
            //设置Cookie的value
            //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
            Date date  = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
            String str_date = sdf.format(date);
            System.out.println("编码前："+str_date);
            //URL编码
            str_date = URLEncoder.encode(str_date,"utf-8");
            System.out.println("编码后："+str_date);

            Cookie cookie = new Cookie("lastTime",str_date);
            //设置cookie的存活时间
            cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
            response.addCookie(cookie);
            response.getWriter().write("<h1>您好，欢迎您首次访问</h1>");
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```

## 3. Session

服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中

> Session的实现是依赖于Cookie的

### 3.1 使用

#### 获取

```java
HttpSession session = request.getSession();
```

#### 使用

```java
Object getAttribute(String name)  
void setAttribute(String name, Object value)
void removeAttribute(String name)
```

### 3.2 session 与 cookie

- 当客户端关闭后，服务器不关闭，两次获取session是否为同一个?

```
1. 由于默认情况下，cookie在 客户端关闭后，会被销毁。而session依赖于 cookie，所以不会是同一个 session
2. 如果需要同一个session，只需要 两次的 cookie一致，即对cookie做持久化存储
	// 注意，Cookie名称固定，为 JSESSIONID
	Cookie c = new Cookie("JSESSIONID",session.getId());
	c.setMaxAge(60*60);
	response.addCookie(c);
```

- 客户端不关闭，服务器关闭后，两次获取的session是同一个吗

```
不是同一个，但是要确保数据不丢失。tomcat自动完成以下工作
    * session的钝化：
        * 在服务器正常关闭之前，将session对象系列化到硬盘上
    * session的活化：
        * 在服务器启动后，将session文件转化为内存中的session对象
```

- session 销毁时机

```
1. 服务器关闭
		2. session对象调用invalidate() 。
		3. session默认失效时间 30分钟
			选择性配置修改	
			<session-config>
		        <session-timeout>30</session-timeout>
		    </session-config>
```

### 3.3 session特点

```
1. session用于存储一次会话的多次请求的数据，存在服务器端
 2. session可以存储任意类型，任意大小的数据
```

- 与 cookie的区别

```
1. session存储数据在服务器端，Cookie在客户端
2. session没有数据大小限制，Cookie有
3. session数据安全，Cookie相对于不安全
```

## 4. 验证码+登录Demo

- 思路
  1. 生成验证码的时候，将验证码对应的内容存储到 session中
  2. 登录的时候，从session中取出 验证码，与传入的对比

### 实现

#### login.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>login</title>


    <script>
        window.onload = function(){
            document.getElementById("img").onclick = function(){
                this.src="${pageContext.request.contextPath}/checkCodeServlet?time="+new Date().getTime();
            }
        }


    </script>
    <style>
        div{
            color: red;
        }

    </style>
</head>
<body>

    <form action="${pageContext.request.contextPath}/loginServlet" method="post">
        <table>
            <tr>
                <td>用户名</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>密码</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td>验证码</td>
                <td><input type="text" name="checkCode"></td>
            </tr>
            <tr>
                <td colspan="2"><img id="img" src="${pageContext.request.contextPath}/checkCodeServlet"></td>
            </tr>
            <tr>
                <td colspan="2"><input type="submit" value="登录"></td>
            </tr>
        </table>


    </form>


    <div><%=request.getAttribute("cc_error") == null ? "" : request.getAttribute("cc_error")%></div>
    <div><%=request.getAttribute("login_error") == null ? "" : request.getAttribute("login_error") %></div>

</body>
</html>

```

#### success.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    <h1><%=request.getSession().getAttribute("user")%>,欢迎您</h1>

</body>
</html>

```

#### 验证码servlet

```java
package cn.itcast.servlet;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Random;

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
        StringBuilder sb = new StringBuilder();
        for (int i = 1; i <= 4; i++) {
            int index = ran.nextInt(str.length());
            //获取字符
            char ch = str.charAt(index);//随机字符
            sb.append(ch);

            //2.3写验证码
            g.drawString(ch+"",width/5*i,height/2);
        }
        String checkCode_session = sb.toString();
        //将验证码存入session
        request.getSession().setAttribute("checkCode_session",checkCode_session);

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

#### Login Servlet

```java
package cn.itcast.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.设置request编码
        request.setCharacterEncoding("utf-8");
        //2.获取参数
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String checkCode = request.getParameter("checkCode");
        //3.先获取生成的验证码
        HttpSession session = request.getSession();
        String checkCode_session = (String) session.getAttribute("checkCode_session");
        //删除session中存储的验证码
        session.removeAttribute("checkCode_session");
        //3.先判断验证码是否正确
        if(checkCode_session!= null && checkCode_session.equalsIgnoreCase(checkCode)){
            //忽略大小写比较
            //验证码正确
            //判断用户名和密码是否一致
            if("zhangsan".equals(username) && "123".equals(password)){//需要调用UserDao查询数据库
                //登录成功
                //存储信息，用户信息
                session.setAttribute("user",username);
                //重定向到success.jsp
                response.sendRedirect(request.getContextPath()+"/success.jsp");
            }else{
                //登录失败
                //存储提示信息到request
                request.setAttribute("login_error","用户名或密码错误");
                //转发到登录页面
                request.getRequestDispatcher("/login.jsp").forward(request,response);
            }


        }else{
            //验证码不一致
            //存储提示信息到request
            request.setAttribute("cc_error","验证码错误");
            //转发到登录页面
            request.getRequestDispatcher("/login.jsp").forward(request,response);

        }

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}

```

