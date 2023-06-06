> 作者：小姐姐味道，来源：https://blog.csdn.net/lycyingO/article/details/117575555
>

## 1、YML前言

> 配置文件界，已经被下面三种所统治：`yaml`，`toml`和`json`，这让一直使用properties文件的我深深的埋下了头。
> 又名：YML or YAML

不要担心，当你读到文章最后，你也会羞愧的埋下头。也可能会有一丝愤怒。

像各种人工智能调参数，k8s调参师，都已经成功升级为yml配置大师。作为一个常年使用yml文件的SpringBoot框架使用者，有时候对yml的表现形式竟然显露出了困惑，这不由得让人羞愧又加了一层。

YAML，竟然是XML的一个子集，所以它的复杂是有源头的，最早诞生于2009年。

使用yml文件，首先遇到的问题，就是它的缩进问题。就如同python语言一样，yml文件的表现层次，是靠嵌套的缩进来完成的。它并不使用TAB，而是使用空格表示缩进。

要命的是，空格的多少，并不重要，只要相同级别元素左侧能够对齐就行。这对于CV党来说，不得不说是一个噩梦哈哈。

那一个配置文件，要解决哪些问题呢？Redis已经做出了回答。就像你学习一门新的语言一样，解决了它的字符串和集合的表示方法，基本上写代码就没问题了。那我们就挨个来看一下。

以下方法以SpringBoot的yml文件格式为准，其他场景的解析器会有些许差异。为了能够debug这些值，我们简单的写了一个测试类，然后再设值完成之后打印以下就可以了。

```java
@EnableAutoConfiguration
@Configuration
public class TestConfig implements InitializingBean {
    @Value("${str1}")
    String str1;
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println(this);
    }
}
```



## 2、字符串

字符串是最简单的配置，也是最常见的配置。再spring中，字符串可以代引号，也可以不带引号。所以下面三行的配置效果，是一样的

```yaml
str1: ksdfjsdlkfjdsf skdfljs
str1: 'ksdfjsdlkfjdsf skdfljs'
str1: "ksdfjsdlkfjdsf skdfljs"
```

那么，如何支持多行文本呢？毕竟有些需求，就是这么作死。写法如下：

```yaml
str1: |
  ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs
```

注意，后面不需要有其他的画蛇添足的结束表示，一切都是靠缩进来证明的。当然，你也可以把 `|`换成`>`，效果是一样的。

```yaml
str1: >
  ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs
```

要命的是，它还有第三种写法。

```yaml
str1: "ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs
  ksdfjsdlkfjdsf skdfljs"
```



## 3、 数字

当我们的接收者，是一个数字的时候，比如下面这个。

```java
@Value("${a}")
int a ;
```

那么，你即使把配置文件写成了字符串，它也会强制转成数字。

```java
a: "014"
```

神奇的是，如果你把引号去掉，也就是下面这样。

```java
a: 014
```

此时，a的数值，竟然变成了12！

我就曾碰到过这样的极品bug，浪费了不少脑细胞，wtf。因为以0开头，代表的是八进制，解析器中间做了一层转换。所以，按照这个逻辑，0x14就是20，使用时一定要注意这一点。机灵的同学可以拿来埋坑哦。

这里也有一些特殊的写法。

```yaml
float: 1.23e+3     # 浮点数
fixed: 13.67       # 固定小数
minmin: -.inf      # 表示负无穷
notNumber: .NaN    # 无效数字

boolean: [true, false] # 布尔值
string: '12345'    # 字符串

date: 2021-06-03   # 日期
```



## 4、 字典

再来看一下常见的字典。其实，把所有的配置罗列开来，本身就是一个字典，也就是kv配置。

它是以`:`进行分割的，所以左半部分要求不能有特殊字符，否则就晕菜了。不不不，它没有晕菜，因为它把乱七八糟的字符，正确的识别了出来。比如下面的yml配置。

```yaml
a&& xk@71: 0x14
```

这样的代码接收。

```java
@Value("${a&& xk@71}")
int a ;
```

嗯，容易被打死的写法。所以，你懂的。还是我太幼稚了，yml文件根本就没规定key不允许有特殊字符，它允许你这么做。



## 5、对象

