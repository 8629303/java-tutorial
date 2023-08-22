> - 作者：SpringForAll；来源：https://mp.weixin.qq.com/s/JzoY4jMyydW_Ae9aIM4ETA
>
> - Spring事务详解：https://blog.csdn.net/csdn_wyl2016/category_11607107.html

# 一、Spring 事务基本原理

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。对于纯JDBC操作数据库，想要用到事务，可以按照以下步骤进行：

1. 获取连接：Connection con = DriverManager.getConnection()
2. 开启事务：con.setAutoCommit(true/false);
3. 执行CRUD
4. 提交事务 or 回滚事务：con.commit(); 或  con.rollback();
5. 关闭连接：conn.close();

使用Spring的事务管理功能后，我们可以不再写步骤 2 和 4 的代码，而是由Spirng 自动完成。那么Spring是如何在我们书写的 CRUD 之前和之后开启事务和关闭事务的呢？解决这个问题，也就可以从整体上理解Spring的事务管理实现原理了。下面简单地介绍下，注解方式为例子

1. 配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional标识。
2. spring 在启动的时候会去解析生成相关的bean，这时候会查看拥有相关注解的类和方法，并且为这些类和方法生成代理，并根据@Transaction的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务）。
3. 真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

# 二、Spring 的事务机制

所有的数据访问技术都有事务处理机制，这些技术提供了API用来开启事务、提交事务来完成数据操作，或者在发生错误的时候回滚数据。

而Spring的事务机制是用统一的机制来处理不同数据访问技术的事务处理。Spring的事务机制提供了一个PlatformTransactionManager接口，不同的数据访问技术的事务使用不同的接口实现，如表所示。

数据访问技术及实现

| 数据访问技术 |             实现             |
| :----------: | :--------------------------: |
|     JDBC     | DataSourceTransactionManager |
|     JPA      |    JapTransactionManager     |
|  Hibernate   | HibernateTransactionManager  |
|     JDO      |    JdoTransactionManager     |
|  分布式事务  |    JtaTransactionManager     |

在程序中定义事务管理器的代码如下：

```java
@Bean 
public PlatformTransactionManager transactionManager() { 
    JpaTransactionManager transactionManager = new JpaTransactionManager(); 
    transactionManager.setDataSource(dataSource()); 
    return transactionManager; 
}
```

# 三、Spring 声名式事务

Spring支持声名式事务，即使用注解来选择需要使用事务的方法，它使用@Transactional注解在方法上表明该方法需要事务支持。这是一个基于AOP的实现操作。

```java
@Transactional 
public void saveSomething(Long  id, String name) { 
    // 数据库操作 
}
```

在此处需要特别注意的是，此@Transactional注解来自org.springframework.transaction.annotation包，而不是javax.transaction。



## 1、AOP 代理的两种实现

- JDK是代理接口，私有方法必然不会存在在接口里，所以就不会被拦截到；
- Cglib是子类，private的方法照样不会出现在子类里，也不能被拦截。

### 1、Java 动态代理

具体有如下四步骤：

1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。

### 2、GCLIB代理

Cglib（Code Generation Library）是一个强大的,高性能,高质量的Code生成类库。它可以在运行期扩展Java类与实现Java接口。

- cglib封装了asm，可以在运行期动态生成新的class（**子类**）。
- cglib用于AOP，jdk中的proxy必须基于接口，cglib却没有这个限制。

### 3、原理区别

Java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

1. 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP
2. 如果目标对象实现了接口，可以强制使用CGLIB实现AOP
3. 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

**如果是类内部方法直接不是走代理，这个时候可以通过维护一个自身实例的代理。**

```java
@Service
public class PersonServiceImpl implements PersonService {
    @Autowired
    PersonRepository personRepository;

    // 注入自身代理对象，在本类内部方法调用事务的传递性才会生效
    @Autowired
    PersonService selfProxyPersonService;

    /**
     * 测试事务的传递性
     *
     * @param person
     * @return
     */
    @Transactional
    public Person save(Person person) {
        Person p = personRepository.save(person);
        try {
            // 新开事务 独立回滚
            selfProxyPersonService.delete();
        } catch (Exception e) {
            e.printStackTrace();
        }
        try {
            // 使用当前事务 全部回滚
            selfProxyPersonService.save2(person);
        } catch (Exception e) {
            e.printStackTrace();
        }
        personRepository.save(person);

        return p;
    }

    @Transactional
    public void save2(Person person) {
        personRepository.save(person);
        throw new RuntimeException();
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void delete() {
        personRepository.delete(1L);
        throw new RuntimeException();
    }
}
```



