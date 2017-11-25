title: spring Boot 填坑手册 - 无web.xml文件时的上下文监听的配置
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/timg.jpeg
date: 2016/09/16 16:21:00
tags: 
    - web
categories:
    - spring
---

&nbsp;&nbsp;&nbsp;&nbsp;Spring Boot提倡Spring 4.x以上版本 使用基于注解的配置代替xml文件配置 , 首当其冲的 , 便是 web.xml 配置全部消失了 。 那么 ,当笔者想像其中注入监听器和上下文时,遇到了难题 -- 如何注入 ? 在哪里注入 ? 

----------
&nbsp;&nbsp;&nbsp;&nbsp;既然出现了问题,那么接下了就是解决问题了,下面阐述笔者的解决方法 , 我就以最简单的项目根文件目录的监听来阐述 :
> *  首先 注意包结构 , 在./src/java/*路径下配置 
> * 新建WebAppRootContext.java 该文件一定要实现 ServletContextInitializer类
> * 实现onStartup方法 , 添加WebAppRootListener监听器 , 并定义文件初始化的上下文参数
> * 最后为该类添加@Configuration
@ComponentScan @EnableAutoConfiguration 
标签声明它为一个上下文配置文件
> * 具体代码如下:

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
public class WebAppRootContext implements ServletContextInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        servletContext.addListener(WebAppRootListener.class);
        servletContext.setInitParameter(ContextParamDictionary.PROJECTPATH.getParamName(),//这里是注入参数的名称
        ContextParamDictionary.PROJECTPATH.getParamValue());
    } 
}
```
> * 对比web.xml文件配置如下:

```xml
	<context-param>
		<param-name>webAppRootKey</param-name>
		<param-value>projectRootPath</param-value>
	</context-param>
 <listener>
		<listener-class>org.springframework.web.util.WebAppRootListener</listener-class>
  </listener> 
```
> * 启动工程,我们可以在日志中看见 如下内容,说明注入成功

```log
2016-09-16 15:26:40.999  INFO 15440 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2016-09-16 15:26:40.999  INFO 15440 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 8489 ms

```
> * 如此一来就可以使用`System.getProperty(ContextParamDictionary.PROJECTPATH.getParamValue());//括号里是 webAppRootKey` 来获取项目路径了 ；（注：ContextParamDictionary.PROJECTPATH.getParamValue()这个 东西 是自己定义的路径罢了，就是“projectRootPath”不要多想。）

### 完

-----


后续补充：

上述是注解辅助手工监听，还有纯注解监听标签，使用 @WebListener

类似于：

```
@WebListener
public class XXXServerListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        xxx
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        xxx
    }
}

注入上下文参数，可使用@WebInitParam()标签方便注入

使用该标签时，需要在 springBoot启动类上使用 @ServletComponentScan标签

```



