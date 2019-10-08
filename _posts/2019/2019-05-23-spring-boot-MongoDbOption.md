---
layout: post
title:  "Spring Boot学习记录：MongoDb操作"
date:   2019-05-23 15:31:00 +0200
categories: SpringBoot
excerpt: Spring Boot中使用MongoDB
tagg: Spring
---

# MongoDb操作

### pom依赖
```
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency> 
```
### application.properties配置
```
spring.data.mongodb.uri=mongodb://name:pass@localhost:27017/test
```
多个IP集群使用下面的配置：
```
spring.data.mongodb.uri=mongodb://user:pwd@ip1:port1,ip2:port2/database
```

### 实体
```
Entity//javax.persistence.Entity;
public class Users implements Serializable {
    private static final long serialVersionUID = 7945207204170209646L;

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String password;
    private Long age;
    
    //...省略get set 和 构造方法
}
```

### Dao
接口就不贴了，直接贴实现类
```
@Component
public class UserMongoDaoImpl implements UserMongoDao {
    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public void saveUser(Users user) {
        mongoTemplate.save(user);
    }

    @Override
    public Users findUserById(Long id) {
        Query query=new Query(Criteria.where("id").is(id));
        Users user =  mongoTemplate.findOne(query , Users.class);
        return user;
    }

    @Override
    public long updateUser(Users user) {
        Query query=new Query(Criteria.where("id").is(user.getId()));
        Update update= new Update().set("name", user.getName()).set("password", user.getPassword()).set("age",user.getAge());
        //更新查询返回结果集的第一条
        UpdateResult result =mongoTemplate.updateFirst(query,update,Users.class);
        //更新查询返回结果集的所有
        // mongoTemplate.updateMulti(query,update,UserEntity.class);
        if(result!=null)
            return result.getMatchedCount();
        else
            return 0;
    }

    @Override
    public void deleteUserById(Long id) {
        Query query=new Query(Criteria.where("id").is(id));
        mongoTemplate.remove(query,Users.class);
    }
}
```
### 测试
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestMongodb {
    @Autowired
    private UserMongoDao umd;

    @Test
    @Ignore
    public void testSave()throws Exception{
        Users user = new Users();
        user.setId(1L);
        user.setName("Hikiy");
        user.setAge(22L);
        user.setPassword("123456");
        umd.saveUser(user);
    }
    @Test
    @Ignore
    public void testFindById()throws Exception{
        Users user = umd.findUserById(1L);
        System.out.println(user.getName());
        System.out.println(user.getPassword());
        System.out.println(user.getAge());
    }

    @Test
    @Ignore
    public void testUpdate()throws  Exception{
        Users user = new Users();
        user.setId(1L);
        user.setName("newHiki");
        user.setPassword("456789");
        user.setAge(22L);
        long result = umd.updateUser(user);
        if( result == 1 ){
            System.out.println("success!");
        }else{
            System.out.println("fail!");
        }
    }
}
```
### 多数据源日后补充


<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.23  
> 更新日期：2019.05.23
