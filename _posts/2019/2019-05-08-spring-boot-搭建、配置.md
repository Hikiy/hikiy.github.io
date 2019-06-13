---
layout: post
title:  "Spring Boot学习记录：搭建、配置"
date:   2019-05-08 12:03:00 +0200
categories: SpringBoot
excerpt: Spring Boot的搭建、配置
tagg: Spring
---

# 搭建、配置

有几种方式可以快速搭建Spring Boot:
- http://start.spring.io
- eclipse通过Spring Tool Suite
- IntelliJ IDEA
- Spring Boot CLI
- Maven手工构建

## maven构建项目
- 访问http://start.spring.io/  
- 选择构建工具Maven Project、Spring Boot版本1.3.6以及一些工程基本信息，点击“Switch to the full version.”java版本选择1.7，可参考下图所示：  
![image](https://note.youdao.com/yws/public/resource/ebe32ace894b7d61f63a3ad5c863b94d/xmlnote/8B034FF4080F49638D45A2046518B846/12345)

spingboot建议的目录结果如下：  
root package结构：com.example.myproject
```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |
```
- Application.java 建议放到跟目录下面,主要用于做一些框架配置

- domain目录主要用于实体（Entity）与数据访问层（Repository）

- service 层主要是业务类代码

- controller 负责页面访问控制
```
myproject
 +-src
 		+- main
 +- java
 +- com.example.myproject
 					+- comm
 					+- model
 					+- repository
 					+- service
 					+- web
    +- Application.java
 +- resources
 +- static
 +- templates
 +- application.properties
 +- test
 +-pom.xml
```
com.example.myproject 目录下：  
- Application.java，建议放到根目录下面，是项目的启动类，Spring Boot 项目只能有1个 main() 方法；  
- comm 目录建议放置公共的类，如全局的配置文件、工具类等；  
- model 目录主要用于实体（Entity）与数据访问层（Repository）；  
- repository 层主要是数据库访问层代码；  
- service 层主要是业务类代码；  
- web 层负责页面访问控制  

resources 目录下：
- static 目录存放 web 访问的静态资源，如 js、css、图片等；
- templates 目录存放页面模板；
- application.properties 存放项目的配置信息

maven
`<scope>test</scope>` ，表示依赖的组件仅仅参与测试相关的⼯作，包括测试代码的编译和执
行，不会被打包包含进去；
spring-boot-starter-test 是 Spring Boot 提供项目测试的工具包，内置了多种测试工具，以
便我们在项目中做单元测试、集成测试。

## 引入web模块

**pom.xml中添加支持web的模块：**

<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
 </dependency>
pom.xml文件中默认有两个模块：
spring-boot-starter ：核心模块，包括自动配置支持、日志和YAML；
spring-boot-starter-test ：测试模块，包括JUnit、Hamcrest、Mockito。

## 编写Controller
```
@RestController
public class HelloWorldController {   
 
    @RequestMapping("/hello")    
    public String index() { 
        return "Hello World";
    }
}
```

如果想传入参数，可以使用
```
@RequestMapping(value="/")
	public String index(String desc) {
		return "hiki"+desc;
	}
```
访问http://localhost:9090/?desc=%20so%20cool 这是get方法

## 开发环境的调试

修改 Controller内相关的代码，需要重新启动项目才能生效，这样做很麻烦是不是？Spring Boot 又给我们提供了另外一个组件来解决

热部署
热启动就需要⽤到我们在⼀开始就引⼊的另外⼀个组件：spring-boot-devtools。它是 Spring Boot 提供的一组开发工具包，其中就包含我们需要的热部署功能，在使用这个功能之前还需要再做一些配置。但springBoot对调试支持很好，修改之后可以实时生效，需要添加以下的配置：
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
该模块在完整的打包环境下运行的时候会被禁用。如果你使用java -jar启动应用或者用一个特定的classloader启动，它会认为这是一个“生产环境”。

如果你使用的是 IDEA 集成开发环境，那么还需要做以下配置。  
- 选择 File | Settings | Compiler 命令  
![image](https://note.youdao.com/yws/public/resource/ebe32ace894b7d61f63a3ad5c863b94d/xmlnote/3E85488221A5457DA568F253344C4161/12450)

- 使用快捷键 Ctrl + Shift + A，在输入框中输入 Registry，勾选compile.automake.allow.when.app.running 复选框  

**为什么 IDEA 需要多配置后面这一步呢**？因为 IDEA 默认不是自动编译的，需要我们手动去配置后才会自动编译，而热部署依赖于项目的自动编译功能。

## 单元测试
普通测试是用@Test  
但是当要测试Web层的时候，Spring Boot 体系中，Spring 给出了一个简单的解决方案，使用MockMVC 进行 Web 测试，MockMVC 内置了很多工具类和方法，可以模拟 post、get 请求，并且判断返回的结果是否正确等，也可以利用 print() 打印执行结果
```
@SpringBootTest
public class DemoApplicationTests {
	private MockMvc mockMvc;

	@Before
	public void setup() {
		mockMvc = MockMvcBuilders.standaloneSetup(new IndexController()).build();
	}
	
	@Test
	public void getname() throws Exception {
		mockMvc.perform(MockMvcRequestBuilders.post("/?desc= so cool").accept(MediaType.APPLICATION_JSON_UTF8)).andDo(print());
	}
}
```

`@Before` 注解的方法表示在测试启动的时候优先执行，一般用作资源初始化。  
`.accept(MediaType.APPLICATION_JSON_UTF8)) `这句是设置 JSON 返回编码，避免出现中文乱码的问题。注意导包需要加入以下代码，
```
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
```
因为里面的print等方法都是静态方法

在类上面要添加@SpringBootTest，系统会自动加载Spring Boot容器

**但是这个方法会返回很多东西，不太容易识别返回结果**。MockMVC提供了更多方法来判断返回结果。如：  
将getname方法改变：  
```
@Test
	public void getname() throws Exception {
		mockMvc.perform(MockMvcRequestBuilders.post("/?desc= so cool").accept(MediaType.APPLICATION_JSON_UTF8))/*.andDo(print());*/
		.andExpect(MockMvcResultMatchers.content().string(Matchers.containsString("px")));
	}
```
会测试失败：  
```
Expected: a string containing "px"
but: was "hiki so cool so cool"
```

`MockMvcResultMatchers.content()`，这段代码的意思是获取到 Web 请求执行后的结果
`Matchers.containsString("px")`判断返回的结果集中是否包含“px”这个字符串



## Maven手工构建

1.Maven项目构建  
2.修改pom.xml
```
<!-- Spring Boot 父级依赖 -->
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
   </parent>

  <dependencies>
 <!-- 核心模块，包括自动配置支持、日志和YAML -->
    	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
 <!-- 测试模块，包括JUnit、Hamcrest、Mockito -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
 <!-- 引入Web模块 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
  </dependencies>
  <build>
        <plugins>
     <!-- Spring Boot 编译插件 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

## Spring Boot的基础结构

共三个文件：
- src/main/java下的程序入口：TestApplication
- src/main/resources下的配置文件：application.properties
- src/test/下的测试入口

`TestApplicationTests`类都可以直接运行来启动当前创建的项目，由于目前该项目未配合任何数据访问或Web模块，程序会在加载完Spring之后结束运行。

程序入口Chapter1Application：
```
@SpringBootApplication
public class TestApplication {
	public static void main(String[] args) {
		SpringApplication.run(TestApplication.class, args);
	}
}
```
在新的包com.hiki.web下编写控制器
```
@RestController
public class HelloController {
	@RequestMapping("/")
	public String index() {
		return "Hello Hiki";
	}
}
```
`@SpringBootApplication`是Spring Boot的核心注解，开启自动配置	

运行入口程序，在浏览器输入`http://localhost:8080`可访问

在src/main/resource源文件夹中创建application.properties(properties类型是新建file然后重命名)
`server.port=9090` 这里将tomcat端口改为了9090

<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.08  
> 更新日期：2019.05.08
