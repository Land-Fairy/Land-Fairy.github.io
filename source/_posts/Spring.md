---
title: Spring
date: 2019-09-09 21:08:21
Categories: Java
---

## 1. Bean

在计算机中，有可重用组件的含义

### JavaBean:

使用Java语言，创建的可重用组件

即，用来创建 service 和 dao 对象的

#### 步骤

1. 需要一个配置文件，配置 service 和dao的全限定类名
2. 读取配置文件内容，使用反射创建对象

## 2. Ioc

### 2.1 工厂模式程序

```
// 正常 程序架构
dao
	- Impl
		- UserDaoImpl.java
	- UserDao.java
service
	- Impl
		- UserServiceImpl.java
	- UserService.java
Servlet
	- UserServlet.java
其中:
	UserServlet.java 中 会创建 service 层的对象，并 调用对应方法去处理；
	同样的，service 会创建 dao层的对象，并 调用对应的方法去处理
	==> 程序间的 耦合很严重。servlet 依赖 service对象， 而 service 又依赖与  dao 对象
	且 如果 service 对象 或 dao 对象发生了更改，需要更改的内容较多，不方便
```

- 如何解耦呢?

```
通过 配置文件 + 反射的方式。
1. 配置文件中，配置 需要创建对象的 全类名
2. 创建一个 BeanFactory类， 这个类 加载配置文件，初始化时就创建好所有对象
3. servlet, service ,dao 需要时，从该类 获取 已经创建好的对象
==> 由于 类名 在配置文件中，因此 更改比较方便
```

### 2.2  spring IOC

