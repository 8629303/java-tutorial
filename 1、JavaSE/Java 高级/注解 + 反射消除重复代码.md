# 利用注解 + 反射消除重复代码

> 3 个绝招 解决 代码重复，你会么？https://mp.weixin.qq.com/s/X5reBiAWGwsqS4yW5eewzA

假设银行提供了一些 API 接口，对参数的序列化有点特殊，不使用 JSON，而是需要我们把参数依次拼在一起构成一个大字符串。

- 按照银行提供的 API 文档的顺序，把所有参数构成定长的数据，然后拼接在一起作为整个字符串。

- 因为每一种参数都有固定长度，未达到长度时需要做填充处理：

- - 字符串类型的参数不满长度部分需要以下划线右填充，也就是字符串内容靠左；
  - 数字类型的参数不满长度部分以 0 左填充，也就是实际数字靠右；
  - 货币类型的表示需要把金额向下舍入 2 位到分，以分为单位，作为数字类型同样进行左填充。

- 对所有参数做 MD5 操作作为签名（为了方便理解，Demo 中不涉及加盐处理）。

比如，创建用户方法和支付方法的定义是这样的：

- 创建用户接口：http://baseUrl/reflection/bank/createUser

  | 参数名称 | 参数顺序 | 参数类型    | 参数长度 |
  | -------- | -------- | ----------- | -------- |
  | 姓名     | 1        | 字符串（S） | 10       |
  | 身份证   | 2        | 字符串（S） | 18       |
  | 年龄     | 3        | 数字（N）   | 5        |
  | 手机号   | 4        | 字符串（S） | 11       |

代码很容易实现，直接根据接口定义实现填充操作、加签名、请求调用操作即可：

```java
public class BankService {

    // 创建用户方法
    public static String createUser(String name, String identity, String mobile, int age) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        // 字符串靠左，多余的地方填充_
        stringBuilder.append(String.format("%-10s", name).replace(' ', '_'));
        // 字符串靠左，多余的地方填充_
        stringBuilder.append(String.format("%-18s", identity).replace(' ', '_'));
        // 数字靠右，多余的地方用0填充
        stringBuilder.append(String.format("%05d", age));
        // 字符串靠左，多余的地方用_填充
        stringBuilder.append(String.format("%-11s", mobile).replace(' ', '_'));
        // 最后加上MD5作为签名
        stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
        return Request.Post("http://localhost:45678/reflection/bank/createUser")
                .bodyString(stringBuilder.toString(), ContentType.APPLICATION_JSON)
                .execute().returnContent().asString();
    }
    
    // 支付方法
    public static String pay(long userId, BigDecimal amount) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        // 数字靠右，多余的地方用0填充
        stringBuilder.append(String.format("%020d", userId));
        // 金额向下舍入2位到分，以分为单位，作为数字靠右，多余的地方用0填充
        stringBuilder.append(String.format("%010d", amount.setScale(2, RoundingMode.DOWN).multiply(new BigDecimal("100")).longValue()));
        // 最后加上MD5作为签名
        stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
        return Request.Post("http://localhost:45678/reflection/bank/pay")
                .bodyString(stringBuilder.toString(), ContentType.APPLICATION_JSON)
                .execute().returnContent().asString();
    }
}
```

可以看到，这段代码的重复粒度更细：

- 三种标准数据类型的处理逻辑有重复，稍有不慎就会出现 Bug；
- 处理流程中字符串拼接、加签和发请求的逻辑，在所有方法重复；
- 实际方法的入参的参数类型和顺序，不一定和接口要求一致，容易出错；
- 代码层面针对每一个参数硬编码，无法清晰地进行核对，如果参数达到几十个、上百个，出错的概率极大。

那应该如何改造这段代码呢？没错，就是要用注解和反射！

使用注解和反射这两个武器，就可以针对银行请求的所有逻辑均使用一套代码实现，不会出现任何重复。

要实现接口逻辑和逻辑实现的剥离，首先需要以 POJO 类（只有属性没有任何业务逻辑的数据类）的方式定义所有的接口参数。比如，下面这个创建用户 API 的参数：

```java
@Data
public class CreateUserAPI {
    private String name;
    private String identity;
    private String mobile;
    private int age;
}
```

有了接口参数定义，我们就能通过自定义注解为接口和所有参数增加一些元数据。如下所示，我们定义一个接口 API 的注解 BankAPI，包含接口 URL 地址和接口说明：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Inherited
public @interface BankAPI {
    String desc() default "";
    String url() default "";
}
```

然后，我们再定义一个自定义注解 @BankAPIField，用于描述接口的每一个字段规范，包含参数的次序、类型和长度三个属性：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
@Inherited
public @interface BankAPIField {
    int order() default -1;
    int length() default -1;
    String type() default "";
}
```

接下来，注解就可以发挥威力了。

如下所示，我们定义了 CreateUserAPI 类描述创建用户接口的信息，通过为接口增加 @BankAPI 注解，来补充接口的 URL 和描述等元数据；通过为每一个字段增加 @BankAPIField 注解，来补充参数的顺序、类型和长度等元数据：

