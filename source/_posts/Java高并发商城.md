---
title: Java高并发商城
date: 2019-10-16 20:40:03
tags: 高并发商城
category: JAVA

---

> Java 高并发商城

<!-- more-->

## 一. Redis

### 1. 加入依赖

Pom 中，添加 jedis 依赖，fastjson 用来 json encode 与 decode 使用

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.38</version>
</dependency>
```

### 2. 添加 Redis 配置信息

类似 数据库的使用，需要从连接池中获取连接，再使用；

同样，Jedis 中，也可以使用 JedisPool的方式，但此种方式需要一直配置项，比如 hostName，port。。。

在 DataSource 中，由于有 DataSourceProperties 的存在，可以直接在 配置文件中配置属性，即可被读取

对于 Jedis 而言，需要 自己配置，方式

1. 创建一个 Bean，并添加到 容器中；该  bean 从 配置文件中 读取 配置的值
2. 创建一个Bean，用来将 JedisPool 放入 容器中（用来自动导入，因为 Pool 只有一个)

**@Data 注解:**

​	用来 自动生成 setter 和 getter ，以及 toString()方法，简化代码

​	依赖:

	1. IDEA 安装 lombock 插件
 	2. SpringBoot 中 导入该依赖

- RedisConfig.java

```java
package com.imooc.miaosha.redis;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix="redis")
@Data
public class RedisConfig {
	private String host;
	private int port;
	private int timeout;//秒
	private String password;
	private int poolMaxTotal;
	private int poolMaxIdle;
	private int poolMaxWait;//秒
}

```

- RedisPool.java

```java
package com.imooc.miaosha.redis;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Service;

import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

@Service
public class RedisPoolFactory {

	@Autowired
	RedisConfig redisConfig;
	
	@Bean
	public JedisPool JedisPoolFactory() {
		JedisPoolConfig poolConfig = new JedisPoolConfig();
		poolConfig.setMaxIdle(redisConfig.getPoolMaxIdle());
		poolConfig.setMaxTotal(redisConfig.getPoolMaxTotal());
		poolConfig.setMaxWaitMillis(redisConfig.getPoolMaxWait() * 1000);
		JedisPool jp = new JedisPool(poolConfig, redisConfig.getHost(), redisConfig.getPort(),
				redisConfig.getTimeout()*1000, redisConfig.getPassword(), 0);
		return jp;
	}
	
}

```

### 3.  提供 常用的 get, set 等方法

外部在使用Redis的时候，直接 调用 get/set 方法即可。具体操作逻辑需要 内部封装(该service中自动注入 JedisPool即可)

由于 Redis 内部存储的是 string，因此 在 get/set 对象时，需要 json encode 和 decode

- string 转 json obj

```java
/*
	1. Class<T> clazz 表示 需要将 string 转换为 什么类型
	2. 对于 int，long，string 等简单类型，需要另外做处理
 */
@SuppressWarnings("unchecked")
	private <T> T stringToBean(String str, Class<T> clazz) {
		if(str == null || str.length() <= 0 || clazz == null) {
			 return null;
		}
		if(clazz == int.class || clazz == Integer.class) {
			 return (T)Integer.valueOf(str);
		}else if(clazz == String.class) {
			 return (T)str;
		}else if(clazz == long.class || clazz == Long.class) {
			return  (T)Long.valueOf(str);
		}else {
			return JSON.toJavaObject(JSON.parseObject(str), clazz);
		}
}
```

- obj 转 string

对于 int， long， string 等类型，并没有使用 json，而是直接转换

```java
private <T> String beanToString(T value) {
		if(value == null) {
			return null;
		}
		Class<?> clazz = value.getClass();
		if(clazz == int.class || clazz == Integer.class) {
			 return ""+value;
		}else if(clazz == String.class) {
			 return (String)value;
		}else if(clazz == long.class || clazz == Long.class) {
			return ""+value;
		}else {
			return JSON.toJSONString(value);
		}
	}
