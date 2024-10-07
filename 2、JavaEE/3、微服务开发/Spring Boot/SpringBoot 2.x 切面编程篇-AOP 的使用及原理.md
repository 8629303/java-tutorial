> 1. Spring AOP：原理、 通知、连接点、切点、切面、表达式：https://glory.blog.csdn.net/article/details/84025425
> 2. Spring Aop 常见注解和执行顺序：https://mp.weixin.qq.com/s/CqzfkaiP3GgQ9b93ugYXcw
> 3. 图文详解 Spring AOP，看完必懂！https://mp.weixin.qq.com/s/0QooD_oCUVxDdW_wNb9uFg



# SpringBoot AOP 切面的使用

> 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：SpringBoot AOP切面：https://blog.csdn.net/m0_58680865/article/details/128461227

## 一、什么是切面

AOP（Aspect OrientedProgramming）：面向切面编程，也是Spring框架中的一个重要内容。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。



## 二、切面的用途

日志记录，性能统计，安全控制，权限管理，事务处理，异常处理，资源池管理。



## 三、AOP 切面常用注解

AOP 切面常用注解如下：

- @Aspect：用来描述这是一个切面类，作用于类上
- @Pointcut：定义一个切入点，所关注的某事件入口
- @Before：在之前做什么
- @After：在之后做什么
-  @Around：环绕增强，能够在方法执行的前后进行增强逻辑，并且可以完全控制目标方法是否执行
- @AfterReturn：在之后对其数据做增强处理
- @AfterThrowing：抛出异常处理
- @Component：表示交给Spring来管理
- @Order：指定切面执行的顺序。值越小优先级越高



### 1、AOP 注解详解

- @Aspect：切面，通常是一个类的注解，里面可以定义切入点和通知。

- @Pointcut：切入点，书写切入点表达式，指明Advice要在什么样的条件下才能被触发。

  由下列方式来定义或者通过 &&、 ||、 !、 的方式进行组合：

  - execution：用于匹配方法执行的连接点；
  - within：用于匹配指定类型内的方法执行；
  - this：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；
  - target：用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；
  - args：用于匹配当前执行的方法传入的参数为指定类型的执行方法；
  - @within：用于匹配所以持有指定注解类型内的方法；
  - @target：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；
  - @args：用于匹配当前执行的方法传入的参数持有指定注解的执行；
  - @annotation：用于匹配当前执行方法持有指定注解的方法；

- Advice：通知，某个连接点所采用的处理逻辑，也就是向连接点注入的代码， AOP在特定的切入点上执行的增强处理。

  - @Before：标识前置增强方法，相当于BeforeAdvice；
  - @Around：环绕增强，相当于MethodInterceptor；
  - @After：final增强，抛出异常和正常退出后都会执行；
  - @AfterReturning：后置增强，正常退出时执行，相当于AfterReturningAdvice；
  - @AfterThrowing：后置增强，抛出异常时执行，相当于ThrowsAdvice。

- JointPoint：连接点，程序运行中的某个阶段点，比如方法的调用、异常的抛出等。

  - Object[] getArgs：返回目标方法的参数；
  - Signature getSignature：返回目标方法的签名；
  - Object getTarget：返回被织入增强处理的目标对象；
  - Object getThis：返回AOP框架为目标对象生成的代理对象。

- Advisor：增强， 是PointCut和Advice的综合体，完整描述了一个advice将会在pointcut所定义的位置被触发。

  ```xml
  <aop:aspectj-autoproxy/>
  <aop:config proxy-target-class="true">
      <aop:pointcut id="servicePointcut"
                    expression="execution(* com.cpic..*Service.*(..))" />
      <aop:advisor pointcut-ref="servicePointcut" advice-ref="txAdvice"
                   order="3" />
  </aop:config>
  <tx:advice id="txAdvice" transaction-manager="transactionManager">
      <tx:attributes>
          <tx:method name="list*" read-only="true" />
          <!-- log方法会启动一个新事务 -->
          <tx:method name="log*" propagation="REQUIRES_NEW"
                     isolation="READ_COMMITTED" />
      </tx:attributes>
  </tx:advice>
  <!-- OK所以一个Spring增强（advisor）= 切面(advice) + 切入点(PointCut) -->
  ```



