---
title:  何为aop
tag: java
---
<!-- toc -->
#  何为aop

　　aop全称Aspect Oriented Programming，面向切面，AOP主要实现的目的是针对业务处理过程中的切面进行提取，它所面对的是处理过程中的某个步骤或阶段，以获得逻辑过程中各部分之间低耦合性的隔离效果。其与设计模式完成的任务差不多，是提供另一种角度来思考程序的结构，来弥补面向对象编程的不足。

　　通俗点讲就是提供一个为一个业务实现提供切面注入的机制，通过这种方式，在业务运行中将定义好的切面通过切入点绑定到业务中，以实现将一些特殊的逻辑绑定到此业务中。

　　比如，若是需要一个记录日志的功能，首先想到的是在方法中通过log4j或其他框架来进行记录日志，但写下来发现一个问题，在整个业务中其实核心的业务代码并没有多少，都是一些记录日志或其他辅助性的一些代码。而且很多业务有需要相同的功能，比如都需要记录日志，这时候又需要将这些记录日志的功能复制一遍，即使是封装成框架，也是需要调用之类的。在此处使用复杂的设计模式又得不偿失。

## aop名词
　　先介绍一些aop的名词，其实这些名词对使用aop没什么影响，但为了更好的理解最好知道一些

* 切面（Aspect）：一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。

* 连接点（Joinpoint）：在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。在Spring AOP中，一个连接点总是表示一个方法的执行。

* 通知（Advice）：在切面的某个特定的连接点上执行的动作。其中包括了“around”、“before”和“after”等不同类型的通知（通知的类型将在后面部分进行讨论）。许多AOP框架（包括Spring）都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。

* 切入点（Pointcut）：匹配连接点的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。

* 引入（Introduction）：用来给一个类型声明额外的方法或属性（也被称为连接类型声明（inter-type declaration））。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。

* 目标对象（Target Object）：被一个或者多个切面所通知的对象。也被称做被通知（advised）对象。既然Spring AOP是通过运行时代理实现的，这个对象永远是一个被代理（proxied）对象。

* AOP代理（AOP Proxy）：AOP框架创建的对象，用来实现切面契约（例如通知方法执行等等）。在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。

* 织入（Weaving）：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

其中重要的名词有：切面，切入点

## 注解

* @Aspect:描述一个切面类，定义切面类的时候需要打上这个注解

* @Configuration：spring-boot配置类

* @Pointcut：声明一个切入点，切入点决定了连接点关注的内容，使得我们可以控制通知什么时候执行。Spring AOP只支持Spring bean的方法执行连接点。所以你可以把切入点看做是Spring bean上方法执行的匹配。一个切入点声明有两个部分：一个包含名字和任意参数的签名，还有一个切入点表达式，该表达式决定了我们关注那个方法的执行。

　注：作为切入点签名的方法必须返回void 类型

## Spring AOP支持在切入点表达式中使用如下的切入点指示符：　　　　

* execution - 匹配方法执行的连接点，这是你将会用到的Spring的最主要的切入点指示符。

* within - 限定匹配特定类型的连接点（在使用Spring AOP的时候，在匹配的类型中定义的方法的执行）。

* this - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中bean reference（Spring AOP 代理）是指定类型的实例。

* target - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中目标对象（被代理的应用对象）是指定类型的实例。

* args - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中参数是指定类型的实例。

* @target - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中正执行对象的类持有指定类型的注解。

* @args - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中实际传入参数的运行时类型持有指定类型的注解。

* @within - 限定匹配特定的连接点，其中连接点所在类型已指定注解（在使用Spring AOP的时候，所执行的方法所在类型已指定注解）。

* @annotation - 限定匹配特定的连接点（使用Spring AOP的时候方法的执行），其中连接点的主题持有指定的注解。

>其中execution使用最频繁，即某方法执行时进行切入。定义切入点中有一个重要的知识，即切入点表达式，我们一会在解释怎么写切入点表达式。


## spring aop支持的通知：

* @Before：前置通知：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。

* @AfterReturning ：后置通知：在某连接点正常完成后执行的通知，通常在一个匹配的方法返回的时候执行。

可以在后置通知中绑定返回值，如：

```java
@AfterReturning（
    pointcut=com.test.service.CacheDemoService.findById(..))",
    returning="retVal"）
  public void doFindByIdCheck（Object retVal） {
    // ...
  }
```

* @AfterThrowing:异常通知：在方法抛出异常退出时执行的通知。　　　　　　　

* @After 最终通知。当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

* @Around：环绕通知：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。

>环绕通知最麻烦，也最强大，其是一个对方法的环绕，具体方法会通过代理传递到切面中去，切面中可选择执行方法与否，执行方法几次等。