```

getter方法，需要提供 key以及 需要的类型，内部会自动将 string 转换为 该类型

由于Redis 可能被多个项目使用，为了防止 key 重复，需要对key添加一个 prefix；并且，对于每个key，都可以设置过期时间

==> 抽取一个类，保存 prefix 和 过期时间，每个模块直接继承该类，定义一些固定的key即可

- BasePrefix.java(基类)

```
package com.imooc.miaosha.redis;

public abstract class BasePrefix implements KeyPrefix{
	
	private int expireSeconds;
	
	private String prefix;
	
	public BasePrefix(String prefix) {//0代表永不过期
		this(0, prefix);
	}
	
	public BasePrefix( int expireSeconds, String prefix) {
		this.expireSeconds = expireSeconds;
		this.prefix = prefix;
	}
	
	public int expireSeconds() {//默认0代表永不过期
		return expireSeconds;
	}

	public String getPrefix() {
		String className = getClass().getSimpleName();
		return className+":" + prefix;
	}

}

```

- Userkey.java

具体模块使用的key (MiaoshaUserKey)

```java
package com.imooc.miaosha.redis;

public class MiaoshaUserKey extends BasePrefix{

	public static final int TOKEN_EXPIRE = 3600*24 * 2;
	private MiaoshaUserKey(int expireSeconds, String prefix) {
		super(expireSeconds, prefix);
	}
	public static MiaoshaUserKey token = new MiaoshaUserKey(TOKEN_EXPIRE, "tk");
}
```

步骤:

	1. 获取连接
 	2. 生成真正的key 

3. 获取值
4. 转换类型
5. 释放连接

- getter

```java
public <T> T get(KeyPrefix prefix, String key,  Class<T> clazz) {
		 Jedis jedis = null;
		 try {
			 jedis =  jedisPool.getResource();
			 //生成真正的key
			 String realKey  = prefix.getPrefix() + key;
			 String  str = jedis.get(realKey);
			 T t =  stringToBean(str, clazz);
			 return t;
		 }finally {
			  returnToPool(jedis);
		 }
}
```

- getter

```java
public <T> boolean set(KeyPrefix prefix, String key,  T value) {
		 Jedis jedis = null;
		 try {
			 jedis =  jedisPool.getResource();
			 String str = beanToString(value);
			 if(str == null || str.length() <= 0) {
				 return false;
			 }
			//生成真正的key
			 String realKey  = prefix.getPrefix() + key;
			 int seconds =  prefix.expireSeconds();
			 if(seconds <= 0) {
				 jedis.set(realKey, str);
			 }else {
				 jedis.setex(realKey, seconds, str);
			 }
			 return true;
		 }finally {
			  returnToPool(jedis);
		 }
	}
```

- incr

```java
public <T> Long incr(KeyPrefix prefix, String key) {
		 Jedis jedis = null;
		 try {
			 jedis =  jedisPool.getResource();
			//生成真正的key
			 String realKey  = prefix.getPrefix() + key;
			return  jedis.incr(realKey);
		 }finally {
			  returnToPool(jedis);
		 }
	}
```

## 二. 登录

### 1. 数据校验

登陆时使用的数据，放入 Vo中（View Object)

使用 @Valid 注解 来进行校验

- 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

-  字段校验规则

```java
package com.imooc.miaosha.vo;

import javax.validation.constraints.NotNull;

import org.hibernate.validator.constraints.Length;

import com.imooc.miaosha.validator.IsMobile;

@Data
public class LoginVo {
	
	@NotNull
	@IsMobile
	private String mobile;
	
	@NotNull
	@Length(min=32)
	private String password;
}

```

- 使用

```java
@RequestMapping("/do_login")
    @ResponseBody
    public Result<Boolean> doLogin(HttpServletResponse response, @Valid LoginVo loginVo) {
    	log.info(loginVo.toString());
    	//登录
    	userService.login(response, loginVo);
    	return Result.success(true);
    }
```

### 2. 自定义校验注解

- 新建注解

```java
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {IsMobileValidator.class })
public @interface  IsMobile {
	
	boolean required() default true;
	
	String message() default "手机号码格式错误";

	Class<?>[] groups() default { };

