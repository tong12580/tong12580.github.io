title: springBoot填坑手册二 redis与cache之坑
date: 2017/07/29 11:24:18
tags: 
    - cache
    - redis
categories:
    - cache
---

### spring Boot 填坑手册 redis与cache之坑

在处理高并发时,我们常常用到缓存,首先,说说redis的配置,与注意事项.
> 在pom.xml中需要引入spring-boot-starter-data-redis和spring-boot-starter-cache 如此引入之后,缓存配置就默认为redis,配置redis 需要在application.properties中配置如下参数

```properties
#redis
spring.redis.database=1 ## 可选
spring.redis.host=192.168.99.100
spring.redis.port=6379
spring.redis.pool.max-active=1024  ## 可选
spring.redis.pool.max-wait=1000 ## 可选
spring.redis.pool.max-idle=200 ## 可选
```
这时大抵上redis就配置好了
redis本身给我们了 RedisTemplate 和 StringRedisTemplate 两块模版 ,实现对redis的操作 , 
这里面一定要注意 , 当使用 opsForValue() 来set对象时 ,首先 RedisTemplate 序列化对象会使用JDK的对象序列化 , 所以该对象一定要实现 Serializable (网上有推荐使用jackson做序列化的方法,并不推荐, 因为使用JDK本身的序列化可得到二进制字符 , 高效快速。)
其次，当opsXXX()来get对象时 , RedisTemplate 和 StringRedisTemplate 要区分开 , 否则得不到想要的结果 .

> 接下来说缓存 , 这里因为使用了redis所以系统会默认redis做缓存, 如果想要redis做别的事情,而用别的缓存框架,该怎么办??

1.那么首先 , 需要在application.properties中配置如下参数

```properties
#Cache
spring.cache.type=guava #指明所用的缓存架构,我这里用的 guava 
spring.cache.cache-names[0]=outsourced # 指明缓存库的名称 
```
2. 在pom.xml中需要引入对应的缓存工具
```
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>19.0</version>
		</dependency>
```
3.对应缓存实例

```
    @Override                                      //key 缓存的键名 '#'不能少
    @Cacheable(value = "outsourced", key = "#name")//value 指缓存库的名称
    public UserInfo fineOne(String name) {
        UserInfo userInfo = userInfoRepository.findFirstByUserName("樱桃");
        System.out.print("缓存了key为"+name+"的鬼");
        return userInfo;
    }
```
完
 
----


后续更新：

文章中写的不太好，应该是不推荐使用JDK来做二进制缓存，JDK自带的序列化工具出来的二进制过长，再分布式环境，特别是跨语言的环境，推荐使用Thrift、Protobuf和Avro 等序列化通信解决方案，他们能友好的规避，XML体积太大，解析性能极差；JSON体积相对较小，解析相对较快，但表达能力较弱的特点。
当然，json对人来说的可读性较好，能如果想使用jackson2.x做缓存，可在redisTemplate中配置setKeySerializer/setValueSerializer/setHashValueSerializer等的序列化方式，这里使`GenericJackson2JsonRedisSerializer` 类来进行配置，具体配置可参考如下方式

```
    @Resource
    private RedisProperties redisProperties;

    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxTotal(100);
        config.setMaxWaitMillis(5000);
        config.setMinIdle(0);
        config.setTestOnBorrow(true);
        config.setTestWhileIdle(true);
        config.setNumTestsPerEvictionRun(2);
        config.setTimeBetweenEvictionRunsMillis(30000);
        config.setMinEvictableIdleTimeMillis(60000);
        config.setSoftMinEvictableIdleTimeMillis(60000);
        return config;
    }

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory redisConnectionFactory = new JedisConnectionFactory(jedisPoolConfig());
        redisConnectionFactory.setDatabase(redisProperties.getDatabase());
        redisConnectionFactory.setHostName(redisProperties.getHost());
        redisConnectionFactory.setPort(redisProperties.getPort());
        redisConnectionFactory.setPassword(redisProperties.getPassword());
        redisConnectionFactory.setTimeout(15000);
        redisConnectionFactory.setUsePool(true);
        return redisConnectionFactory;
    }

    @Bean
    public <K, V> RedisTemplate<K, V> redisTemplate() {
        RedisTemplate<K, V> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
//        template.setEnableTransactionSupport(true);
        template.afterPropertiesSet();
        return template;
    }

```


