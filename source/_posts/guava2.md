title: 在spring中利用Guava实现本地Cache的具体配置 第二篇
date: 2016/09/18 14:07:00
tags: 
    - cache
    - Guava
    - spring
categories:
    - cache
---

### 通过直接实现org.springframework.cache.Cache 接口来管理和实现缓存

*****
springframework 为我们提供了一套公用的接口模版标准 本文将通过Guava工程 利用单例实现Cache 接口来完成本地缓存的配置

具体代码如下:

```
public class GuavaCacheUtil implements Cache {

    private static Logger log = LoggerFactory.getLogger(GuavaCacheUtil.class);
    private static final Object NULL_HOLDER = new NullHolder();
    private static final String DEFAULT_NAME = "system_cache";
    private final String name;
    private final com.google.common.cache.Cache<Object, Object> cache;
    private final boolean allowNullValues;
    private static GuavaCacheUtil guavaCacheUtil = new GuavaCacheUtil();

    public static GuavaCacheUtil builder() {
        return guavaCacheUtil;
    }

    private GuavaCacheUtil() {
        this(DEFAULT_NAME, CacheBuilder.newBuilder().maximumSize(100).expireAfterWrite(1, TimeUnit.HOURS).build());
    }

    private GuavaCacheUtil(String name, com.google.common.cache.Cache<Object, Object> cache) {
        this(name, cache, true);
    }

    private GuavaCacheUtil(String name, com.google.common.cache.Cache<Object, Object> cache, boolean allowNullValues) {
        Assert.notNull(name, "Name must not be null");
        Assert.notNull(cache, "Cache must not be null");
        this.name = name;
        this.cache = cache;
        this.allowNullValues = allowNullValues;
    }

    @Override
    public String getName() {
        return this.name;
    }

    @Override
    public Object getNativeCache() {
        return this.cache;
    }

    @Override
    public ValueWrapper get(Object key) {
        key = getKey(key.toString());
        Object value = this.cache.getIfPresent(key);
        log.info("getKey = {}, getObject = {}", key, JsonUtil.objectToJson(value));
        return toWrapper(value);
    }

    @Override
    public <T> T get(Object key, Class<T> aClass) {
        key = getKey(key.toString());
        T value = fromStoreValue(this.cache.getIfPresent(key), aClass);
        log.info("getKey = {}, getObject = {}", key, JsonUtil.objectToJson(value));
        return value;
    }

    @Override
    public <T> T get(Object o, Callable<T> callable) {
        return null;
    }

    @Override
    public void put(Object key, Object value) {
        key = getKey(key.toString());
        this.cache.put(key, toStoreValue(value));
        log.info("getKey = {}, getObject = {}", key, JsonUtil.objectToJson(value));
    }

    @Override
    public ValueWrapper putIfAbsent(Object key, Object value) {
        PutIfAbsentCallable callable = new PutIfAbsentCallable(value);
        try {
            Object result = this.cache.get(key, callable);
            return (callable.called ? null : toWrapper(result));
        } catch (ExecutionException e) {
            throw new IllegalStateException(e);
        }
    }

    @Override
    public void evict(Object key) {
        this.cache.invalidate(key);
        log.info("deleteKey = {}", key);
    }

    @Override
    public void clear() {
        this.cache.invalidateAll();
        log.info("clearAll");
    }

    private ValueWrapper toWrapper(Object value) {
        return (value != null ? new SimpleValueWrapper(fromStoreValue(value, null)) : null);
    }

    private static class NullHolder implements Serializable {
    }

    private Object toStoreValue(Object userValue) {
        if (this.allowNullValues && userValue == null) {
            return NULL_HOLDER;
        }
        return userValue;
    }

    private <T> T fromStoreValue(Object storeValue, Class<T> aClass) {
        if (this.allowNullValues && storeValue == NULL_HOLDER) {
            return null;
        }
        if (storeValue != null && aClass != null && !aClass.isInstance(storeValue)) {
            throw new IllegalStateException("Cached value is not of required type [" + aClass.getName() + "]: " + storeValue);
        }
        return (T) storeValue;
    }


    private class PutIfAbsentCallable implements Callable<Object> {

        private final Object value;

        private boolean called;

        public PutIfAbsentCallable(Object value) {
            this.value = value;
        }

        @Override
        public Object call() throws Exception {
            this.called = true;
            return toStoreValue(this.value);
        }
    }

    private String getKey(String key) {
        return name + ":" + key;
    }
}
```
测试代码 以及测试结果:

```
public static String testCachePut(){
        Cache cache = GuavaCacheUtil.builder();
        cache.put("user_id:1","20");
        String user_id = cache.get("user_id:1", String.class);
        System.out.println(user_id);
        return user_id;
    }

    public static String testCacheGet(){
        Cache cache = GuavaCacheUtil.builder();
        cache.put("user_id:2","30");
        String user_id = cache.get("user_id:1", String.class);
        System.out.println(user_id);
        System.out.println(cache.get("user_id:2",String.class));
        return user_id;
    }

    public static void main(String[] args) {
        testGuavaCachePut();
        testGuavaCacheGet();
    }
```
测试结果 : ![测试结果](http://img.blog.csdn.net/20161224143049467?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjNva2Vy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)