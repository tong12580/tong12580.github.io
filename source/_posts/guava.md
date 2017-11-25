title: 在spring中利用Guava实现本地Cache的具体配置 第一篇
date: 2016/12/24 14:13:00
tags: 
    - cache
    - Guava
    - spring
categories:
    - cache
---

## 基于注解的配置实现spring Cache本地缓存 操作
`spring` `Guava` `cache`
$begin$
> - Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库
> - 通常来说，Guava Cache适用于：
> 
    1.你愿意消耗一些内存空间来提升速度。
    2.你预料到某些键会被查询一次以上。
    3.缓存中存放的数据总量不会超出内存容量。
    **注意!!!** **（Guava Cache是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果这不符合你的需求，请尝试Memcached, redis缓存架构!!! 个人更推荐redis）**
> - 具体Guava cache说明 [可参阅这里](http://ifeve.com/google-guava-cachesexplained/)

****

接下来就该说说spring中Guava的基于注解的实现的配置

当然 ,这里只说基于注解的配置和使用 我不会去说注解的原理 这不是本文要说的 .

> 1.首先 新建一个spring-cache.xml配置文件 并将文件配置于web.xml中
> 2. spring-cahe文件配置如下:

```xml
<!-- 声明缓存注解的开启 -->
<cache:annotation-driven cache-manager="cacheManager" proxy-target-class="true"/>
<!-- 声明使用spring管理缓存组 -->
    <bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
        <property name="cacheManagers">
            <list>
                <ref bean="guavaCacheManager"/>
            </list>
        </property>
        <property name="fallbackToNoOpCache" value="true"/>
    </bean>
    <!-- 缓存的创建的具体实现类注入 class为自定义的类 -->
    <bean id="guavaCacheManager" class="cn.springmvc.cache.GuavaCacheManagerConfig">
    <!-- 此处可以配置一组缓存池对应不同的业务类型 这里我先实现了一个交"likr"的默认缓存并构建 -->
        <property name="configMap">
            <map key-type="java.lang.String" value-type="com.google.common.cache.CacheBuilder">
                <entry key="likr" value-ref="defaultCacheBuilder"/>
            </map>
        </property>
    </bean>
    
    <!-- 此处直接构建"likr"默认缓存 -->
    <bean id="defaultCacheBuilder"
          class="com.google.common.cache.CacheBuilder"
          factory-method="from">
          <!-- 缓存池大小 时间(定时回收 缓存项在给定时间内没有被'写'访问 回收 还有refreshAfterWrite expireAfterAccess可供使用) 当然还有一些其他可选组件(weakKeys,removalListener and so on!) -->
        <constructor-arg value="maximumSize=10000, expireAfterWrite=1h"/>
    </bean>
```
>3.GuavaCacheManagerConfig的具体实现

```java
public class GuavaCacheManagerConfig extends AbstractTransactionSupportingCacheManager {

    private final ConcurrentMap<String, Cache> cacheMap = Maps.newConcurrentMap();
    private Map<String, CacheBuilder> builderMap = Maps.newHashMap();
    
    @Override
    protected Collection<? extends Cache> loadCaches() {
        return cacheMap.values();
    }
	
	//获取缓存单例
    @Override
    public Cache getCache(String name) {
        Cache cache = this.cacheMap.get(name);
        if (null == cache) {
            synchronized (this.cacheMap) {
                cache = this.cacheMap.get(name);
                if (null == cache && this.builderMap.containsKey(name)) {
                    CacheBuilder builder = this.builderMap.get(name);
                    cache = createGuavaCache(name, builder);
                    this.cacheMap.put(name, cache);
                }
            }
        }
        return cache;
    }

    private Cache createGuavaCache(String name, CacheBuilder builder) {
        com.google.common.cache.Cache<Object, Object> cache;
        if(builder == null){
            cache = CacheBuilder.newBuilder().build();
        }else{
            cache = builder.build();
        }
        return new GuavaCache(name, cache, isAllowNullValues());
    }


    private boolean isAllowNullValues() {
        return true;
    }

	//配置中多组缓存池注入
    public void setConfigMap(Map<String, CacheBuilder> configMap) {
        this.builderMap = configMap;
    }
}
```

>4.代码中使用:

```java
//spring EL
//LikrUserQuestionnaire 普通的DTO
//@Cacheable(value = "likr", key = "'selectByRecordOne:userId:' + #record.user_id")
//value 为配置文件中的缓存池名称 key 为键名
	@Override
    @Cacheable(value = "likr", key = "'selectByRecordOne:userId:' + #record.user_id")
    public LikrUserQuestionnaire selectByRecordOne(LikrUserQuestionnaire record) {
        return likrUserQuestionnaireMapper.selectByRecordOne(record);
    }
```

$end$

 