由字典，很容易可以扩展到对象。因为对象，也是一堆属性的集合。json已经证明，这些属性，就是一堆KV，我们的yaml也是如此。

假设有如下的代码，我们需要构造dog中的数据。

```java
@Data
public static class Dog{
    private String xjjdog1;
    private String xjjdog2;
}

@Bean
@ConfigurationProperties(prefix = "dog")
public Dog getDog(){
    return new Dog();
}
```

第一种yml的写法，是这样。

```yaml
dog:
  xjjdog1: i am xjjdog1
  xjjdog2: i am xjjdog1++
```

而另一种方式，是把json数据直接给写到文件里。

```yaml
dog: {xjjdog1: 'i am xjjdog1',xjjdog2: 'i am xjjdog++'}
```

当然，多个层次，可以在一行之中平铺开。比如prefix是super.dog，那么yml文件就可以这么写。

```yaml
super.dog: {xjjdog1: 'i am xjjdog1',xjjdog2: 'i am xjjdog++'}
```



## 6、列表支持

列表，就是list，我们可以使用数组接收，也可以使用List等。

它也有两种写法。这是最常见的一种。

```yaml
animal:
  - dog
  - cat
  - monkey
```

当然，也可以放在一行。

```yaml
animal: [dog,cat,monkey]
```

这没什么问题，关键是yml文件支持嵌套。比如List里嵌套Map，或者Map里嵌套List。当嵌套层次比较深的时候，或者缩进没什么规律的时候，就显得非常的乱。

比如下面这个k8s的pod配置。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: xjjdog-Pod
  labels:
    app: front-web
spec:
  containers:
    - name: front-web
      image: nginx
      ports:
        - containerPort: 80
    - name: front-app
      image: xjjdog/frontapp
      ports:
        - containerPort: 14000
  storages:
  # ...
```

比较复杂的是spec，里面有`containers`、`storages`等配置。其中containers是一个列表，列表之间是一个map，map中其中的ports属性，又是一个列表...如此嵌套，如果配置文件比较长的化，不熟悉业务属性的同学就会容易晕菜。



## 7、特殊数据

即使是这样，yaml也比xml简单的多。它也有很多特殊的写法。比如这个。

```yaml
str1: !!str 2021-06-03
```

它的意思是，把2021-06-04，强制转化成字符串。这样的强制转化有很多，但大多数时候你不会用。但如果你想要把你的yaml文件变得复杂，让别人不敢动，那就可以这么做。

```yaml
 !!int               # 整数类型
 !!float             # 浮点类型
 !!bool              # 布尔类型
 !!str               # 字符串类型
 !!binary            # 也是字符串类型
 !!timestamp         # 日期时间类型
 !!null              # 空值
 !!set               # 集合
 !!omap, !!pairs     # 键值列表或对象列表
 !!seq               # 序列，也是列表
 !!map               # 键值表
```

既然yml文件有这么多复杂的写法，那么我们就可以去玩一把。比如下面的写法。

```yaml
from: &d !!str 2021-06-04
str1: *d
```

这个配置，和上面的配置，效果是一样的，`&`的意思是标记，我们给它起了个名字，叫做`d`；`*`的意思是引用，我们在需要它的地方引用一把就可以了。

yml中的key，竟然也可以用对象或者复杂的结构作为key。为了标识是一个特殊的key，我们还要做一点处理。

```yaml
?[blue, reg, green]: Color
```

上面这个配置的`?`，就是说，我下面要进行一个比较复杂的配置了，你准备好了么？



## 7、YML总结

学会了这些招数的你，是不是跃跃欲试了？想要在你的SpringBoot项目里搞一点有意思的东西？为了让你的基础架构部门无法扫描出你的配置，为什么不呢？

这是我改造的一个普通datasource的配置文件。

```yaml
h2: &sa !!str sa
driver: &driver !!str org.h2.Driver
defaults: &defaults
  ?username: *sa
  ?password:
  ?driverClassName: *driver
spring:
  datasource:
    <<: *defaults
    ?url: !!str >
      jdbc:h2:mem:h2test;
      DB_CLOSE_DELAY=-1;
      DB_CLOSE_ON_EXIT=FALSE
```