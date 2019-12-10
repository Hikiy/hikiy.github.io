---
layout: post
title:  "Spring Boot学习记录:Mybatis使用总结"
date:   2019-05-15 16:19:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Mybatis使用总结
有两种模式：
- **注解模式**
- **XML模式**  

### Maven:
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
### application.properties:
```
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?serverTimezone=GMT&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
## 注解模式
### 1.Mapper
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
### 2.Mapper开发
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

#### #{} 和 ${} 的区别

`#{}` 可以防止Sql 注入，它会将所有传入的参数作为一个字符串来处理。

 `${}`  则将传入的参数拼接到Sql上去执行，一般用于表名和字段名参数，`${}` 所对应的参数应该由服务器端提供，前端可以用参数进行选择，避免 Sql 注入的风险

### 3.动态sql

#### 方法一：

```
    @UpdateProvider(type = DistrictProvider.class, method = "updateDistrict")
    int updateDistrict(DistrictEntity districtEntity);

    class DistrictProvider {
        public String updateDistrict(DistrictEntity districtEntity) {
            String sql = "UPDATE districts SET updated = #{updated}";
            if( districtEntity.getCityid() != null ){
                sql += ", cityid = #{cityid} ";
            }
            if( districtEntity.getName() != null ){
                sql += ", name = #{name} ";
            }
            if( districtEntity.getStatus() != null ){
                sql += ", status = #{status} ";
            }
            sql += "WHERE id = #{id}";
            return sql;
        }
```

#### 方法二：

```
    @Select({
            "<script>",
            "select",
                "id, mobile, name, status, type",
            "from parents",
                "where id in",
                "<foreach collection='downPids' item='pid' open='(' separator=',' close=')'>",
                "#{pid}",
                "</foreach>",
            "AND (status != 0 OR status != 3)",
            "ORDER BY id DESC",
            "</script>"})
    List<ParentEntity> queryByPids(@Param("downPids") List<Integer> downPids);
```

### 4.测试
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
### 1.application.properties配置
在上面配置的基础上，新增如下配置：
```
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```

### 2.mybatis-config.xml配置：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
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

### 3.映射文件配置：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
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
### 4.Mapper层代码：
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
> 更新日期：2019.12.10
