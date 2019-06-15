---
layout: post
title:  "Spring Boot学习记录：Tomcat配置"
date:   2019-06-15 16:25:00 +0200
categories: SpringBoot
excerpt: 
tagg: Spring
---

# Tomcat配置

## application.properties配置Tomcat
Tomcat的属性在`org.springframework.boot.autoconfigure.web.ServerProperties`配置类中定义。一般在`application.properties`中配置即可。
- `server`前缀：配置Servlet容器，常用：

```
server.port=9090
server.session-timeout = # 用户会话session过期时间，以秒为单位
server.comtext-path = # 配置访问路径，默认为/
```

- `server.tomcat`前缀：配置Tomcat特有属性，常用：

```
 server.tomcat.uri-encoding = # 配置Tomcat编码 默认为UTF-8
server.tomcat.compression = # Tomcat是否开启压缩，默认为关闭 off
```

## 代码配置Tomcat

```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
  @Bean
    public TomcatServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint constraint = new SecurityConstraint();
                constraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                constraint.addCollection(collection);
                context.addConstraint(constraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(httpConnector());
        return tomcat;
    }
    @Bean
    public Connector httpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        //Connector监听的http的端口号
        connector.setPort(8321);
        connector.setSecure(false);
        //监听到http的端口号后转向到的https的端口号
        connector.setRedirectPort(8432);
        return connector;
    }
}
```
## 替换Tomcat
### 替换成Jetty

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- 将tomcat更换成jetty，前提是上面先排除掉tomcat -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
```

### 替换成Undertow

和替换成Jetty一样，只是换一个依赖包

## SSL配置
### 生成证书
证书有自签名的也有SSL证书授权中心获得的。

JDK或JRE中有个工具叫keytool，是一个证书管理工具，可以用来生成自签名的证书：
- 配置好JAVA_HOME和其bin目录到环境变量

> 新建变量->变量名：JAVA_HOME 变量值：jdk根目录  
> 编辑Path变量-> 添加变量：%JAVA_HOME%\bin

- 在控制台调用命令:

```
keytool -genkey -alias tomcat
```

- 根据提示填好内容后会在执行该命令的目录生成一个.keystore文件

### 配置到Spring Boot
- 将.keystore放到项目根目录
- 在application.properties中配置：

```
server.ssl.key-store = .keystore
server.ssl.key-store-password = 123456
server.ssl.keyStoreType = JKS
server.ssl.ketyAlias:tomcat
```

- 然后就可以通过https访问了

### http自动转https

只要配置前面关于 **代码配置Tomcat** 的方法就行了


<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.06.15  
> 更新日期：2019.06.15