### 3、自定义注解回顾

@Target：描述了注解修饰的对象范围，取值在java.lang.annotation.ElementType定义，常用的包括：

- METHOD：用于描述方法
- PACKAGE：用于描述包
- PARAMETER：用于描述方法变量
- TYPE：用于描述类、接口或enum类型


@Retention: 表示注解保留时间长短。取值在java.lang.annotation.RetentionPolicy中，取值为：

- SOURCE：在源文件中有效，编译过程中会被忽略
- CLASS：随源文件一起编译在class文件中，运行时忽略
- RUNTIME：在运行时有效

> 注意：
>
> - 只有定义为`RetentionPolicy.RUNTIME`时，我们才能通过注解反射获取到注解。
> - 所以，假设我们要自定义一个注解，它用在字段上，并且可以通过反射获取到，功能是用来描述字段的长度和作用。

**@Documented**：表示这个注解是由 javadoc记录的，在默认情况下也有类似的记录工具。 如果一个类型声明被注解了文档化，它的注解成为公共API的一部分。

操作示例：定义注解 + 通过反射获取注解

```java
@Target(ElementType.FIELD)           // 注解用于字段上
@Retention(RetentionPolicy.RUNTIME)  // 保留到运行时，可通过注解获取
public @interface MyField {
    String description();
    int length();
}

public class MyFieldTest {
    // 使用我们的自定义注解
    @MyField(description = "用户名", length = 12)
    private String username;

    @Test
    public void testMyField(){

        // 获取类模板
        Class c = MyFieldTest.class;

        // 获取所有字段
        for(Field f : c.getDeclaredFields()){
            // 判断这个字段是否有MyField注解
            if(f.isAnnotationPresent(MyField.class)){
                MyField annotation = f.getAnnotation(MyField.class);
                System.out.println(
                    "字段:[" + f.getName() + "], 描述:[" 
                    + annotation.description() + "], 长度:[" 
                    + annotation.length() +"]");
            }
        }
    }
}
```



## 四、切面详细内容

切面【Aspect】

- 连接点【Joinpoint】
- 通知【Advice】
  - 前置通知【Before advice】
  - 后置通知【After advice】
  - 返回通知【After return advice】
  - 环绕通知【Around advice】
  - 抛出异常后通知【After throwing advice】
- 切入点【Pointcut】

**详细内容：**

1、切面（Aspect）

2、连接点（Joinpoint）：程序执行过程中的某一行为动作，例如，某一个方法的调用或者抛出异常等行为。

3、通知（Advice）：“切面”对于某个“连接点”所产生的动作。比如，切面类对项目中service包下的所有类的方法进行日志记录的动作就是一个Advice。一个切面可以包含多个Advice：

1. 前置通知：在某连接点（JoinPoint）之前执行的通知，但这个通知不能阻止连接点前的执行。注解中使用@Before声明。

2. 后通知：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。注解中使用@After声明。

3. 返回后通知：在某连接点正常完成后执行的通知，不包括抛出异常的情况。注解中使用@AfterReturning声明。

4. 环绕通知：包围一个连接点的通知，可以在方法的调用前后完成自定义的行为，也可以选择不执行。注解中使用@Around声明。

5. 抛出异常后通知： 在方法抛出异常退出时执行的通知。注解中使用@AfterThrowing声明。

   **通知执行顺序**：

   前置通知 → 环绕通知连接点之前 → 连接点执行 → 环绕通知连接点之后 → 返回通知 → 后通知 → (如果发生异常)异常通知 → 后通知。

4、切入点（Pointcut）：匹配连接点的断言，在 AOP 中通知和一个切入点表达式关联。



## 五、切面代码演示

Maven 依赖