>环绕通知使用一个代理ProceedingJoinPoint类型的对象来管理目标对象，所以此通知的第一个参数必须是ProceedingJoinPoint类型，在通知体内，调用ProceedingJoinPoint的proceed()方法会导致后台的连接点方法执行。proceed 方法也可能会被调用并且传入一个Object[]对象-该数组中的值将被作为方法执行时的参数。

## 切入点表达式

切入点表达式的格式：execution([可见性] 返回类型 [声明类型].方法名(参数) [异常])

其中【】中的为可选，其他的还支持通配符的使用:

*：匹配所有字符
..：一般用于匹配多个包，多个参数
+：表示类及其子类

运算符有：&&、||、!

```
* execution：用于匹配子表达式。
//匹配com.cjm.model包及其子包中所有类中的所有方法，返回类型任意，方法参数任意
@Pointcut("execution(* com.cjm.model..*.*(..))")
public void before(){}

* within：用于匹配连接点所在的Java类或者包。
//匹配Person类中的所有方法
@Pointcut("within(com.cjm.model.Person)")
public void before(){}
            
//匹配com.cjm包及其子包中所有类中的所有方法
@Pointcut("within(com.cjm..*)")
public void before(){}

* this：用于向通知方法中传入代理对象的引用。
@Before("before() && this(proxy)")
public void beforeAdvide(JoinPoint point, Object proxy){
   //处理逻辑
}

* target：用于向通知方法中传入目标对象的引用。
@Before("before() && target(target)
public void beforeAdvide(JoinPoint point, Object proxy){
   //处理逻辑
}

* args：用于将参数传入到通知方法中。
@Before("before() && args(age,username)")
public void beforeAdvide(JoinPoint point, int age, String username){
   //处理逻辑
}
 
* @within ：用于匹配在类一级使用了参数确定的注解的类，其所有方法都将被匹配。 
@Pointcut("@within(com.cjm.annotation.AdviceAnnotation)")
//所有被@AdviceAnnotation标注的类都将匹配
public void before(){}

* @target ：和@within的功能类似，但必须要指定注解接口的保留策略为RUNTIME。 @Pointcut("@target(com.cjm.annotation.AdviceAnnotation)")
public void before(){}

* @args ：传入连接点的对象对应的Java类必须被@args指定的Annotation注解标注。   @Before("@args(com.cjm.annotation.AdviceAnnotation)")
public void beforeAdvide(JoinPoint point){
   //处理逻辑
}

* @annotation ：匹配连接点被它参数指定的Annotation注解的方法。也就是说，所有被指定注解标注的方法都将匹配。
@Pointcut("@annotation(com.cjm.annotation.AdviceAnnotation)")
public void before(){}

* bean：通过受管Bean的名字来限定连接点所在的Bean。该关键词是Spring2.5新增的。
@Pointcut("bean(person)")
public void before(){}
```

## 准备工作
因为需要对web请求做切面来记录日志，所以先引入web模块，并创建一个简单的hello请求的处理。

* pom.xml 中引入aop模块

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

* 或者gradle.build 引入

```
    compile("org.springframework.boot:spring-boot-starter-aop")
```

在完成了引入AOP依赖包后，一般来说并不需要去做其他配置。也许在Spring中使用过注解配置方式的人会问是否需要在程序主类中增加 @EnableAspectJAutoProxy 来启用，实际并不需要。

可以看下面关于AOP的默认配置属性，其中 spring.aop.auto 属性默认是开启的，也就是说只要引入了AOP依赖后，默认已经增加了 @EnableAspectJAutoProxy 。

```
# AOP
spring.aop.auto=true # Add @EnableAspectJAutoProxy.  
spring.aop.proxy-target-class=false # Whether subclass-based (CGLIB) proxies are to be created (true) as  
 opposed to standard Java interface-based proxies (false).
```

>而当我们需要使用CGLIB来实现AOP的时候，需要配置 spring.aop.proxy-target-class=true ，不然默认使用的是标准Java的实现。


## AOP方法
实现AOP的切面主要有以下几个要素：

* 使用 @Aspect 注解将一个java类定义为切面类
* 使用 @Pointcut 定义一个切入点，可以是一个规则表达式，比如下例中某个package下的所有函数，也可以是一个注解等。
* 根据需要在切入点不同位置的切入内容
	* 使用 @Before 在切入点开始处切入内容
	* 使用 @After 在切入点结尾处切入内容
	* 使用 @AfterReturning 在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）
	* 使用 @Around 在切入点前后切入内容，并自己控制何时执行切入点自身的内容
	* 使用 @AfterThrowing 用来处理当切入内容部分抛出异常之后的处理逻辑

	
```java
@Aspect
@Component
public class WebLogAspect {

    private Logger logger = Logger.getLogger(getClass());

    @Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 记录下请求内容
        logger.info("URL : " + request.getRequestURL().toString());
        logger.info("HTTP_METHOD : " + request.getMethod());
        logger.info("IP : " + request.getRemoteAddr());
        logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));

    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
        logger.info("RESPONSE : " + ret);
    }

}

```	