	Class<? extends Payload>[] payload() default { };
}

```

- 注解解析器

```java
package com.imooc.miaosha.validator;
import  javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.apache.commons.lang3.StringUtils;

import com.imooc.miaosha.util.ValidatorUtil;

public class IsMobileValidator implements ConstraintValidator<IsMobile, String> {

	private boolean required = false;
	
	public void initialize(IsMobile constraintAnnotation) {
		required = constraintAnnotation.required();
	}

	public boolean isValid(String value, ConstraintValidatorContext context) {
		if(required) {
			return ValidatorUtil.isMobile(value);
		}else {
			if(StringUtils.isEmpty(value)) {
				return true;
			}else {
				return ValidatorUtil.isMobile(value);
			}
		}
	}

}

```

### 3. 登陆处理步骤

1. 数据校验
2. md5(输入的密码 + salt) 与 数据库中密码比较
3. 成功后，创建token，存储到 Redis中
4. 返回cookie

## 三. cookie 获取 user

对于很多业务请求，都是带有cookie，需要从cookie中解析出 用户信息

由于在很多请求中都使用到了，因此 可以抽取出来，放在拦截器中

- 用户参数解析器

从请求中获取 cookie

根据 cookie中 token，获取 user 对象

resolveArgument 对象返回的为 object，是自动放入到 request 域中了？ 然后 隐藏模型得到，将其类型首字母小写作为key，进行存储。然后再 controller 中，参数中写入该类型，就可以再 隐藏模型中根据类型获取到？

```java

@Service
public class UserArgumentResolver implements HandlerMethodArgumentResolver {

	@Autowired
	MiaoshaUserService userService;
	
	public boolean supportsParameter(MethodParameter parameter) {
		Class<?> clazz = parameter.getParameterType();
		return clazz==MiaoshaUser.class;
	}

	public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
		HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
		HttpServletResponse response = webRequest.getNativeResponse(HttpServletResponse.class);
		
		String paramToken = request.getParameter(MiaoshaUserService.COOKI_NAME_TOKEN);
		String cookieToken = getCookieValue(request, MiaoshaUserService.COOKI_NAME_TOKEN);
		if(StringUtils.isEmpty(cookieToken) && StringUtils.isEmpty(paramToken)) {
			return null;
		}
		String token = StringUtils.isEmpty(paramToken)?cookieToken:paramToken;
		return userService.getByToken(response, token);
	}

	private String getCookieValue(HttpServletRequest request, String cookiName) {
		Cookie[]  cookies = request.getCookies();
		for(Cookie cookie : cookies) {
			if(cookie.getName().equals(cookiName)) {
				return cookie.getValue();
			}
		}
		return null;
	}

}

```

- 添加ArgumentResolvers

```java
@Configuration
public class WebConfig  extends WebMvcConfigurerAdapter{
	
	@Autowired
	UserArgumentResolver userArgumentResolver;
	
	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
		argumentResolvers.add(userArgumentResolver);
	}
}
```

## 四. 程序调用规则

1. Controller 调用  Service
2. Service 调用Dao
3. 即使 Controller 需要用到 其他 Service 的 Dao，也需要 自动导入  对应的 Service 来使用，而不应该直接导入对应的 Dao

## 五.  页面缓存

在Request中
	ContentType 用来告诉服务器当前发送的数据是什么格式
	Accept      用来告诉服务器，客户端能认识哪些格式,最好返回这些格式中的其中一种

对应的，在 @RequestMapping 中:

​	consumes 用来限制ContentType
​	produces 用来限制Accept   

```java
    @RequestMapping(value="/to_list", produces="text/html")
    @ResponseBody
    public String list(HttpServletRequest request, HttpServletResponse response, Model model,MiaoshaUser user) {
    	model.addAttribute("user", user);
    	List<GoodsVo> goodsList = goodsService.listGoodsVo();
    	model.addAttribute("goodsList", goodsList);
    	SpringWebContext ctx = new SpringWebContext(request,response,
    			request.getServletContext(),request.getLocale(), model.asMap(), applicationContext );
    	//手动渲染
    	String html = thymeleafViewResolver.getTemplateEngine().process("goods_list", ctx);
    	if(!StringUtils.isEmpty(html)) {
    		redisService.set(GoodsKey.getGoodsList, "", html);
    	}
    	return html;
    }
