

- https://www.cnblogs.com/binbingg/p/17771923.html
- https://mp.weixin.qq.com/s?__biz=MzAxMjY5NDU2Ng==&mid=2651858469&idx=1&sn=fb123bf1e13a14905139ffaacd9432c8&chksm=8049636cb73eea7adaa46c67ff4f26e65ee4744d8dc5c4ef0268b58603c9cbaa98dff596b1ca&scene=27







## 0、名词解释

领域模型中的实体类分为四种模型：VO、DTO、DO 和 PO，各种实体类用于不同业务层次间的交互，并会在层次内实现实体类之间的转化。新项目使用了新的框架和开发规范，特意集体讨论了DTO，DO，VO，BO，POJO，PO和Entity以及DAO、Model和View的基本概念和使用场景，为了深入理解，这里整理为一篇笔记。它们的大致分类：

- 领域对象
  - 按包含关系划分
    - POJO
      - BO
      - VO
      - DTO
      - PO
    - BO
      - PO
      - VO
      - ENTITY
  - 按传输层划分
    - DTO（商业逻辑层和表示层）
    - PO（数据层）
  - 按业务对象划分
    - ENTITY（有唯一标识）
    - VO（无唯一标识）

**简明扼要地综述如下**：VO 用于后端向前端传输数据，DTO用于前端向后端传输数据，BO用于微服务之间传输数据。PO等同于Entity，DO是Entity的一种，三者用于表示数据库的一条记录，通常用Entity。

***

简单介绍所能接触及使用到的模型类：

- POJO（ Plain Ordinary Java Object）纯普通Java对象
- VO（View Object）视图模型
- DTO（Data Transfer Object）数据传输对象
- DO （Domain Object）领域对象
- PO（Persistent Object）持久化对象
- Entity（应用程序域中的一个概念）实体
- BO（ Business Object）业务对象
- AO（Application Object）应用对象
- Model （概念实体模型）实体类和模型
- View （概念视图模型）视图模型
- DAO （Data Access Object）数据访问对象



## 1、POJO

总的来说，普通 Java 对象POJO（Pure Old Java Object 、 Plain Ordinary Java Object），按照 Martin Fowler 的解释是“Plain Old Java Object”，从字面上翻译为“纯洁老式的java对象”，但大家都使用“简单 Java 对象”来称呼它。包含 DO、DTO、BO、VO 和 PO 等，它们本质上都是一个简单的 Java 对象，实际就是普通的 JavaBeans。没有业务逻辑，有时可以作为 VO 或 DTO 来使用。当然，这里特意说明纯普通 Java 对象，如果你有一个简单的运算属性也是可以的，但不允许有业务方法。

POJO 是指这样的 Java 对象：

- 有一些 private 的参数作为对象的属性
- 针对每一个参数定义get和set方法
- 没有从任何类继承
- 没有实现任何接口
- 没有被其它框架侵入。

许多开发者把 JavaBean 看作遵从特定命名约定的 POJO。POJO 其实是比 JavaBean 更纯净的简单类或接口。POJO 严格地遵守简单对象的概念，而不具有业务逻辑处理的能力，而一些 JavaBean 中往往会封装一些简单逻辑。例如，改造 User 后，可以得到一个JavaBean。



## 2、VO

VO（View Object）视图模型，用于展示层，它的作用是把某个指定页面（或组件）的所有数据封装起来。如果是一个DTO对应一个VO，则DTO等价于VO;但是如果一个DTO对应多个VO，则展示层需要把VO转换为服务层对应方法所要求的DTO，传送给服务层，从而达到服务层与展示层解耦的效果。

一般用在业务逻辑层（Service）对前端（Web） 的视图模型效果控制的展示上，说白了就是**后台向前端传输数据**。示例：xxxVO，xxx一般为网页名称。注：**在展示业务不复杂的系统，可直接使用DTO**。



## 3、DTO

数据传输对象 DTO（Data Transfer Object）是一个比较特殊的对象，它有两种存在形式：**在后端，它的存在形式是 Java 对象，也就是在 Controller 里面定义的请求参数**，通常在后端不需要关心怎么从 json 转成 Java 对象的，这个都是由一些成熟的框架帮你完成啦，比如spring框架；在前端，它的存在形式通常是js里面的对象（也可以简单理解成json），即通过ajax请求的那个数据体。这也是为什么把他画成横跨两层的原因。举个例子，xxxDTO，xxx 为业务领域相关的名称。

现在微服务盛行，服务和服务之间调用的传输对象能叫 DTO 吗？我的理解是看情况，DTO 的一个隐含意义是要能够完整的表达一个业务模块的输出。如果服务和服务之间相对独立，那就可以叫 DTO；如果服务和服务之间不独立，每个都不是一个完整的业务模块，拆开可能仅仅是因为计算复杂度或者性能的问题，那这就不能够叫做 DTO，只能是 BO。

DTO 与 BO 或者 DO 的区别是 DTO 没有任何行为（方法），只是存储和提供它所拥有数据的查询（访问器和修改器）。DTO 是简单对象，不包含任何需要测试的业务逻辑。



## 4、Entity / PO