```xml
<!-- 切面依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 1、定义一个切面类

```java
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class AopAspect {

}
```



### 2、@Pointcut 的使用

**@Pointcut 使用有两种**，在需要切面的方法上加入 @Pointcut 注解

1. 方式一：execution()

   ```java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Pointcut;
   import org.springframework.stereotype.Component;
   
   @Aspect
   @Component
   public class AopAspect {
   
       /**
        *  @Pointcut：定义一个切面，所关注的某件事入口
        *  execution()：表达式主体
        *  第一个 *：表示返回值类型， *表示所有类型
        *  com.example.service：包名
        *  ..：表示当前包和当前包子包
        *  第二个*：表示类名，*表示所有类
        *  *（..）：*表示方法名，括弧里边表示参数，..表示任何参数
        */
       @Pointcut("execution(* com.example.service..*.*(..))")
       public void pointCut(){
           // 业务逻辑
       }
   }
   ```

2. 方式二： @annotation()

   ```java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Pointcut;
   import org.springframework.stereotype.Component;
   
   @Aspect
   @Component
   public class AopAspect {
   
       /**
        * @annotation()：针对否个注解来定义切面，如下根据@GetMapping进行切面 如：@GetMapping、@PostMapping
        */
       @Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
       public void pointCut() {
           // 业务逻辑
       }
   }
   ```



### 3、通知中获取被调方法信息

通知中如果想获取被调用方法的信息，分2种情况

1. 非环绕通知，可以将`org.aspectj.lang.JoinPoint`作为通知方法的第1个参数，通过这个参数获取被调用方法的信息
2. 如果是环绕通知，可以将`org.aspectj.lang.ProceedingJoinPoint`作为方法的第1个参数，通过这个参数获取被调用方法的信息



#### 1）JoinPoint：连接点信息

```java
org.aspectj.lang.JoinPoint
```

提供访问当前被通知方法的目标对象、代理对象、方法参数等数据：

```java
package org.aspectj.lang;  
import org.aspectj.lang.reflect.SourceLocation;
public interface JoinPoint {
    // 连接点所在位置的相关信息
    String toString();
    // 连接点所在位置的简短相关信息
    String toShortString();
    // 连接点所在位置的全部相关信息
    String toLongString();
    // 返回AOP代理对象
    Object getThis();
    // 返回目标对象
    Object getTarget();
    // 返回被通知方法参数列表，也就是目前调用目标方法传入的参数
    Object[] getArgs();
    // 返回当前连接点签名，这个可以用来获取目标方法的详细信息，如方法Method对象等
    Signature getSignature();
    // 返回连接点方法所在类文件中的位置
    SourceLocation getSourceLocation();
    // 连接点类型
    String getKind();
    // 返回连接点静态部分
    StaticPart getStaticPart();
}
```



#### 2）ProceedingJoinPoint：环绕通知连接点信息

用于环绕通知，内部主要关注2个方法，一个有参的，一个无参的，用来继续执行拦截器链上的下一个通知。

```java
package org.aspectj.lang;
import org.aspectj.runtime.internal.AroundClosure;
public interface ProceedingJoinPoint extends JoinPoint {
    /**
     * 继续执行下一个通知或者目标方法的调用
     */
    public Object proceed() throws Throwable;
    /**
     * 继续执行下一个通知或者目标方法的调用
     */
    public Object proceed(Object[] args) throws Throwable;
}
```



#### 3）Signature：连接点签名信息

注意`JoinPoint#getSignature()`这个方法，用来获取连接点的签名信息，这个比较重要

```java
Signature getSignature();
```

通常情况，Spring 中的 AOP 都是用来对方法进行拦截，所以通常情况下连接点都是一个具体的方法，`Signature`有个子接口

```java
org.aspectj.lang.reflect.MethodSignature
```

`JoinPoint#getSignature()`都可以转换转换为`MethodSignature`类型，然后可以通过这个接口提供的一些方法来获取被调用的方法的详细信息。

操作示例：这里演示@Before获取被调用方法的详细信息，

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import java.lang.reflect.Method;