```

## 六. 验证码

步骤:

	1. 随机生成几个 运算表达式 (类似1+1)，并存储到 redis中，key为 userId+goodsId, value 为 运算表达式的值 
 	2. 返回 图片 到 客户端

- controller

```java
@RequestMapping(value="/verifyCode", method=RequestMethod.GET)
    @ResponseBody
    public Result<String> getMiaoshaVerifyCod(HttpServletResponse response,MiaoshaUser user,
    		@RequestParam("goodsId")long goodsId) {
    	if(user == null) {
    		return Result.error(CodeMsg.SESSION_ERROR);
    	}
    	try {
            /*
            	生成 一个 验证码 图片
            	使用 流的方式，返回给 客户端
             */
    		BufferedImage image  = miaoshaService.createVerifyCode(user, goodsId);
    		OutputStream out = response.getOutputStream();
    		ImageIO.write(image, "JPEG", out);
    		out.flush();
    		out.close();
    		return null;
    	}catch(Exception e) {
    		e.printStackTrace();
    		return Result.error(CodeMsg.MIAOSHA_FAIL);
    	}
    }
```

- service

```java
public BufferedImage createVerifyCode(MiaoshaUser user, long goodsId) {
		if(user == null || goodsId <=0) {
			return null;
		}
		int width = 80;
		int height = 32;
		//create the image
		BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
		Graphics g = image.getGraphics();
		// set the background color
		g.setColor(new Color(0xDCDCDC));
		g.fillRect(0, 0, width, height);
		// draw the border
		g.setColor(Color.black);
		g.drawRect(0, 0, width - 1, height - 1);
		// create a random instance to generate the codes
		Random rdm = new Random();
		// make some confusion
		for (int i = 0; i < 50; i++) {
			int x = rdm.nextInt(width);
			int y = rdm.nextInt(height);
			g.drawOval(x, y, 0, 0);
		}
		// generate a random code
		String verifyCode = generateVerifyCode(rdm);
		g.setColor(new Color(0, 100, 0));
		g.setFont(new Font("Candara", Font.BOLD, 24));
		g.drawString(verifyCode, 8, 24);
		g.dispose();
		//把验证码存到redis中
		int rnd = calc(verifyCode);
		redisService.set(MiaoshaKey.getMiaoshaVerifyCode, user.getId()+","+goodsId, rnd);
		//输出图片	
		return image;
	}

private String generateVerifyCode(Random rdm) {
    int num1 = rdm.nextInt(10);
    int num2 = rdm.nextInt(10);
    int num3 = rdm.nextInt(10);
    char op1 = ops[rdm.nextInt(3)];
    char op2 = ops[rdm.nextInt(3)];
    String exp = ""+ num1 + op1 + num2 + op2 + num3;
    return exp;
}

private static int calc(String exp) {
    try {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("JavaScript");
        return (Integer)engine.eval(exp);
    }catch(Exception e) {
        e.printStackTrace();
        return 0;
    }
}