```java
@BankAPI(url = "/bank/createUser", desc = "创建用户接口")
@Data
public class CreateUserAPI extends AbstractAPI {
    @BankAPIField(order = 1, type = "S", length = 10)
    private String name;
    @BankAPIField(order = 2, type = "S", length = 18)
    private String identity;
    @BankAPIField(order = 4, type = "S", length = 11) // 注意这里的order需要按照API表格中的顺序
    private String mobile;
    @BankAPIField(order = 3, type = "N", length = 5)
    private int age;
}
```

另一个 PayAPI 类也是类似的实现：

```java
@BankAPI(url = "/bank/pay", desc = "支付接口")
@Data
public class PayAPI extends AbstractAPI {
    @BankAPIField(order = 1, type = "N", length = 20)
    private long userId;
    @BankAPIField(order = 2, type = "M", length = 10)
    private BigDecimal amount;
}
```

这 2 个类继承的 AbstractAPI 类是一个空实现，因为这个案例中的接口并没有公共数据可以抽象放到基类。

通过这 2 个类，我们可以在几秒钟内完成和 API 清单表格的核对。理论上，如果我们的核心翻译过程（也就是把注解和接口 API 序列化为请求需要的字符串的过程）没问题，只要注解和表格一致，API 请求的翻译就不会有任何问题。

以上，我们通过注解实现了对 API 参数的描述。接下来，我们再看看反射如何配合注解实现动态的接口参数组装：

- 第 3 行代码中，我们从类上获得了 BankAPI 注解，然后拿到其 URL 属性，后续进行远程调用。
- 第 6~9 行代码，使用 stream 快速实现了获取类中所有带 BankAPIField 注解的字段，并把字段按 order 属性排序，然后设置私有字段反射可访问。
- 第 12~38 行代码，实现了反射获取注解的值，然后根据 BankAPIField 拿到的参数类型，按照三种标准进行格式化，将所有参数的格式化逻辑集中在了这一处。
- 第 41~48 行代码，实现了参数加签和请求调用。

```java
private static String remoteCall(AbstractAPI api) throws IOException {
    // 从BankAPI注解获取请求地址
    BankAPI bankAPI = api.getClass().getAnnotation(BankAPI.class);
    bankAPI.url();
    StringBuilder stringBuilder = new StringBuilder();
    Arrays.stream(api.getClass().getDeclaredFields()) // 获得所有字段
            .filter(field -> field.isAnnotationPresent(BankAPIField.class)) // 查找标记了注解的字段
            .sorted(Comparator.comparingInt(a -> a.getAnnotation(BankAPIField.class).order())) // 根据注解中的order对字段排序
            .peek(field -> field.setAccessible(true)) // 设置可以访问私有字段
            .forEach(field -> {
                // 获得注解
                BankAPIField bankAPIField = field.getAnnotation(BankAPIField.class);
                Object value = "";
                try {
                    // 反射获取字段值
                    value = field.get(api);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                // 根据字段类型以正确的填充方式格式化字符串
                switch (bankAPIField.type()) {
                    case "S": {
                        stringBuilder.append(String.format("%-" + bankAPIField.length() + "s", value.toString()).replace(' ', '_'));
                        break;
                    }
                    case "N": {
                        stringBuilder.append(String.format("%" + bankAPIField.length() + "s", value.toString()).replace(' ', '0'));
                        break;
                    }
                    case "M": {
                        if (!(value instanceof BigDecimal))
                            throw new RuntimeException(String.format("{} 的 {} 必须是BigDecimal", api, field));
                        stringBuilder.append(String.format("%0" + bankAPIField.length() + "d", ((BigDecimal) value).setScale(2, RoundingMode.DOWN).multiply(new BigDecimal("100")).longValue()));
                        break;
                    }
                    default:
                        break;
                }
            });
    // 签名逻辑
   stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
    String param = stringBuilder.toString();
    long begin = System.currentTimeMillis();
    // 发请求
    String result = Request.Post("http://localhost:45678/reflection" + bankAPI.url())
            .bodyString(param, ContentType.APPLICATION_JSON)
            .execute().returnContent().asString();
    log.info("调用银行API {} url:{} 参数:{} 耗时:{}ms", bankAPI.desc(), bankAPI.url(), param, System.currentTimeMillis() - begin);
    return result;
}
```

可以看到，所有处理参数排序、填充、加签、请求调用的核心逻辑，都汇聚在了 remoteCall 方法中。有了这个核心方法，BankService 中每一个接口的实现就非常简单了，只是参数的组装，然后调用 remoteCall 即可。

```java
// 创建用户方法
public static String createUser(String name, String identity, String mobile, int age) throws IOException {
    CreateUserAPI createUserAPI = new CreateUserAPI();
    createUserAPI.setName(name);
    createUserAPI.setIdentity(identity);
    createUserAPI.setAge(age);
    createUserAPI.setMobile(mobile);
    return remoteCall(createUserAPI);
}
// 支付方法
public static String pay(long userId, BigDecimal amount) throws IOException {
    PayAPI payAPI = new PayAPI();
    payAPI.setUserId(userId);
    payAPI.setAmount(amount);
    return remoteCall(payAPI);
}
```

其实，**许多涉及类结构性的通用处理，都可以按照这个模式来减少重复代码**。

反射给予了我们在不知晓类结构的时候，按照固定的逻辑处理类的成员；而注解给了我们为这些成员补充元数据的能力，使得我们利用反射实现通用逻辑的时候，可以从外部获得更多我们关心的数据。