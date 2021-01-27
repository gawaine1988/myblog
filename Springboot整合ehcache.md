# EhCache简介

Ehcache主要基于内存缓存，磁盘缓存为辅的。

# Springboot 整合Ehcache的方法

## Maven依赖

```
<!--开启 cache 缓存 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- ehcache缓存 -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.9.1</version><!--$NO-MVN-MAN-VER$ -->
</dependency>
```



## ehcache配置

1. ehcache的配置需要通过xml文件的方式，文件示例如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <!--timeToIdleSeconds 当缓存闲置n秒后销毁 -->
    <!--timeToLiveSeconds 当缓存存活n秒后销毁 -->
    <!-- 缓存配置 
        name:缓存名称。 
        maxElementsInMemory：缓存最大个数。 
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。 
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。 
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。 
        overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。 diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。 
        maxElementsOnDisk：硬盘最大缓存个数。 
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk 
        store persists between restarts of the Virtual Machine. The default value 
        is false. 
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。  memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是 
LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。 
        clearOnFlush：内存数量最大时是否清除。 -->
    <!-- 磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir" />
    <!-- 默认缓存 -->
    <defaultCache 
        maxElementsInMemory="10000" 
        eternal="false"
        timeToIdleSeconds="120" 
        timeToLiveSeconds="120" 
        maxElementsOnDisk="10000000"
        diskExpiryThreadIntervalSeconds="120" 
        memoryStoreEvictionPolicy="LRU">

        <persistence strategy="localTempSwap" />
    </defaultCache>

    <!-- 测试 -->
    <cache name="GoodsType" 
        eternal="false" 
        timeToIdleSeconds="2400"
        timeToLiveSeconds="2400" 
        maxEntriesLocalHeap="10000"
        maxEntriesLocalDisk="10000000" 
        diskExpiryThreadIntervalSeconds="120"
        overflowToDisk="false" 
        memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>
```





2. 还需要在spring的配置文件中配置

   ```
   spring.cache.ehcache.config=classpath:ehcache.xml
   spring.cache.type=ehcache
   
   ```







## 代码示例

```
@Service
public class DemoEhcacheService implements DemoService {

    @Override
    //value是指在ehcahce中指定的cache的名称
    @Cacheable(key = "#id", value = "GoodsType")
    public String demoCache(String id) {
        System.out.println("cacheAble " + id);
        return "testEhCacheValue" + id;
    }

    @Override
    @CachePut(cacheNames = "GoodsType")



    public String demoCachePut(String id) {
        return "cachePut " + id;
    }

    @Override
    @CacheEvict(cacheNames = "GoodsType")
    public String demoCacheDelete(String id) {
        return "cacheDelete" + id;
    }
}

```

程序说明：

1. Cacheable注解中的参数说明

   | 参数名称      | 参数值                                                       |
   | ------------- | ------------------------------------------------------------ |
   | value         | 缓存名称，在ehcache的xml中由name指定                         |
   | cacheNames    | 和value一样                                                  |
   | key           | key就是缓存中的key值。key有三种方法执行：<br />1. 默认情况下不指定key的时候，spring会根据所有的方法参数计算出key。<br />2. 通过spel表达式获取参数中的值作为key，如上述的代码示例。<br />3. 使用keyGenerator参数。 |
   |               |                                                              |
   | keyGenenrator | key生成器。                                                  |
   |               |                                                              |
   | cacheManager  | 指定cacheManager                                             |
   |               |                                                              |
   | cacheResolver | 指定cacheResolver                                            |
   | Condition     | 缓存数据的条件                                               |
   | unless        | 否定缓存, 但是和condition不同，使用unless的时候会先调用方法。所以unless也可以使用spel表达式中的result |
   | sync          | 多线程同步开关。将使用了cacheAble注解的方法进行多线程同步操作，也就是说，如果多个线程同时传入相同key，并且该key还没有被缓存，则只有一个线程的会真正调用被注解的方法，其他的会直接从缓存中取出数据。 |

