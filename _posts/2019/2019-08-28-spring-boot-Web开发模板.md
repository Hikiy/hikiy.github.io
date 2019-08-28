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

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.8.28  
> 更新日期：2019.8.28