持久化对象PO（Persistent Object）等同于 Entity，它们的概念是一致的。数据库表中的记录在 Java 对象中的显示状态。最形象的理解就是一个 PO 对象对应数据库中的一条记录，一个 PO 的数据结构对应着库中一张表的表结构，即自身属性与数据表字段一一对应。好处是可以把一条记录作为一个对象处理，方便的转为其它对象。

例如我们有一条数据，现在有一个简单类而且已经是被赋予了这条数据的实例，那么这条数据在这个简单类中的存在状态就是 PO，不管这个简单类是 DO 还是 BO 抑或其它。PO 只是数据持久化的一个状态。

通常 PO 里面除了get，set 之外没有别的方法。对于 PO 来说，数量是相对固定的，一定不会超过数据库表的数量。



## 5、DO

领域对象 DO（Domain Object） 是从现实世界中抽象出来的有形或无形的业务实体，它用来接收数据库对应的实体，是一种抽象化的数据状态，介于数据库与业务逻辑之间。

一般在业务逻辑层（Service）对数据库（SQL） 进行访问时，用于接收数据。xxxDO，xxx即为数据表名。另外，DO与Entity的不同点就是DO是与数据库存在着某种映射关系的Entity，总的来说DO是Entity的一种。

现在主要有两个版本一个是阿里巴巴的开发手册中定义的DO（ Data Object），这个等同于上面的 PO；另一个是在 DDD（Domain-Driven Design）领域驱动设计中定义的DO（Domain Object），这个等同于下面的 BO。



## 6、BO

业务对象（Business Object，BO）是对数据进行检索和处理的组件，主要作用是把业务逻辑封装为一个对象，这个对象可以包括一个或多个其它的对象。形象描述为一个对象的形为和动作，当然也有涉及到其它对象的一些形为和动作。

BO 通常位于中间层或者业务逻辑层。BO 支持序列化和反序列化，可以轻易地将 BO 的 Java 实例转换为一个 XML 文件或者一个流保存起来，并且在需要的时候，将这个 BO 从 XML 或者流中转换回一个 Java 实例。举个简单的例子，一个简历包含教育经历、工作经历、社会关系等三个模块，每个模块对应一个 PO；建立一个 BO 对象处理简历，则每个 BO 包含这三个 PO。

应用中的所有实体（Entity）都是 BO，但并不是所有 BO 都是实体。BO 包括包含方法的实体对象（Entity Object）和不包含方法的值对象（VO）。



## 7、Model

Model 是数学逻辑名词，包括有限操作的集合以及定义于其上的关系，主要用于分析、设计过程。

实体类和模型 Model 在计算机程序设计中有两个概念：一个是三层架构中的实体类，另一个是 MVC 架构中的模型。在“三层架构”中，为了面向对象编程，将各层传递的数据封装成实体类，便于数据传递和提高可读性。在 MVC（模型 Model - 视图 View - 控制器 Controller）模式中，Model 代表模型，是业务流程/状态的处理以及业务规则的制定，接受视图请求的数据，并返回最终的处理结果。业务模型的设计可以说是 MVC 最主要的核心。



## 8、View

在 MVC 模式中，View 代表视图，用来解析 Model 带来的数据模型，以展示视图数据，View 的模型觉决定了需要什么样的 Model 来对接，相互联系。



## 9、DAO

数据访问对象 DAO （Data Access Object）是一个数据访问接口，所谓数据访问，顾名思义，就是与数据库打交道，夹在业务逻辑与数据库资源中间。一般在业务逻辑层对数据库进行访问时使用。

xxxDAO，xxx 即为实体类名（Entity 实体）。在核心 J2EE 模式中是这样介绍 DAO 模式的：为了建立一个健壮的 J2EE 应用，应该将所有对数据源的访问操作抽象封装在一个公共 API 中。用程序设计的语言来说，就是建立一个接口，接口中定义了此应用程序中将会用到的所有事务方法。在这个应用程序中，当需要和数据源进行交互的时候就使用这个接口，并且编写一个单独的类或者 xml 文件，来实现这个接口在逻辑上对特定数据的操作。



## 10、放置目录

- PO 通常放在名为 bean、entity、model 目录中
- DAO 通常在 dao、mapper 目录中
- BO 通常在 service、manager、business 目录中



## 11、POJO 的扩展

POJO 仅包含最简单的字段属性，没有多余的东西，它本质上就是一个普通的 JavaBean。但是在 POJO 的基础上，能够扩展出不同的对象。

- 为 POJO 增加了持久化的方法（Insert、Update、Delete……）之后，POJO 就变成了 PO。
- 为 POJO 增加了数据绑定功能之后，POJO 就变成了 View Object，即 UI Model。
- 为 POJO 增加业务逻辑的方法（比如单据审核、转帐……）之后，POJO 就变成了 Domain Model。
- 且 POJO 还可以当作 DTO 使用。



## 12、调用关系图

```
前端 ===DTO===》Controller ===DTO===》Service ===DO/实体===》Domain ===DO/实体===》Mapper

Mapper ===DO/实体===》Domain ===DO/实体===》Service ===VO/PO/实体===》Controller ===VO/PO/实体===》Mapper
```



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