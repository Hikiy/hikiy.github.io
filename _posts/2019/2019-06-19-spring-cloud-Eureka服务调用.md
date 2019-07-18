---
layout: post
title:  "Spring Cloud学习记录：Eureka 服务调用"
date:   2019-06-19 16:40:00 +0200
categories: SpringCloud
excerpt: 
tagg: Spring
---

# Eureka 服务调用
##  使用Feign实现

声明式REST客户端(伪RPC)。本身还是http远程调用

调用方快速上手：

1. 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-openfeign</artifactId>
</dependency>
```
2. 启动类添加注解
```
@EnableFeignClients
```
3. 创建接口
```
@FeignClient(name  = "servername")
public interface serverClient {
    @GetMapping("/add")
    String add();
}
```
其中`@FeignClient(name  = "servername")`中的servername是微服务中其中一个应用的名称。`@GetMapping`映射的已经是servername应用中的接口了

4.在controller中调用接口的方法即可

```
    @Autowired
    private ServerClient serverClient;
    
    @GetMapping("/add")
    public String add(){
        return serverClient.add();
    }
```

### 服务提供方

#### 主要依赖

```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```
#### 配置文件

因为使用了两个注册中心，所以配置里有两个注册中心的url，注意`server.port`：

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
spring:
  application:
    name: client
server:
  port: 9000
```

#### 启动类
```
@SpringBootApplication
@EnableDiscoveryClient
public class ClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }

}
```

#### Controller层
```
/**
 * @author ：Hiki
 * 2019/6/19 15:17
 */
@RestController
public class HikiController {
    @RequestMapping("/hello")
    public String hello(@RequestParam String name){
        return "Hello " + name + ",I'm Hiki!";
    }
}

```

### 服务调用方

#### 主要依赖
```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

#### 配置文件
注意端口跟提供方不一致。

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
spring:
  application:
    name: consumer
server:
  port: 9001
```

#### 启动类

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

- `@EnableFeignClients`：启用feign进行远程调用

> Feign是一个声明式Web Service客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

#### 实现feign

```
/**
 * @author ：Hiki
 * 2019/6/19 15:31
 */
@FeignClient(name= "client")
public interface HelloFeign {
    @RequestMapping("/hello")
    public String hello(@RequestParam String name);
}
```

- 注意`@FeignClient`注解的name参数与服务提供方的名称要一致

#### Controller层通过feign远程调用

```
/**
 * @author ：Hiki
 * 2019/6/19 15:37
 */
@RestController
public class HelloController {
    @Autowired
    HelloFeign hf;
    @RequestMapping("/hello")
    public String hello(@RequestParam String name){
        return hf.hello(name);
    }
}
```

### 测试

- 启动注册中心、服务提供方、服务调用方
- 先在注册中心检查服务是否注册成功
- 先访问http://localhost:9000/hello?name=Tom测试服务提供方是否成功
- 访问http://localhost:9001/hello?name=Sam测试服务调用方是否成功

如果都成功，那恭喜啦

### 服务提供的负载均衡

- 如果多新建一个服务提供方，名称与原本的服务提供方一样
- 修改一下controller返回的内容，但保持参数和request url不变
- 则每次访问http://localhost:9001/hello?name=Sam时，将会发现返回的内容不断交替变化。

这是因为服务中心提供的负载均衡功能

---------------↓2019.07.18↓---------------

## 使用RestTemplate实现
```
RestTemplate restTemplate = new RestTemplate();
        String response = restTemplate.getForObject("http://localhost:8989/msg",String.class);
```
### 缺点
URL写死了，如果不知道ip也无法访问。

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.19  
> 更新日期：2019.07.18