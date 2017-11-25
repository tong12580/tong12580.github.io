title: spring4 + log4j2+ slf4j的配置
date: 2016/12/23 15:05:22
tags: 
    - slf4j
    - 日志
    - log4j2
    - spring
categories:
    - 日志
---

### log4j2的一些配置介绍
log4j2 相比log4j 解决了一些 死锁问题  改进了性能
下来 我直接挂配置留作本人日后参考

> 1.maven配置

```xml
	<properties>
		<!-- spring版本号 -->
		<spring.version>4.3.2.RELEASE</spring.version>
		<!-- log4j日志文件管理包版本 -->
		<slf4j.version>1.7.21</slf4j.version>
		<log4j.version>2.7</log4j.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
	<dependencies>
			<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<!-- spring核心包 -->
		<!-- springframe start -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
		</dependency>
				<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${slf4j.version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-slf4j-impl</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-web</artifactId>
			<version>${log4j.version}</version>
			<scope>runtime</scope>
		</dependency>
	</dependencies>
```

> 2.log4j2.xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
#这一行我设置为OFF 表示不记录log4j2本身的日志
<configuration status="OFF"> 
    <properties>
        <property name="LOG_HOME">${web:rootDir}/logs</property>
        <property name="FILE_NAME">likR</property>
    </properties>
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
        #这ThresholdFilter是个类似拦截器 level 填写日志显示类型
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </Console>
        <File name="Error" fileName="${LOG_HOME}/error.log">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>
        <RollingFile name="RollingFile" fileName="${LOG_HOME}/${FILE_NAME}.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/${FILE_NAME}-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
            #日志文件最大值 第二天压缩
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
        </RollingFile>
    </appenders>
    #root 默认加载
    <loggers>
        <root level="info">
            <appender-ref ref="RollingFile"/>
            <appender-ref ref="Console"/>
            <appender-ref ref="Error" />
        </root>
    </loggers>
</configuration>
```
> 3.web.xml配置
注意!! 是***log4jConfiguration***！！！
```xml
	<context-param>
		<!-- 日志配置文件路径 -->
		<param-name>log4jConfiguration</param-name>
		<param-value>classpath:conf/log4j2.xml</param-value>
	</context-param>
```

4.代码中使用
private Logger logger = LoggerFactory.getLogger(getClass());
logger .error("{}",Object);

文章太简单?没办法 ,您想了解更多 请谷歌!