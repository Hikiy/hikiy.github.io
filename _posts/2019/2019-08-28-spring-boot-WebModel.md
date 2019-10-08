---
layout: post
title:  "Spring Boot学习记录：Web开发模板"
date:   2019-08-28 17:28:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Web开发模板
集合一些常用的功能

</br>
</br>
</br>

## Mapper层

### 1.使用Mybatis

#### 依赖

```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### 启动类Application上添加 `@MapperScan` 注解
```
@MapperScan("com.hiki.mapper")
@SpringBootApplication
public class AlbumApplication {
    public static void main(String[] args) {
		SpringApplication.run(AlbumApplication.class, args);
	}
}
```

#### Mapper类
```
@Mapper
public interface UserMapper {
}
```

#### Insert

```
    @Insert("INSERT INTO users (id, mobile, created, updated) values ( #{id}, #{mobile}, #{name}, #{created}, #{updated})")
    @Options(useGeneratedKeys = true,keyProperty = "id", keyColumn = "id")
    Integer insertUser(UserEntity userEntity);
```

> 这里使用**实体类**作为参数</br>
> `@Options` 注解定义了自增id

#### Update

```
    @Update("UPDATE users SET name = #{name} , updated = #{updated} WHERE id = #{id}")
    Integer updateName(UserEntity userEntity);
```

#### Select

单个查询：

```
    @Select("SELECT id From users where mobile = #{mobile}")
    UserEntity queryUserByMobile(@Param("mobile") String mobile);
```

多个查询：

```
    @Select("SELECT iid, name FROM investors WHERE status = 1 ORDER BY iid ASC LIMIT #{limitStart}, #{limitEnd}")
    List<InvestorEntity> getInvestorList(int limitStart, int limitEnd);
```

### 2.使用JPA

#### 实现JpaRepository

```
public interface UsersRepository extends JpaRepository<Users, Integer> {
    public List<Users> findAll();

    public Users findAllByUsername(String username);

    public Users findByUid(int uid);

    @Transactional
    public void deleteByUid(int uid);
}
```

#### Insert

使用接口的 `.save()`方法即可：
```
    Users user = new Users();
    user.setUsername(username);
    user.setName(name);
    user.setPassSalt(passSalt);
    user.setPassHash(passHash);
    user.setStatus(1);
    user.setCreated(time);
    user.setUpdated(time);

    try {
        usersRepository.save(user);
    }catch (Exception e){
        return false;
    }
```

#### Delete
再进行删除的时候，需要添加事务注解 `@Transactional` ，否则会报错:
```
    @Transactional
    public void deleteByUid(int uid);
```

#### Select

```
    //查全部
    List<AlbumCategory> findAll();
    
    //指定查询字段
    @Query(value = "select acid, banner from AlbumCategory")
    List<Object> findBannerList();
    
    //ORDER BY
    List<AlbumCategory> findAllByAidOrderByPriorityDesc(int aid);
```

#### Update
也是使用 `.save()` 方法，需要将id填进去


</br>
</br>
</br>

## Service 层
实现类添加 `@Service` 注解即可



</br>
</br>
</br>

## Json返回
### Json包装类
```
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseBase {

    private Integer ret;
    private String msg;
    private Object data;
    
    public ResponseBase() {
        
    }
    
    public ResponseBase(Integer ret, String msg, Object data) {
        super();
        this.ret = ret;
        this.msg = msg;
        this.data = data;
    }
    

    public ResponseBase(Integer ret, String msg) {
        super();
        this.ret = ret;
        this.msg = msg;
    }

    @Override
    public String toString() {
        if(data == null ) {
            return "ResponseBase [ret=" + ret + ", msg=" + msg +  "]";
        } else {
            return "ResponseBase [ret=" + ret + ", msg=" + msg + ", data=" + data + "]";
        }
    }

}
```

</br>
</br>
</br>

## Controller层
### 返回Json的接口
```
@RestController
@RequestMapping(value = "/hiki", produces="application/json;charset=UTF-8")
public class HikiController {

    @Autowired
    private HikiService hikiService;

    @RequestMapping(value = "/index")
    public String index() {
        return null;
    }

}
```
注意`produces="application/json;charset=UTF-8"`是可以省略的。

</br>
</br>
</br>

## redis

### 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 配置

```
# Redis
spring.redis.database = 0
spring.redis.host = 
spring.redis.port = 6379
spring.redis.password = 

#最大可用连接数（默认为8，负数表示无限） 
spring.redis.jedis.pool.max-active = 100 

# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait = -1

# 最大空闲连接数（默认为8，负数表示无限）
spring.redis.jedis.pool.max-idle = 10

# 最小空闲连接数（默认为0，该值只有为正数才有作用） 
spring.redis.jedis.pool.min-idle = 1

# 连接超时时间（毫秒）
spring.redis.timeout = 500
```

### 使用RedisTemplate

```
@Autowired
private RedisTemplate redisTemplate;
    
    //写入
    if (!redisTemplate.hasKey("WEATHER_" + city)) {
        redisTemplate.opsForValue().set("WEATHER_" + city, WeatherUtil.getWeather(city), 3600000, TimeUnit.MILLISECONDS);
    }
    
    //获取(需要转换类型 get()方法获取的是Object类型)
    String value = redisTemplate.opsForValue().get("WEATHER_" + city).toString();