@Aspect
public class BeforeAspectTest {
    @Pointcut("execution(* com.example.aop.demo10.test1.Service1.*(..))")
    public void pc() {
    }
    @Before("com.javacode2018.aop.demo10.test2.BeforeAspect2.pc()")
    public void before(JoinPoint joinPoint) {
        // 获取连接点签名
        Signature signature = joinPoint.getSignature();
        // 将其转换为方法签名
        MethodSignature methodSignature = (MethodSignature) signature;
        // 通过方法签名获取被调用的目标方法
        Method method = methodSignature.getMethod();
        // 输出方法信息
        System.out.println(method);
    }
}
```



### 4、指定方法前处理 @Before

1. 任意方法上使用`@Before`标注，将这个方法作为前置通知，目标方法被调用之前，会自动回调这个方法
2. 被`@Before`标注的方法参数可以为空，或者为`JoinPoint`类型，当为`JoinPoint`类型时，必须为第一个参数
3. 被`@Before`标注的方法名称可以随意命名，符合java规范就可以，其他通知也类似

```java
/**
 * @Before：在之前做什么，指定的方法在切面切入目标方法之前执行
 * pointcut()：定义切面入口方法
 * JoinPoint:当为JoinPoint类型时，必须为第一个参数
 */
@Before("pointCut()")
public void before(JoinPoint joinPoint) {
    // 业务逻辑
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    log.info("{}, 准备执行!", methodSignature.getMethod());
}
```

@Before通知最后会被解析为下面这个通知类：`org.springframework.aop.aspectj.AspectJMethodBeforeAdvice`



### 5、指定方法之后处理 @After

后置通知，在方法执行之后执行，用法和前置通知类似。特点如下：

- **不管目标方法是否有异常，后置通知都会执行**
- 这种通知无法获取方法返回值
- 可以使用`JoinPoint`作为方法的第一个参数，用来获取连接点的信息

```java
/**
 * @After：指定的方法在切面切入目标方法之后执行、和@Before注解对应
 * pointcut()：定义切面入口方法
 * JoinPoint:当为JoinPoint类型时，必须为第一个参数
 */
@After("pointCut()")
public void after(JoinPoint joinPoint) {
    // 业务逻辑
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    log.info("{}, 执行完毕!", methodSignature.getMethod());
}
```

@After通知最后会被解析为下面这个通知类：`org.springframework.aop.aspectj.AspectJAfterAdvice`

这个类中有`invoke`方法，这个方法内部会调用被通知的方法，其内部采用`try..finally`的方式实现的，所以不管目标方法是否有异常，通知一定会被执行。

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        // 继续执行下一个拦截器
        return mi.proceed();
    }
    finally {
        // 内部通过反射调用被@After标注的方法
        invokeAdviceMethod(getJoinPointMatch(), null, null);
    }
}
```



### 6、数据增强处理 @AfterReturning

返回通知，在方法返回结果之后执行。**特点如下**：

- 可以获取到方法的返回值
- 当目标方法返回异常的时候，这个通知不会被调用，这点和@After通知是有区别的

```java
/**
 * @AfterReturning ：与 @After 类似，
 * 区别在于 @AfterReturning 注解可以用来捕获切入方法执行完之后的返回值，对返回值进行业务逻辑上的增强处理
 * pointcut：切面方法名
 * JoinPoint:当为JoinPoint类型时，必须为第一个参数
 * returning:被切方法的返回值，必须要和参数保持一致，否则会检测不到
 */
@AfterReturning(pointcut = "pointCut()", returning = "result")
public void afterReturning(JoinPoint joinPoint, Object result) {
    // 业务逻辑
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    log.info("{} 返回值：{}", methodSignature.getMethod(), result);
}
```

@AfterReturning通知最后会被解析为下面这个通知类：`org.springframework.aop.aspectj.AspectJAfterReturningAdvice`



### 7、数据异常处理 @AfterThrowing

@AfterThrowing 标注的方法可以指定异常的类型，当被调用的方法触发该异常及其子类型的异常之后，会触发异常方法的回调。也可以不指定异常类型，此时会匹配所有异常。

1. 未指定异常类型，可以匹配所有异常类型，如下：

   ```java
   @AfterThrowing(pointcut = "切入点")
   public void afterThrowing() {
   }
   ```

2. 指定异常类型：通过`@AfterThrowing`的`throwing`指定参数异常参数名称，我们用方法的第二个参数用来接收异常，第二个参数名称为e，下面的代码，当目标方法发生`IllegalArgumentException`异常及其子类型异常时，下面的方法会被回调。

   ```java
   @AfterThrowing(pointcut = "com.javacode2018.AfterThrowingAspect6.pc()", throwing = "e")
   public void afterThrowing(JoinPoint joinPoint, IllegalArgumentException e) {
   }
   ```

