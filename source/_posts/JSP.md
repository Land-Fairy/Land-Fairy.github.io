---
title: JSP
date: 2019-09-07 18:07:29
categories: Java
---

> JSP

<!-- more -->

## 1. JSP

### 1.1 概念

Java Server Pages： java服务器端页面

- 一个特殊的页面，其中既可以指定定义html标签，又可以定义java代码

> JSP本质上就是一个Servlet

### 1.2 脚本

- <%  代码 %>

> 该处的java代码，与 service 方法一致；在 service方法中定义什么，在该处就可以写什么

- <%! 代码 %>

> 相当于 java类的成员

- <%= 代码 %>

> 用来输出到 页面上，输出语句可以定义什么，该处就可以写什么

### 1.3 内置对象

在jsp页面中不需要获取和创建，可以直接使用的对象

```
// 1. 当前页面共享数据，还可以获取其他八个内置对象
* pageContext				PageContext		
// 2. 一次请求访问的多个资源(转发)
* request					HttpServletRequest	
// 3. 一次会话的多个请求间
* session					HttpSession			
// 4. 所有用户间共享数据
* application				ServletContext		
// 5. 响应对象
* response					HttpServletResponse		
// 6. 当前页面(Servlet)的对象  this
* page						Object				
// 7. 输出对象，数据输出到页面上
* out						JspWriter	
// 8. Servlet的配置对象
* config					ServletConfig		
// 9. 异常对象
* exception					Throwable					
```



### 1.4 Demo

```java
<%@ page import="java.util.Date" %>
<%@ page import="java.text.SimpleDateFormat" %>
<%@ page import="java.net.URLEncoder" %>
<%@ page import="java.net.URLDecoder" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>itcast</title>
</head>
<body>

<%
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
                %>
            <h1>欢迎回来，您上次访问时间为:<%=value%></h1>
            <input>

<%

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

%>

        <h1>您好，欢迎您首次访问</h1>
<span></span>

<%
    }

%>

<input>




</body>
</html>

```

### 1.5 指令

作用：用于配置JSP页面，导入资源文件

- 格式

```
<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>
```

#### 1.1 Page 指令

用于配置 JSP 页面

- contentType 属性

```
等同于response.setContentType()
1. 设置响应体的mime类型以及字符集
2. 设置当前jsp页面的编码（只能是高级的IDE才能生效，如果使用低级工具，则需要设置pageEncoding属性设置当前页面的字符集）
```

- import 属性

```
用于导入 包
```

- errorPage 属性

```
当前页面发生异常后，会自动跳转到指定的错误页面
```

- isErrorPage 属性

```
标识当前也是是否是错误页面。
* true：是，可以使用内置对象exception
* false：否。默认值。不可以使用内置对象exception
```

#### 1.2 include 指令

导入页面的资源文件

```jsp
<%@include file="top.jsp"%>
```

#### 1.3 taglib 指令

导入资源

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

### 1.6 注释

- html 注释

```
<!-- -->:只能注释html代码片段
```

- jsp 注释

```
<%-- --%>：可以注释所有
```

## 2. MVC

### 2.1 jsp 演变

- servlet阶段

```
早期只有servlet，只能使用response输出标签数据，非常麻烦
```

- jsp 阶段

```
jsp，简化了Servlet的开发，如果过度使用jsp，在jsp中即写大量的java代码，有写html表，造成难于维护，难于分工协作
```

- mvc 阶段

### 2.2 MVC 架构

- Model

```
完成具体的业务操作，如：查询数据库，封装对象
```

- View

```
展示数据 JSP
```

- Controller

```
// Servlet
* 获取用户的输入
* 调用模型
* 将数据交给视图进行展示
```

## 3. EL表达式

Expression Language 表达式语言

#### 作用

> 替换和简化jsp页面中java代码的编写

#### 语法

> ${表达式}

#### 注意

