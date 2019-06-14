---
layout: post
title:  "Spring Boot学习记录：测试方法和热部署"
date:   2019-05-23 16:02:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# 测试方法和热部署

### 单元测试

#### pom依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```
#### 测试类
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {

	@Test
	public void hello() {
		System.out.println("hello world");
	}

}
```
#### Dao层和Service层测试
```
//简单验证结果集是否正确
Assert.assertEquals(22, user.getAge());

//验证结果集，提示
Assert.assertTrue("后面的判断错了就打印这句话", user.getName().equals("newHikii");
Assert.assertFalse("后面的判断对了就打印这句话",user.getName().equals("newHiki"));
```

#### Controller层测试:MockMvc
```
public class HelloControlerTests {

    private MockMvc mvc;

    //初始化执行
    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }

    //验证controller是否正常响应并打印返回结果
    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
    }
    
    //验证controller是否正常响应并判断返回结果是否正确
    @Test
    public void testHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello World")));
    }

}
```
#### 热部署
```
 <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
</plugins>
</build>
```
<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.23  
> 更新日期：2019.05.23