**特点**：不论异常是否被异常通知捕获，异常还会继续向外抛出。

```java
/**
 * @AfterThrowing：被切方法执行时抛出异常时会进入
 * pointcut：切面方法名
 * JoinPoint:当为JoinPoint类型时，必须为第一个参数
 * throwing：属性的值必须要和参数一致
 */
@AfterThrowing(pointcut = "pointCut()", throwing = "e")
public void afterThrowing(JoinPoint joinPoint, Throwable e) {
    // 业务处理
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    log.info("{} 发生异常,异常信息：{}", methodSignature.getMethod(), e.getMessage());
}
```

@AfterThrowing通知最后会被解析为下面这个通知类：`org.springframework.aop.aspectj.AspectJAfterThrowingAdvice`

来看一下这个类的`invoke`方法，这个方法是关键

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        // 继续调用下一个拦截器链
        return mi.proceed();
    }
    catch (Throwable ex) {
        // 判断ex和需要不糊的异常是否匹配
        if (shouldInvokeOnThrowing(ex)) {
            // 通过反射调用@AfterThrowing标注的方法
            invokeAdviceMethod(getJoinPointMatch(), null, ex);
        }
        // 继续向外抛出异常
        throw ex;
    }
}
```



### 8、指定方法环绕处理 @Around

环绕通知会包裹目标目标方法的执行，可以在通知内部调用`ProceedingJoinPoint.process`方法继续执行下一个拦截器。

用起来和其他几个类似，但是有2点不一样

1. 若需要获取目标方法的信息，需要将ProceedingJoinPoint作为第一个参数
2. 通常使用Object类型作为方法的返回值，返回值也可以为void

特点：环绕通知比较特殊，其他4种类型的通知都可以用环绕通知来实现。

```java
/**
 * @Around：环绕增强，能够在方法执行的前后进行增强逻辑，并且可以完全控制目标方法是否执行
 * pointcut()：定义切面入口方法
 * JoinPoint:当为JoinPoint类型时，必须为第一个参数
 */
@Around("pointCut()")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    // 获取连接点签名,并且将其转换为方法签名
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    // 通过方法签名获取被调用的目标方法
    Method method = methodSignature.getMethod();
    long startTime = System.nanoTime();
    // 调用proceed方法，继续调用下一个通知
    Object returnVal = joinPoint.proceed();
    long endTime = System.nanoTime();
    long costTime = endTime - startTime;
    // 输出方法信息
    log.info("{}, 耗时(纳秒): {}", method.toString(), costTime);
    // 返回方法的返回值
    return returnVal;
}
```

@Around通知最后会被解析为下面这个通知类：`org.springframework.aop.aspectj.AspectJAroundAdvice`



### 9、Aspect 切面类完整示例

```java
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

@Slf4j
@Aspect
@Component
public class AopAspect {

    /**
     * @annotation()：针对某个注解来定义切面，如下根据 @GetMapping 进行切面
     */
    @Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
    public void pointCut() {
        log.info("@Pointcut：定义一个切面，所关注的某件事入口");
    }

    /**
     * @Before：在之前做什么，指定的方法在切面切入目标方法之前执行
     * pointcut()：定义切面入口方法
     * JoinPoint:当为JoinPoint类型时，必须为第一个参数
     */
    @Before("pointCut()")
    public void before(JoinPoint joinPoint) {
        // 业务逻辑
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        log.info("{}, 准备执行!", methodSignature.getMethod());
    }

    /**
     * @After：指定的方法在切面切入目标方法之后执行、和@Before注解对应
     * pointcut()：定义切面入口方法
     * JoinPoint:当为JoinPoint类型时，必须为第一个参数
     */
    @After("pointCut()")
    public void after(JoinPoint joinPoint) {
        // 业务逻辑
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        log.info("{}, 执行完毕!", methodSignature.getMethod());
    }

