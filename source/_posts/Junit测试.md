---
title: Junit测试
date: 2019-09-03 22:03:26
categories: Java
---

>Java 中Junit的使用

### 1. 使用

- 定义测试类

>测试类名： 被测试类别Test  (例如 CalucatorTest)
>
>测试包名: xx.xx.test  (例如  cn.day.test)

- 定义测试方法

> 方法名: test 测试的方法名
>
> 返回值: void
>
> 参数列表: 空参

- 给方法添加 @Test 注解 并 导入Junit 依赖
- 执行测试方法

```java
public class CalulatorTest {
    /*
    *  测试add方法
    * */
    @Test
    public void testAdd(){
        Calulator c = new Calulator();
        int result = c.add(1, 2);
        /*
        * 使用断言来判断是否成功
        * */
        Assert.assertEquals(3, result);
    }
}
```

#### Before 和 After 注解 

- Before

> Before 用于在 所有的 Test方法之前执行，一般做初始化操作

- After

> After 用于在 所有的 Test 方法之后执行，一般做 资源释放操作

```java
/**
     * 在所有测试方法执行之前进行执行
     * 一般用于 申请资源，做初始化
     */
    @Before
    public void init(){
        System.out.println("init....");
    }

    /**
     * 在所有测试方法执行完成之后，进行执行
     * 一般用于 释放资源
     */
    @After
    public void close(){
        System.out.println("close.....");
    }
```