## 2、@Transactional 注解详解

### 1、@Transactional 介绍及注意事项

**@Transactional 加在类上：**说明该事务作用于类中的所有方法

**@Transactional 加载方法上：**说明该事务只作用域该方法，只能加在public方法上

**@Transactional 避坑注意事项**： 

1. **@Transactional 注解只能加在 public 方法上**，这是由 Spring AOP 的本质动态代理决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。 Spring默认使用的是jdk自带的基于接口的代理，而没有使用基于类的代理CGLIB。
2. **@Transactional注解底层使用的是动态代理来进行实现的**，如果在调用本类中的方法，此时不添加@Transactional注解，而是在调用类中使用this调用本类中的另外一个添加了@Transactional注解，此时this调用的方法上的@Transactional注解是不起作用的，**因为 @Transactional其实是为你的类做了一个代理，既然是代理类那么只有调用代理类才能真正的执行到增强的方法，如果是在类的内部调用的化意味着没有调用增强的方法，所以声明式事务是不生效的。**
3. **【重点中的重点】 Spring 在扫描 Bean 的时候会扫描方法上是否包含 @Transactional 注解，如果包含，Spring 会为这个Bean动态地生成一个子类（即代理类，proxy），代理类是继承原来那个Bean的。此时，当这个有注解的方法被调用的时候，实际上是由代理类来调用的，代理类在调用之前就会启动transaction。然而，如果这个有注解的方法是被同一个类中的其他方法调用的，那么该方法的调用并没有通过代理类，而是直接通过原来的那个Bean，所以就不会启动transaction，我们看到的现象就是@Transactional注解无效。**（总结来说就是：同类中，无事务的方法，调用添加事务的方法，这个方法的事务并不生效）
4. **如果异常被try｛｝catch｛｝了**，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw Exception｝。
5. @Transactional 注解标识的方法，处理过程尽量的简单。尤其是带锁的事务方法，能不放在事务里面的最好不要放在事务里面。可以将常规的数据库查询操作放在事务前面进行，而事务内进行增、删、改、加锁查询等操作。
6. @Transactional既可以作用于接口，接口方法上以及类已经类的方法上。**但是Spring官方不建议接口或者接口方法上使用该注解**，因为这只有在使用基于接口的代理时它才会生效。
7. 你当然可以在接口上使用 @Transactional 注解，但是这将只能当你设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果你正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装（将被确认为严重的）。**因此，请接受Spring团队的建议并且在具体的类上使用 @Transactional 注解。**



