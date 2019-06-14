---
layout: post
title:  "Spring Boot学习记录：使用RabbitMQ"
date:   2019-05-17 11:17:00 +0200
categories: SpringBoot
excerpt: 在Spring Boot中使用RabbitMQ
tagg: Spring
---

# Spring Boot 中使用RabbitMQ

### 消息队列中间件
之前一直不知道消息队列是啥，这里举例子解释：  
生产者生产苹果，消费者消费苹果。有几种情况：
>1. 生产者生产1个苹果，消费者消费1个苹果。如果消费者消费苹果时噎住（系统宕机），生产者还在生产，则生产的苹果就丢失了。  
>2. 生产者1秒生产10个苹果，消费者1秒消费5个苹果。消费者会吃不消（消息堵塞，导致系统超时），最后苹果丢失  

如果我们在生产者和消费者之间放一个篮子，生产者将苹果放进篮子，消费者从篮子拿苹果，则苹果就不会丢失了。消息队列中间件即是这个篮子。一下明朗了。

### RabbitMQ介绍

[RabbitMQ介绍](https://github.com/Hikiy/Notes/blob/master/%E6%A1%86%E6%9E%B6/Spring%20Boot/RabbitMQ.md)

RabbitMQ是要先安装的，直接百度就好了

### Maven
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### application.properties
```
spring.application.name=Spring-boot-rabbitmq

spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456
```

### 队列配置
```
@Configuration
public class RabbitMQConfig {

    @Bean
    public Queue Queue(){
        //org.springframework.amqp.core.Queue;
        return new Queue("hello");
    }
}
```

### Sender
```
@Component
public class TestSender {
    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send(String message){
        String context = message + "  " + new Date();
        System.out.println("Sender:" + context);
        this.rabbitTemplate.convertAndSend("hello",context);
    }

}
```

### Receiver
```
@Component
@RabbitListener(queues = "hello")
public class TestReceiver {
    @RabbitHandler
    public void handler(String message){
        System.out.println("Receiver:  You say :" + message);
    }
}
```

### Test
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestRabbitMQ {
    @Autowired
    private TestSender ts;

    @Test
    public void hello() throws Exception{
        ts.send("Hello,Hiki");
    }
}
```
**注意Sender和Receiver用的queue要一样**
### 一对多
多加两个Receiver进行测试，结果为轮流分配给Receiver
```
    @Test
    //一对多测试,结果为轮流分配给Receiver
    public void one2many()throws Exception{
        for (int i=0;i<20;i++){
            ts2.send(i + ": Hello,Hiki");
        }
    }
```

### 多对多
多加一个Sender并发送到同一个queue，结果也为轮流分配给Receiver，也就是怎么操作都是一个队列，sender发一个，receiver也只能拿一个。
```
    @Test
    //多对多测试，结果也为轮流分配给Receiver，也就是怎么操作都是一个队列，sender发一个，receiver也只能拿一个
    public void many2many() throws  Exception{
        for (int i=0;i<20;i++){
            ts2.send(i + ": Hello,Hiki");
            ts3.send(i + ": Hello,Hiki2");
        }
    }
```

### 传对象
其实就是把要传的类型换成对象就行了。

**一个Recevier只能监听一个queue（还未核实）**

### Topic Exchange
Topic 是RabbitMQ中最灵活的方式。  

**配置**
```
    //Topic Exchange

    @Bean
    public Queue queueMessage(){
        return new Queue("topic.message");
    }
    @Bean
    //要记住，Spring的Bean是怎么生成的，所以名字也要起好，以便容器拿的时候能识别出来，
    // 这里的名字跟后面的绑定就是要用同一个名字，不然绑定无法Autowired
    //接收所有topic消息
    public Queue queueAll(){
        return new Queue("topic.all");
    }
    @Bean
    //按我的理解，这个exchange就像一个规则，sender使用这个规则，就会去跟这个规则绑定的key进行匹配
    //根据sender给的key，在绑定中寻找匹配的。
    public TopicExchange exchange(){
        return new TopicExchange("exchange");
    }
    //下面这两个Bean，第一个只匹配topickey.message,并发到topic.message。
    //第二个只要以topickey开头都匹配，并发到topic.all。
    //也就是所有都发到topic.all，而topic.message只接收topickey.message
    @Bean
    public Binding bindingExchangeMessage(Queue queueMessage,TopicExchange exchange){
        return BindingBuilder.bind(queueMessage).to(exchange).with("topickey.message");
    }
    @Bean
    public Binding bindingExchangeAll(Queue queueAll,TopicExchange exchange){
        return BindingBuilder.bind(queueAll).to(exchange).with("topickey.#");
    }
```
注意其中的`return BindingBuilder.bind(queueMessage).to(exchange).with("topickey.message"); `这里的 `"topickey.message"`就是`routing_key`故意写的不一样为了区分队列的标识。

**Sender**
```
@Component
public class TopicSender {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 发送给单信息
     * @param message
     */
    public void send1(String message){
        String context = "TopicSender: " + message;
        System.out.println(context);
        rabbitTemplate.convertAndSend("exchange","topickey.message",message);
    }

    public void send2(String message){
        String context =  "TopicSenders: " + message;
        System.out.println(context);
        rabbitTemplate.convertAndSend("exchange","topickey.all",message);
    }
}
```

**Receiver**
```
@Component
@RabbitListener(queues = "topic.message" )
public class TopicReceiver {
    @RabbitHandler
    public void receive(String message){
        System.out.println("topic.message: "+message);
    }
}

@Component
@RabbitListener(queues = "topic.all")
public class TopicReceivers {
    @RabbitHandler
    public void receiver(String message){
        System.out.println("topic.all: " + message);
    }
}
```

**测试**
```
    @Test
    //测试Topic Exchange
    public void testTopic() throws Exception{
        topics.send1("发给message的信息，但是all也能拿到，所以两个Receiver都拿到了");
        topics.send2("发给all的，所以只有一个Receiver拿到了");
    }
```

### Fanout Exchange
广播

**配置**
```
    //Fanout Exchange

    @Bean
    public Queue Message1() {
        return new Queue("fanout.1");
    }
    @Bean
    public Queue Message2() {
        return new Queue("fanout.2");
    }
    @Bean
    public Queue Message3() {
        return new Queue("fanout.3");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }
    @Bean
    Binding bindingExchangeA(Queue Message1,FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(Message1).to(fanoutExchange);
    }
    @Bean
    Binding bindingExchangeB(Queue Message2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(Message2).to(fanoutExchange);
    }
    @Bean
    Binding bindingExchangeC(Queue Message3, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(Message3).to(fanoutExchange);
    }
```
**Sender**
```
@Component
public class FanoutSender {
    @Autowired
    private RabbitTemplate rt;

    public void send(String message){
        System.out.println("发送广播啦：" + message);
        rt.convertAndSend("fanoutExchange","",message);
    }
}
```
**Receiver**

建立三个Receiver，但是他们几乎一样，只贴其中一个：
```
@Component
@RabbitListener(queues = "fanout.1")
public class FanoutReceiver1 {
    @RabbitHandler
    public void receiver(String message){
        System.out.println("FanoutReceiver1收到广播： " + message);
    }
}
```
**测试**
```
    @Test
    //测试Fanout Exchange
    public void testFanout() throws Exception{
        fs.send("hiki来啦");
    }
```



<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.17  
> 更新日期：2019.05.23