    /**
     * @AfterReturning ：与 @After 类似，
     * 区别在于 @AfterReturning 注解可以用来捕获切入方法执行完之后的返回值，对返回值进行业务逻辑上的增强处理
     * pointcut：切面方法名
     * JoinPoint:当为JoinPoint类型时，必须为第一个参数
     * returning:被切方法的返回值，必须要和参数保持一致，否则会检测不到
     */
    @AfterReturning(pointcut = "pointCut()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        // 业务逻辑
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        log.info("{} 返回值：{}", methodSignature.getMethod(), result);
    }

    /**
     * @AfterThrowing：被切方法执行时抛出异常时会进入
     * pointcut：切面方法名
     * JoinPoint:当为JoinPoint类型时，必须为第一个参数
     * throwing：属性的值必须要和参数一致
     */
    @AfterThrowing(pointcut = "pointCut()", throwing = "e")
    public void afterThrowing(JoinPoint joinPoint, Throwable e) {
        // 业务处理
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        log.info("{} 发生异常,异常信息：{}", methodSignature.getMethod(), e.getMessage());
    }

    /**
     * @Around：环绕增强，能够在方法执行的前后进行增强逻辑，并且可以完全控制目标方法是否执行
     * pointcut()：定义切面入口方法
     * JoinPoint:当为JoinPoint类型时，必须为第一个参数
     */
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        // 获取连接点签名,并且将其转换为方法签名
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        // 通过方法签名获取被调用的目标方法
        Method method = methodSignature.getMethod();
        long startTime = System.nanoTime();
        // 调用proceed方法，继续调用下一个通知
        Object returnVal = joinPoint.proceed();
        long endTime = System.nanoTime();
        long costTime = endTime - startTime;
        // 输出方法信息
        log.info("{}, 耗时(纳秒): {}", method.toString(), costTime);
        // 返回方法的返回值
        return returnVal;
    }
}
```



### 10、几种通知对比

| 通知类型        | 执行时间点     | 可获取返回值 | 目标方法异常时是否会执行 |
| --------------- | -------------- | ------------ | ------------------------ |
| @Before         | 方法执行之前   | 否           | 是                       |
| @Around         | 环绕方法执行   | 是           | 自己控制                 |
| @After          | 方法执行后     | 否           | 是                       |
| @AfterReturning | 方法执行后     | 是           | 否                       |
| @AfterThrowing  | 方法发生异常后 | 否           | 是                       |



## 六、切面实际业务实现—拦截功能

### 1、基于注解拦截

1、引入Maven依赖

```xml
<!-- web依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- 切面依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2、定义拦截注解 @InterceptAnnotation

```java
package com.example.annotation;
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InterceptAnnotation {

}
```

3、定义切面类及方法

```java
package com.example.intercept;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Slf4j
@Aspect
@Component
public class InterceptAnnotationAspect {

    @Pointcut("@annotation(com.example.annotation.InterceptAnnotation)")
    public void interceptPointcut() {

    }

    @Around("interceptPointcut()")
    public Object interceptAround(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("基于注解拦截的切面类");
        return joinPoint.proceed();
    }
}
```

4、定义Controller类，加上注解进行测试

```java
package com.example.controller;

import com.example.annotation.InterceptAnnotation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class InterceptAopController {

    @InterceptAnnotation
    @GetMapping("/testAspect")
    public String testAspect() {
        log.info("testAspect");
        return "ok";
    }
}
```

5、call api 验证

```bash
curl -X GET "localhost:8080/testAspect"
ok
```

6、控制台输出

```bash
2024-08-26 23:46:00.992  INFO 54043 --- [nio-8080-exec-2] c.e.intercept.InterceptAnnotationAspect  : 基于注解拦截的切面类
2024-08-26 23:46:00.994  INFO 54043 --- [nio-8080-exec-2] c.e.controller.InterceptAopController    : testAspect
```



### 2、基于表达式拦截

```bash
表达式：* com.example.controller..*.*(..)
```

1、定义切面类

```java
package com.example.intercept;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Slf4j
@Aspect
@Component
public class InterceptControllerAspect {

    @Pointcut("execution(* com.example.controller..*.*(..))")
    public void controllerIntercept() {

    }

    @Around("controllerIntercept()")
    public Object controllerInterceptAround(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("基于表达式进行拦截切面类");
        return joinPoint.proceed();
    }
}
```

2、切点多个表达式或注解示例

