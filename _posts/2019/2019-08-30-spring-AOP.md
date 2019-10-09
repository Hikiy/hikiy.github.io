---
layout: post
title:  "Spring 学习记录：AOP"
date:   2019-08-30 11:27:00 +0200
categories: Spring
excerpt: 
tagg: Spring
---

# AOP:bread:

面向切面编程:bread:

</br>
</br>
</br>

## 依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

</br>
</br>
</br>

## 实现切面类
两步实现切面类：
- @Component注解
- @Aspect注解

```
@Aspect
@Component
public class WebAspect {
}
```

</br>
</br>
</br>

## 定义切入点
切入点不是必须要先定义的，先定义应该是为了方便维护

```
    @Pointcut("execution(public * com.example.aop..*.*(..))")
    public void webLog(){}
```

### 使用方法
```
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        ...
    }
```

</br>
</br>
</br>

## 支持的通知

每个通知实现的位置:
```
try{
	try{
		//@Around
		//@Before
		method.invoke(..);
		//@Around
	}catch(){
		throw.....;
	}finally{
		//@After
	}
	//@AfterReturning
}catch(){
	//@AfterThrowing
}
```

### 1.前置通知@Before

```
@Before(value = "execution(public * com.example.aop..*.*(..))")
public void before(JoinPoint joinPoint){

}
```

### 2.后置通知@After

```
/** 
 * 后置最终通知（目标方法只要执行完了就会执行后置通知方法） 
 * @param joinPoint 
 */  
@After(value = "execution(public * com.example.aop..*.*(..))")  
public void after(JoinPoint joinPoint){ 
    logger.info("after");  
}  
```

### 3.后置通知@AfterReturning
在某连接点之后执行的通知，通常在一个匹配的方法返回的时候执行（可以在后置通知中绑定返回值）。

```
/** 
 * 后置返回通知 
 * 这里需要注意的是: 
 *      如果参数中的第一个参数为JoinPoint，则第二个参数为返回值的信息 
 *      如果参数中的第一个参数不为JoinPoint，则第一个参数为returning中对应的参数 
 *       returning：限定了只有目标方法返回值与通知方法相应参数类型时才能执行后置返回通知，否则不执行，
 *       对于returning对应的通知方法参数为Object类型将匹配任何目标返回值 
 * @param joinPoint 
 * @param keys 
 */  
@AfterReturning(value = POINT_CUT,returning = "keys")  
public void afterReturning1(JoinPoint joinPoint,Object keys){  
    logger.info("1："+keys);  
}  

@AfterReturning(value = POINT_CUT,returning = "keys",argNames = "keys")  
public void afterReturning2(String keys){  
    logger.info("2："+keys);  
}
```

### 4.后置异常通知

```
@AfterThrowing(value = POINT_CUT,throwing = "exception")  
public void doAfterThrowingAdvice(JoinPoint joinPoint,Throwable exception){  
    //目标方法名：  
    logger.info(joinPoint.getSignature().getName());  
    if(exception instanceof NullPointerException){  
        logger.info("发生了空指针异常!!!!!");  
    }  
} 
```

### 5.环绕通知@Around
环绕通知使用一个代理ProceedingJoinPoint类型的对象来管理目标对象，所以此通知的第一个参数必须是ProceedingJoinPoint类型。在通知体内调用ProceedingJoinPoint的proceed()方法会导致后台的连接点方法执行。proceed()方法也可能会被调用并且传入一个Object[]对象，该数组中的值将被作为方法执行时的入参。

```
@Around(value = POINT_CUT)  
public Object doAroundAdvice(ProceedingJoinPoint proceedingJoinPoint){  
    logger.info("环绕通知的目标方法名："+proceedingJoinPoint.getSignature().getName());  
    try {  
        Object obj = proceedingJoinPoint.proceed();  
        return obj;  
    } catch (Throwable throwable) {  
        throwable.printStackTrace();  
    }  
    return null;  
} 
```

</br>
</br>
</br>

## 切入点表达式

- `*`：匹配所有字符 
- `..`：一般用于匹配多个包，多个参数 
- `+`：表示类及其子类 
- 运算符有：`&&`, `||`, `!`

### 关键词
1.execution：用于匹配子表达式。 主要使用这个。

```
//匹配com.cjm.model包及其子包中所有类中的所有方法，返回类型任意，方法参数任意
@Pointcut(“execution(* com.cjm.model...(..))”) 
public void before(){}
```

2.within：用于匹配连接点所在的Java类或者包。 

```
//匹配Person类中的所有方法 
@Pointcut(“within(com.cjm.model.Person)”) 
public void before(){} 
//匹配com.cjm包及其子包中所有类中的所有方法 
@Pointcut(“within(com.cjm..*)”) 
public void before(){}
```

3. this：用于向通知方法中传入代理对象的引用。 

```
@Before(“before() && this(proxy)”) 
public void beforeAdvide(JoinPoint point, Object proxy){ 
//处理逻辑 
}
```

4.target：用于向通知方法中传入目标对象的引用。 

```
@Before(“before() && target(target) 
public void beforeAdvide(JoinPoint point, Object proxy){ 
//处理逻辑 
}
```

5.args：用于将参数传入到通知方法中。 

```
@Before(“before() && args(age,username)”) 
public void beforeAdvide(JoinPoint point, int age, String username){ 
//处理逻辑 
}
```

6.@within ：用于匹配在类一级使用了参数确定的注解的类，其所有方法都将被匹配。 

```
@Pointcut(“@within(com.cjm.annotation.AdviceAnnotation)”) 
－ 所有被@AdviceAnnotation标注的类都将匹配 
public void before(){}
```

7.@target ：和@within的功能类似，但必须要指定注解接口的保留策略为RUNTIME。 

```
@Pointcut(“@target(com.cjm.annotation.AdviceAnnotation)”) 
public void before(){}
```

8.@args ：传入连接点的对象对应的Java类必须被@args指定的Annotation注解标注。 

```
@Before(“@args(com.cjm.annotation.AdviceAnnotation)”) 
public void beforeAdvide(JoinPoint point){ 
//处理逻辑 
}
```

9.@annotation ：匹配连接点被它参数指定的Annotation注解的方法。也就是说，所有被指定注解标注的方法都将匹配。 

```
@Pointcut(“@annotation(com.cjm.annotation.AdviceAnnotation)”) 
public void before(){}
```

10.bean：通过受管Bean的名字来限定连接点所在的Bean。该关键词是Spring2.5新增的。

```
@Pointcut(“bean(person)”) 
public void before(){}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.8.30  
> 更新日期：2019.8.30