private static char[] ops = new char[] {'+', '-', '*'};
```

## 七. 隐藏秒杀接口

获取秒杀地址，需要 另外调用接口 + 验证码 ，返回值为一个 randomtoken （随机 uuid)，并存入Redis中，key为 userId + miaosha_addr, value 为  randomtoken 

controller 对应的路由为  {randomtoken}/do_miaosha, 该controller 首先判断 该用户 的 randomtoken 与  redis 中 存储的是否一致，如果不一致，直接返回错误

## 八. 秒杀

步骤:

	1. 程序启动后，将所有商品的库存预加载到 Redis中
 	2. 判断秒杀地址是否正确
 	3. 判断是否已经无库存(内存中标志，不用请求redis，更快)
 	4. Redis 预 减少 库存
 	5. 判断是否已经秒杀过
 	6. 添加到 amqp对了，等待 真正 下单操作
 	7. 客户端轮询，是否秒杀成功

- 程序启动时，预加载所有库存

1. 实现 InitializingBean 接口即可
2. 重写 afterPropertiesSet 方法

```
public class MiaoshaController implements InitializingBean {
	/**
	 * 系统初始化
	 * */
	public void afterPropertiesSet() throws Exception {
		List<GoodsVo> goodsList = goodsService.listGoodsVo();
		if(goodsList == null) {
			return;
		}
		for(GoodsVo goods : goodsList) {
			redisService.set(GoodsKey.getMiaoshaGoodsStock, ""+goods.getId(), goods.getStockCount());
			localOverMap.put(goods.getId(), false);
		}
	}
}
```

- 秒杀接口处理

问题： 先预减少库存，在进行 是否已经秒杀过判断，如果已经秒杀过，应当 在加回库存

```java
@RequestMapping(value="/{path}/do_miaosha", method=RequestMethod.POST)
    @ResponseBody
    public Result<Integer> miaosha(Model model,MiaoshaUser user,
    		@RequestParam("goodsId")long goodsId,
    		@PathVariable("path") String path) {
    	model.addAttribute("user", user);
    	if(user == null) {
    		return Result.error(CodeMsg.SESSION_ERROR);
    	}
    	//验证path
    	boolean check = miaoshaService.checkPath(user, goodsId, path);
    	if(!check){
    		return Result.error(CodeMsg.REQUEST_ILLEGAL);
    	}
    	//内存标记，减少redis访问
    	boolean over = localOverMap.get(goodsId);
    	if(over) {
    		return Result.error(CodeMsg.MIAO_SHA_OVER);
    	}
    	//预减库存
    	long stock = redisService.decr(GoodsKey.getMiaoshaGoodsStock, ""+goodsId);//10
    	if(stock < 0) {
    		 localOverMap.put(goodsId, true);
    		return Result.error(CodeMsg.MIAO_SHA_OVER);
    	}
    	//判断是否已经秒杀到了
    	MiaoshaOrder order = orderService.getMiaoshaOrderByUserIdGoodsId(user.getId(), goodsId);
    	if(order != null) {
    		return Result.error(CodeMsg.REPEATE_MIAOSHA);
    	}
    	//入队
    	MiaoshaMessage mm = new MiaoshaMessage();
    	mm.setUser(user);
    	mm.setGoodsId(goodsId);
    	sender.sendMiaoshaMessage(mm);
    	return Result.success(0);//排队中
    }
```

## 九. 页面限流

为了简单且通用的方式，可以新增一个注解，只有添加了改注解的 接口，才会被限流

==> 使用 拦截器的方式，判断方法上是否有该注解，如果有，判断该注解的属性

==> 将 接口访问次数放在  Redis中，使用 Incr 增加访问次数；

==> 对于 n秒限制m次，只需要 将 n 设置为 该key 的 过期时间即可

对于原来 在 ArgumentResolver 中 进行 cookie ==> user 解析的操作，放在了 拦截器中(接口限流可能需要用户信息)

==》 那么在 ArgumentResolver  就不应当再进行一次 cookie ==> user 的解析操作了，由于 一个 请求的处理过程都是再一个线程中执行的

==> 将 user 存储到 ThreadLocal 中，在 ArgumentResolver  中，直接进行 获取即可

- 注解

```
@Retention(RUNTIME)
@Target(METHOD)
public @interface AccessLimit {
	int seconds();
	int maxCount();
	boolean needLogin() default true;
}

```

- 拦截器

```java
package com.imooc.miaosha.access;
@Service
public class AccessInterceptor  extends HandlerInterceptorAdapter{
	
	@Autowired
	MiaoshaUserService userService;
	