```

### 使用StringRedisTemplate
使用这个的好处是，拿到的value是String类型，不必再转换
#### 获取：

```
String redisKey = "HIKI_NAME";
String cache = stringRedisTemplate.opsForValue().get(redisKey);
```

#### 写入

```
@Autowired
private StringRedisTemplate stringRedisTemplate;

    //写入
    stringRedisTemplate.opsForValue().set(redisKey, theJson);
    
    //设置key失效时间
    int cacheDay = 86400;
    stringRedisTemplate.expire(redisKey, cacheDay, TimeUnit.SECONDS);
```

</br>
</br>
</br>

## 发起Http请求

### 使用RestTemplate发起请求

```
    import com.alibaba.fastjson.JSONObject;

    RestTemplate restTemplate = new RestTemplate();
    JSONObject resJson = restTemplate.getForEntity(url,JSONObject.class).getBody();
    String name = resJson.getString("name");
    String id = resJson.getString("id");
```

其中用的 `JSONObject` 依赖：
```
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.57</version>
        <scope>compile</scope>
    </dependency>
```

</br>
</br>
</br>

## 环境打包
### 多环境配置

```
    <!-- 多环境配置 -->
    <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <!-- 默认情况下使用dev配置 -->
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <profileActive>dev</profileActive>
            </properties>
        </profile>

        <!-- 生产环境，打包命令"maven clean package -P prod" -->
        <profile>
            <id>prod</id>
            <!-- applicateion-dev.properties -->
            <properties>
                <profileActive>prod</profileActive>
            </properties>
        </profile>
    </profiles>
```

### 开启Maven指定环境配置

```
# 使用Maven时指定环境配置（dev/prod）
spring.profiles.active = @profileActive@
```

### 打包配置

```
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <!-- 项目打包完成的包中只包含当前环境文件 -->
                    <include>application.properties</include>
                    <include>application-${profileActive}.properties</include>
                </includes>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- 配置修改文件保存后自动重启 -->
                <configuration>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

</br>
</br>
</br>

## 加密

### sha256加密
```
    public static String getSHA256(String str){
        MessageDigest messageDigest;
        String encodestr = "";
        try {
            messageDigest = MessageDigest.getInstance("SHA-256");
            messageDigest.update(str.getBytes("UTF-8"));
            encodestr = byte2Hex(messageDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return encodestr;
    }

    /**
    * 将byte转为16进制
    * @param bytes
    * @return
    */
    private static String byte2Hex(byte[] bytes){
        StringBuffer stringBuffer = new StringBuffer();
        String temp = null;
        for (int i=0;i<bytes.length;i++){
            temp = Integer.toHexString(bytes[i] & 0xFF);
            if (temp.length()==1){
            //1得到一位的进行补0操作
            stringBuffer.append("0");
            }
        stringBuffer.append(temp);
        }
        return stringBuffer.toString();
    }
```

### hash对称加密
#### 依赖
```
<dependency>
    <groupId>org.hashids</groupId>
    <artifactId>hashids</artifactId>
    <version>1.0.3</version>
</dependency>
```

#### 使用
```
    //加密
    Hashids hashids = new Hashids(salt, 15);
    String hashid = hashids.encodeHex(String.valueOf(id));
    
    //解密
    Hashids hashids = new Hashids(salt, 15);
    long id = Long.valueOf(hashids.decodeHex(encodeId));
```

### MD5加密
#### 使用封装类 `DigestUtils` 实现
```
import org.springframework.util.DigestUtils;

    DigestUtils.md5DigestAsHex("1".getBytes());
```
源码实现其实是使用 `MessageDigest` ，所以可以使用MessageDigest直接实现

#### 使用MessageDigest加密
```
public static String encode(String str){
        try {
        MessageDigest digest = MessageDigest.getInstance("md5");
        byte[] buffer = digest.digest(str.getBytes());

        StringBuffer sb = new StringBuffer();
        for (byte b : buffer) {
            int a = b & 0xff;
            // Log.d(TAG, "" + a);
            String hex = Integer.toHexString(a);

            if (hex.length() == 1) {
                hex = 0 + hex;
            }
            sb.append(hex);
        }
        return sb.toString();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
    return null;
    }
```

## 拦截器
### 1.实现 `HandlerInterceptor`

这里用自己写过的一个权限拦截做例子：

```
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 目标方法执行之前执行
     *
     * @param request
     * @param response
     * @param handler
     * @return
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        Object uid = request.getSession().getAttribute("uid");
        //没有登录，返回错误页面
        if( uid == null || (int)uid < 1 ){
            try {
                response.sendRedirect("/auth_error");     //没有uid信息的话进行路由重定向
            } catch (IOException e) {
                e.printStackTrace();
            }
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

    }
}
```

### 2.配置拦截器

```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    /**
     * 自定义拦截规则
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // addPathPatterns - 用于添加拦截规则
        // excludePathPatterns - 用户排除拦截

        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/admin/**");

        //      .excludePathPatterns("/index.html", "/", "/user/login");
    }
}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.8.28  
> 更新日期：2019.8.29