```java
@Pointcut("execution(* com.example.controller..*.*(..)) || @annotation(com.example.annotation.InterceptAnnotation)")
```

3、定义Controller类，基于表达式测试

```java
package com.example.controller;

import com.example.annotation.InterceptAnnotation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class InterceptAopController {

    @InterceptAnnotation
    @GetMapping("/testAspect")
    public String testAspect() {
        log.info("testAspect");
        return "ok";
    }
}
```

5、call api 验证

```bash
curl -X GET "localhost:8080/testAspect"
ok
```

6、控制台输出

```bash
2024-08-27 00:00:01.294  INFO 54796 --- [nio-8080-exec-1] c.e.intercept.InterceptAnnotationAspect  : 基于注解拦截的切面类
2024-08-27 00:00:01.296  INFO 54796 --- [nio-8080-exec-1] c.e.intercept.InterceptControllerAspect  : 基于表达式进行拦截切面类
2024-08-27 00:00:01.325  INFO 54796 --- [nio-8080-exec-1] c.e.controller.InterceptAopController    : testAspect
```



### 3、通过 @Order 注解规定切面类执行顺序

上面控制台打印结果可以看到先执行了注解切面类然后执行表达式切面类，现实中，可以`通过@Order(0)注解规定切面类执行顺序，数字越小，执行越靠前`。

```java
@Slf4j
@RestController
@Order(value = 0)
public class InterceptAopController {}
```

**测试结果：**表达式切面类先于注解切面类执行。

```java
2024-08-27 00:00:01.294  INFO 54796 --- [nio-8080-exec-1] c.e.intercept.InterceptAnnotationAspect  : 基于表达式进行拦截切面类
2024-08-27 00:00:01.296  INFO 54796 --- [nio-8080-exec-1] c.e.intercept.InterceptControllerAspect  : 基于注解拦截的切面类
2024-08-27 00:00:01.325  INFO 54796 --- [nio-8080-exec-1] c.e.controller.InterceptAopController    : testAspect
```



## 七、定义切面步骤总结

定义切面的步骤：

1. 切面类加 @Aspect、@Service 注解，告诉 SpringBoot 这是一个切面类，让 Spring 容器管理起来
2. 定义一个切点 @Pointcut
3. 定义切入点切入的时机及完成的功能（@Around、@Before、@After、@AfterReturning、@AfterThrowing）



## 八、参考文献 & 鸣谢

- Spring高效实践（AOP 教程非常详细）：https://www.zhihu.com/column/c_1444056002182750208
- SpringAOP专题：http://itsoku.com/course/11





# SpringBoot AOP + 自定义线程池实现异步日志记录

> SpringBoot 使用自定义注解和自定义线程池实现异步日志记录：https://blog.csdn.net/lhmyy521125/article/details/139428055

在我们日常开发工作中，日志记录是至关重要的部分。它不仅有助于调试和故障排除，还能提供系统运行的历史记录，帮助进行性能优化和安全监控。然而，日志记录也可能对系统性能产生影响，特别是在高并发环境下。因此，使用异步日志记录技术可以有效地提升系统性能和可靠性。

下面将带着大家一起探讨 SpringBoot 异步日志记录的优雅实现方法。



1、异步日志记录的重要性
---------------------------------------------------------------------------

### 1、提高性能

在传统的同步日志记录方式中，每次日志记录操作都需要等待日志写入完成，才能继续执行后续操作。这种方式在高并发环境下会导致明显的性能瓶颈。异步日志记录通过将日志写入操作放入独立的线程中执行，避免了主线程的阻塞，从而大幅提高了系统的整体性能。

### 2、减少延迟

在需要快速响应的应用程序中，如实时系统或高频交易系统，任何形式的延迟都会影响系统的响应时间。异步日志记录能将日志记录操作从主业务流程中剥离出来，减少了响应时间，提升了用户体验。

### 3、提升稳定性

在高负载情况下，同步日志记录可能会导致系统资源的争用，影响系统的稳定性。异步日志记录通过使用独立的线程池管理日志写入操作，避免了这种资源争用，提高了系统的稳定性和可靠性。

### 4 资源管理优化