可以看上面的例子，通过 @Pointcut 定义的切入点为 com.didispace.web 包下的所有函数（对web层所有请求处理做切入点），然后通过 @Before 实现，对请求内容的日志记录（本文只是说明过程，可以根据需要调整内容），最后通过 @AfterReturning 记录请求返回的对象。

通过运行程序并访问： http://localhost:8080/hello?name=didi ，可以获得下面的日志输出

```java
2016-05-19 13:42:13,156  INFO WebLogAspect:41 - URL : http://localhost:8080/hello  
2016-05-19 13:42:13,156  INFO WebLogAspect:42 - HTTP_METHOD : http://localhost:8080/hello  
2016-05-19 13:42:13,157  INFO WebLogAspect:43 - IP : 0:0:0:0:0:0:0:1  
2016-05-19 13:42:13,160  INFO WebLogAspect:44 - CLASS_METHOD : com.didispace.web.HelloController.hello  
2016-05-19 13:42:13,160  INFO WebLogAspect:45 - ARGS : [didi]  
2016-05-19 13:42:13,170  INFO WebLogAspect:52 - RESPONSE:Hello didi
```

## 优化：AOP切面中的同步问题

在WebLogAspect切面中，分别通过doBefore和doAfterReturning两个独立函数实现了切点头部和切点返回后执行的内容，若我们想统计请求的处理时间，就需要在doBefore处记录时间，并在doAfterReturning处通过当前时间与开始处记录的时间计算得到请求处理的消耗时间。

那么我们是否可以在WebLogAspect切面中定义一个成员变量来给doBefore和doAfterReturning一起访问呢？是否会有同步问题呢？

的确，直接在这里定义基本类型会有同步问题，所以我们可以引入ThreadLocal对象，像下面这样进行记录：

```
@Aspect
@Component
public class WebLogAspect {

    private Logger logger = Logger.getLogger(getClass());

    ThreadLocal<Long> startTime = new ThreadLocal<>();

    @Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        startTime.set(System.currentTimeMillis());

        // 省略日志记录内容
    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
        logger.info("RESPONSE : " + ret);
        logger.info("SPEND TIME : " + (System.currentTimeMillis() - startTime.get()));
    }


}
```


## 优化：AOP切面的优先级

由于通过AOP实现，程序得到了很好的解耦，但是也会带来一些问题，比如：我们可能会对Web层做多个切面，校验用户，校验头信息等等，这个时候经常会碰到切面的处理顺序问题。

所以，我们需要定义每个切面的优先级，我们需要 @Order(i) 注解来标识切面的优先级。 i的值越小，优先级越高 。假设我们还有一个切面是 CheckNameAspect 用来校验name必须为didi，我们为其设置 @Order(10) ，而上文中WebLogAspect设置为 @Order(5) ，所以WebLogAspect有更高的优先级，这个时候执行顺序是这样的：

* 在 @Before 中优先执行 @Order(5) 的内容，再执行 @Order(10) 的内容
* 在 @After 和 @AfterReturning 中优先执行 @Order(10) 的内容，再执行 @Order(5) 的内容
所以我们可以这样子总结：

* 在切入点前的操作，按order的值由小到大执行
* 在切入点后的操作，按order的值由大到小执行


## AOP实现web签名验证


```java
@Aspect
@Component
public class SignatureAspect{

    private final String secret = "test";

    private Logger log = LoggerFactory.getLogger(this.getClass());

    @Pointcut("execution(public * com.mailejifen.msg.controllers..*.*(..))")
    public void signatureVerification() {}

    @Around("signatureVerification()")
    public Object invoke(ProceedingJoinPoint pjp) throws Throwable {

        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 记录下请求内容
        log.info("URL : " + request.getRequestURL().toString());
        log.info("HTTP_METHOD : " + request.getMethod());
        log.info("IP : " + request.getRemoteAddr());
        log.info("CLASS_METHOD : " + pjp.getSignature().getDeclaringTypeName() + "." + pjp.getSignature().getName());
        log.info("ARGS : " + Arrays.toString(pjp.getArgs()));
        
        //获取签名
        String sign= request.getParameter("sign");
        //获取所有值
        Object[] keys = request.getParameterMap().keySet().toArray();
        Arrays.sort(keys);
        String str = "";
        for (Object key : keys) {
            if(!key.equals("sign")) {
                str += request.getParameter(key.toString());
            }
        }
        str += secret;
        System.out.println(str);
        System.out.println(DigestUtils.md5Hex(str));
        System.out.println(sign);

        //判断签名
        if(DigestUtils.md5Hex(str).equals(sign)) {
            Object result = pjp.proceed();
            return result;
        }
        else {
            return new HandleResult(ErrorCode.SIGN_FAIL,ErrorCode.getErrorMsg(ErrorCode.SIGN_FAIL));

        }
    }
}
```

