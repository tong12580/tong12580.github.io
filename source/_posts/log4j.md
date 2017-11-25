title: 一个关于Log4j警告的解决办法
date: 2016/06/11 20:32:00
tags:
    - spring
    - log4j
categories:
    - log4j
---

  <br><p style="text-align:center;color:#123456;font-size:30px;">&nbsp;&nbsp;&nbsp;&nbsp;最近在搭载springMVC运行环境并配置Log4j时启动时，发现控制台报出如下警告：</p>
```
log4j:WARN No appenders could be found for logger (org.springframework.web.context.ContextLoader).
log4j:WARN Please initialize the log4j system properly.
```
为解决这样的警告，现提出如下方案：
<p>
&nbsp;&nbsp;&nbsp;&nbsp;首先检查log4j.properties文件中是否在log4j.rootLoger中配置日志输出级别；例如：
&nbsp;&nbsp;&nbsp;&nbsp;

```
log4j.rootLogger=WARN,Console
```

<br>在此段代码中的日志响应级别就为WARN警告级；其次要注意在web.xml文件中log监听器的配置：
<br>

```
    <context-param>  
        <param-name>log4jConfigLocation</param-name>  
        <param-value>classpath:log4j.properties</param-value>  
    </context-param>  
      
    <context-param>  
        <param-name>log4jRefreshInterval</param-name>  
        <param-value>60000</param-value>  
    </context-param>
  <listener>  
      <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>  
  </listener>  
```

该段配置一定要配置在ContextLoaderListener监听器之前；

```
  <listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```

这样问题就解决了。

</p>