### 2、@Transactional 属性介绍

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```

| 属性                   | 类型                               | 描述                                       |
| :--------------------- | :--------------------------------- | :----------------------------------------- |
| value                  | String                             | 可选的限定描述符，指定使用的事务管理器     |
| propagation            | enum: Propagation                  | 定义事务的生命周期，详细可查看下方。       |
| isolation              | enum: Isolation                    | 可选的事务隔离级别设置，决定了事务的完整性 |
| readOnly               | boolean                            | 读写或只读事务，默认读写                   |
| timeout                | int (in seconds granularity)       | 事务超时时间设置                           |
| rollbackFor            | Class对象数组，必须继承自Throwable | 导致事务回滚的异常类数组                   |
| rollbackForClassName   | 类名数组，必须继承自Throwable      | 导致事务回滚的异常类名字数组               |
| noRollbackFor          | Class对象数组，必须继承自Throwable | 不会导致事务回滚的异常类数组               |
| noRollbackForClassName | 类名数组，必须继承自Throwable      | 不会导致事务回滚的异常类名字数组           |

- propagation：定义事务的生命周期，有REQUIRED、SUPPORTS、MANDATORY、REQUIRES_NEW、NOT_SUPPORTED、NEVER、NESTED，详细含义可查阅枚举类org.springframework.transaction.annotation.Propagation源码。

#### 1、propagation属性（事务传播行为）

**Spring事务有7种传播行为**：使用 propagation 指定事务的传播行为, 即当前的事务方法被另外一个事务方法调用时如何使用事务, 常用属性值有: SUPPORTS ：对于只包含查询的方法一般使用这个。

@Transactional(propagation = Propagation.REQUIRED)：默认之为 Propagation.REQUIRED。

| 属性                      | 描述                                                         |
| :------------------------ | ------------------------------------------------------------ |
| Propagation.REQUIRED      | 如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置 |
| Propagation.SUPPORTS      | 支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行 |
| Propagation.MANDATORY     | 支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常 |
| Propagation.REQUIRES_NEW  | 创建新事务，无论当前存不存在事务，都创建新事务               |
| Propagation.NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起（不需要事务管理的(只查询的)） |
| Propagation.NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常               |
| Propagation.NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作 |



#### 2、isolation属性（事务隔离级别）

使用 isolation 指定事务的隔离级别, 属性值为：**@Transactional(isolation = Isolation.READ_COMMITTED)**

| 属性             | 描述                                                         |
| :--------------- | ------------------------------------------------------------ |
| DEFAULT          | （默认值）使用底层数据库的默认隔离级别，对大多数据库来说默认隔离级别都是READ_COMMITTED |
| READ_UNCOMMITTED | 允许事务读取其它事务未提交的数据变更 (会出现脏读, 不可重复读) 基本不使用 |
| READ_COMMITTED   | 只允许事务读取已提交的数据 (可以避免脏读，但可能出现不可重复读和幻读) |
| REPEATABLE_READ  | 可重复读，确保事务可以多次从一个字段中读取相同的值，在这个事务延续期间，禁止其他事务对这个字段进行更新 (可以避免脏读和不可重复读，但可能出现幻读) |
| SERIALIZABLE     | 串行化，确保事务可以从一个表中读取相同的行，在这个事务延续期间，不允许其他事务对该表执行插入、修改、删除的操作。（所有并发问题都可避免，但效率十分低下） |

最常用的取值为：**READ_COMMITTED**。事务的隔离级别要得到底层数据库引擎的支持, 而不是应用程序或者框架的支持. 

- Oracle 支持的 2 种事务隔离级别：READ_COMMITED , SERIALIZABLE 。
- MySQL 支持 4 中事务隔离级别。



#### 3、rollbackFor属性（事务回滚设置）

Spring 框架的事务管理默认地只在发生不受控异常（RuntimeException 和 Error）时才进行事务回滚。

也就是说，当事务方法抛出受控异常（ Exception 中除了 RuntimeException 及其子类以外的）时不会进行事务回滚。

- 受控异常（checked exceptions）：就是非运行时异常，即Exception中除了RuntimeException及其子类以外的。

- 不受控异常（unchecked exceptions）：RuntimeException和Error。


rollbackFor 属性在这里就可以发挥它的作用了，这里我们修改成 Exception 异常就回滚。

-  @Transactional(rollbackFor = Exception.class)


 这里你可以使用 java 已声明的异常；也可以使用自定义异常；也可同时设置多个异常类，中间用逗号间隔。

-  @Transactional(rollbackFor = {SQLException.class，UserAccountException.class}) 



#### 4、noRollbackFor属性（事务不回滚设置）

默认情况下 Spring 的声明式事务对所有的运行时异常进行回滚. 可以通过noRollbackFor属性进行设置例外异常。如： 

- @Transactional(noRollbackFor = {UserAccountException.class}) 

上面设置了遇到UserAccountException异常不回滚。 一般不建议设置这个属性，通常情况下默认即可。



#### 5、readOnly 属性

@Transactional(readOnly = false)：默认为 ：false 

- 指定事务是否为只读，表示这个事务只读取数据但不更新数据。
- 这样可以帮助数据库引擎优化事务。
- 如果只有查询数据操作，应设置 readOnly=true。



#### 6、timeout 属性

@Transactional(timeout = 5)：

- 指定强制回滚之前事务可以占用的时间。单位：秒。
- 如果执行时间超过这个时间就强制回滚。



#### 7、操作示例

```java
/**
 * @Transactional默认只有遇到 RuntimeException 和 error 的时候才回滚
 * 可以通过 rollbackFor 的设置来增加回滚的条件
 */
