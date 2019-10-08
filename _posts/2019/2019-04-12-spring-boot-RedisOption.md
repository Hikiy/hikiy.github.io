---
layout: post
title:  "Spring Boot学习记录：Redis操作"
date:   2019-04-12 09:50:00 +0200
categories: SpringBoot
excerpt: 在SpringBoot中操作Redis
tagg: Spring
---

# Redis操作

依赖包：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

### Application配置

```
# Redis 数据库索引（默认为 0）
spring.redis.database=0
# Redis 服务器器地址
spring.redis.host=localhost
# Redis 服务器器连接端⼝口
spring.redis.port=6379
# Redis 服务器器连接密码（默认为空）
spring.redis.password=
# 连接池最⼤大连接数（使⽤用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最⼤大阻塞等待时间（使⽤用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最⼤大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最⼩小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0
```

Spring Boot 默认⽀支持 Lettuce 连接池  

### 测试使用

直接上代码：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestRedisTemplate {
    @Autowired
    private RedisTemplate redisTemplate;
    @Test
    public void testString() {
        redisTemplate.opsForValue().set("hiki", "knight");
        Assert.assertEquals("knight", redisTemplate.opsForValue().get("hiki"));
    }
}
```

**如果发现**`private RedisTemplate redisTemplate;`**报错：**

```
Could not autowire. There is more than one bean of 'RedisTemplate' type.Beans:redisTemplate&nbsp;&nbsp; (RedisAutoConfiguration.class)stringRedisTemplate&nbsp;&nbsp; (RedisAutoConfiguration.class)
```

可能是写成了`redistemplate`,说明要规范。或者我偶然在网上看到别的解决方案：

在一个配置类中定义这个类即可修复：

```
 @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```



### 实体

Redis支持Pojo类型

```
public void testObj(){
    User user=new User("hiki@cool.com", "good", "shoot", "knight","2019");
    ValueOperations<String, User> operations=redisTemplate.opsForValue();
    operations.set("com.hiki", user);
    User u=operations.get("com.hiki");
    System.out.println("user: "+u.toString());
}
```

### 超时

就是每个数据的超时时间，时间到了会自动删除。直接上代码：：

```
ValueOperations<String, User> operations=redisTemplate.opsForValue();
operations.set("hiki", user,100,TimeUnit.MILLISECONDS);//单位毫秒
```

### 删除

```
@Test
public void testDelete() {
    ValueOperations<String, User> operations=redisTemplate.opsForValue();
    redisTemplate.opsForValue().set("deletekey", "hiki");
    redisTemplate.delete("deletekey");
    boolean exists=redisTemplate.hasKey("deletekey");
    if(exists){
        System.out.println("exists is true");
    }else{
        System.out.println("exists is false");
    }
}
```

### Hash（哈希）

Hash Set 就在哈希表 Key中的域（Field）的值设为value。如果Key不存在，一个新的哈希表被创建并进行 HSET 操作；如果域（field）已经存在于哈希表中，旧值将被覆盖。

```
@Test
public void testHash() {
    HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
    hash.put("hash","you","you");
    String value=(String) hash.get("hash","you");
    System.out.println("hash value :"+value);
}
```

第一个为可以，第二个为field，第三个为存储的值。也就是类似二维数组,仔细想想就是hash的结构。  

### List

使用List可以轻松实现队列，典型应用场景是消息队列，利用Push、Pop操作。

```
@Test
public void testList() {
    ListOperations<String, String> list = redisTemplate.opsForList();
    list.leftPush("list","hiki");
    list.leftPush("list","px");
    list.leftPush("list","knight");
    String value=(String)list.leftPop("list");
    System.out.println("list value :"+value.toString());
}
```

结果：

```
list value :knight
```

这不是栈么。。。  
上面的例子是左插，还可以右插（右插的话应该就是队列了），Redis是双向链表，因此带来了额外的内存开销

### Set

与List类似，但是自动排重，Set 可以判断某个成员是否在Set集合内的重要接口。

```
@Test
public void testSet() {
    String key="set";
    SetOperations<String, String> set = redisTemplate.opsForSet();
    set.add(key,"hiki");
    set.add(key,"knight");
    set.add(key,"knight");
    set.add(key,"cool");
    Set<String> values=set.members(key);
    for (String v:values){
    System.out.println("set value :"+v);
    }
}
```

结果：

```
set value :hiki
set value :knight
set value :cool
```

#### difference方法

```
    SetOperations<String, String> set = redisTemplate.opsForSet();
    String key1="setMore1";
    String key2="setMore2";
    set.add(key1,"hiki");
    set.add(key1,"cool");
    set.add(key1,"knight");
    set.add(key1,"knight");
    set.add(key2,"xxx");
    set.add(key2,"knight");
    Set<String> diffs=set.difference(key1,key2);
    for (String v:diffs){
        System.out.println("diffs set value :"+v);
    }
```

结果：

```
diffs set value :hiki
diffs set value :cool
```

根据上面这个例⼦子可以看出，difference()函数会把key1中不同于key2的数据对比出来，这个特性适合我们在金融场景中对账的时候使用。

#### unions方法

将两个集合合起来，不贴代码了

> Set 的内部实现是一个 Value 永远为 null 的 HashMap，实际就是通过计算 Hash 的方式来快速排重，这也是 Set 能提供判断一个成员是否在集合内的原因。

### ZSet

ZSet是自动有序的，用户可以通过优先级的参数（Score）来为成员排序

```
@Test
public void testZset(){
    String key="zset";
    redisTemplate.delete(key);
    ZSetOperations<String, String> zset = redisTemplate.opsForZSet();
    zset.add(key,"it",1);
    zset.add(key,"you",6);
    zset.add(key,"know",4);
    zset.add(key,"neo",3);
    Set<String> zsets=zset.range(key,0,3);
    for (String v:zsets){
        System.out.println("zset value :"+v);
    }
    Set<String> zsetB=zset.rangeByScore(key,0,3);
    for (String v:zsetB){
        System.out.println("zsetB value :"+v);
    }
}
```

结果：

```
zset value :it
zset value :neo
zset value :know
zset value :you
zsetB value :it
zsetB value :neo
```

> Redis Sorted Set 的内部使用HashMap和跳跃表（SkipList）来保证数据的存储和有序，HashMap 里放的是成员到Score的映射，而跳跃表⾥存放的是所有的成员，排序依据是HashMap 里存的 Score，⽤用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。

### 封装

实际使用时，一般将redisTemplate封装到一个类中使用，例如：

```
@Service
public class RedisService {
    @Autowired
    private RedisTemplate redisTemplate;
}
```

```
public boolean set(final String key, Object value) {
    boolean result = false;
    try {
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        operations.set(key, value);
        result = true;
    } catch (Exception e) {
        logger.error("set error: key {}, value {}",key,value,e);
    }
    return result;
}
```

<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.04.12  
> 更新日期：2019.05.15