```
* jsp默认支持el表达式的。如果要忽略el表达式
		1. 设置jsp中page指令中：isELIgnored="true" 忽略当前jsp页面中所有的el表达式
		2. \${表达式} ：忽略当前这个el表达式
```

### 使用

#### 运算符

```
1. 算数运算符： + - * /(div) %(mod)
2. 比较运算符： > < >= <= == !=
3. 逻辑运算符： &&(and) ||(or) !(not)
4. 空运算符： empty
	* 功能：用于判断字符串、集合、数组对象是否为null或者长度是否为0
	* ${empty list}:判断字符串、集合、数组对象是否为null或者长度为0
	* ${not empty str}:表示判断字符串、集合、数组对象是否不为null 并且 长度>0
```

#### 获取值

el表达式只能从域对象中获取值

```
1. ${域名称.键名}：从指定域中获取指定键的值
	* 域名称：
		1. pageScope		--> pageContext
		2. requestScope 	--> request
		3. sessionScope 	--> session
		4. applicationScope --> application（ServletContext）
			* 举例：在request域中存储了name=张三
			* 获取：${requestScope.name}
			
2. ${键名}：表示依次从最小的域中查找是否有该键对应的值，直到找到为止。
3. 获取对象、List集合、Map集合的值
	1. 对象：${域名称.键名.属性名}
		* 本质上会去调用对象的getter方法
	2. List集合：${域名称.键名[索引]}
	3. Map集合：
		* ${域名称.键名.key名称}
		* ${域名称.键名["key名称"]}
```

#### 隐士对象

```
el表达式中有11个隐式对象
pageContext：
	* 获取jsp其他八个内置对象
	* ${pageContext.request.contextPath}：动态获取虚拟目录
```

### Demo1(获取域中的数据)

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>el获取域中的数据</title>
</head>
<body>
    <%
        //在域中存储数据
        session.setAttribute("name","李四");
        request.setAttribute("name","张三");
        session.setAttribute("age","23");
        request.setAttribute("str","");
    %>
<h3>el获取值</h3>
${requestScope.name}
${sessionScope.age}
${sessionScope.haha}
${name}
${sessionScope.name}
</body>
</html>

```

### Demo2(获取数据)

```jsp
<%@ page import="cn.itcast.domain.User" %>
<%@ page import="java.util.*" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>el获取数据</title>
</head>
<body>

    <%
        User user = new User();
        user.setName("张三");
        user.setAge(23);
        user.setBirthday(new Date());


        request.setAttribute("u",user);


        List list = new ArrayList();
        list.add("aaa");
        list.add("bbb");
        list.add(user);

        request.setAttribute("list",list);


        Map map = new HashMap();
        map.put("sname","李四");
        map.put("gender","男");
        map.put("user",user);

        request.setAttribute("map",map);

    %>

<h3>el获取对象中的值</h3>
${requestScope.u}<br>

<%--
    * 通过的是对象的属性来获取
        * setter或getter方法，去掉set或get，在将剩余部分，首字母变为小写。
        * setName --> Name --> name
--%>

    ${requestScope.u.name}<br>
    ${u.age}<br>
    ${u.birthday}<br>
    ${u.birthday.month}<br>

    ${u.birStr}<br>

    <h3>el获取List值</h3>
    ${list}<br>
    ${list[0]}<br>
    ${list[1]}<br>
    ${list[10]}<br>

    ${list[2].name}

    <h3>el获取Map值</h3>
    ${map.gender}<br>
    ${map["gender"]}<br>
    ${map.user.name}

</body>
</html>

```

### Demo3(隐士对象)

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>el隐式对象</title>
</head>
<body>


    ${pageContext.request}<br>
    <h4>在jsp页面动态获取虚拟目录</h4>
    ${pageContext.request.contextPath}

<%


%>
</body>
</html>

```





## 4. JSTL

JavaServer Pages Tag Library  JSP标准标签库

是由Apache组织提供的开源的免费的jsp标签		<标签>

