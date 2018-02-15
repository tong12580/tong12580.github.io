title: 基于注解的AOP日志切面配置
date: 2018/02/16 00:33:22
tags: 
    - 日志
    - aop
    - spring
categories:
    - 日志
---

#基于注解的AOP日志切面配置

## 前言
日志是我们用来发现问题和解决问题的好帮手，在业务越来越繁杂的大中型应用中，引入日志切面可以使我们用最少量的代码，记录下我们所需要的日志。

## 实现Web层的日志切面
* 使用@Aspect注解将一个java类定义为切面类
* 使用@Pointcut定义一个切入点，可以是一个规则表达式，比如下例中某个package下的所有函数，也可以是一个注解等。
* 根据需要在切入点不同位置的切入内容
* 使用@Before在切入点开始处切入内容
* 使用@After在切入点结尾处切入内容
* 使用@AfterReturning在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）
* 使用@Around在切入点前后切入内容，并自己控制何时执行切入点自身的内容
* 使用@AfterThrowing用来处理当切入内容部分抛出异常之后的处理逻辑    

````

    @Aspect//1
    @Order(1)//2
    @Component
    @Slf4j//3
    public class WebLogHeadAspect {
    
    
        @Pointcut("execution(public * com.business.controller..*.*(..))") //4
        public void webLog() {
        }
    
        @Before("webLog()")//5
        public void doBefore(JoinPoint joinPoint) {
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder
            .getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            log.info("URL: {}, Method: {}, params: {}", request.getRequestURL(),
                request.getMethod(), Arrays.toString(joinPoint.getArgs()));
        }
    
        @AfterReturning(returning = "ret", pointcut = "webLog()") //6
        public void doAfterReturning(Object ret) {
            log.info("Result : {}", JsonUtil.objectToJson(ret));
        }
    }

````

如上述代码所示

1.注解将一个java类定义为切面类
2.表示该切面的加载顺序，在切入点前的操作，按order的值由小到大执行,在切入点后的操作，按order的值由大到小执行
3.基于lombok的统一日志配置标签 需引入 [lombok](https://projectlombok.org/) 的相关配置.
4.定义一个切入点,为该 Package下的所有控制器。
5.在webLog()函数执行前，也就是在控制器controller 前先执行doBefore()函数中的内容。这里面记录了URL，请求方式和入参
6.统计回调方式，记录result值。

日志打印如下：

```

2018-02-14 at 21:10:03 CST INFO  org.thymeleaf.TemplateEngine 838 initialize - [THYMELEAF] TEMPLATE ENGINE INITIALIZED
2018-02-14 at 21:10:17 CST INFO  com.business.config.WebLogHeadAspect 37 doBefore - URL: http://127.0.0.1:8080/, params [{}]
2018-02-14 at 21:10:17 CST INFO  com.business.config.WebLogHeadAspect 42 doAfterReturning - Result : "index"
2018-02-14 at 21:10:33 CST INFO  com.business.config.WebLogHeadAspect 37 doBefore - URL: http://127.0.0.1:8080/api/xcl, params []
2018-02-14 at 21:10:33 CST INFO  com.business.config.WebLogHeadAspect 42 doAfterReturning - Result : "xcl"

```

文末记录一下别的东西， 当node.js升级时，需要重新安装hexo-cil否则无法正常编译。