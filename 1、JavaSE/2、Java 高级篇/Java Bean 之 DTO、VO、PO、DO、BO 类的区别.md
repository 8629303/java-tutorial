> 你还不知道 DTO、VO、PO、DO、BO 的区别？https://mp.weixin.qq.com/s/t6xojlgkv1pQ48FUhj7yww
>
> PO BO VO DTO POJO DAO DO这些Java中的概念分别指一些什么？https://www.zhihu.com/question/39651928

# 一、概念

POJO、DTO、VO、PO、DO、BO、Entity 这些都是在 MVC 三层结构之间传输的实体对象。

先来看看阿里巴巴开发规约的定义：

- AO（ Application Object）：应用对象。 在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
- BO（ Business Object）：业务对象。 由Service层输出的封装业务逻辑的对象。
- DO（ Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。
- VO（ View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。
- DTO（ Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。
- POJO（ Plain Ordinary Java Object）：POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。
- Query：数据查询对象，各层接收上层的查询请求。 注意超过2个参数的查询封装，禁止使用Map类来传输。

POJO、DTO、VO、PO、DO、BO、Entity 详细介绍：

- DTO（Data Transfer Object）数据传输对象：只作为简单的数据传输，没有业务逻辑处理。DTO 一般用在前端向后端传输数据，也就是 MVC 模式的 Controller 接口入参。
- VO（View Object）视图对象：用于展示层，把某个指定页面（或组件）的所有数据封装起来。一般用在后端向前端传输数据，也就是 Controller 接口的出参。
- PO（Persistent Object）持久化对象：PO 跟持久层（通常是关系型数据库）的数据结构形成一一对应的映射关系，一个 PO 对象数据就是表的一条记录，表的每个字段就对应着  PO 的一个属性。PO 只是数据的对象，不包含任何操作。
- DO（Data Object）数据对象：在阿里巴巴 Java 开发手册对 MVC 分层领域模型的规范中，DO 等同于 PO。在 DDD（Domain-Driven Design）领域驱动设计中，DO 等同于 BO。
- Entity：应用程序域中的一个概念，对象实体。等同于 PO / DO。（好像用这个名比较多）
- BO（Business Object）业务对象：BO 把业务逻辑封装为一个对象，一类业务对应一个 BO 对象，这个对象可以包括一个或多个其它的对象。BO 是 PO / DO 的组合。举个例子，BO是个人网站行为业务对象，PO1是交易记录，PO2是登录记录，PO3是商品浏览记录，PO4是添加购物车记录，PO5是搜索记录，所以 BO 就是这些 PO 的组合，包含了多张表的数据记录。一般用在包含业务功能模块的具体实例上，比如写了一个Controller、一个Service、一个DAO、一个工具类等等这一系列实例组合后能实现一些功能，这一系列实例组合为一个组件，这个组件就是 BO。
- POJO（Plain Ordinary Java Object）纯普通Java对象：POJO 专指只有 setter、getter、toString 方法的简单 Java 对象，包括 DO、DTO、BO、VO 等，实际就是普通 JavaBeans，是为了避免和 EJB 混淆所创造的简称。

领域模型命名规范：

- 数据对象：xxxPO / xxxDO / xxxEntity，xxx 即为数据表名。以前用 PO 很多，现在普遍喜欢用 DO / Entity 命名。
- 数据传输对象：xxxDTO，xxx为业务领域相关的名称，比如 addXxxDTO、updateXxxDTO 等。
- 展示对象：xxxVO，xxx一般为网页名称。
- 业务对象：xxxBO，xxx是业务名称。
- POJO：DO/DTO/BO/VO 的统称，禁止命名成 xxxPOJO。

Java 实体类为什么需要分那么多类型呢？

- 例如，用户信息包含：用户姓名name、用户密码password、用户的年龄age，首先数据库层获取PO数据包含这三个字段的数据，可是password不应该暴露出去，怎么做呢，在发送给服务层的时候做一次处理，转成只有name和password的DTO，这样就能减少出数据的传输，同时将name可以改为username，就可以保证数据库结构的安全。
- 怎么解耦呢，用户数据库结构改变了，把name改为displayName，PO对象的字段也修改了，如果没有DTO做数据转换的话，所有使用po来存储用户信息的服务都要受到影响。



# 二、易混点

## 1、DTO 和 VO 的区别

DTO 一般是前端向后端传输的数据，VO 一般是后端向前端传输的数据，大部分场景下，属性值基本是一样的。这就疑惑了，有了 DTO 为什么还需要有 VO？

主要有两方面的区别：

- VO 在部分场景下不需要 DTO 所有字段，有时需要删除或者增加部分字段，比如增加中文解释。
- 根据业务场景需求，VO 有些值可能跟DTO的值不一样。



举个例子，有个业务场景，需要根据时间范围筛选出用户列表。

Entity 表示数据库表的映射对象。

```java
@Data
class UserEntity {
    private Long userId;
    private String userName;
    private String passWord;
    private Datetime createTime；
    private Long deptId;
}
```



DTO 包含前端传过来的筛选开始时间和筛选结束时间。

```java
@Data
public class UserQueryConditionDTO {
    private String userName;
    private Datetime startTime;
    private Datetime endTime;
}
```



前端展示时需要显示部门名称，而不是部门 id，VO如下。

```java
@Data
class UserVO {
    private Long userId;
    private String userName;
    private Long deptId;
    private String deptName;
    private Datetime createTime；
}
```



## 2、BO 和 DTO 的区别

BO 和 DTO 都可以对 PO / DO 进行组合，但他们的区别主要是用途和字段。

BO 是业务对象，主要是对内提供数据的，可能包含所有数据，包括辅助数据等。而 DTO 是数据传输对象，主要是对外提供数据的。DTO 在 BO 的基础上，在对外提供数据时，删减掉不需要用到的某些属性，抽取自己需要的属性后对外提供数据。



# 三、总结

最后，做个通俗易懂的区分：

- **POJO：**普通 Java 对象，包括 DO、DTO、BO、VO 等。
- **PO / DO / Entity：**其数据结构对应数据表中的一条记录。
- **BO：**可以理解为 PO / DO / Entity 的组合。
- **DTO：**传输数据，可以前端传给后端，也可以后端传给前端时用 DTO。
- **VO：**视图对象，前端最后所需要的数据，即后端传给前端返回 VO。



概念规范很多，但没有必要教条主义，把这些全都用上。

引入这些，是为了在团队成员协作开发业务复杂的项目时，能够满足不同业务展示需求，避免有人修改某个POJO对象属性而影响到大范围的功能，提高代码的可维护性和可读性。



如果业务场景很简单，团队协作人员少，照搬全上反而会增加开发的复杂度和工作量。可以考虑以下方案：

- 无论是 PO，DO，还是 Entity，用于映射数据库记录，必须要有，并且只选一种。
- DTO 和 BO 在业务不复杂情况下，BO 可以用 DTO 代替。
- 在不需要新增或删减字段情况下，VO 可以用 DTO 代替。
- 在简单的 MVC 项目中，可以直接用 DTO 和 POJO 就行。



不管用哪种方式，只要团队内部达成统一标准即可，适合自己的才是最好的。