---
layout: post
title:  "Spring Boot学习记录:Mybatis"
date:   2019-05-15 16:19:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Mybatis

有两种模式：

- **注解模式**
- **XML模式**  

注解模式开发快，但是要动态SQL还是要XML模式。

### Maven

```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.0.0</version>
	</dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

### application.properties

```
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?serverTimezone=GMT&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
## 注解模式

### Mapper

Mapper可以用两种方式配置：

- **1.在启动类中添加包扫描`@MapperScan`**
- **2.在Mapper类上添加注解`@Mapper`**

建议使用包扫描，才不用每个Mapper都加注解：

```
@SpringBootApplication
@MapperScan("com.hiki.springbootlearn.mapper")
public class SpringbootlearnApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootlearnApplication.class, args);
    }
}
```

### Mapper开发

```
public interface UserMapper {
    @Select("SELECT * FROM users")
    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "name", column = "name"),
            @Result(property = "password", column = "password"),
            @Result(property = "age", column = "age")
    })//如果对象的字段名和数据库的字段名一样，则不需要这个Results注解
    public List<Users> getAll();

    @Select("SELECT * FROM users WHERE id = #{id}")
    public Users getOne(@Param("id") Long id);

    @Insert("INSERT INTO users(name, password, age) VALUES(#{name},#{password}, #{age})")
    public void add(Users user);

    @Update("UPDATE users SET name = #{name},password = #{password}, age = #{age} WHERE id = #{id}")
    public void update(Users user);

    @Delete("DELETE FROM users WHERE id = #{id}")
    public void remove(Long id);
}
```

**注意`@Results`和`@Param`注解，`@Param`注解是用于多参数时，单参数可以不用**

### 测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestMybatis {
    @Autowired
    private UserMapper userMapper;

    @Test
    //@Ignore
    public void testMybatis1(){
        List<Users> users = userMapper.getAll();
        for (Users user:users) {
            System.out.println(user.getName());
            System.out.println(user.getAge());
            System.out.println(user.getId());
            System.out.println(user.getPassword());
        }
    }

    @Test
    @Ignore
    public void testGetOne(){
        Users user = userMapper.getOne(14L);
        System.out.println(user.getName());
    }

    @Test
    @Ignore
    public void testAdd(){
        Users user = new Users();
        user.setAge(22L);
        user.setName("hiki");
        user.setPassword("123456");
        userMapper.add(user);
    }

    @Test
    @Ignore
    public void testUpdate(){
        Users user = new Users();
        user.setName("hikiy");
        user.setId(16L);
        user.setPassword("456789");
        user.setAge(22L);
        userMapper.update(user);
    }

    @Test
    @Ignore
    public void testRemove(){
        userMapper.remove(16L);
    }
}
```

## XML模式

### application.properties配置

在上面配置的基础上，新增如下配置：

```
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```

### mybatis-config.xml配置

```
<configuration>
    <typeAliases>
        <typeAlias alias="Integer" type="java.lang.Integer" />
        <typeAlias alias="Long" type="java.lang.Long" />
        <typeAlias alias="HashMap" type="java.util.HashMap" />
        <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
        <typeAlias alias="ArrayList" type="java.util.ArrayList" />
        <typeAlias alias="LinkedList" type="java.util.LinkedList" />
    </typeAliases>
</configuration>
```

### 映射文件配置

```
<mapper namespace="com.hiki.springbootlearn.mybatis.mapper.UsersMapper" >
    <resultMap id="BaseResultMap" type="com.hiki.springbootlearn.entity.Users" >
        <id column="id" property="id" jdbcType="BIGINT" />
        <result column="name" property="name" jdbcType="VARCHAR" />
        <result column="password" property="password" jdbcType="VARCHAR" />
        <result column="age" property="age" javaType="LONG"/>
    </resultMap>

    <sql id="Base_Column_List" >
        id, name, password, age
    </sql>

    <select id="getAll" resultMap="BaseResultMap"  >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
    </select>

    <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
        WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="com.hiki.springbootlearn.entity.Users" >
        INSERT INTO
        users
        (name,password,age)
        VALUES
        (#{name}, #{password}, #{age})
    </insert>

    <update id="update" parameterType="com.hiki.springbootlearn.entity.Users" >
        UPDATE
        users
        SET
        <if test="name != null">name = #{name},</if>
        <if test="password != null">password = #{password},</if>
        name = #{name}
        WHERE
        id = #{id}
    </update>

    <delete id="delete" parameterType="java.lang.Long" >
        DELETE FROM
        users
        WHERE
        id =#{id}
    </delete>
</mapper>
```


### Mapper层代码

```
public interface UsersMapper {
    public List<Users> getAll();

    public Users getOne(Long id);

    public void insert(Users user);

    public void update(Users user);

    public void delete(Long id);
}
```

<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.15  
> 更新日期：2019.05.23