@Transactional(propagation = Propagation.REQUIRED,
               isolation = Isolation.DEFAULT,
               rollbackFor = {Exception.class},
               timeout = 5)
@Service
public class UsersServiceImpl implements UsersService {

    @Resource
    UsersBeanMapper usersBeanMapper;

    @Transactional(readOnly = true)
    @Override
    public UsersBean getUserByLogin(String username, String pwd) throws Exception {
        UsersBean UsersBean=usersBeanMapper.selectByUsersBean(new UsersBean(username, pwd));
        return UsersBean;
    }

    @Override
    public int deleteById(Integer id) {
        try {
            int row = usersBeanMapper.deleteByPrimaryKey(id);
            if (row<1) {
                throw new MyException("我没有找到对应的ID，无法进行删除");
            }
            return row;
        } catch (Exception e) {
            throw new RuntimeException("我执行的时候出错了");
            // throw new SQLException("我执行sql的时候出错了");
        }
    }
}
```



# 四、Spring 编程式事务

```java
/**
 * 注入
 */
@Autowired
private DataSourceTransactionManager transactionManager;

@Override
public String interfaceTest(Map<String, Object> map) {
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    // 设置事务名称
    def.setName("test-Transactional");
    // 读写或只读事务、默认：读写
    def.setReadOnly(true);
    // 事务超时时间，默认：不超时 单位：秒
    def.setTimeout(10);
    /**
     * 设置事务的隔离级别
     *
     * 1、TransactionDefinition.ISOLATION_DEFAULT
     *    这是一个默认的隔离级别，默认使用数据库的隔离级别。
     * 2、TransactionDefinition.ISOLATION_READ_UNCOMMITTED
     *    这是事务最低的隔离界别，他允许别的事务可以看到这个事务未提交的数据。
     *    这种隔离级别可能会产生脏读，不可重复读和幻像读。
     * 3、TransactionDefinition.ISOLATION_READ_COMMITTED
     *    这个级别可以保证在数据提交以后才能被另个一个事务读取。
     *    这种隔离级别可以避免脏读，但是可能会出现不可重复读和幻读。
     * 4、TransactionDefinition.ISOLATION_REPEATABLE_READ
     *    这个级别可以禁止一个事务读取一个没有提交变化的行。
     *    这种隔离级别可以防止脏读和不可重复读，但是可能会出现幻读。
     * 5、TransactionDefinition.ISOLATION_SERIALIZABLE
     *    这种隔离级别可以防止脏读、不可重复读、幻读。
     *
     */
    def.setIsolationLevel(TransactionDefinition.ISOLATION_DEFAULT);
    /**
     * 设置事务的传播行为
     *
     * 1、TransactionDefinition.PROPAGATION_REQUIRED
     *    如果当前存在事务，则加入该事务，如果当前没有事务，则创建一个新的事务。默认值
     * 2、TransactionDefinition.PROPAGATION_REQUIRES_NEW
     *    创建一个新的事务，如果当前存在事务，则把当前的事务挂起。
     * 3、TransactionDefinition.PROPAGATION_SUPPORTS
     *    如果当前存在事务，则加入该事务，如果当前没有事务，则以非事务的方式继续运行。
     * 4、TransactionDefinition.PROPAGATION_NOT_SUPPORTED
     *    以非事务方式运行，如果当前存在事务，则把当前事务挂起。
     * 5、TransactionDefinition.PROPAGATION_NEVER
     *    以非事务方式运行，如果当前存在事务，则抛出异常。
     * 6、TransactionDefinition.PROPAGATION_MANDATORY
     *    如果当前存在事务、则加入该事务，如果当前没有事务，则抛出异常
     * 7、TransactionDefinition.PROPAGATION_NESTED
     *    如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行。
     *    如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。
     */
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    TransactionStatus transactionStatus = transactionManager.getTransaction(def);
    try {
        //
        // 提交事务
        transactionManager.commit(transactionStatus);
    } catch (Exception e) {
        // 回滚事务
        transactionManager.rollback(transactionStatus);
    }
    return null;
}
```



# 五、Spring 事务的传播属性

所谓spring事务的传播属性，就是定义在存在多个事务同时存在的时候，spring应该如何处理这些事务的行为。这些属性在TransactionDefinition中定义，具体常量的解释见下表：

|         常量名称          |                           常量解释                           |
| :-----------------------: | :----------------------------------------------------------: |
|   PROPAGATION_REQUIRED    | 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是 Spring 默认的事务的传播。 |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作 |
|   PROPAGATION_SUPPORTS    |     支持当前事务，如果当前没有事务，就以非事务方式执行。     |
|   PROPAGATION_MANDATORY   |         支持当前事务，如果当前没有事务，就抛出异常。         |
| PROPAGATION_NOT_SUPPORTED |  以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  |
|     PROPAGATION_NEVER     |       以非事务方式执行，如果当前存在事务，则抛出异常。       |
|    PROPAGATION_NESTED     | 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。 |

# 六、数据库隔离级别

| 隔离级别         | 隔离级别的值 | 导致的问题                                                   |
| :--------------- | :----------- | :----------------------------------------------------------- |
| Read-Uncommitted | 0            | 导致脏读                                                     |
| Read-Committed   | 1            | 避免脏读，允许不可重复读和幻读                               |
| Repeatable-Read  | 2            | 避免脏读，不可重复读，允许幻读                               |
| Serializable     | 3            | 串行化读，事务只能顺序执行，避免了脏读、不可重复读、幻读。执行效率慢，使用时慎重 |

- 脏读：一事务对数据进行了增删改，但未提交，另一事务可以读取到未提交的数据。如果第一个事务这时候回滚了，那么第二个事务就读到了脏数据。
- 不可重复读：一个事务中发生了两次读操作，第一次读操作和第二次操作之间，另外一个事务对数据进行了修改，这时候两次读取的数据是不一致的。
- 幻读：第一个事务对一定范围的数据进行批量修改，第二个事务在这个范围增加一条数据，这时候第一个事务就会丢失对新增数据的修改。

总结：

1. 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。
2. 大多数的数据库默认隔离级别为 Read Commited，比如 SqlServer、Oracle
3. 少数数据库默认隔离级别为：Repeatable Read 比如：MySQL InnoDB

# 七、Spring 中的隔离级别

|            常量            |                             解释                             |
| :------------------------: | :----------------------------------------------------------: |
|     ISOLATION_DEFAULT      | 这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与 JDBC 的隔离级别相对应。 |
| ISOLATION_READ_UNCOMMITTED | 这是事务最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 |
|  ISOLATION_READ_COMMITTED  | 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。 |
| ISOLATION_REPEATABLE_READ  | 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。 |
|   ISOLATION_SERIALIZABLE   | 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。 |

# 八、事务的嵌套

通过上面的理论知识的铺垫，我们大致知道了数据库事务和Spring事务的一些属性和特点，接下来我们通过分析一些嵌套事务的场景，来深入理解Spring事务传播的机制。

假设**外层事务 Service A** 的 Method A() 调用 **内层Service B** 的 Method B()



## 1、PROPAGATION_REQUIRED（Spring 默认）

如果ServiceB.methodB() 的事务级别定义为 PROPAGATION_REQUIRED，那么执行 ServiceA.methodA() 的时候Spring已经起了事务，这时调用 ServiceB.methodB()，ServiceB.methodB() 看到自己已经运行在 ServiceA.methodA() 的事务内部，就不再起新的事务。

假如 ServiceB.methodB() 运行的时候发现自己没有在事务中，他就会为自己分配一个事务。

这样，在 ServiceA.methodA() 或者在 ServiceB.methodB() 内的任何地方出现异常，事务都会被回滚。



## 2、PROPAGATION_REQUIRES_NEW

比如我们设计 ServiceA.methodA() 的事务级别为 PROPAGATION_REQUIRED，ServiceB.methodB() 的事务级别为 PROPAGATION_REQUIRES_NEW。

那么当执行到 ServiceB.methodB() 的时候，ServiceA.methodA() 所在的事务就会挂起，ServiceB.methodB() 会起一个新的事务，等待 ServiceB.methodB() 的事务完成以后，它才继续执行。

他与 PROPAGATION_REQUIRED 的事务区别在于事务的回滚程度了。因为 ServiceB.methodB() 是新起一个事务，那么就是存在两个不同的事务。如果 ServiceB.methodB() 已经提交，那么 ServiceA.methodA() 失败回滚，ServiceB.methodB() 是不会回滚的。如果 ServiceB.methodB() 失败回滚，如果他抛出的异常被 ServiceA.methodA() 捕获，ServiceA.methodA() 事务仍然可能提交(主要看B抛出的异常是不是A会回滚的异常)。



## 3、PROPAGATION_SUPPORTS

假设ServiceB.methodB() 的事务级别为 PROPAGATION_SUPPORTS，那么当执行到ServiceB.methodB()时，如果发现ServiceA.methodA()已经开启了一个事务，则加入当前的事务，如果发现ServiceA.methodA()没有开启事务，则自己也不开启事务。这种时候，内部方法的事务性完全依赖于最外层的事务。



## 4、PROPAGATION_NESTED

现在的情况就变得比较复杂了, ServiceB.methodB() 的事务属性被配置为 PROPAGATION_NESTED, 此时两者之间又将如何协作呢? ServiceB#methodB 如果 rollback, 那么内部事务(即 ServiceB#methodB) 将回滚到它执行前的 SavePoint 而外部事务(即ServiceA#methodA) 可以有以下两种处理方式：

1. 捕获异常，执行异常分支逻辑

   ```java
   void methodA() { 
       try { 
           ServiceB.methodB(); 
       } catch (SomeException) { 
           // 执行其他业务, 如 ServiceC.methodC(); 
       } 
   }
   ```

   这种方式也是嵌套事务最有价值的地方, 它起到了分支执行的效果, 如果 ServiceB.methodB 失败, 那么执行 ServiceC.methodC(), 而 ServiceB.methodB 已经回滚到它执行之前的 SavePoint, 所以不会产生脏数据(相当于此方法从未执行过), 这种特性可以用在某些特殊的业务中, 而 PROPAGATION_REQUIRED 和 PROPAGATION_REQUIRES_NEW 都没有办法做到这一点。

2. 外部事务回滚/提交 代码不做任何修改, 那么如果内部事务(ServiceB#methodB) rollback, 那么首先 ServiceB.methodB 回滚到它执行之前的 SavePoint(在任何情况下都会如此), 外部事务(即 ServiceA#methodA) 将根据具体的配置决定自己是 commit 还是 rollback

   另外三种事务传播属性基本用不到，在此不做分析。



# 九、事务的总结

对于项目中需要使用到事务的地方，我建议开发者还是使用Spring的TransactionCallback接口来实现事务，不要盲目使用Spring事务注解，如果一定要使用注解，那么一定要对Spring事务的传播机制和隔离级别有个详细的了解，否则很可能发生意想不到的效果。

## 1、SpringBoot 对事务的支持

通过org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration类。我们可以看出Spring Boot自动开启了对注解事务的支持 Spring

## 2、只读事务的概念（readOnly = true）

> Spring 的只读事务开启：@Transactional(readOnly = true)

- 概念：从这一点设置的时间点开始（时间点A）到这个事务结束的过程中，其他事务所提交的数据，该事务将看不见！（查询中不会出现别人在时间点A之后提交的数据）

  **@Transcational(readOnly=true)** 这个注解一般会写在业务类上，或者其方法上，用来对其添加事务控制。当括号中添加readOnly=true, 则会告诉底层数据源，这个是一个只读事务，对于JDBC而言，只读事务会有一定的速度优化。而这样写的话，事务控制的其他配置则采用默认值，事务的隔离级别(isolation) 为DEFAULT,也就是跟随底层数据源的隔离级别，事务的传播行为(propagation)则是REQUIRED，所以还是会有事务存在，一代在代码中抛出RuntimeException，依然会导致事务回滚。

- 应用场合：

  1. 如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持SQL执行期间的读一致性；
  2. 如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询SQL必须保证整体的读一致性，否则，在前条SQL查询之后，后条SQL查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

【注意是一次执行多次查询来统计某些信息，这时为了保证数据整体的一致性，要用只读事务】



# 十、参考文献 & 鸣谢

1. Spring事务详解：https://blog.csdn.net/csdn_wyl2016/category_11607107.html
5. Spring 声名式事务@Transactional注解详解：https://blog.csdn.net/fox_bert/article/details/99460057