特别强调：推荐使用 GenericJackson2JsonRedisSerializer 去替代 Jackson2JsonRedisSerializer<Object>以及JacksonJsonRedisSerializer<Object>
因为GenericJackson2JsonRedisSerializer 可以保留泛型。
在处理高并发时,我们常常用到缓存,首先,说说redis的配置,与注意事项.
> 在pom.xml中需要引入spring-boot-starter-data-redis和spring-boot-starter-cache 如此引入之后,缓存配置就默认为redis,配置redis 需要在application.properties中配置如下参数

```properties
#redis
spring.redis.database=1 ## 可选
spring.redis.host=192.168.99.100
spring.redis.port=6379
spring.redis.pool.max-active=1024  ## 可选
spring.redis.pool.max-wait=1000 ## 可选
spring.redis.pool.max-idle=200 ## 可选
```
这时大抵上redis就配置好了
redis本身给我们了 RedisTemplate 和 StringRedisTemplate 两块模版 ,实现对redis的操作 , 
这里面一定要注意 , 当使用 opsForValue() 来set对象时 ,首先 RedisTemplate 序列化对象会使用JDK的对象序列化 , 所以该对象一定要实现 Serializable (网上有推荐使用jackson做序列化的方法,并不推荐, 因为使用JDK本身的序列化可得到二进制字符 , 高效快速。)
其次，当opsXXX()来get对象时 , RedisTemplate 和 StringRedisTemplate 要区分开 , 否则得不到想要的结果 .

> 接下来说缓存 , 这里因为使用了redis所以系统会默认redis做缓存, 如果想要redis做别的事情,而用别的缓存框架,该怎么办??

1.那么首先 , 需要在application.properties中配置如下参数

```properties
#Cache
spring.cache.type=guava #指明所用的缓存架构,我这里用的 guava 
spring.cache.cache-names[0]=outsourced # 指明缓存库的名称 
```
2. 在pom.xml中需要引入对应的缓存工具
```
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>19.0</version>
		</dependency>
```
3.对应缓存实例

```
    @Override                                      //key 缓存的键名 '#'不能少
    @Cacheable(value = "outsourced", key = "#name")//value 指缓存库的名称
    public UserInfo fineOne(String name) {
        UserInfo userInfo = userInfoRepository.findFirstByUserName("樱桃");
        System.out.print("缓存了key为"+name+"的鬼");
        return userInfo;
    }
```
完
 
----


后续更新：

文章中写的不太好，应该是不推荐使用JDK来做二进制缓存，JDK自带的序列化工具出来的二进制过长，再分布式环境，特别是跨语言的环境，推荐使用Thrift、Protobuf和Avro 等序列化通信解决方案，他们能友好的规避，XML体积太大，解析性能极差；JSON体积相对较小，解析相对较快，但表达能力较弱的特点。
当然，json对人来说的可读性较好，能如果想使用jackson2.x做缓存，可在redisTemplate中配置setKeySerializer/setValueSerializer/setHashValueSerializer等的序列化方式，这里使`GenericJackson2JsonRedisSerializer` 类来进行配置，具体配置可参考如下方式

```
    @Resource
    private RedisProperties redisProperties;

    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxIdle(10);
        config.setMaxTotal(100);
        config.setMaxWaitMillis(5000);
        config.setMinIdle(0);
        config.setTestOnBorrow(true);
        config.setTestWhileIdle(true);
        config.setNumTestsPerEvictionRun(2);
        config.setTimeBetweenEvictionRunsMillis(30000);
        config.setMinEvictableIdleTimeMillis(60000);
        config.setSoftMinEvictableIdleTimeMillis(60000);
        return config;
    }

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory redisConnectionFactory = new JedisConnectionFactory(jedisPoolConfig());
        redisConnectionFactory.setDatabase(redisProperties.getDatabase());
        redisConnectionFactory.setHostName(redisProperties.getHost());
        redisConnectionFactory.setPort(redisProperties.getPort());
        redisConnectionFactory.setPassword(redisProperties.getPassword());
        redisConnectionFactory.setTimeout(15000);
        redisConnectionFactory.setUsePool(true);
        return redisConnectionFactory;
    }

    @Bean
    public <K, V> RedisTemplate<K, V> redisTemplate() {
        RedisTemplate<K, V> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
//        template.setEnableTransactionSupport(true);
        template.afterPropertiesSet();
        return template;
    }

```


特别强调：推荐使用 GenericJackson2JsonRedisSerializer 去替代 Jackson2JsonRedisSerializer<Object>以及JacksonJsonRedisSerializer<Object>
因为GenericJackson2JsonRedisSerializer 可以保留泛型。