异步日志记录允许更灵活地管理系统资源。通过配置线程池的大小和任务队列，可以更好地控制系统资源的使用，避免了因为日志记录过多导致的内存和磁盘 I/O 资源耗尽问题。



2、构建项目
---------------------------------------------------------------------

### 1、创建 SpringBoot 项目

可以使用 Spring Initializr 创建项目，确保在 `pom.xml` 文件中添加了必要的依赖：

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Boot Starter AOP -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```



### 2、创建自定义注解类

定义一个自定义注解类 LogAsync 用于标记需要异步记录的日志方法，并且系统会记录功能组、操作人、方法名、传递的参数等（仅模拟，大家根据自己项目需求定制） 代码如下：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogAsync {
    // 可以根据需要添加属性 比如功能组
    String funGroup() default "";
}
```



### 3、配置自定义线程池

为了优化异步日志记录，我们需要配置一个自定义线程池

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.Executor;

@EnableAsync
@Configuration
public class AsyncConfig {

    @Bean(name = "logExecutor")
    public Executor logExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("LogExecutor-");
        executor.initialize();
        return executor;
    }
}
```



### 4、编写日志记录切面

创建一个AOP切面 LoggingAspect 来处理异步日志记录

```java
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;
import java.util.concurrent.Executor;

@Slf4j
@Aspect
@Component
public class LoggingAspect {

    private final Executor logExecutor;

    public LoggingAspect(@Qualifier("logExecutor") Executor logExecutor) {
        this.logExecutor = logExecutor;
    }

    @Pointcut("@annotation(LogAsync)")
    public void loggableMethods() {
    }

    @Async("logExecutor")
    @AfterReturning(pointcut = "loggableMethods()", returning = "result")
    public void logMethodCall(JoinPoint joinPoint, Object result) {
        // 获取注解类
        LogAsync logAsync = ((MethodSignature) joinPoint.getSignature()).getMethod().getAnnotation(LogAsync.class);
        // 获取注解上的功能组
        String funGroup = logAsync.funGroup();
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取参数
        Object[] args = joinPoint.getArgs();
        // 方法返回结果  Object result
        log.info("funGroup: {}, Method: {}, Args: {}, Result: {}", funGroup, methodName, args, result);

        // TODO 这里可以加入日志入库操作
    }
}
```



### 5、编写 Controller 中使用 @LogAsync

创建一个 TestUser 类，作为接收参数

```java
import lombok.Data;

@Data
public class TestUser {
    private String username;
    private String password;
}
```

创建 LogSampleController ，标注 @LogAsync 注解

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class LogSampleController {

    /**
     * 模拟保存
     * @param user 自定义用户类
     * @return
     */
    @LogAsync(funGroup = "用户模块")
    @PostMapping("/save-user")
    public ResponseEntity<String> saveUser(@RequestBody  TestUser user) {
        return new ResponseEntity<>("操作成功", HttpStatus.OK);
    }
}
```



4、接口测试
---------------------------------------------------------------------

运行 Spring Boot 应用程序，使用测试工具访问 `http:/your_host/api/save-user`。观察控制台中可以看到异步日志记录的信息

```bash
curl -X POST "localhost:8080/api/save-user" --max-time 30 -d '{ "username": "zhangsan",  "password": "123456"}'
操作成功

### 控制台输出
[LogExecutor-1]....... : funGroup: 用户模块, Method: saveUser, Args: [TestUser(username=zhangsan, password=123456)], Result: <200 OK OK,操作城, []>
```

> **温馨提示**：演示代码中，只是获取了：功能组、方法名、传递的参数、返回结果； 正常我们日志记录还会有很多主要信息，比如：操作人、修改前数据、修改后数据、修改时间等等，大家可以根据自己系统情况进行调整



5、使用总结
-------------------------------------------------------------------

通过自定义注解、Spring AOP 和自定义线程池，我们可以在 SpringBoot 应用中实现高效的异步日志记录。这种方法不仅提高了日志记录的灵活性，还能减小对主业务线程的影响。希望本文对您在实际项目中实现日志记录有所帮助。

这种方式在实际生产环境中非常实用，特别是在需要高效处理大量日志记录的场景下。通过合理配置线程池，可以确保日志记录的性能和稳定性。