	@Autowired
	RedisService redisService;
	
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		if(handler instanceof HandlerMethod) {
			MiaoshaUser user = getUser(request, response);
			UserContext.setUser(user);
			HandlerMethod hm = (HandlerMethod)handler;
			AccessLimit accessLimit = hm.getMethodAnnotation(AccessLimit.class);
			if(accessLimit == null) {
				return true;
			}
			int seconds = accessLimit.seconds();
			int maxCount = accessLimit.maxCount();
			boolean needLogin = accessLimit.needLogin();
			String key = request.getRequestURI();
			if(needLogin) {
				if(user == null) {
					render(response, CodeMsg.SESSION_ERROR);
					return false;
				}
				key += "_" + user.getId();
			}else {
				//do nothing
			}
			AccessKey ak = AccessKey.withExpire(seconds);
			Integer count = redisService.get(ak, key, Integer.class);
	    	if(count  == null) {
	    		 redisService.set(ak, key, 1);
	    	}else if(count < maxCount) {
	    		 redisService.incr(ak, key);
	    	}else {
	    		render(response, CodeMsg.ACCESS_LIMIT_REACHED);
	    		return false;
	    	}
		}
		return true;
	}
	
	private void render(HttpServletResponse response, CodeMsg cm)throws Exception {
		response.setContentType("application/json;charset=UTF-8");
		OutputStream out = response.getOutputStream();
		String str  = JSON.toJSONString(Result.error(cm));
		out.write(str.getBytes("UTF-8"));
		out.flush();
		out.close();
	}

	private MiaoshaUser getUser(HttpServletRequest request, HttpServletResponse response) {
		String paramToken = request.getParameter(MiaoshaUserService.COOKI_NAME_TOKEN);
		String cookieToken = getCookieValue(request, MiaoshaUserService.COOKI_NAME_TOKEN);
		if(StringUtils.isEmpty(cookieToken) && StringUtils.isEmpty(paramToken)) {
			return null;
		}
		String token = StringUtils.isEmpty(paramToken)?cookieToken:paramToken;
		return userService.getByToken(response, token);
	}
	
	private String getCookieValue(HttpServletRequest request, String cookiName) {
		Cookie[]  cookies = request.getCookies();
		if(cookies == null || cookies.length <= 0){
			return null;
		}
		for(Cookie cookie : cookies) {
			if(cookie.getName().equals(cookiName)) {
				return cookie.getValue();
			}
		}
		return null;
	}
	
}

```

- 保存在 ThreadLocal 中的 用户信息

```java
public class UserContext {
	
	private static ThreadLocal<MiaoshaUser> userHolder = new ThreadLocal<MiaoshaUser>();
	
	public static void setUser(MiaoshaUser user) {
		userHolder.set(user);
	}
	
	public static MiaoshaUser getUser() {
		return userHolder.get();
	}

}
```

- 限流的使用

```java
@AccessLimit(seconds=5, maxCount=5, needLogin=true)
@RequestMapping(value="/path", method=RequestMethod.GET)
@ResponseBody
public Result<String> getMiaoshaPath(HttpServletRequest request, MiaoshaUser user,
	... 
}
```

## 十. 错误处理

对于 业务错误，需要也返回 code, msg, data 的 json形式

==> 自定义错误，并添加 错误处理

- 自定义错误

```java
public class GlobalException extends RuntimeException{

	private static final long serialVersionUID = 1L;
	
	private CodeMsg cm;
	
	public GlobalException(CodeMsg cm) {
		super(cm.toString());
		this.cm = cm;
	}

	public CodeMsg getCm() {
		return cm;
	}

}
```

- 错误处理

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
	@ExceptionHandler(value=Exception.class)
	public Result<String> exceptionHandler(HttpServletRequest request, Exception e){
		e.printStackTrace();
		if(e instanceof GlobalException) {
			GlobalException ex = (GlobalException)e;
			return Result.error(ex.getCm());
		}else if(e instanceof BindException) {
			BindException ex = (BindException)e;
			List<ObjectError> errors = ex.getAllErrors();
			ObjectError error = errors.get(0);
			String msg = error.getDefaultMessage();
			return Result.error(CodeMsg.BIND_ERROR.fillArgs(msg));
		}else {
			return Result.error(CodeMsg.SERVER_ERROR);
		}
	}
}
```

## 十一. Jmeter 使用