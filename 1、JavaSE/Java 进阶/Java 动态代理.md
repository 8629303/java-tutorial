> Spring AOP的实现原理及应用场景（通过动态代理）：https://mp.weixin.qq.com/s/l_UuzWNmAfvKNUDAVii0tA

# 1、AOP 的作用

作用：在不修改源代码的情况下，可以实现功能的增强。

AOP 思想：基于代理思想，对原来目标对象，创建代理对象，在不修改原对象代码情况下，通过代理对象，调用增强功能的代码，从而对原有业务方法进行增强 ！

# 2、AOP应用场景

- 场景一：记录日志
- 场景二：监控方法运行时间 （监控性能）
- 场景三：权限控制
- 场景四：缓存优化 （第一次调用查询数据库，将查询结果放入内存对象， 第二次调用， 直接从内存对象返回，不需要查询数据库 ）
- 场景五：事务管理 （调用方法前开启事务， 调用方法后提交关闭事务 ）



# 3、AOP的实现原理

Spring中AOP是怎么实现的呢？Spring中AOP的有两种实现方式：

- JDK动态代理
- Cglib动态代理



## 1、JDK动态代理

1、引入依赖，有spring，单元测，日志管理

```xml
<dependencies>
    <!-- Spring -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>

    <!-- 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- 日志 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
    </dependency>
</dependencies>
```

2、UserDao 接口

```java
public interface UserDao {
    public void saveUser();
}
```

3、UserDao 实现类

```java
public class UserDaoImpl implements UserDao {
    @Override
    public void saveUser() {
        System.out.println("持久层：用户保存");
    }
}
```

4、动态代理

```java
@Test
public void test1() {

    final UserDao userDao = new UserDaoImpl();
    // newProxyInstance的三个参数解释：
    // 参数1：代理类的类加载器，同目标类的类加载器
    // 参数2：代理类要实现的接口列表，同目标类实现的接口列表
    // 参数3：回调，是一个InvocationHandler接口的实现对象，当调用代理对象的方法时，执行的是回调中的invoke方法
    //proxy为代理对象
    UserDao proxy = (UserDao) Proxy.newProxyInstance(userDao.getClass().getClassLoader(),
            userDao.getClass().getInterfaces(), new InvocationHandler() {
                @Override
                // 参数proxy:被代理的对象
                // 参数method:执行的方法，代理对象执行哪个方法，method就是哪个方法
                // 参数args:执行方法的参数
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("记录日志");
                    Object result = method.invoke(userDao, args);
                    return result;
                }
            });
    //代理对象执行方法
    proxy.saveUser();
}
```

5、结果，在没有修改原有类的代码的情况下，对原有类的功能进行了增强

```
记录日志
持久层：用户保存
```



## 2、Cglib动态代理

在实际开发中，可能需要对没有实现接口的类增强，用JDK动态代理的方式就没法实现。采用[Cglib动态代理](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247492039&idx=1&sn=215e51a412ec94d51f24b70fb8d0f9cf&chksm=ebd5deebdca257fd96db36f28880d526f89f7130adbef7df5e7baf54194540c72544515668a9&scene=21#wechat_redirect)可以对没有实现接口的类产生代理，实际上是生成了目标类的子类来增强。

首先，需要导入Cglib所需的jar包。提示：spring已经集成了cglib，我们已经导入了spring包，所以不需要再导入其它包了。

1、创建LinkManDao类，没有实现任何接口

```java
public class LinkManDao {
    public void save(){
        System.out.println("持久层：联系人保存....");
    }
}
```

2、动态代理

```java
@Test
public void test2() {
    final LinkManDao linkManDao = new LinkManDao();
    // 创建cglib核心对象
    Enhancer enhancer = new Enhancer();
    // 设置父类
    enhancer.setSuperclass(linkManDao.getClass());
    // 设置回调
    enhancer.setCallback(new MethodInterceptor() {
        /**
         * 当你调用目标方法时，实质上是调用该方法
         * intercept四个参数：
         * proxy:代理对象
         * method:目标方法
         * args：目标方法的形参
         * methodProxy:代理方法
        */
        @Override
        public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy)
                throws Throwable {
            System.out.println("记录日志");
            Object result = method.invoke(linkManDao, args);
            return result;
        }
    });
    // 创建代理对象
    LinkManDao proxy = (LinkManDao) enhancer.create();
    proxy.save();
}
```

3、结果

```
记录日志
持久层：用户保存
```