### 使用

```
1. 导入jstl相关jar包
2. 引入标签库：taglib指令：  <%@ taglib %>
3. 使用标签
```

### 常用的JSTL标签

#### if

相当于java代码的if语句

```
1. 属性：
  * test 必须属性，接受boolean表达式
  * 如果表达式为true，则显示if标签体内容，如果为false，则不显示标签体内容
     * 一般情况下，test属性值会结合el表达式一起使用
2. 注意：
   * c:if标签没有else情况，想要else情况，则可以在定义一个c:if标签
```

- Demo

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>if标签</title>
</head>
<body>

    <%--

    c:if标签
        1. 属性：
            * test 必须属性，接受boolean表达式
                * 如果表达式为true，则显示if标签体内容，如果为false，则不显示标签体内容
                * 一般情况下，test属性值会结合el表达式一起使用

        2. 注意：c:if标签没有else情况，想要else情况，则可以在定义一个c:if标签


    --%>

    <c:if test="true">
        <h1>我是真...</h1>
    </c:if>
    <br>

    <%
        //判断request域中的一个list集合是否为空，如果不为null则显示遍历集合

        List list = new ArrayList();
        list.add("aaaa");
        request.setAttribute("list",list);

        request.setAttribute("number",4);

    %>

    <c:if test="${not empty list}">
        遍历集合...

    </c:if>
    <br>

    <c:if test="${number % 2 != 0}">

            ${number}为奇数

    </c:if>

    <c:if test="${number % 2 == 0}">

        ${number}为偶数

    </c:if>

</body>
</html>

```



#### choose

相当于java代码的switch语句

```
1. 使用choose标签声明         			相当于switch声明
2. 使用when标签做判断         			相当于case
3. 使用otherwise标签做其他情况的声明    	相当于default
```

- Demo

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>choose标签</title>
</head>
<body>

    <%--
        完成数字编号对应星期几案例
            1.域中存储一数字
            2.使用choose标签取出数字         相当于switch声明
            3.使用when标签做数字判断         相当于case
            4.otherwise标签做其他情况的声明  相当于default
    --%>

    <%
        request.setAttribute("number",51);
    %>

    <c:choose>
        <c:when test="${number == 1}">星期一</c:when>
        <c:when test="${number == 2}">星期二</c:when>
        <c:when test="${number == 3}">星期三</c:when>
        <c:when test="${number == 4}">星期四</c:when>
        <c:when test="${number == 5}">星期五</c:when>
        <c:when test="${number == 6}">星期六</c:when>
        <c:when test="${number == 7}">星期天</c:when>

        <c:otherwise>数字输入有误</c:otherwise>
    </c:choose>


</body>
</html>

```



#### foreach

相当于java代码的for语句

- Demo

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<html>
<head>
    <title>foreach标签</title>
</head>
<body>

<%--

    foreach:相当于java代码的for语句
        1. 完成重复的操作
            for(int i = 0; i < 10; i ++){

            }
            * 属性：
                begin：开始值
                end：结束值
                var：临时变量
                step：步长
                varStatus:循环状态对象
                    index:容器中元素的索引，从0开始
                    count:循环次数，从1开始
        2. 遍历容器
            List<User> list;
            for(User user : list){

            }

            * 属性：
                items:容器对象
                var:容器中元素的临时变量
                varStatus:循环状态对象
                    index:容器中元素的索引，从0开始
                    count:循环次数，从1开始


--%>

<c:forEach begin="1" end="10" var="i" step="2" varStatus="s">
    ${i} <h3>${s.index}<h3> <h4> ${s.count} </h4><br>

</c:forEach>

    <hr>


    <%
        List list = new ArrayList();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");

        request.setAttribute("list",list);


    %>

    <c:forEach items="${list}" var="str" varStatus="s">

            ${s.index} ${s.count} ${str}<br>

    </c:forEach>


</body>
</html>

```





