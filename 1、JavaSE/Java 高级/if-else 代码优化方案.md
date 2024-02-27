# 优化 if-else 代码的八种方案

前言：代码中如果if-else比较多，阅读起来比较困难，维护起来也比较困难，很容易出bug，本文将介绍优化if-else代码的八种方案。

## 优化一：提前 return，去除不必要的 else

**优化前：**

```java
if(condition){
    // 代码
}else{
    return ;
}
```

**优化后：**

如果if-else代码块包含return语句，可以考虑通过提前return，把多余else干掉，使代码更加优雅。

```java
if（!condition）{
    return ;
}
// 代码
```



## 优化二：使用条件三目运算符

**优化前：**

```java
int  price;
if(condition){
    price = 80;
}else{
    price = 100;
}
```

**优化后：**

使用条件三目运算符可以简化某些if-else,使代码更加简洁，更具有可读性

```java
int price = condition ? 80 : 100;
```



## 优化三：使用枚举

**优化前：**

```java
String OrderStatusDes;
if (orderStatus==0) {
    OrderStatusDes ="订单未支付";
} else if (OrderStatus == 1) {
    OrderStatusDes ="订单已支付";
} else if(OrderStatus == 2) {
    OrderStatusDes ="已发货";
}
...
```

**优化后：**

使用条件三目运算符可以简化某些if-else,使代码更加简洁，更具有可读性

```java
public enum OrderStatusEnum {
    UN_PAID(0, "订单未支付"),
    PAIDED(1, "订单已支付"),
    SENDED(2, "已发货");
    private int index;
    private String desc;

    public int getIndex() {
        return index;
    }

    public String getDesc() {
        return desc;
    }

    OrderStatusEnum(int index, String desc) {
        this.index = index;
        this.desc = desc;
    }

    static OrderStatusEnum of(int orderStatus) {
        for (OrderStatusEnum temp : OrderStatusEnum.values()) {
            if (temp.getIndex() == orderStatus) {
                return temp;
            }
        }
        return null;
    }
}
```

有了枚举之后，以上if-else逻辑分支，可以优化为一行代码

```java
String OrderStatusDes = OrderStatusEnum.of(1).getDesc();
```



## 优化四：合并条件表达式

**优化前：**

```java
double getVipDiscount() {
    if(age<18){
        return 0.8;
    }
    if("深圳".equals(city)){
        return 0.8;
    }
    if(isStudent){
        return 0.8;
    }
    // .....
}
```

**优化后：**

> 如果有一系列条件返回一样的结果，可以将它们合并为一个条件表达式，让逻辑更加清晰

```java
double getVipDiscount(){
    if(age<18|| "深圳".equals(city)||isStudent){
        return 0.8;
    }
    //  ......
}
```





## 优化五：使用 Optional

有时候if-else比较多，是因为非空判断导致的，这时候你可以使用java8的Optional进行优化。

**优化前：**

```java
String str = "jay@huaxiao";
if (str != null) {
    System.out.println(str);
} else {
    System.out.println("Null");
}
```

**优化后：**

```java
Optional<String> strOptional = Optional.of("jay@huaxiao");
strOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Null"));
```



## 优化六：表驱动法

**表驱动法**，又称之为表驱动、表驱动方法。表驱动方法是一种使你可以在表中查找信息，而不必用很多的逻辑语句（if或case）来把它们找出来的方法。以下的demo，把map抽象成表，在map中查找信息，而省去不必要的逻辑语句。

**优化前：**

```java
if (param.equals(value1)) {
    doAction1(someParams);
} else if (param.equals(value2)) {
    doAction2(someParams);
} else if (param.equals(value3)) {
    doAction3(someParams);
}
// ...
```

**优化后：**

```java
Map<?, Function<?> action> actionMappings = new HashMap<>(); // 这里泛型 ? 是为方便演示，实际可替换为你需要的类型
// 初始化
actionMappings.put(value1, (someParams) -> { doAction1(someParams)});
actionMappings.put(value2, (someParams) -> { doAction2(someParams)});
actionMappings.put(value3, (someParams) -> { doAction3(someParams)});
// 省略多余逻辑语句
actionMappings.get(param).apply(someParams);
```



## 优化七：优化逻辑结构，让正常流程走主干

**优化前：**

```java
public double getAdjustedCapital(double a, double b) {
    if (a <= 0.0) {
        return 0.0;
    }
    if (a > 0 && b > 0) {
        return (a / b) * 100;
    }
    return 0.0;
}
```

**优化后：**

将条件反转使异常情况先退出，让正常流程维持在主干流程，可以让代码结构更加清晰

```java
public double getAdjustedCapital(double a, double b) {
    if (a <= 0.0) {
        return 0.0;
    }
    if (a <= 0 && b <= 0) {
        return 0.0;
    }
    return (a / b) * 100;
}
```



## 优化八：策略模式 + 工厂方法消除

**优化前：**

```java
String medalType = "vip0";
if ("vip0".equals(medalType)) {
    System.out.println("VIP0");
} else if ("vip1".equals(medalType)) {
    System.out.println("VIP1");
} else if ("vip2".equals(medalType)) {
    System.out.println("VIP2");
}
```

**优化后：**

首先，我们把每个条件逻辑代码块，抽象成一个公共的接口，可以得出以下代码：

```java
// 接口
public interface IMedalService {
    void showMedal();
}
```

我们根据每个逻辑条件，定义相对应的策略实现类，可得以下代码：

```java
public class Vip0MedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("VIP0");
    }
}
public class Vip1MedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("VIP1");
    }
}
public class Vip2MedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("VIP2");
    }
}
```

接下来，我们再定义策略工厂类，用来管理这些VIP实现策略类，如下：

```java
// 服务工产类
public class MedalServicesFactory {
    private static final Map<String, IMedalService> map = new HashMap<>();
    static {
        map.put("vip0", new Vip0MedalServiceImpl());
        map.put("vip1", new Vip1MedalServiceImpl());
        map.put("vip2", new Vip2MedalServiceImpl());
    }
    public static IMedalService getMedalService(String medalType) {
        return map.get(medalType);
    }
}
```

使用了策略 + 工厂模式之后，代码变得简洁多了，如下：

```java
public class Test {
    public static void main(String[] args) {
        String medalType = "vip0";
        IMedalService medalService = MedalServicesFactory.getMedalService(medalType);
        medalService.showMedal();
    }
}
```