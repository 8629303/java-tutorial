> 1. 作者：三分恶；来源：Jackson用法详解：https://fighter3.blog.csdn.net/article/details/106207324
> 2. 作者：进击的阿晨；来源：Jackson注解大全：https://blog.csdn.net/weixin_44610216/article/details/118978414
> 3. 作者：萧明；来源：SpringBoot中使用Jackson总结：https://blog.csdn.net/u010192145/article/details/115231894
> 4. 来源：Spring Boot 3.0 日期时间处理总结：https://blog.csdn.net/qq_44866828/article/details/125073769

# 一、Jackson 简介

Jackson 是当前用的比较广泛的，用来序列化和反序列化 json 的 Java 的开源框架。Jackson 社区相对比较活跃，更新速度也比较快， 从 Github 中的统计来看，Jackson 是最流行的 json 解析器之一 。 Spring MVC 的默认 json 解析器便是 Jackson。 Jackson 优点很多。Jackson 所依赖的 jar 包较少 ，简单易用。与其他 Java 的 json 的框架 Gson 等相比， Jackson 解析大的 json 文件速度比较快；Jackson 运行时占用内存比较低，性能比较好；Jackson 有灵活的 API，可以很容易进行扩展和定制。

Jackson 的 1.x 版本的包名是 org.codehaus.jackson ，当升级到 2.x 版本时，包名变为 com.fasterxml.jackson。

Jackson 的核心模块由三部分组成。

- jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
- jackson-annotations，注解包，提供标准注解功能；
- jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

得益于 Jackson 高扩展性的设计，有很多常见的文本格式以及工具都有对 Jackson 的相应适配，如 CSV、XML、YAML 等。

源码地址：FasterXML/jackson：https://github.com/FasterXML/jackson



# 二、Jackson 依赖

Jackson 有三个核包，分别是 Streaming、Databid、Annotations，通过这些包可以方便的对 JSON 进行操作。

- Streaming[1] 在 jackson-core 模块。定义了一些流处理相关的 API 以及特定的 JSON 实现
- Annotations[2] 在 jackson-annotations 模块，包含了 Jackson 中的注解
- Databind[3] 在 jackson-databind 模块， 在 Streaming 包的基础上实现了数据绑定，依赖于 Streaming 和 Annotations 包

使用Maven构建项目，需要添加依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.13.3</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.13.3</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

当然了，jackson-databind 依赖 jackson-core 和 jackson-annotations，所以可以只显示地添加jackson-databind依赖，jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

为了方便这篇文章后续的代码演示，我们同时引入 Junit 进行单元测试和 Lombok 以减少 Get/Set 的代码编写。

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.2</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
</dependency>
```



# 三、ObjectMapper

Jackson 最常用的 API 就是基于"对象绑定" 的 ObjectMapper：

- ObjectMapper可以从字符串，流或文件中解析JSON，并创建表示已解析的JSON的Java对象。 将JSON解析为Java对象也称为从JSON反序列化Java对象。

- ObjectMapper也可以从Java对象创建JSON。 从Java对象生成JSON也称为将Java对象序列化为JSON。

- Object映射器可以将JSON解析为自定义的类的对象，也可以解析置JSON树模型的对象。


之所以称为ObjectMapper是因为它将JSON映射到Java对象（反序列化），或者将Java对象映射到JSON（序列化）。



## 1、从JSON中获取Java对象（反序列化）

### 1、Jackson 快速入门示例

将Json转换为Car类对象：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;

@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
        ObjectMapper objectMapper = new ObjectMapper();
        
        Car car = objectMapper.readValue(carJson, Car.class);
        System.out.println("car brand = " + car.getBrand());
        System.out.println("car doors = " + car.getDoors());
    }
}
```

打印结果如下：

```
car brand = Mercedes
car doors = 5
```



### 2、JSON字段和Java属性如何匹配

> ObjectMapper如何匹配JSON对象的字段和Java对象的属性：

默认情况下，Jackson通过将JSON字段的名称与Java对象中的getter和setter方法进行匹配，将JSON对象的字段映射到Java对象中的属性。 Jackson删除了getter和setter方法名称的“ get”和“ set”部分，并将其余名称的第一个字符转换为小写。

例如，名为brand的JSON字段与名为getBrand()和setBrand()的Java getter和setter方法匹配。 名为engineNumber的JSON字段将与名为getEngineNumber()和setEngineNumber()的getter和setter匹配。

如果需要以其他方式将JSON对象字段与Java对象字段匹配，则需要使用自定义序列化器和反序列化器，或者使用一些Jackson注解。



### 3、JSON 字符串 »» Java 对象

从JSON字符串读取Java对象非常容易。 上面已经有了一个示例：JSON字符串作为第一个参数传递给ObjectMapper的readValue()方法。 

```java
ObjectMapper objectMapper = new ObjectMapper();
String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
Car car = objectMapper.readValue(carJson, Car.class);
System.out.println(car);
// 输出内容: Car(brand=Mercedes, doors=5)
```



### 4、JSON 字符输入流 »» Java 对象

还可以从通过Reader实例加载的JSON中读取对象。示例如下：

```java
ObjectMapper objectMapper = new ObjectMapper();
String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 4 }";
Reader reader = new StringReader(carJson);

Car car = objectMapper.readValue(reader, Car.class);
System.out.println(car);
// 输出内容: Car(brand=Mercedes, doors=4)
```



### 5、JSON 字节输入流 »» Java 对象

也可以使用ObjectMapper通过InputStream从JSON读取对象。 这是一个从InputStream读取JSON的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
// 文件内容为: { "brand" : "Mercedes", "doors" : 4 }
InputStream input = new FileInputStream("data/car.json");

Car car = objectMapper.readValue(input, Car.class);
System.out.println(car);
// 输出结果:  Car(brand=Mercedes, doors=4)
```



### 6、JSON 文件 »» Java 对象

从文件读取JSON当然可以通过FileReader（而不是StringReader）来完成，也可以通过File对象来完成。 这是从文件读取JSON的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
// 文件内容为: { "brand" : "Mercedes", "doors" : 4 }
File file = new File("D:\\car.json");

Car car = objectMapper.readValue(file, Car.class);
System.out.println(car);
// 输出结果:  Car(brand=Mercedes, doors=4)
```



### 7、JSON via URL »» Java 对象

可以通过URL（java.net.URL）从JSON读取对象，如下所示：

```java
ObjectMapper objectMapper = new ObjectMapper();
// 文件内容为: { "brand" : "Mercedes", "doors" : 4 }
URL url = new URL("file:data/car.json");

Car car = objectMapper.readValue(url, Car.class);
System.out.println(car);
// 输出结果:  Car(brand=Mercedes, doors=4)
```

示例使用文件URL，也可以使用HTTP URL（类似于：http://jenkov.com/some-data.json）



### 8、JSON 数组 »» Java 对象数组

Jackson ObjectMapper也可以从JSON数组字符串读取对象数组。 这是从JSON数组字符串读取对象数组的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";

Car[] cars2 = objectMapper.readValue(jsonArray, Car[].class);
System.out.println(cars2[0] + "、" + cars2[1]);
// 输出结果:  Car(brand=ford, doors=0)、Car(brand=Fiat, doors=0)
```

需要将Car数组类作为第二个参数传递给readValue()方法。读取对象数组还可以与字符串以外的其他JSON源一起使用。 例如，文件，URL，InputStream，Reader等。



### 9、JSON 数组 »» List 对象

Jackson ObjectMapper还可以从JSON数组字符串读取对象的Java List。 这是从JSON数组字符串读取对象列表的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";

List<Car> cars1 = objectMapper.readValue(jsonArray, new TypeReference<List<Car>>(){});
System.out.println(cars1);
// 输出结果: [Car(brand=ford, doors=0), Car(brand=Fiat, doors=0)]
```


### 10、JSON 字符串 »» Map 对象

Jackson ObjectMapper还可以从JSON字符串读取Java Map。 如果事先不知道将要解析的确切JSON结构，这种方法是很有用的。 通常，会将JSON对象读入Java Map。 JSON对象中的每个字段都将成为Java Map中的键，值对。

这是一个使用Jackson ObjectMapper从JSON字符串读取Java Map的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
String jsonObject = "{\"brand\":\"ford\", \"doors\":5}";

Map<String, Object> jsonMap = objectMapper.readValue(jsonObject, new TypeReference<Map<String, Object>>(){});
System.out.println(jsonMap);
// 输出结果: {brand=ford, doors=5}
```



### 11、忽略未知的JSON字段

有时候，与要从JSON读取的Java对象相比，JSON中的字段更多。 默认情况下，Jackson在这种情况下会抛出异常，报不知道XYZ字段异常，因为在Java对象中找不到该字段。

但是，有时应该允许JSON中的字段多于相应的Java对象中的字段。 例如，要从REST服务解析JSON，而该REST服务包含的数据远远超出所需的。 在这种情况下，可以使用Jackson配置忽略这些额外的字段。 以下是配置Jackson ObjectMapper忽略未知字段的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```

错误示例：

```java
@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5, \"name\" : \"Sam\"}";
        
        Car car = objectMapper.readValue(carJson, Car.class);
        System.out.println(car);
    }
}
```

打印内容如下（报错）：

```
Exception in thread "main" com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "name" (class Car), not marked as ignorable (2 known properties: "doors", "brand"])
 at [Source: (String)"{ "brand" : "Mercedes", "doors" : 5, "name" : "Sam"}"; line: 1, column: 48] (through reference chain: Car["name"])
....
```

正确示例：

```java
@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5, \"name\" : \"Sam\"}";
        Car car = objectMapper.readValue(carJson, Car.class);
        System.out.println(car);
        // 输出结果: Car(brand=Mercedes, doors=5)
    }
}
```



### 12、不允许基本类型为null

如果JSON字符串包含其值设置为null的字段（对于在相应的Java对象中是基本数据类型（int，long，float，double等）的字段），Jackson ObjectMapper默认会处理基本数据类型为null的情况，我们可以可以将Jackson ObjectMapper默认配置为失效，这样基本数据为null就会转换失败。 例如以下Car类：

现在，假设有一个与Car对象相对应的JSON字符串，如下所示：

```json
{ "brand":"Toyota", "doors":null }
```

请注意，doors字段值为null。 Java中的基本数据类型不能为null值。 默认情况下，Jackson ObjectMapper会忽略原始字段的空值。 但是，可以将Jackson ObjectMapper配置设置为失败。

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
```

在FAIL_ON_NULL_FOR_PRIMITIVES配置值设置为true的情况下，尝试将空JSON字段解析为基本类型Java字段时会遇到异常。 这是一个Java Jackson ObjectMapper示例，该示例将失败，因为JSON字段包含原始Java字段的空值：

```java
@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);

        String carJson = "{ \"brand\":\"Toyota\", \"doors\":null }";
        Car car = objectMapper.readValue(carJson, Car.class);
        System.out.println(car);
    }
}
```

打印结果如下（报错）：

```
Exception in thread "main" com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot map `null` into type `int` (set DeserializationConfig.DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES to 'false' to allow)
 at [Source: (String)"{ "brand":"Toyota", "doors":null }"; line: 1, column: 29] (through reference chain: Car["doors"])
```



### 13、自定义反序列化

有时，可能希望以不同于Jackson ObjectMapper缺省方式的方式将JSON字符串读入Java对象。 可以将自定义反序列化器添加到ObjectMapper，可以按需要执行反序列化。

这是在Jackson的ObjectMapper中注册和使用自定义反序列化器的方式：

```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;
import com.fasterxml.jackson.core.Version;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;
import com.fasterxml.jackson.databind.module.SimpleModule;
import lombok.Data;

@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

/**
 * 自定义反序列化器CarDeserializer类：
 */
class CarDeserializer extends StdDeserializer<Car> {

    public CarDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Car deserialize(JsonParser parser, DeserializationContext deserializer) throws Exception {
        Car car = new Car();
        while (!parser.isClosed()) {
            JsonToken jsonToken = parser.nextToken();

            if (JsonToken.FIELD_NAME.equals(jsonToken)) {
                String fieldName = parser.getCurrentName();
                System.out.println(fieldName);

                jsonToken = parser.nextToken();

                if ("brand".equals(fieldName)) {
                    car.setBrand(parser.getValueAsString());
                } else if ("doors".equals(fieldName)) {
                    car.setDoors(parser.getValueAsInt());
                }
            }
        }
        return car;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{ \"brand\" : \"Ford\", \"doors\" : 6 }";

        SimpleModule module = new SimpleModule("CarDeserializer", new Version(3, 1, 8, null, null, null));
        module.addDeserializer(Car.class, new CarDeserializer(Car.class));

        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(module);

        Car car = mapper.readValue(json, Car.class);
        System.out.println(car);
    }
}
```

打印结果如下：

```
brand
doors
Car(brand=Ford, doors=6)
```



## 2、将Java对象写入JSON中（序列化）

### 1、Java对象 »» JSON

Jackson ObjectMapper也可以用于从对象生成JSON。 可以使用以下方法之一进行操作：

- writeValue(参数, obj)：直接将传入的对象序列化为json，并且返回给客户端
- writeValueAsString(obj)：从一个对象生成JSON，并将生成的JSON作为String返回给调用者
- writeValueAsBytes(obj)：从一个对象生成JSON，并将生成的JSON作为字节数组返回给调用者

三者的共同点：将将对象转为json字符串。

三者的不同点：writeValue（参数，obj）方法的参数有四种重载形式：

- 第一种：file 将转换后的json字符串保存到指定的file文件中
- 第二种：writer 将转换后的json字符串保存到字符输出流中
- 第三重：dataOutput 将转换后的json字符串保存到数据输出流中
- 第四种：outputStream 将转换后的json字符串保存到字节输出流中
- 第五种：jsonGenerator 类

这是一个从Car对象生成JSON的示例：


```java
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;
import com.fasterxml.jackson.databind.module.SimpleModule;
import lombok.Data;
import java.io.*;

@Data
class Car {
    private String brand = null;
    private int doors = 0;
}


public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();

        Car car = new Car();
        car.setBrand("BMW");
        car.setDoors(4);

        // 第一种：file 将转换后的json字符串保存到指定的file文件中
        objectMapper.writeValue(new File("D:\\car-1.json"), car);
        // 第二种：writer 将转换后的json字符串保存到字符输出流中
        objectMapper.writeValue(new FileWriter("D:\\car-2.json"), car);
        // 第三重：dataOutput 将转换后的json字符串保存到数据输出流中
        objectMapper.writeValue(new FileOutputStream("D:\\car-3.json"), car);
        // 第四种：outputStream 将转换后的json字符串保存到字节输出流中
        objectMapper.writeValue((OutputStream) new DataOutputStream(new FileOutputStream("D:\\car-4.json")), car);

        // 第五种：jsonGenerator 类
        JsonFactory factory = new JsonFactory();
        JsonGenerator generator = factory.createGenerator(new File("D:\\car-5.json"), JsonEncoding.UTF8);

        generator.writeStartObject();
        generator.writeStringField("brand", "Mercedes");
        generator.writeNumberField("doors", 5);
        generator.writeEndObject();

        generator.close();
    }
}
```

此示例首先创建一个ObjectMapper，然后创建一个Car实例，最后调用ObjectMapper的writeValue()方法，该方法将Car对象转换为JSON并将其写入给定的FileOutputStream。

ObjectMapper的writeValueAsString()和writeValueAsBytes()都从一个对象生成JSON，并将生成的JSON作为String或字节数组返回。 示例如下：

```java
ObjectMapper objectMapper = new ObjectMapper();

Car car = new Car();
car.setBrand("BMW");
car.setDoors(4);

String json = objectMapper.writeValueAsString(car);
System.out.println(json);
byte[] bytes = objectMapper.writeValueAsBytes(car);
System.out.println(new String(bytes));
// 输出内容: 
// {"brand":"BMW","doors":4}
// {"brand":"BMW","doors":4}
```



### 2、自定义序列化

有时，想要将Java对象序列化为JSON的方式与使用Jackson的默认方式不同。 例如，可能想要在JSON中使用与Java对象中不同的字段名称，或者希望完全省略某些字段。

Jackson可以在ObjectMapper上设置自定义序列化器。 该序列化器已为某个类注册，然后在每次要求ObjectMapper序列化Car对象时将调用该序列化器。

这是为Car类注册自定义序列化器的示例：

```java
import com.fasterxml.jackson.core.JsonEncoding;
import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.Version;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.StdSerializer;
import lombok.Data;
import java.io.*;

@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

/**
 * 自定义反序列化器CarDeserializer类：
 */
class CarSerializer extends StdSerializer<Car> {

    protected CarSerializer(Class<Car> t) {
        super(t);
    }

    public void serialize(Car car, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField("producer", car.getBrand());
        jsonGenerator.writeNumberField("doorCount", car.getDoors());
        jsonGenerator.writeEndObject();
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        CarSerializer carSerializer = new CarSerializer(Car.class);
        ObjectMapper objectMapper = new ObjectMapper();

        SimpleModule module = new SimpleModule("CarSerializer", new Version(2, 1, 3, null, null, null));
        module.addSerializer(Car.class, carSerializer);

        objectMapper.registerModule(module);

        Car car = new Car();
        car.setBrand("Mercedes");
        car.setDoors(5);

        String carJson = objectMapper.writeValueAsString(car);
        System.out.println(carJson);
        // 输出内容: {"producer":"Mercedes","doorCount":5}
    }
}
```



## 3、Jackson 日期转化

默认情况下，Jackson会将java.util.Date对象序列化为其long型的值，该值是自1970年1月1日以来的毫秒数。但是，Jackson还支持将日期格式化为字符串。



### 1、Date «» Long

默认的Jackson日期格式，该格式将Date序列化为自1970年1月1日以来的毫秒数（long类型）所以反序列化也必须要是毫秒数（long）

这是一个包含Date字段的Java类示例（Date 转 Long 以及 Long 转 Date）：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.text.SimpleDateFormat;
import java.util.Date;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
class Transaction {
    private String type = null;
    private Date date = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        /**
         * Date 序列化成 Long
         */
        ObjectMapper objectMapper = new ObjectMapper();
        String output = objectMapper.writeValueAsString(new Transaction("transfer", new Date()));
        System.out.println(output);
        // 输出内容: {"type":"transfer","date":1668744922635}

        /**
         * Long 反序列化成 Date
         */
        String json = "{ \"type\" : \"transfer\", \"date\" : 1668738063531 }";
        Transaction transaction = objectMapper.readValue(json, Transaction.class);
        System.out.println(transaction);
        // 输出内容: Transaction(type=transfer, date=Fri Nov 18 10:21:03 CST 2022)
    }
}
```

打印内容如下：

```json
{"type":"transfer","date":1668744922635}
Transaction(type=transfer, date=Fri Nov 18 10:21:03 CST 2022)
```



### 2、Date «» String

日期的long序列化格式不符合人类的时间查看格式。 因此，Jackson也支持文本日期格式。 可以通过在ObjectMapper上设置SimpleDateFormat来指定要使用的确切Jackson日期格式。 这是在Jackson的ObjectMapper上设置SimpleDateFormat的示例：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Transaction {
    private String type = null;
    private Date date = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        /**
         * Date 序列化成 String
         */
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
        String output = objectMapper.writeValueAsString(new Transaction("transfer", new Date()));
        System.out.println(output);
        // 输出内容: {"type":"transfer","date":"2022-11-18"}

        /**
         * String 反序列化成 Date
         * Long  反序列化成 Date
         */
        String json1 = "{ \"type\" : \"Dev\", \"date\" : \"2022-11-18\" }";
        Transaction transaction = objectMapper.readValue(json1, Transaction.class);
        System.out.println(transaction);
        // 输出内容: Transaction(type=Dev, date=Fri Nov 18 00:00:00 CST 2022)
        String json2 = "{ \"type\" : \"transfer\", \"date\" : 1668738063531 }";
        System.out.println(objectMapper.readValue(json2, Transaction.class));
        // 输出内容: Transaction(type=transfer, date=Fri Nov 18 10:21:03 CST 2022)
    }
}
```

打印结果如下：

```json
{"type":"transfer","date":"2022-11-18"}
Transaction(type=Dev, date=Fri Nov 18 00:00:00 CST 2022)
Transaction(type=transfer, date=Fri Nov 18 10:21:03 CST 2022)
```



### 3、LocalDateTime 类型

为什么没有尝试设置 LocalDateTime 类型的时间呢？默认情况下进行 LocalDateTime 类的 JSON 转换会遇到报错。错误示例如下：

```java
import com.fasterxml.jackson.databind.ObjectMapper;

import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.util.Date;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
class Transaction {
    private String type = null;
    private LocalDateTime localDateTime = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        /**
         * LocalDateTime 序列化
         */
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            String output = objectMapper.writeValueAsString(new Transaction("transfer", LocalDateTime.now()));
            System.out.println(output);
        } catch (Exception e) {
            e.printStackTrace();
        }

        /**
         * LocalDateTime 反序列化 
         */
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            String json = "{ \"type\" : \"transfer\", \"date\" : 1668738063531 }";
            Transaction transaction = objectMapper.readValue(json, Transaction.class);
            System.out.println(transaction);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

打印信息如下：

```shell
## 序列化报错
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Java 8 date/time type `java.time.LocalDateTime` not supported by default: add Module "com.fasterxml.jackson.datatype:jackson-datatype-jsr310" to enable handling (through reference chain: Transaction["localDateTime"])

## 反序列化报错
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "date" (class Transaction), not marked as ignorable (2 known properties: "type", "localDateTime"])
 at [Source: (String)"{ "type" : "transfer", "date" : 1668738063531 }"; line: 1, column: 46] (through reference chain: Transaction["date"])## 
```

这里我们需要添加相应的数据绑定支持包。添加依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.13.3</version>
</dependency>
```

然后在定义 ObjectMapper 时通过 findAndRegisterModules() 方法来注册依赖。这里需要注意：Jackson 默认序列化LocalDateTime返回是Integer数组：[2022,11,18,12,36,10,133708200]

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.util.Date;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
class Transaction {
    private String type = null;
    private LocalDateTime localDateTime = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        /**
         * LocalDateTime 序列化
         */
        try {
            ObjectMapper objectMapper = new ObjectMapper().findAndRegisterModules();
            String output = objectMapper.writeValueAsString(new Transaction("transfer", LocalDateTime.now()));
            System.out.println(output);
            // 输出内容: {"type":"transfer","localDateTime":[2022,11,18,12,36,10,133708200]}
        } catch (Exception e) {
            e.printStackTrace();
        }

        /**
         * LocalDateTime 反序列化
         */
        try {
            ObjectMapper objectMapper = new ObjectMapper().findAndRegisterModules();
            String json = "{\"type\":\"transfer\",\"localDateTime\":[2022,11,18,12,34,2,844805800]}";
            Transaction transaction = objectMapper.readValue(json, Transaction.class);
            System.out.println(transaction);
            // 输出内容: Transaction(type=transfer, localDateTime=2022-11-18T12:34:02.844805800)
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

打印内容如下，可以得到正常序列化与反序列化日志，不过序列化后的时间格式依旧奇怪。

```
{"type":"transfer","localDateTime":[2022,11,18,12,36,10,133708200]}
Transaction(type=transfer, localDateTime=2022-11-18T12:34:02.844805800)
```



### 4、时间格式化注解

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.time.LocalDateTime;
import java.util.Date;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
class Transaction {
    private String type = null;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Shanghai")
    private Date date = null;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Shanghai")
    private LocalDateTime localDateTime = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        /**
         * LocalDateTime、Date 序列化
         */
        ObjectMapper objectMapper = new ObjectMapper().findAndRegisterModules();
        String output = objectMapper.writeValueAsString(new Transaction("transfer", new Date(), LocalDateTime.now()));
        System.out.println(output);
        
        /**
         * LocalDateTime、Date 反序列化
         */
        objectMapper = new ObjectMapper().findAndRegisterModules();
        String json1 = "{\"type\":\"transfer\" , \"date\" : 1668738063531, \"localDateTime\":[2022,11,18,12,34,2]}";
        System.out.println(objectMapper.readValue(json1, Transaction.class));

        String json2 = "{\"type\":\"transfer\" , \"date\" : \"2022-11-18 12:42:45\", \"localDateTime\":[2022,11,18,12,34,2]}";
        System.out.println(objectMapper.readValue(json2, Transaction.class));

        String json3 = "{\"type\":\"transfer\" , \"date\" : \"2022-11-18 12:42:45\", \"localDateTime\":\"2022-11-18 12:34:02\"}";
        System.out.println(objectMapper.readValue(json3, Transaction.class));
    }
}
```

打印结果如下

```
{"type":"transfer","date":"2022-11-18 12:48:30","localDateTime":"2022-11-18 12:48:30"}
Transaction(type=transfer, date=Fri Nov 18 10:21:03 CST 2022, localDateTime=2022-11-18T12:34:02)
Transaction(type=transfer, date=Fri Nov 18 12:42:45 CST 2022, localDateTime=2022-11-18T12:34:02)
Transaction(type=transfer, date=Fri Nov 18 12:42:45 CST 2022, localDateTime=2022-11-18T12:34:02)
```



## 4、Jackson JSON 树模型

Jackson具有内置的树模型，可用于表示JSON对象。 如果不知道接收到的JSON的格式，或者由于某种原因而不能（或者只是不想）创建一个类来表示它，那么就要用到Jackson的树模型。 如果需要在使用或转化JSON之前对其进行操作，也需要被用到Jackson树模型。 所有这些情况在数据流场景中都很常见。

Jackson树模型由JsonNode类表示。 您可以使用Jackson ObjectMapper将JSON解析为JsonNode树模型，就像使用您自己的类一样。

以下将展示如何使用Jackson ObjectMapper读写JsonNode实例。

### 1、Jackson Tree Model 示例

下面是一个简单的例子：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;

@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);
            System.out.println(jsonNode);
            // 输出内容: {"brand":"Mercedes","doors":5}
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

只需将JsonNode.class作为第二个参数传递给readValue()方法，而不是本教程前面的示例中使用的Car.class，就可以将JSON字符串解析为JsonNode对象而不是Car对象。 。

ObjectMapper类还具有一个特殊的readTree()方法，该方法返回JsonNode。 这是使用ObjectMapper readTree()方法将JSON解析为JsonNode的示例：

```java
@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            JsonNode jsonNode = objectMapper.readTree(carJson);
            System.out.println(jsonNode);
            // 输出内容: {"brand":"Mercedes","doors":5}
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



### 2、Jackson JsonNode 类

通过JsonNode类，可以以非常灵活和动态的方式将JSON作为Java对象导航。这里了解一些如何使用它的基础知识。

将JSON解析为JsonNode（或JsonNode实例树）后，就可以浏览JsonNode树模型。 这是一个JsonNode示例，显示了如何访问JSON字段，数组和嵌套对象：

```json
{
    "brand": "Mercedes",
    "doors": 5,
    "owners": ["John", "Jack", "Jill"],
    "nestedObject": {
        "field": "value"
    }
}
```

```java
@Data
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5," +
                "  \"owners\" : [\"John\", \"Jack\", \"Jill\"]," +
                "  \"nestedObject\" : { \"field\" : \"value\" } }";

        ObjectMapper objectMapper = new ObjectMapper();
        try {
            JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);

            JsonNode brandNode = jsonNode.get("brand");
            String brand = brandNode.asText();
            System.out.println("brand = " + brand);

            JsonNode doorsNode = jsonNode.get("doors");
            int doors = doorsNode.asInt();
            System.out.println("doors = " + doors);

            JsonNode array = jsonNode.get("owners");
            JsonNode jsonArray = array.get(0);
            String john = jsonArray.asText();
            System.out.println("john  = " + john);

            JsonNode child = jsonNode.get("nestedObject");
            JsonNode childField = child.get("field");
            String field = childField.asText();
            System.out.println("field = " + field);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

打印内容如下

```
brand = Mercedes
doors = 5
john  = John
field = value
```

请注意，JSON字符串现在包含一个称为owners的数组字段和一个称为nestedObject的嵌套对象字段。

无论访问的是字段，数组还是嵌套对象，都可以使用JsonNode类的get()方法。 通过将字符串作为参数提供给get()方法，可以访问JsonNode的字段。 如果JsonNode表示数组，则需要将索引传递给get()方法。 索引指定要获取的数组元素。



### 3、Java 对象  »» JsonNode

可以使用Jackson ObjectMapper将Java对象转换为JsonNode，而JsonNode是转换后的Java对象的JSON表示形式。 可以通过Jackson ObjectMapper valueToTree()方法将Java对象转换为JsonNode。 这是一个使用ObjectMapper valueToTree()方法将Java对象转换为JsonNode的示例：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        Car car = new Car("Cadillac", 18);
        JsonNode carJsonNode = objectMapper.valueToTree(car);
        System.out.println(carJsonNode);
        // 输出内容: {"brand":"Cadillac","doors":18}
    }
}
```



### 4、JsonNode »» Java 对象

可以使用Jackson ObjectMapper treeToValue()方法将JsonNode转换为Java对象。 这类似于使用Jackson Jackson的ObjectMapper将JSON字符串（或其他来源）解析为Java对象。 唯一的区别是，JSON源是JsonNode。 这是一个使用Jackson ObjectMapper treeToValue()方法将JsonNode转换为Java对象的示例：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Car {
    private String brand = null;
    private int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
        ObjectMapper objectMapper = new ObjectMapper();

        // JSON字符串 转换 JsonNode对象
        JsonNode carJsonNode = objectMapper.readTree(carJson);
        System.out.println(carJsonNode);
        // 输出内容: {"brand":"Mercedes","doors":5}

        // JsonNode对象 转 Java对象
        Car car = objectMapper.treeToValue(carJsonNode, Car.class);
        System.out.println(car);
        // 输出内容: Car(brand=Mercedes, doors=5)
    }
}
```

上面的示例有点“人为”，因为我们首先将JSON字符串转换为JsonNode，然后将JsonNode转换为Car对象。 显然，如果我们有对原始JSON字符串的引用，则最好将其直接转换为Car对象，而无需先将其转换为JsonNode。



# 四、JsonNode

Jackson JsonNode类com.fasterxml.jackson.databind.JsonNode是Jackson的JSON树形模型（对象图模型）。 Jackson可以将JSON读取到JsonNode实例中，然后将JsonNode写入JSON。 因此，这一节将说明如何将JSON反序列化为JsonNode以及将JsonNode序列化为JSON。 此Jackson JsonNode教程还将说明如何从头开始构建JsonNode对象图，因此以后可以将它们序列化为JSON。



## 1、JsonNode vs ObjectNode

Jackson JsonNode类是不可变的。 这意味着，实际上不能直接构建JsonNode实例的对象图。 而是创建JsonNode子类ObjectNode的对象图。 作为JsonNode的子类，可以在可以使用JsonNode的任何地方使用ObjectNode。



## 2、JSON转JsonNode

要使用Jackson将JSON读取到JsonNode中，首先需要创建一个Jackson ObjectMapper实例。 在ObjectMapper实例上，调用readTree()并将JSON源作为参数传递。 这是将JSON反序列化为JsonNode的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{ \"f1\" : \"v1\" } ";
        ObjectMapper objectMapper = new ObjectMapper();

        JsonNode jsonNode = objectMapper.readTree(json);
        System.out.println(jsonNode.get("f1").asText());
        // 输出内容: v1
    }
}
```



## 3、JsonNode转JSON

要将Jackson的JsonNode写入JSON，还需要一个Jackson ObjectMapper实例。 在ObjectMapper上，调用writeValueAsString()方法或任何适合需要的写入方法。 这是将JsonNode写入JSON的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode jsonNode = new ObjectMapper().readTree("{ \"f1\" : \"v1\" }");

        ObjectMapper objectMapper = new ObjectMapper();
        String json = objectMapper.writeValueAsString(jsonNode);
        System.out.println(json);
        // 输出内容: 
    }
}
```



## 4、获取JsonNode字段

JsonNode可以像JSON对象一样具有字段。 假设已将以下JSON解析为JsonNode：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode jsonNode = new ObjectMapper().readTree("{\"field1\" : \"value1\", \"field2\" : 999}");
        JsonNode field1 = jsonNode.get("field1");
        JsonNode field2 = jsonNode.get("field2");
        System.out.println(field1 + "、" + field2);
        // 输出内容: "value1"、999
    }
}
```

请注意，即使两个字段都是String字段，get()方法也始终返回JsonNode来表示该字段。



## 5、路径中获取JsonNode字段

Jackson JsonNode有一个称为at()的特殊方法。 at()方法可以从JSON图中以给定JsonNode为根的任何位置访问JSON字段。 假设JSON结构如下所示：

```json
{
    "identification" :  {
        "name" : "James",
        "ssn": "ABC123552"
    }
}
```

如果将此JSON解析为JsonNode，则可以使用at()方法访问名称字段，如下所示：

```java
JsonNode nameNode = jsonNode.at("/identification/name");
```

注意传递给at()方法的参数：字符串/ identification / name。 这是一个JSON路径表达式。 此路径表达式指定从根JsonNode到您要访问其值的字段的完整路径。 这类似于从文件系统根目录到Unix文件系统中文件的路径。

**请注意**：JSON路径表达式必须以斜杠字符（/字符）开头。

at()方法返回一个JsonNode，它表示请求的JSON字段。 要获取该字段的实际值，需要调用下一部分介绍的方法之一。 如果没有节点与给定的路径表达式匹配，则将返回null。

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\n" +
                "    \"identification\" :  {\n" +
                "        \"name\" : \"James\",\n" +
                "        \"ssn\": \"ABC123552\"\n" +
                "    }\n" +
                "}";
        JsonNode jsonNode = new ObjectMapper().readTree(json);

        JsonNode identificationNode = jsonNode.at("/identification");
        System.out.println(identificationNode);
        // 输出内容: {"name":"James","ssn":"ABC123552"}

        JsonNode nameNode = jsonNode.at("/identification/name");
        System.out.println(nameNode);
        // 输出内容: "James"
    }
}
```



## 6、转换JsonNode字段

Jackson JsonNode类包含一组可以将字段值转换为另一种数据类型的方法。 例如，将String字段值转换为long或相反。 这是将JsonNode字段转换为一些更常见的数据类型的示例：

```java
String f2Str = jsonNode.get("f1").asText();
double f2Dbl = jsonNode.get("f2").asDouble();
int    f2Int = jsonNode.get("f3").asInt();
long   f2Lng = jsonNode.get("f4").asLong();
```

**使用默认值转换：**如果JsonNode中的字段可以为null，则在尝试转换它时可以提供默认值。 这是使用默认值调用转换方法的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{ \"f1\":\"Hello\", \"f2\":null }";
        ObjectMapper objectMapper = new ObjectMapper();

        JsonNode jsonNode = objectMapper.readTree(json);
        String f2Value = jsonNode.get("f2").asText("Default");
        System.out.println(f2Value);
        // 输出内容: Default
    }
}
```

在示例的JSON字符串中可以看到，声明了f2字段，但将其设置为null。 在这种情况下，调用jsonNode.get（“ f2”）。asText（“ Default”）将返回默认值，在此示例中为字符串Default。asDouble()，asInt()和asLong()方法还可以采用默认参数值，如果尝试从中获取值的字段为null，则将返回默认参数值。

请注意：如果该字段在JSON中未显式设置为null，但在JSON中丢失，则调用jsonNode.get("fieldName") 将返回Java null值，您无法在该Java值上调用asInt() ，asDouble()，asLong()或asText()。 如果尝试这样做，将会导致NullPointerException。 这是说明这种情况的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{ \"f1\":\"Hello\" }";
        ObjectMapper objectMapper = new ObjectMapper();

        JsonNode jsonNode = objectMapper.readTree(json);
        JsonNode f2FieldNode = jsonNode.get("f2");
        System.out.println(f2FieldNode);
        // 输出内容: null
    }
}
```



## 7、创建ObjectNode对象

如前所述，JsonNode类是不可变的。 要创建JsonNode对象图，必须能够更改图中的JsonNode实例，例如 设置属性值和子JsonNode实例等。由于是不可变的，因此无法直接使用JsonNode来实现。

而是创建一个ObjectNode实例，该实例是JsonNode的子类。 这是一个通过Jackson ObjectMapper createObjectNode()方法创建ObjectNode的示例：

```java
ObjectMapper objectMapper = new ObjectMapper();
ObjectNode objectNode = objectMapper.createObjectNode();
```



## 8、Set ObjectNode字段

要在Jackson ObjectNode上设置字段，可以调用其set()方法，并将字段名称String和JsonNode作为参数传递。 这是在Jackson的ObjectNode上设置字段的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode childNode = new ObjectMapper().readTree("{ \"f1\" : \"v1\" }");

        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode parentNode = objectMapper.createObjectNode();
        parentNode.set("child1", childNode);
        System.out.println(parentNode);
        // 输出内容: {"child1":{"f1":"v1"}}
    }
}
```



##  9、Put ObjectNode字段

ObjectNode类还具有一组方法，可以直接为字段put(设置)值。 这比尝试将原始值转换为JsonNode并使用set()进行设置要容易得多。 以下是使用put()方法为ObjectNode上的字段设置字符串值的示例：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

public class TestMain {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode objectNode = objectMapper.createObjectNode();
        objectNode.put("field1", "value1");
        objectNode.put("field2", 123);
        objectNode.put("field3", 999.999);
        System.out.println(objectNode);
        // 输出内容: {"field1":"value1","field2":123,"field3":999.999}
    }
}
```



## 10、Get ObjectNode字段

ObjectNode类可以直接通过get(field)方法获取value的值，返回的是JsonNode对象，不过只能一层一层获取，多层结构无法一次性就获取到最内部的value。示例如下：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode childNode = new ObjectMapper().readTree("{ \"f1\" : \"v1\" }");
        ObjectNode parentNode = new ObjectMapper().createObjectNode();
        parentNode.set("child1", childNode);
        JsonNode child1 = parentNode.get("child1");
        System.out.println(child1);
        // 输出内容: {"f1":"v1"}
        System.out.println(parentNode.get("f1"));
        // 输出内容: null
    }
}
```



## 11、删除ObjectNode字段

ObjectNode类具有一个称为remove()的方法，该方法可用于从ObjectNode中删除字段。 这是一个通过其remove()方法从Jackson ObjectNode删除字段的示例：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

public class TestMain {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode objectNode = objectMapper.createObjectNode();
        objectNode.put("field1", "value1");
        objectNode.put("field2", 123);
        objectNode.put("field3", 999.999);
        System.out.println(objectNode);
        // 输出内容: {"field1":"value1","field2":123,"field3":999.999}

        // 删除字段
        objectNode.remove("field3");
        System.out.println(objectNode);
        // 输出内容: {"field1":"value1","field2":123}

    }
}
```



## 12、循环ObjectNode字段

ObjectNode类具有一个名为fieldNames()的方法，该方法返回一个Iterator，可以迭代JsonNode的所有字段名称。 我们可以使用字段名称来获取字段值。 这是一个迭代Jackson ObjectNode的所有字段名称和值的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.util.Iterator;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode childNode = new ObjectMapper().readTree("{\"f1\" : \"v1\", \"f2\" : \"v2\"}");
        ObjectNode parentNode = new ObjectMapper().createObjectNode();
        parentNode.set("child1", childNode);

        // 首先获取所以key名称, 然后通过key名后去value值
        Iterator<String> fieldNames1 = parentNode.fieldNames();
        while(fieldNames1.hasNext()) {
            String fieldName = fieldNames1.next();
            JsonNode field = parentNode.get(fieldName);
            System.out.println(field);
        }
    }
}
```

打印结果如下

```
{"f1":"v1","f2":"v2"}
```



## 13、循环JsonNode字段

JsonNode类具有一个名为fieldNames()的方法，该方法返回一个Iterator，可以迭代JsonNode的所有字段名称。 我们可以使用字段名称来获取字段值。 这是一个迭代Jackson JsonNode的所有字段名称和值的示例：

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Iterator;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonNode jsonNode = new ObjectMapper().readTree("{\"f1\" : \"v1\", \"f2\" : \"v2\"}");
        // 首先获取所以key名称, 然后通过key名后去value值
        Iterator<String> fieldNames = jsonNode.fieldNames();
        while(fieldNames.hasNext()) {
            String fieldName = fieldNames.next();
            JsonNode field = jsonNode.get(fieldName);
            System.out.println(field);
        }

        System.out.println("=================================");

        String json = "{\n" +
                "    \"identification\" :  {\n" +
                "        \"name\" : \"James\",\n" +
                "        \"ssn\": \"ABC123552\"\n" +
                "    }\n" +
                "}";
        JsonNode jsonNode2 = new ObjectMapper().readTree(json);
        // 首先获取所以key名称, 然后通过key名后去value值
        Iterator<String> fieldNames2 = jsonNode2.fieldNames();
        while(fieldNames2.hasNext()) {
            String fieldName = fieldNames2.next();
            JsonNode field = jsonNode2.get(fieldName);
            System.out.println(field);
        }
    }
}
```



# 五、JsonParser

Jackson JsonParser类是一个底层一些的JSON解析器。 它类似于XML的Java StAX解析器，差别是JsonParser解析JSON而不解析XML。JsonParser的运行层级低于Jackson ObjectMapper。 这使得JsonParser比ObjectMapper更快，但使用起来也比较麻烦。



## 1、创建一个JsonParser

为了创建Jackson JsonParser，首先需要创建一个JsonFactory。 JsonFactory用于创建JsonParser实例。 JsonFactory类包含几个createParser()方法，每个方法都使用不同的JSON源作为参数。

这是创建一个JsonParser来从字符串中解析JSON的示例：

```java
import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"f1\" : \"v1\", \"f2\" : \"v2\"}";
        JsonFactory factory = new JsonFactory();
        JsonParser parser  = factory.createParser(json);
    }
}
```



## 2、用JsonParser转化JSON

一旦创建了Jackson JsonParser，就可以使用它来解析JSON。 JsonParser的工作方式是将JSON分解为一系列令牌，可以一个一个地迭代令牌。

这是一个JsonParser示例，它简单地循环遍历所有标记并将它们输出到System.out。 这是一个实际上很少用示例，只是展示了将JSON分解成的令牌，以及如何遍历令牌的基础知识。

```java
import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"f1\" : \"v1\", \"f2\" : \"v2\"}";
        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(json);
        while (!parser.isClosed()) {
            JsonToken jsonToken = parser.nextToken();
            System.out.println("jsonToken = " + jsonToken);
        }
    }
}
```

```
jsonToken = START_OBJECT
jsonToken = FIELD_NAME
jsonToken = VALUE_STRING
jsonToken = FIELD_NAME
jsonToken = VALUE_STRING
jsonToken = END_OBJECT
jsonToken = null
```

只要JsonParser的isClosed()方法返回false，那么JSON源中仍然会有更多的令牌。

可以使用JsonParser的nextToken()获得一个JsonToken。 您可以使用此JsonToken实例检查给定的令牌。 令牌类型由JsonToken类中的一组常量表示。 这些常量是：

```
START_OBJECT
END_OBJECT
START_ARRAY
END_ARRAY
FIELD_NAME
VALUE_EMBEDDED_OBJECT
VALUE_FALSE
VALUE_TRUE
VALUE_NULL
VALUE_STRING
VALUE_NUMBER_INT
VALUE_NUMBER_FLOAT
```

可以使用这些常量来找出当前JsonToken是什么类型的令牌。 可以通过这些常量的equals()方法进行操作。 这是一个例子：

```java
import com.fasterxml.jackson.core.*;
import lombok.Data;

@Data
class Car {
    public String brand = null;
    public int doors = 0;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(carJson);

        Car car = new Car();
        while (!parser.isClosed()) {
            JsonToken jsonToken = parser.nextToken();

            if (JsonToken.FIELD_NAME.equals(jsonToken)) {
                String fieldName = parser.getCurrentName();
                System.out.println(fieldName);

                jsonToken = parser.nextToken();

                if ("brand".equals(fieldName)) {
                    car.brand = parser.getValueAsString();
                } else if ("doors".equals(fieldName)) {
                    car.doors = parser.getValueAsInt();
                }
            }
        }

        System.out.println("car.brand = " + car.brand);
        System.out.println("car.doors = " + car.doors);
    }
}
```

打印结果如下

```
brand
doors
car.brand = Mercedes
car.doors = 5
```

如果指向的标记是字段名称，则JsonParser的getCurrentName()方法将返回当前字段名称。

如果指向的令牌是字符串字段值，则getValueAsString()返回当前令牌值作为字符串。 如果指向的令牌是整数字段值，则getValueAsInt()返回当前令牌值作为int值。 JsonParser具有更多类似的方法来获取不同类型的curren令牌值（例如boolean，short，long，float，double等）。





# 六、JsonGenerator

Jackson JsonGenerator用于从Java对象（或代码从中生成JSON的任何数据结构）生成JSON。

## 1、创建一个JsonGenerator

为了创建Jackson JsonGenerator，必须首先创建JsonFactory实例。 这是创建JsonFactory的方法：`new JsonFactory();`

一旦创建了JsonFactory，就可以使用JsonFactory的createGenerator()方法创建JsonGenerator。 这是创建JsonGenerator的示例：

```java
JsonFactory factory = new JsonFactory();
JsonGenerator generator = factory.createGenerator(new File("D:\\output.json"), JsonEncoding.UTF8);
```

createGenerator()方法的第一个参数是生成的JSON的目标。 在上面的示例中，参数是File对象。 这意味着生成的JSON将被写入给定文件。 createGenerator()方法已重载，因此还有其他版本的createGenerator()方法采用例如OutputStream等，提供了有关将生成的JSON写入何处的不同选项。

createGenerator()方法的第二个参数是生成JSON时使用的字符编码。 上面的示例使用UTF-8。

## 2、JsonGenerator生成JSON

一旦创建了JsonGenerator，就可以开始生成JSON。 JsonGenerator包含一组write …()方法，可以使用这些方法来编写JSON对象的各个部分。 这是一个使用Jackson JsonGenerator生成JSON的简单示例：

```java
import com.fasterxml.jackson.core.*;
import lombok.Data;
import java.io.File;

public class TestMain {
    public static void main(String[] args) throws Exception {
        JsonFactory factory = new JsonFactory();
        JsonGenerator generator = factory.createGenerator(new File("D:\\output.json"), JsonEncoding.UTF8);

        generator.writeStartObject();
        generator.writeStringField("brand", "Mercedes");
        generator.writeNumberField("doors", 5);
        generator.writeEndObject();

        generator.close();
    }
}
```

文件内容如下：

```json
{"brand":"Mercedes","doors":5}
```

此示例首先调用writeStartObject()，将{写入输出。 然后，该示例调用writeStringField()，将品牌字段名称+值写入输出。 之后，将调用writeNumberField()方法，此方法会将Doors字段名称+值写入输出。 最后，调用writeEndObject()，将}写入输出。

JsonGenerator还可以使用许多其他写入方法。 这个例子只显示了其中一些。


## 3、关闭JsonGenerator

完成生成JSON后，应关闭JsonGenerator。 您可以通过调用其close()方法来实现。 这是关闭JsonGenerator的样子：

```java
generator.close();
```



# 七、Jackson 常用注解

## 1、序列化注解

### 1、@JsonGetter

**简单说：就是给字段取别名。可以用来代替@JsonProperty**

@JsonGetter 注解用于告诉Jackson，应该通过调用getter方法而不是通过直接字段访问来获取某个字段值。 如果您的Java类使用jQuery样式的getter和setter名称，则@JsonGetter注解很有用。

例如：JSON中有一个字段叫id，准备序列化Java对象中的studentId，就可以使用此注解

```java
import com.fasterxml.jackson.annotation.JsonGetter;
import com.fasterxml.jackson.annotation.JsonSetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private Integer studentId = 0;
    private String name = "Sam";
    private Integer age = 18;

    // 此注解只能用在方法上
    @JsonGetter("id")
    public Integer getStudentId() {
        return studentId;
    }
    @JsonSetter("id")
    public void setStudentId(Integer studentId) {
        this.studentId = studentId;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","age":18,"id":0}
    }
}
```



### 2、@JsonAnyGetter

**简单说：序列化普通字段到 Map 集合中**

@JsonAnyGetter 可以在对 Java 对象进行序列化时，使其中的 Map 集合作为 JSON 中属性的来源。像普通属性一样序列化Map：@JsonAnyGetter 注解使您可以将Map用作要序列化为JSON的属性的容器。下面做个示例。

```java
import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.*;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private String name = "Sam";
    private Integer age = 18;
    @JsonAnyGetter
    private Map<String, Object> diyMap = new HashMap<>() {{
        put("a", 111);
        put("b", 222);
        put("c", 333);
    }};
    // 两种方式都支持(建议使用此种方式,方法名称可以随意命名)
    // @JsonAnyGetter
    // public Map<String, Object> getDiyMap () {
    //     return diyMap;
    // }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","age":18,"diyMap":{"a":111,"b":222,"c":333},"a":111,"b":222,"c":333}
    }
}
```



### 3、@JsonPropertyOrder

用在类上，在序列化的时候自定义属性输出顺序

```java
import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
@JsonPropertyOrder({ "age", "name", "id"})
class Student {
    private Integer id = 0;
    private String name = "Sam";
    private Integer age = 18;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"age":18,"name":"Sam","id":0}
    }
}
```



### 4、@JsonRawValue

**简单说：完全按照原样序列化属性的值**

@JsonRawValue 注解告诉Jackson该属性值应直接写入JSON输出。 如果该属性是字符串，Jackson通常会将值括在引号中，但是如果使用@JsonRawValue属性进行注解，Jackson将不会这样做。

```java
import com.fasterxml.jackson.annotation.JsonRawValue;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private String name_1 = "Sam";
    @JsonRawValue
    private String name_2 = "Sam";
    private String json_1 = "{\"attr\":false}";
    @JsonRawValue
    private String json_2 = "{\"attr\":false}";
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name_1":"Sam","name_2":Sam,"json_1":"{\"attr\":false}","json_2":{"attr":false}}
    }
}
```

可以发现Java对象的一个字段为JSON字符串并且带转义符，如果没有加@JsonRawValue，那么序列化会把转义符带上并加上引号，如果带上有@JsonRawValue注解，就会显示正常的JSON字符串。

### 5、@JsonValue

**简单说：定义整个实体的序列化方法，Jackson将会使用该方法的输出作为序列化输出**

@JsonValue告诉Jackson不应该尝试序列化对象本身，而应在对象上调用将对象序列化为JSON字符串的方法。 请注意，Jackson将在自定义序列化返回的String内转义任何引号，因此不能返回例如完整的JSON对象。 为此，应该改用 @JsonRawValue。

@JsonValue注解已添加到Jackson调用的方法中，以将对象序列化为JSON字符串。 这是显示如何使用@JsonValue注解的示例：

```java
import com.fasterxml.jackson.annotation.JsonValue;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private String name = "Sam";
    private String type = "Java";
    @JsonValue
    public String toJson() {
        return this.name + this.type;
    }
}

@Getter
@AllArgsConstructor
enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type B");
    private Integer id;
    private String name;
    @JsonValue
    public String getJson() {
        return this.name + this.id;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String student = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(student);
        // 输出内容: "SamJava"
        String typeEnum = new ObjectMapper().writeValueAsString(TypeEnumWithValue.TYPE1);
        System.out.println(typeEnum);
        // 输出内容: "Type A1"
    }
}
```

引号由Jackson添加。 请记住，对象返回的值字符串中的所有引号均会转义。

> @JsonValue 与 @JsonRawValue 的区别：
>
> 1. @JsonRawValue 表示原样输出，输出时不会加引号，所以有转义符会自动转义。
> 2. @JsonRawValue 既可以放在字段上，也能放在方法上
> 3. @JsonValue 只能作用在方法上，序列化的结果就为方法返回结果，并且会带上引号，如果有转移会在引号中显示出来



### 6、@JsonRootName

**简单说：如果需要将实体包装一层，可以使用@JsonRootName来指定根包装器的名称**

@JsonRootName 仅用于指定 JSON 根属性的名称，并不能启用装功能。只有当 **SerializationFeature.WRAP_ROOT_VALUE**、**DeserializationFeature.UNWRAP_ROOT_VALUE** 启用时才有效。若只开启了 Feature 功能但未使用 @JsonRootName 注解，默认会使用类名作为 RootName。

1、使用 @JsonRootName 注解 + 开启 Feature 功能

```java
import com.fasterxml.jackson.annotation.JsonRootName;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import lombok.*;

@Data
@JsonRootName(value = "data")
class Student {
    private String name = "Sam";
    private String type = "Java";
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        Student student = new Student();

        System.out.println("-- before serialization --");
        String json = objectMapper.writeValueAsString(student);
        System.out.println(json);
        // 输出内容: {"name":"Sam","type":"Java"}

        // 启用 SerializationFeature.WRAP_ROOT_VALUE
        objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
        System.out.println("-- after serialization --");
        json = objectMapper.writeValueAsString(student);
        System.out.println(json);
        // 输出内容: {"data":{"name":"Sam","type":"Java"}}

        // 启用 DeserializationFeature.UNWRAP_ROOT_VALUE
        objectMapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);
        System.out.println("-- after deserialization --");
        Student student2 = objectMapper.readValue(json, Student.class);
        System.out.println(student2);
        // 输出内容: Student(name=Sam, type=Java)
    }
}
```

打印结果如下

```java
-- before serialization --
{"name":"Sam","type":"Java"}
-- after serialization --
{"data":{"name":"Sam","type":"Java"}}
-- after deserialization --
Student(name=Sam, type=Java)
```



2、未使用 @JsonRootName 注解 + 开启 Feature 功能

打印结果如下：可以看出默认 RootName 为 PersonEntity（类名）

```
-- before serialization --
{"name":"Sam","type":"Java"}
-- after serialization --
{"Student":{"name":"Sam","type":"Java"}}
-- after deserialization --
Student(name=Sam, type=Java)
```



### 7、@JsonSerialize

**简单说：用于指定自定义序列化器来序列化实体**

```java
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import lombok.*;
import java.io.IOException;

@Data
class Student {
    private String name = "Sam";
    private String type = "Java";
    // 注意启用字段上方的@JsonSerialize注解
    @JsonSerialize(using = OptimizedBooleanSerializer.class)
    public boolean enabled = false;
}

/**
 * OptimizedBooleanSerializer将序列的真值序列化为1，将假值序列化为0
 */
class OptimizedBooleanSerializer extends JsonSerializer<Boolean> {
    @Override
    public void serialize(Boolean enable,
                          JsonGenerator jsonGenerator,
                          SerializerProvider serializerProvider) throws IOException {
        if (enable) {
            jsonGenerator.writeNumber(1);
        } else {
            jsonGenerator.writeNumber(0);
        }
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","type":"Java","enabled":0}
    }
}
```



## 2、反序列化注解

### 1、@JsonSetter

**简单说：可以指定属性别名，当时只能用在方法上**

@JsonSetter告诉Jackson，当将JSON读入对象时，应将此setter方法的名称与JSON数据中的属性名称匹配。 如果Java类内部使用的属性名称与JSON文件中使用的属性名称不同，这个注解就很有用了。

以下Student类用studentId名称对应JSON中名为id的字段（JSON对象中，使用名称id代替studentId）：

```java
import com.fasterxml.jackson.annotation.JsonGetter;
import com.fasterxml.jackson.annotation.JsonSetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private Integer studentId = 0;
    private String name = "Sam";
    private Integer age = 18;

    // 此注解只能用在方法上
    @JsonGetter("id")
    public Integer getStudentId() {
        return studentId;
    }
    @JsonSetter("id")
    public void setStudentId(Integer studentId) {
        this.studentId = studentId;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"name\":\"Sam\",\"age\":18,\"id\":0}";
        Student student = new ObjectMapper().readValue(json, Student.class);
        System.out.println(student);
        // 输出内容: Student(studentId=0, name=Sam, age=18)
    }
}
```



### 2、@JsonAnySetter

**简单说：就是把不存在Java对象的属性都放进加了此注解的Map集合中**

使用 @JsonAnySetter 可以在对 JSON 进行反序列化时，对所有在 Java 对象中不存在的属性进行逻辑处理，下面的代码演示把不存在的属性存放到一个 Map 集合中。

```java
import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.*;
import lombok.*;

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private String name;
    private Integer age;
    @JsonAnySetter
    private Map<String, Object> diyMap = new HashMap<>();
    // 两种方式都支持(建议使用此种方式,方法名称可以随意命名)
    // @JsonAnySetter
    // public void otherField(String key, String value) {
    //     this.diyMap.put(key, value);
    // }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"name\":\"Sam\", \"age\":18, \"key1\":\"java\", \"key2\":\"python\"}";
        System.out.println(json);
        Student student = new ObjectMapper().readValue(json, Student.class);
        System.out.println(student);
        // 输出内容: Student(name=Sam, age=18, diyMap={key1=java, key2=python})
    }
}
```

打印结果如下

```
{"name":"Sam", "age":18, "key1":"java", "key2":"python"}
Student(name=Sam, age=18, diyMap={key1=java, key2=python})
```



### 3、@JsonCreator

@JsonCreator告诉Jackson该Java对象具有一个构造函数（“创建者”），该构造函数可以将JSON对象的字段与Java对象的字段进行匹配。@JsonCreator注解在无法使用@JsonSetter注解的情况下使用。 例如，不可变对象没有任何设置方法，因此它们需要将其初始值注入到构造函数中。

要告诉Jackson应该调用Student的构造函数，我们必须在构造函数中添加@JsonCreator注解。 但是，仅凭这一点还不够。 我们还必须注解构造函数的参数，以告诉Jackson将JSON对象中的哪些字段传递给哪些构造函数参数。

添加了@JsonCreator和@JsonProperty注解的Student类的示例如下：

```java
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@ToString
@NoArgsConstructor
class Student {
    @Getter
    @Setter
    private Integer id = 0;
    @Getter
    private String name = "Sam";
    @Getter
    private Integer age = 18;

    @JsonCreator
    public Student(@JsonProperty("name") String name,
                   @JsonProperty("age") Integer age) {
        this.name = name;
        this.age = age;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"name\":\"Sam\",\"age\":18,\"id\":0}";
        Student student = new ObjectMapper().readValue(json, Student.class);
        System.out.println(student);
        // 输出内容: Student(id=0, name=Sam, age=18)
    }
}
```

总结：使用 @JsonCreator + @JsonProperty 时，反序列化可以省略 Setter 方法。



### 4、@JacksonInject

**简单说：指定某个字段从注入赋值，而不是从JSON**

@JacksonInject 用于将值注入到解析的对象中，而不是从JSON中读取这些值。 例如，假设正在从各种不同的源下载Student JSON对象，并且想知道给定Student对象来自哪个源。 源本身可能不包含该信息，但是可以让Jackson将其注入到根据JSON对象创建的Java对象中。要将Java类中的字段标记为需要由Jackson注入其值的字段，请在该字段上方添加@JacksonInject注解。

这是一个示例Student类，在属性上方添加了@JacksonInject注解：

```java
import com.fasterxml.jackson.annotation.JacksonInject;
import com.fasterxml.jackson.databind.InjectableValues;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.*;

@Data
class Student {
    @JacksonInject
    private Integer id;
    private String name = "Sam";
    private Integer age = 18;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = "{\"name\":\"Sam\",\"age\":18}";
        // 如下如果未设置忽略Java对象不存在的字段配置，会显示报错
        // System.out.println(new ObjectMapper().readValue(json, Student.class));

        InjectableValues inject = new InjectableValues.Std().addValue(Integer.class, 1);
        Student student = new ObjectMapper().reader(inject).forType(Student.class).readValue(json);
        System.out.println(student);
        // 输出内容: Student(id=1, name=Sam, age=18)
    }
}
```

请注意，如何在InjectableValues.addValue()方法中设置要注入到ID属性中的值。 还要注意，该值仅绑定到字符串类型-而不绑定到任何特定的字段名称。 @JacksonInject注解指定将值注入到哪个字段。

如果要从多个源下载人员JSON对象，并为每个源注入不同的源值，则必须为每个源重复以上代码。



### 5、@JsonDeserialize

**简单说：用于为Java对象中给定的属性指定自定义反序列化器类**

例如，假设想优化布尔值false和true的在线格式，使其分别为0和1。首先，需要将@JsonDeserialize注解添加到要为其使用自定义反序列化器的字段。 这是将@JsonDeserialize注解添加到字段的示例：


```java
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import lombok.*;
import java.io.IOException;

@Data
class Student {
    private String name = "Sam";
    private String type = "Java";
    // @JsonSerialize 为序列化注解，@JsonDeserialize 为反序列化注解
    @JsonSerialize(using = OptimizedBooleanSerializer.class)
    @JsonDeserialize(using = OptimizedBooleanDeserializer.class)
    public boolean enabled = false;
}

/**
 * OptimizedBooleanSerializer将序列的真值序列化为1，将假值序列化为0
 */
class OptimizedBooleanSerializer extends JsonSerializer<Boolean> {
    @Override
    public void serialize(Boolean enable,
                          JsonGenerator jsonGenerator,
                          SerializerProvider serializerProvider) throws IOException {
        if (enable) {
            jsonGenerator.writeNumber(1);
        } else {
            jsonGenerator.writeNumber(0);
        }
    }
}

/**
 * OptimizedBooleanDeserializer将序列的1反序列化为真值，将0反序列化为假值
 */
class OptimizedBooleanDeserializer
        extends JsonDeserializer<Boolean> {

    @Override
    public Boolean deserialize(JsonParser jsonParser,
                               DeserializationContext deserializationContext) throws IOException {
        String text = jsonParser.getText();
        if ("0".equals(text)) return false;
        return true;
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","type":"Java","enabled":0}
        System.out.println(new ObjectMapper().readValue(json, Student.class));
        // 输出内容: Student(name=Sam, type=Java, enabled=false)
    }
}
```



## 3、属性包含注解

### 1、@JsonIgnore

**简单说：在具体属性上忽略，使其不参与序列化过程**

@JsonIgnore用于告诉Jackson忽略Java对象的某个属性（字段）。 在将JSON读取到Java对象中以及将Java对象写入JSON时，都将忽略该属性。

```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
class Student {
    @JsonIgnore
    private String name = "Sam";
    @JsonIgnore
    private String type = "Java";
    public boolean enabled = false;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"enabled":false}
    }
}
```

与**@JsonIgnoreProperties**是等效的。



### 2、@JsonIgnoreProperties

**简单说：在类上指定要忽略的属性**

@JsonIgnoreProperties 注解用于指定要忽略的类的属性列表。 @JsonIgnoreProperties注解放置在类声明上方，而不是要忽略的各个属性（字段）上方。

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
@JsonIgnoreProperties({"name", "type"})
class Student {
    private String name = "Sam";
    private String type = "Java";
    public boolean enabled = false;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"enabled":false}
    }
}
```



### 3、@JsonIgnoreType

**简单说：使用此注解可以忽略整个类**。@JsonIgnoreType 注解用于将整个类型（类）标记为在使用该类型的任何地方都将被忽略。

使用@JsonIgnoreType注解，在类上将忽略该类所有属性：

```java
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
class Student {
    private String name = "Sam";
    private String type = "Java";
    private Address address = new Address();
    @JsonIgnoreType
    public static class Address {
        public String streetName = "1";
        public String houseNumber = "2";
        public String zipCode = "3";
        public String city = "4";
        public String country = "5";
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","type":"Java"}
    }
}
```

如果去掉@JsonIgnoreType注解那么打印如下：

```
{"name":"Sam","type":"Java","address":{"streetName":"1","houseNumber":"2","zipCode":"3","city":"4","country":"5"}}
```



### 4、@JsonInclude

**简单说：用于排除值为`empty/null/default`的属性**。@JsonInclude告诉Jackson仅在某些情况下包括属性，如果属性值为empty/null/default，那么将忽略该字段。例如：仅当属性为非null，非空或具有非默认值时，才应包括该属性。 

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
@JsonInclude(JsonInclude.Include.NON_EMPTY)
class Student {
    private Long id = 1L;
    private String name = null;
    private String type = null;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"id":1}
    }
}
```

当前注解表示：如果有字段为null或空字符串的话，将忽略这个字段的生成。



### @JsonAutoDetect

**简单说：强制序列化私有属性，不管它有没有getter方法**。@JsonAutoDetect告诉Jackson在读写对象时包括非public修饰的属性。

这是一个示例类，展示如何使用@JsonAutoDetect注解：

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.databind.*;

@JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)
class Student {
    private Long id = 1L;
    private String name = "Sam";
    private String type = "Java";
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"id":1,"name":"Sam","type":"Java"}
    }
}
```

注意：JsonAutoDetect.Visibility类包含与Java中的可见性级别匹配的常量，表示ANY，DEFAULT，NON_PRIVATE，NONE，PROTECTED_AND_PRIVATE和PUBLIC_ONLY。



## 4、多态处理注解

一般都是组合起来使用，有下面三个注解：

- @JsonTypeInfo：指定序列化中包含的类型信息的详细信息
- @JsonSubTypes：指定带注释类型的子类型
- @JsonTypeName：指定用于带注释的类的逻辑类型名称

下面例子中，指定属性type为判断具体子类的依据，例如：type=dog，将被序列化为Dog类型。

```java
public class Zoo {
    public Animal animal;

    @JsonTypeInfo(use = JsonTypeInfo.Id.NAME, 
                  include = JsonTypeInfo.As.PROPERTY, 
                  property = "type")
    @JsonSubTypes({@JsonSubTypes.Type(value = Dog.class, name = "dog"), 
                   @JsonSubTypes.Type(value = Cat.class, name = "cat")})
    public static class Animal {
        public String name;
    }

    @JsonTypeName("dog")
    public static class Dog extends Animal {
        public double barkVolume;
    }

    @JsonTypeName("cat")
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;
    }
}
```



## 5、通用注解（序列化反序列化都生效）

### 1、@JsonProperty

指定JSON中的属性名称，又称取别名。即可以序列化又可以反序列化，可以在方法和属性上使用

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.*;
import lombok.Data;

@Data
class Student {
    @JsonProperty("id")
    private Long studentId = 1L;
    private String name = "Sam";
    private String type = "Java";
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"name":"Sam","type":"Java","id":1}
        System.out.println(new ObjectMapper().readValue(json, Student.class));
        // 输出内容: Student(studentId=1, name=Sam, type=Java)
    }
}
```



### 2、@JsonFormat

用于在序列化日期/时间值时指定格式

```java
@Data
public class Student {
    private Long id = 1L;
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    public Date date;
}
```



### 3、@JsonUnwrapped

将对象中所有的属性与当前平级，不太好描述，简单说就是拆开包装。简单说：就是扁平化

```java
import com.fasterxml.jackson.annotation.JsonUnwrapped;
import com.fasterxml.jackson.databind.*;
import lombok.ToString;
import lombok.Data;

@Data
class Student {
    private Long id = 1L;
    @JsonUnwrapped
    public Name name = new Name();
    @ToString
    public static class Name {
        public String firstName = "Sam";
        public String lastName = "Liu";
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"id":1,"firstName":"Sam","lastName":"Liu"}
        System.out.println(new ObjectMapper().readValue(json, Student.class));
        // 输出内容: Student(id=1, name=Student.Name(firstName=Sam, lastName=Liu))
    }
}
```

打印结果如下

```
{"id":1,"firstName":"Sam","lastName":"Liu"}
Student(id=1, name=Student.Name(firstName=Sam, lastName=Liu))
```

对比一下加于不加注解的区别：

1. 如果加@JsonUnwrapped注解，将被序列化为：

   ```json
   {
       "id":1,
       "firstName":"John",
       "lastName":"Doe"
   }
   ```

2. 如果不加@JsonUnwrapped注解，将被序列化为：

   ```json
   {
       "id":1,
       "name": {
           "firstName":"John",
           "lastName":"Doe"
       }
   }
   ```



### 4、@JsonView

指定视图，类似分组进行序列化/反序列化。分组指定序列化/反序列化时是否包含属性。

```java
import com.fasterxml.jackson.annotation.JsonView;
import com.fasterxml.jackson.databind.*;
import lombok.*;

/**
 * 定义视图
 */
class Views {
    public static class Public {}
    public static class Internal extends Public {}
}

/**
 * 定义实体
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
class Item {
    @JsonView(Views.Public.class)
    public int id;
    @JsonView(Views.Public.class)
    public String itemName;
    @JsonView(Views.Internal.class)
    public String ownerName;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        Item item = new Item(2, "book", "Sam");
        String result = new ObjectMapper().writerWithView(Views.Public.class).writeValueAsString(item);
        System.out.println(result);
        // 输出内容: {"id":2,"itemName":"book"}
    }
}
```

最终，将只会序列化 id 和 itemName 字段



### 5、@JsonManagedReference@JsonBackReference

@JsonManagedReference 和 @JsonBackReference 注解用于处理父/子关系并解决循环问题。

```java
@Data
public class ItemWithRef {
    public int id;
    public String itemName;
    @JsonManagedReference
    public UserWithRef owner;
}
```

```java
public class UserWithRef {
    public int id;
    public String name;
    @JsonBackReference
    public List<ItemWithRef> userItems;
}
```

不加注解，会循环调用，导致内存溢出，这时候可以使用@JsonManagedReference和@JsonBackReference来避免内存溢出。



### 6、@JsonIdentityInfo

用于指定在序列化 或者 反序列化值时使用对象标识，例如，处理无限递归类型的问题。

```java
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
public class ItemWithIdentity {
    public int id;
    public String itemName;
    public UserWithIdentity owner;
}
```



### 7、@JsonFilter

指定序列化期间要使用的过滤器

```java
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.annotation.JsonFilter;
import com.fasterxml.jackson.databind.ser.FilterProvider;
import com.fasterxml.jackson.databind.ser.impl.SimpleBeanPropertyFilter;
import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
import lombok.*;

@Data
@AllArgsConstructor
@JsonFilter("myFilter")
class BeanWithFilter {
    public int id;
    public String name;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        BeanWithFilter bean = new BeanWithFilter(1, "My bean");

        // 指定filter需要序列化输出那些属性,这里只序列化name
        SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter.filterOutAllExcept("name");
        FilterProvider filters = new SimpleFilterProvider().addFilter("myFilter", filter);

        String result = new ObjectMapper().writer(filters).writeValueAsString(bean);
        System.out.println(result);
        // 输出内容: {"name":"My bean"}
    }
}
```



## 6、自定义注解

可以使用**@JacksonAnnotationsInside**来开发自定义注解

```java
import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import lombok.Data;

/**
 * 一般自定义注解不会单独使用,会组合其他Jackson一起使用
 * 这里加上了忽略NULL值配置 以及 序列化属性排序注解
 */
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({ "name", "id", "age" })
@interface CustomAnnotation {}

@Data
@CustomAnnotation
class Student {
    private Integer id = 0;
    private String name;
    private Integer age;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String json = new ObjectMapper().writeValueAsString(new Student());
        System.out.println(json);
        // 输出内容: {"id":0}
    }
}
```

作用：自定义注解可以**增强代码复用**，把一些通用的Jackson注解组合起来，形成一个新注解，新注解可以代替组合的注解。

***

## 7、自定义注解案例（实现数据脱敏）

> 下面完成一个案例：**Jackson 自定义注解实现数据脱敏**
>
> - https://mp.weixin.qq.com/s/RujXqk0MWhmtQKpf_zaTnQ

1、自定义一个Jackson注解 + 定制脱敏策略 （这里偷懒了，把注解于枚举定义在一起了。实际上需要分开定义）

```java
import com.fasterxml.jackson.annotation.JacksonAnnotationsInside;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.function.Function;

/**
 * 自定义jackson注解，标注在属性上
 * 
 * 需要自定义一个脱敏注解，一旦有属性被标注，则进行对应得脱敏，
 * 针对项目需求，定制不同字段的脱敏规则，比如手机号中间几位用*替代
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD) // 针对成员属性进行脱敏
@JacksonAnnotationsInside  // 表示和其他Jackson注解联合使用，如果缺少则无法执行数据脱敏流程
@JsonSerialize(using = SensitiveJsonSerializer.class) // 表明使用的序列化的类，定义在后面
public @interface Sensitive {
    /**
     * 脱敏策略
     */
    SensitiveStrategy strategy();

    /**
     * 脱敏策略，枚举类，针对不同的数据定制特定的策略
     */
    enum SensitiveStrategy {
        /**
         * 用户名
         */
        USERNAME(s -> s.replaceAll("(\\S)\\S(\\S*)", "$1*$2")),
        /**
         * 身份证
         */
        ID_CARD(s -> s.replaceAll("(\\d{4})\\d{10}(\\w{4})", "$1****$2")),
        /**
         * 手机号
         */
        PHONE(s -> s.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2")),
        /**
         * 地址
         */
        ADDRESS(s -> s.replaceAll("(\\S{3})\\S{2}(\\S*)\\S{2}", "$1****$2****"));
        
        private final Function<String, String> desensitize;

        SensitiveStrategy(Function<String, String> desensitize) {
            this.desensitize = desensitize;
        }

        public Function<String, String> desensitize() {
            return desensitize;
        }
    }
}
```

2、自定义JsonSerializer从而实现数据脱敏，对标注注解@Sensitive的字段进行脱敏，实现如下：

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.BeanProperty;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.ContextualSerializer;
import java.io.IOException;
import java.util.Objects;

/**
 * 序列化注解自定义实现
 * JsonSerializer<String>：指定String 类型，serialize()方法用于将修改后的数据载入
 */
public class SensitiveJsonSerializer extends JsonSerializer<String> implements ContextualSerializer {
    private Sensitive.SensitiveStrategy strategy;

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        // 在序列化时进行数据脱敏
        gen.writeString(strategy.desensitize().apply(value));
    }

    /**
     * 获取属性上的注解属性, 又叫自定义注解被拦截后的回调函数
     */
    @Override
    public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property) throws JsonMappingException {

        Sensitive annotation = property.getAnnotation(Sensitive.class);
        if (Objects.nonNull(annotation) && Objects.equals(String.class, property.getType().getRawClass())) {
            this.strategy = annotation.strategy();
            return this;
        }
        return prov.findValueSerializer(property.getType(), property);
    }
}
```

4、定义Person类，对其数据脱敏

```java
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    /**
     * 真实姓名
     */
    @Sensitive(strategy = Sensitive.SensitiveStrategy.USERNAME)
    private String realName;
    /**
     * 地址
     */
    @Sensitive(strategy = Sensitive.SensitiveStrategy.ADDRESS)
    private String address;
    /**
     * 电话号码
     */
    @Sensitive(strategy = Sensitive.SensitiveStrategy.PHONE)
    private String phoneNumber;
    /**
     * 身份证号码
     */
    @Sensitive(strategy = Sensitive.SensitiveStrategy.ID_CARD)
    private String idCard;
}
```

5、模拟接口测试，以上4个步骤完成了数据脱敏的Jackson注解，这里模拟main方法测试，接口测试效果一样。代码如下：

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestMain {
    public static void main(String[] args) throws Exception {
        Student student = new Student("路边烧卖", "广东广州", "12306", "4333333333334334333");
        String json = new ObjectMapper().writeValueAsString(student);
        System.out.println(json);
        // 输出内容: {"realName":"路*烧卖","address":"广东广州","phoneNumber":"12306","idCard":"4333****34333"}
    }
}
```



## 8、MixIn 注解

> Jackson的Mixins,真香：https://blog.csdn.net/wjw465150/article/details/127187859

Jackson Mixins 是一种在类中添加 Jackson 注解而不修改实际类的机制。 它是为我们无法修改类的情况而创建的，例如在使用第三方类时。

我们可以使用任何Jackson注释，但不直接将它们添加到类中。我们在mixin类中使用它们，而mixin类既可以是抽象类，也可以是接口。它们既可以用于Jackson序列化也可以用于反序列化，并且必须将它们添加到ObjectMapper配置中。

在接下来的部分中，将展示一些使用 Jackson Mixin 有用的常见案例。

### 1、使第三方类 Jackson 可序列化

我们展示的第一个案例是当我们需要使用无法由 Jackson 序列化或反序列化的第三方类时，因为它们不遵循 Jackson 约定。 由于我们无法修改这些类，我们必须使用 mixins 来添加 Jackson 序列化所需的所有必要部分。

假设我们要序列化这个第三方类：

```java
public class Person {
    private final String firstName;
    private final String lastName;

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Person.class.getSimpleName() + "[", "]")
                .add("firstName='" + firstName + "'")
                .add("lastName='" + lastName + "'")
                .toString();
    }
}
```

我们不能用 Jackson 序列化这个类，因为属性是私有的并且没有 getter 和 setter。 因此，Jackson 不会识别任何属性并会抛出异常：

```java
String jsonSting = new ObjectMapper().writeValueAsString(new Person("Sam", "Liu"));
System.out.println(jsonSting);
```

```shell
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class com.grabanotherbyte.jackson.Person and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS)
```

现在让我们创建一个 mixin 来解决这个问题！ 在我们的 mixin 中，我们将添加我们想要序列化的属性：

```java
public abstract class PersonMixin {
    @JsonProperty private String firstName;
    @JsonProperty private String lastName;
}
```

在这种情况下，我们创建了一个抽象类，因为稍后我们需要添加一个构造函数。之后，我们需要告诉 Jackson 使用我们的 Mixin。 为此，我们需要在 *ObjectMapper* 中进行设置：

```java
// 序列化
        String jsonSting = new ObjectMapper()
                .addMixIn(Person.class, PersonMixin.class)
                .writeValueAsString(new Person("Sam", "Liu"));
        System.out.println(jsonSting);
        // 输出内容: {"firstName":"Sam","lastName":"Liu"}

        // 反序列化
        Person person = new ObjectMapper()
                .addMixIn(Person.class, PersonMixin.class)
                .readValue(jsonSting, Person.class);
        System.out.println(person);
        // .readValue(jsonSting, Person.class); 这里已经报错
```

现在，Jackson将能够序列化我们的*Person*，并将序列化*firstName*和*lastName*。另一方面，如果我们还想使用*Person*类进行反序列化，我们需要在mixin类中添加一个Jackson友好的构造函数:

```java
public abstract class PersonMixin {
    @JsonProperty private String firstName;
    @JsonProperty private String lastName;
    @JsonCreator
    public PersonMixin(@JsonProperty("firstName") String firstName, @JsonProperty("lastName") String lastName) {}
}
```

这样配置后反序列化就不会报错了，否则，Jackson将抛出异常。



### 2、忽略属性

现在，在序列化第三方类时，我们将考虑不同的场景。让我们假设现在我们的*Person*类拥有所需的所有getter、setter和构造函数，并且它的所有字段都将被序列化：

```java
public class Person {
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```

但是，我们希望只序列化firstName，并且和以前一样不能修改这个类。同样，我们可以通过使用mixin类来解决这个问题：

```java
public abstract class PersonMixin {
    @JsonIgnore private String lastName;
}
```

这样，Jackson将忽略这个属性，只序列化firstName。



### 3、改变属性名

按照前面的例子，我们还希望在序列化某些属性时更改它们的名称。

让我们修改mixin类，重命名属性lastName:

```java
public abstract class PersonMixin {
    @JsonProperty("surname") private String lastName;
}
```

现在，我们的 *lastName* 属性将被序列化为 *surname*。



### 4、重写自定义序列化器和反序列化器

在其他情况下，我们会发现使用自定义序列化器和反序列化器的类，但我们希望重写它们。当然，我们不能也不想修改这些类。

让我们扩展我们的*Person*类来包含一个自定义序列化器和反序列化器：

```java
@JsonSerialize(using = PersonSerializer.class)
@JsonDeserialize(using = PersonDeserializer.class)
public class Person {
    private final String firstName;
    private final String lastName;

    // getters, setters, constructors...
    public static class PersonSerializer extends JsonSerializer<Person> {
        static final String SEPARATOR = " ";
        @Override
        public void serialize(
            Person person, JsonGenerator jsonGenerator, SerializerProvider serializerProvider)
            throws IOException {
            jsonGenerator.writeString(person.getFirstName() + SEPARATOR + person.getLastName());
        }
    }

    public static class PersonDeserializer extends JsonDeserializer<Person> {
        @Override
        public Person deserialize(JsonParser jsonParser, DeserializationContext deserializationContext)
            throws IOException {
            String[] fields = jsonParser.getValueAsString().split(PersonSerializer.SEPARATOR);
            return new Person(fields[0], fields[1]);
        }
    }
}
```

正如我们所见，*Person* 类现在通过连接*firstName* 和*lastName* 进行序列化。

然而，在某些情况下，我们希望以不同的方式序列化这个类。 让我们为我们的类创建一个不同的序列化器和反序列化器：

```java
public static class PersonReversedSerializer extends JsonSerializer<Person> {
    static final String SEPARATOR = ", ";
    @Override
    public void serialize(
        Person person, JsonGenerator jsonGenerator, SerializerProvider serializerProvider)
        throws IOException {
        jsonGenerator.writeString(person.getLastName() + SEPARATOR + person.getFirstName());
    }
}

public static class PersonReversedDeserializer extends JsonDeserializer<Person> {
    @Override
    public Person deserialize(JsonParser jsonParser, DeserializationContext deserializationContext)
        throws IOException {
        String[] fields = jsonParser.getValueAsString().split(PersonReversedSerializer.SEPARATOR);
        return new Person(fields[1], fields[0]);
    }
}

```

现在，我们的 *Person* 将被序列化为“*lastName*, *firstName*”。

正如我们之前所做的那样，要在不修改 *Person* 的情况下使用这个新的序列化器和反序列化器，我们需要在我们的 mixin 类中指定它：

```java
@JsonSerialize(using = PersonReversedSerializer.class)
@JsonDeserialize(using = PersonReversedDeserializer.class)
public abstract class PersonMixin {}
```

现在，Jackson将使用这些序列化器，并将忽略*Person*中指定的序列化器。

如果我们只是想对一个特定的属性这样做也是一样的。



## 9、禁用Jackson注解

我们可以这样来禁用该实体上的所有Jackson注解：

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.databind.json.JsonMapper;
import lombok.Data;

// 假设我们有一个带Jackson注解的实体
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
@JsonPropertyOrder({ "name", "id" })
class Student {
    private Long id = 1L;
    private String name;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        // 让注解失效,此方法已经被标记@Deprecated
        objectMapper.disable(MapperFeature.USE_ANNOTATIONS);
        System.out.println(objectMapper.writeValueAsString(new Student()));
        // 输出内容: {"id":1,"name":null}

        // 方式二（2.10 后新增）:
        JsonMapper mapper = JsonMapper.builder().disable(MapperFeature.USE_ANNOTATIONS).build();
        System.out.println(mapper.writeValueAsString(new Student()));
        // 输出内容: {"id":1,"name":null}
    }
}
```



# 八、Jackson 注解扩展

## @JsonIdentityReference

使用指定的标识来序列化Java对象，而不是序列化整个对象

```java
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
@JsonIdentityReference(alwaysAsId = true)
public class BeanWithoutIdentityReference {
    private int id;
    private String name;
}
```



## @JsonAppend

运行在序列化时添加额外的属性

```java
import com.fasterxml.jackson.databind.annotation.JsonAppend;
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@JsonAppend(attrs = {@JsonAppend.Attr(value = "version")})
class BeanWithAppend {
    private Integer id;
    private String name;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        BeanWithAppend bean = new BeanWithAppend(2, "Bean With Append Annotation");
        ObjectWriter writer = new ObjectMapper().writerFor(BeanWithAppend.class).withAttribute("version", "1.0");
        String jsonString = writer.writeValueAsString(bean);
        System.out.println(jsonString);
        // 输出内容: {"id":2,"name":"Bean With Append Annotation","version":"1.0"}
    }
}
```



## @JsonNaming

指定序列化的时候属性命名方式

有四种选项：

- KEBAB_CASE：由连字符分割，例如：kebab-case

- LOWER_CASE：所有的字母都转换为小写，例如：lowercase

- SNAKE_CASE：所有的字母都转换为小写，并且由下划线分割，例如：snake_case

- UPPER_CAMEL_CASE：所有名称元素，包括第一个元素，都以大写字母开头，后跟小写字母，并且没有分隔符，例如：UpperCamelCase

```java
import com.fasterxml.jackson.databind.annotation.JsonNaming;
import com.fasterxml.jackson.databind.*;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
class NamingBean {
    private Integer id;
    private String beanName;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        NamingBean bean = new NamingBean(2, "Naming Bean");
        String jsonString = new ObjectMapper().writeValueAsString(bean);
        System.out.println(jsonString);
        // 输出内容: {"id":2,"bean_name":"Naming Bean"}
    }
}
```



## @JsonPropertyDescription

用于生成字段的描述信息。Jackson的独立模块JSON Schema提供了创建Json信息表(Json schemas)来描述Java的类型信息. 信息表可用于输出我们期望的序列化Java对象, 或者在反序列化前验证JSON文档（Document）

首先需要导入JSON Schema依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jsonSchema</artifactId>
    <version>2.13.3</version>
</dependency>
```

注解@JsonPropertyDescription允许把人类可读的描述信息, 附加在要创建的Json信息表的description属性

```java
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.module.jsonSchema.JsonSchema;
import com.fasterxml.jackson.module.jsonSchema.factories.SchemaFactoryWrapper;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
class PropertyDescriptionBean {
    private int id;
    @JsonPropertyDescription("This is a description of the name property")
    private String name;
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        SchemaFactoryWrapper wrapper = new SchemaFactoryWrapper();
        objectMapper.acceptJsonFormatVisitor(PropertyDescriptionBean.class, wrapper);
        JsonSchema jsonSchema = wrapper.finalSchema();
        String jsonString = objectMapper.writeValueAsString(jsonSchema);
        System.out.println(jsonString);
    }
}
```

打印结果如下（JSON经过格式化）

```json
{
    "type": "object",
    "id": "urn:jsonschema:PropertyDescriptionBean",
    "properties": {
        "id": {
            "type": "integer"
        },
        "name": {
            "type": "string",
            "description": "This is a description of the name property"
        }
    }
}
```



## @JsonPOJOBuilder

自定义生成器类，来控制JSON的反序列化行为。该注解用来配置一个builder类用于定制反序列化过程，尤其是当JSON文档中属性命名习惯和POJO类对象的属性不同

@JsonPOJOBuilder有两个属性

- buildMethodName：将JSON字段绑定到bean的属性后，用于实例化预期bean的无参构造的名称。默认名称为build。

- withPrefix：用于自动检测json和bean属性之间匹配的名称前缀。默认前缀为with。
  

1、假设我们要反序列化的JSON如下

```json
{
    "id": 5,
    "name": "POJO Builder Bean"
}
```

2、对应的POJO（可以看到Bean属性的名称和JSON字符串中对应属性的名称不同. 这就是@JsonPOJOBuilder发挥作用的地方）：

```java
@JsonDeserialize(builder = BeanBuilder.class)
public class POJOBuilderBean {
    private int identity;
    private String beanName;
}
```
3、对应的生成器：

```java
@JsonPOJOBuilder(buildMethodName = "createBean", withPrefix = "construct")
public class BeanBuilder {
    private int idValue;
    private String nameValue;
    public BeanBuilder constructId(int id) {
        idValue = id;
        return this;
    }

    public BeanBuilder constructName(String name) {
        nameValue = name;
        return this;
    }

    public POJOBuilderBean createBean() {
        return new POJOBuilderBean(idValue, nameValue);
    }
}
```
4、使用ObjectMapper反序列化：

```java
String jsonString = "{\"id\":5,\"name\":\"POJO Builder Bean\"}";
POJOBuilderBean bean = mapper.readValue(jsonString, POJOBuilderBean.class);
```

5、完整代码：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonPOJOBuilder;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@JsonDeserialize(builder = BeanBuilder.class)
class POJOBuilderBean {
    private int identity;
    private String beanName;
}

/**
 * 我们配置了注解@JsonPOJOBuilder的参数, 用createBean方法作为build方法, 用construct前缀来匹配属性名
 */
@JsonPOJOBuilder(buildMethodName = "createBean", withPrefix = "construct")
class BeanBuilder {
    private int idValue;
    private String nameValue;

    public BeanBuilder constructId(int id) {
        idValue = id;
        return this;
    }

    public BeanBuilder constructName(String name) {
        nameValue = name;
        return this;
    }

    public POJOBuilderBean createBean() {
        return new POJOBuilderBean(idValue, nameValue);
    }
}

public class TestMain {
    public static void main(String[] args) throws Exception {
        String jsonString = "{\"id\":5,\"name\":\"POJO Builder Bean\"}";
        POJOBuilderBean bean = new ObjectMapper().readValue(jsonString, POJOBuilderBean.class);
        System.out.println(bean);
        // 输出内容: POJOBuilderBean(identity=5, beanName=POJO Builder Bean)
    }
}
```



# 九、SpringBoot & Jackson

通常我们在使用SpringBoot框架时，如果没有特别指定接口的序列化类型，则会使用SpringBoot框架默认集成的Jackson框架进行处理，通过Jackson框架将服务端响应的数据序列化成JSON格式的数据。

目前市面上针对JSON序列化的框架很多,比较出名的就是[Jackson](https://github.com/FasterXML/jackson)、[Gson](https://github.com/google/gson)、[FastJson](https://github.com/alibaba/fastjson)。如果开发者对序列化框架没有特别的要求的情况下，个人建议是直接使用SpringBoot框架默认集成的Jackson，**没有必要**进行更换。



## 1、序列化 & 反序列化 时间格式

在我们的接口中，针对时间类型的字段序列化是最常见的需求之一，一般前后端开发人员会针对时间字段统一进行约束，这样有助于在编码开发时，统一编码规范。

在Spring Boot框架中，如果使用Jackson处理框架，并且没有任何配置的情况下，Jackson针对不同时间类型字段，序列化的格式也会不尽相同。

先来看一个简单示例，User.java实体类编码如下：

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Calendar;
import java.util.Date;
import java.util.Random;

@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    private LocalDateTime localDateTime;
    private LocalDate localDate;
    private Date date;
    private Timestamp timestamp;
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

接口代码层也很简单，返回一个User的实体对象即可，代码如下：

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(User.buildOne());
    }
}
```

如果我们对框架代码没有任何的配置，此时我们通过调用接口/queryOne，拿到的返回结果数据如下：

```json
{
    "name": "姓名-0",
    "age": 1,
    "localDateTime": "2022-11-22T20:40:20.3144084",
    "localDate": "2037-11-22",
    "date": "2032-11-22T12:40:20.314+00:00",
    "timestamp": "2027-11-22T12:40:20.314+00:00",
    "calendar": "2022-11-22T12:40:20.318+00:00"
}
```

Jackson序列化框架针对四个不同的时间类型字段，序列化处理的操作是不同的，如果我们对时间字段有格式化的要求时，我们应该如何处理呢？



### 1、@JsonFormat 注解配置

最直接也是最简单的一种方式，是我们通过使用Jackson提供的@JsonFormat注解，对需要格式化处理的时间字段进行标注，在@JsonFormat注解中写上我们的时间格式化字符。

1、User.java代码如下：

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Calendar;
import java.util.Date;
import java.util.Random;

@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    // 需要注意的是: LocalDateTime 格式化必须带上HH mm ss, 不能只有yyyy MM dd, 不然反序列化会报错,序列化还是正常
    // 总结: 格式可以变, 年月日时分秒不可少.
    @JsonFormat(pattern = "yyyy_MM_dd HH_mm_ss")
    private LocalDateTime localDateTime;
    @JsonFormat(pattern = "yyyy_MM_dd")
    private LocalDate localDate;
    @JsonFormat(pattern = "yyyyMMdd")
    private Date date;
    @JsonFormat(pattern = "yyyyMMdd")
    private Timestamp timestamp;
    @JsonFormat(pattern = "yyyyMMdd HHmmss")
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

2、调用Controller接口序列化对象

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(User.buildOne());
    }
}
```

3、返回结果如下，可以发现时间格式都按照指定格式序列化输出成字符串了

```json
{
    "name": "姓名-4",
    "age": 27,
    "localDateTime": "2022_11_22 20_51_35",
    "localDate": "2037_11_22",
    "date": "20321122",
    "timestamp": "20271122",
    "calendar": "20221122 125135"
}
```

4、现在开始尝试反序列化对象，还是使用如上的User.java。我们把/queryOne返回的结果当成request Body。现在写一个Post接口

```java
@RestController
public class TestController {
    // ...GET接口省略了
    @PostMapping("/queryTwo")
    public ResponseEntity<User> queryTwo(@RequestBody User user) {
        System.out.println(user);
        return ResponseEntity.ok(user);
    }
}
```

5、然后调用当前接口查看控制台输出：

```java
User(name=姓名-4, 
     age=27, 
     localDateTime=2022-11-22T20:51:35, 
     localDate=2037-11-22, 
     date=Mon Nov 22 08:00:00 CST 2032, 
     timestamp=2027-11-22 08:00:00.0,
     calendar=java.util.GregorianCalendar[time=1669121495000,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="UTC",offset=0,dstSavings=0,useDaylight=false,transitions=0,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2022,MONTH=10,WEEK_OF_YEAR=48,WEEK_OF_MONTH=4,DAY_OF_MONTH=22,DAY_OF_YEAR=326,DAY_OF_WEEK=3,DAY_OF_WEEK_IN_MONTH=4,AM_PM=1,HOUR=0,HOUR_OF_DAY=12,MINUTE=51,SECOND=35,MILLISECOND=0,ZONE_OFFSET=0,DST_OFFSET=0])
```

可以看得出来增加@JsonFormat直接后，不管是序列化或者反序列化都能正常格式化时间了。



### 2、application.yml 全局配置

通过@JsonFormat注解的方式虽然能解决问题，但是我们在实际的开发当中，涉及到的时间字段会非常多，如果全部都用注解的方式对项目中的时间字段进行标注，那开发的工作量也会很大，并且多团队一起协同编码时，难免会存在遗漏的情况，因此，@JsonFormat注解只适用于针对特定的接口，特定的场景下，对序列化响应的时间字段进行约束，而在全局的角度来看，开发者应该考虑通过在application.yml配置文件中进行全局配置。针对SpringBoot框架中Jackson的全局配置，我们在application.yml进行配置时，IDEA等编辑器会给出相应的提示，只需要输入`spring.jackson.`IDEA就会自动弹窗提示。

开发者可以通过`org.springframework.boot.autoconfigure.jackson.JacksonProperties.java`查看所有配置的源码信息

| 配置属性      | 说明                                       |
| ------------- | ------------------------------------------ |
| `date-format` | 日期字段格式化，例如:`yyyy-MM-dd HH:mm:ss` |

我们从Spring Boot的源码中可以看到对Jackson的时间处理逻辑，JacksonAutoConfiguration.java中部分代码如下：

```java
private void configureDateFormat(Jackson2ObjectMapperBuilder builder) {
    // We support a fully qualified class name extending DateFormat or a date
    // pattern string value
    String dateFormat = this.jacksonProperties.getDateFormat();
    if (dateFormat != null) {
        try {
            Class<?> dateFormatClass = ClassUtils.forName(dateFormat, null);
            builder.dateFormat((DateFormat) BeanUtils.instantiateClass(dateFormatClass));
        }
        catch (ClassNotFoundException ex) {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat(dateFormat);
            // Since Jackson 2.6.3 we always need to set a TimeZone (see
            // gh-4170). If none in our properties fallback to the Jackson's
            // default
            TimeZone timeZone = this.jacksonProperties.getTimeZone();
            if (timeZone == null) {
                timeZone = new ObjectMapper().getSerializationConfig().getTimeZone();
            }
            simpleDateFormat.setTimeZone(timeZone);
            builder.dateFormat(simpleDateFormat);
        }
    }
}
```

从上面的代码中，我们可以看到的处理逻辑：

- 从yml配置文件中拿到dateFormat属性字段
- 首先通过ClassUtils.forName方法来判断开发者配置的是否是格式化类，如果配置的是格式化类，则直接配置dateFormat属性
- 类找不到的情况下,捕获ClassNotFoundException异常，默认使用JDK自带的SimpleDateFormat类进行初始化

***

1、针对全局日期字段的格式化处理，我们只需要使用`date-format`属性进行配置即可，`application.yml`配置如下：

```yaml
spring:
  jackson:
    date-format: yyyy_MM_dd HH_mm_ss
    # 此配置对 LocalDateTime、LocalDate、LocalTime 无效（不管配置成什么格式, yyyy_MM_dd 也是无效）
```

2、修改User.java类，删除所有@JsonFormat注解

```java
import lombok.Data;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Calendar;
import java.util.Date;
import java.util.Random;

@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    private LocalDateTime localDateTime;
    private LocalDate localDate;
    private Date date;
    private Timestamp timestamp;
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

3、调用Controller的queryOne和queryTwo接口（序列化于反序列化）

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(User.buildOne());
    }

    @PostMapping("/queryTwo")
    public ResponseEntity<User> queryTwo(@RequestBody User user) {
        System.out.println(user);
        return ResponseEntity.ok(user);
    }
}
```

4、queryOne接口序列化输出结果如下：

```json
{
    "name": "姓名-1",
    "age": 68,
    "localDateTime": "2022-11-22T22:07:47.0203599",
    "localDate": "2037-11-22",
    "date": "2032_11_22 14_07_47",
    "timestamp": "2027_11_22 14_07_47",
    "calendar": "2022_11_22 14_07_47"
}
```

注意：可以查看到 LocalDateTime 与 LocalDate 时间没有被格式化到。

5.1、接下来再来调用queryTwo接口反序列化，requestBody 如下：

```json
{
    "name": "姓名-1",
    "age": 68,
    "localDateTime": "2022-11-22 22_07_47",
    "localDate": "2037_11_22",
    "date": "2032_11_22 14_07_47",
    "timestamp": "2027_11_22 14_07_47",
    "calendar": "2022_11_22 14_07_47"
}
```

5.2、可以看到控制台报如下错误（这是因为Jackson SpringBoot全局配置不支持 LocalDateTime、LocalDate、LocalTime）

```shell
2022-11-22 21:32:39.830  WARN 11552 --- [nio-8080-exec-2] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `java.time.LocalDateTime` from String "2022_11_22": Failed to deserialize java.time.LocalDateTime: (java.time.format.DateTimeParseException) Text '2022_11_22' could not be parsed at index 4; nested exception is com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type `java.time.LocalDateTime` from String "2022_11_22": Failed to deserialize java.time.LocalDateTime: (java.time.format.DateTimeParseException) Text '2022_11_22' could not be parsed at index 4
 at [Source: (PushbackInputStream); line: 4, column: 20] (through reference chain: com.example.springbootjackson.model.User["localDateTime"])]
```

**解决方法一**：

5.3、重新修改User.java，在开启全局配置情况下，使用@JsonFormat重写LocalDateTime等失效字段，代码如下：

```java
package com.example.springbootjackson.model;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Calendar;
import java.util.Date;
import java.util.Random;

@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    // 需要注意的是: LocalDateTime 格式化必须带上HH mm ss, 不能只有yyyy MM dd, 不然反序列化会报错,序列化还是正常
    // 总结: 格式可以变, 年月日时分秒不可少.
    @JsonFormat(pattern = "yyyy_MM_dd HH_mm_ss")
    private LocalDateTime localDateTime;
    @JsonFormat(pattern = "yyyy_MM_dd")
    private LocalDate localDate;
    private Date date;
    private Timestamp timestamp;
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

5.4、调用queryOne接口（序列化）返回结果如下：

```json
{
    "name": "姓名-3",
    "age": 91,
    "localDateTime": "2022_11_22 22_23_19",
    "localDate": "2037_11_22",
    "date": "2032_11_22 14_23_19",
    "timestamp": "2027_11_22 14_23_19",
    "calendar": "2022_11_22 14_23_19"
}
```

5.5、调用queryTwo接口反序列化，requestBody 为上面的json，调用接口返回结果如下：

```java
User(name=姓名-3, 
     age=91, 
     localDateTime=2022-11-22T22:23:19, 
     localDate=2037-11-22, 
     date=Mon Nov 22 22:23:19 CST 2032, timestamp=2027-11-22 22:23:19.0, 
     calendar=java.util.GregorianCalendar[time=1669126999000,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="UTC",offset=0,dstSavings=0,useDaylight=false,transitions=0,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2022,MONTH=10,WEEK_OF_YEAR=48,WEEK_OF_MONTH=4,DAY_OF_MONTH=22,DAY_OF_YEAR=326,DAY_OF_WEEK=3,DAY_OF_WEEK_IN_MONTH=4,AM_PM=1,HOUR=2,HOUR_OF_DAY=14,MINUTE=23,SECOND=19,MILLISECOND=0,ZONE_OFFSET=0,DST_OFFSET=0])
```



### 3、Jackson 注解与全局格式化时间总结

1. 可以发现SpringBoot在格式化时间格式的时候可以使用两种方式，一个是注解单一字段格式化，另一种是全局配置
2. @JsonFormat 注解单一字段格式化优缺点：
   - 优点：可以格式化（序列化与反序列化）任意时间类型
   - 缺点：如果项目时间格式比较统一，那么每个字段都配置相同的操作，显示很冗余。
3. application.yml 全局时间格式化配置优缺点：
   - 优点：可以不需要每个字段都配置，配置一个全局的，如果有个别需要单独处理使用@JsonFormat重写即可
   - 缺点：无法格式化（序列化与反序列化）LocalDateTime、LocalDate、LocalTime 时间类型（可以搭配@JsonFormat解决）
4. LocalDateTime、LocalDate、LocalTime 全局配置失效，除了 使用全局配置 + @JsonFormat，还可以使用其他方案请看下篇



### 4、SpringBoot LocalDateTime 时间格式失效解决

> 方案一：Jackson序列化LocalDateTime，引包 + 配置ObjectMapper

1、Java序列化工具用的是Jackson，实体类上加注解，可以让Date序列化生效，但是LocalDateTime属性不生效

```yaml
spring:
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    default-property-inclusion: always
```

2、添加使LocalDateTime生效的依赖（SpringBoot-2.5.0版本的mvc-starter已经包含该依赖，所以可以不引入）

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.12.3</version>
</dependency>
```

3、SpringBoot 配置 ObjectMapper

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import org.springframework.stereotype.Component;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Component
public class LocalDateTimeSerializerConfig {
    DateTimeFormatter localDateTimeFormatter = DateTimeFormatter.ofPattern("yyyy_MM_dd HH_mm_ss");
    DateTimeFormatter localDateFormatter = DateTimeFormatter.ofPattern("yyyy_MM_dd");

    @Bean
    public ObjectMapper serializingObjectMapper() {
        JavaTimeModule module = new JavaTimeModule();
        LocalDateTimeDeserializer dateTimeDeserializer = new LocalDateTimeDeserializer(localDateTimeFormatter);
        LocalDateTimeSerializer dateTimeSerializer = new LocalDateTimeSerializer(localDateTimeFormatter);
        // LocalDateTime类型 序列化与反序列化同时处理
        module.addDeserializer(LocalDateTime.class, dateTimeDeserializer);
        module.addSerializer(LocalDateTime.class, dateTimeSerializer);
        // LocalDate类型 序列化与反序列化同时处理
        module.addDeserializer(LocalDate.class, new LocalDateDeserializer(localDateFormatter));
        module.addSerializer(LocalDate.class, new LocalDateSerializer(localDateFormatter));
        return Jackson2ObjectMapperBuilder.json().modules(module)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS).build();
    }
}
```

***

> 方案二：Jackson序列化LocalDateTime，依赖 + 重写 WebMvcConfigurer 方法 + 配置ObjectMapper

1、添加使LocalDateTime生效的依赖（SpringBoot-2.5.0版本的mvc-starter已经包含该依赖，所以可以不引入）

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.12.3</version>
</dependency>
```

2、出参入参处理，**实现**WebMvcConfigurer的extendMessageConverters方法

```java
package com.example.springbootjackson.config;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

@Configuration
public class DateTimeSerializerConfig implements WebMvcConfigurer {

    @Value("${spring.jackson.date-format}")
    private String pattern;

    /**
     * 时间处理
     * 使用此方法, 以下 spring-boot: jackson时间格式化 配置 将会失效
     * spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
     * spring.jackson.time-zone=GMT+8
     * 原因: 会覆盖 @EnableAutoConfiguration 关于 WebMvcAutoConfiguration 的配置
     */
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();

        // LocalDateTime 格式化
        JavaTimeModule module = new JavaTimeModule();
        LocalDateTimeDeserializer dateTimeDeserializer = new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(pattern));
        LocalDateTimeSerializer dateTimeSerializer = new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
        module.addDeserializer(LocalDateTime.class, dateTimeDeserializer);
        module.addSerializer(LocalDateTime.class, dateTimeSerializer);
        ObjectMapper objectMapper = Jackson2ObjectMapperBuilder.json().modules(module)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS).build();
        
        // LocalDate 格式化如果有需要可以自行定义

        // Date 时间格式化
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        objectMapper.setDateFormat(new SimpleDateFormat(pattern));

        // 设置格式化内容
        converter.setObjectMapper(objectMapper);
        converters.add(0, converter);
    }
}
```

> 配置方案一 或者 方案二：然后执行如下代码。

```java
@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    private LocalDateTime localDateTime;
    private LocalDate localDate;
    private Date date;
    private Timestamp timestamp;
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(User.buildOne());
    }

    @PostMapping("/queryTwo")
    public ResponseEntity<User> queryTwo(@RequestBody User user) {
        System.out.println(user);
        return ResponseEntity.ok(user);
    }
}
```

执行后可以发现结果都是正常



> 5、SpringBoot  GET 接口入参时间格式化填(天)坑
>
> GET请求及POST表单日期时间字符串格式转换的区别：这种情况要和时间作为JSON字符串时区别对待，因为前端JSON转后端POJO底层使用的是JSON序列化Jackson工具（HttpMessgeConverter）；而时间字符串作为普通请求参数传入时，转换用的是Converter，两者在处理方式上是有区别。
>
> - POST 请求时：时间参数作为JSON字符串，从前端JSON转到后端POJO，底层使用的是JSON序列化Jackson工具（HttpMessgeConverter），Header中Content-Type类型为application/json。
> - GET 请求时：而时间字符串作为普通请求参数传入时，转换用的是SpringWeb框架的Converter。



## 2、SpringBoot Jackson 配置选项

在上面的时间字段序列化处理，我们已经知道了如何配置，那么在SpringBoot的框架中，针对Jackson的各个配置项主要包含哪些呢？我们通过IDEA的提示可以看到：会出现12个配置项，并且有一个已经过时了。这里就讲解11个正在使用的（本人使用的是SpringBoot 2.5.0 version）

```properties
spring.jackson.locale=zh_CN
spring.jackson.time-zone=Asia/Shanghai
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.property-naming-strategy=SNAKE_CASE
spring.jackson.default-property-inclusion=always
spring.jackson.serialization.write-empty-json-arrays=true
spring.jackson.deserialization.read-enums-using-to-string=true
spring.jackson.mapper.use-annotations=true
spring.jackson.parser.ignore-undefined=true
spring.jackson.generator.ignore-unknown=true
spring.jackson.visibility.field=any
```



### 1、date-format 日期格式化

date-format 在前面我们已经知道了该属性的作用，主要是针对日期字段的格式化

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```



### 2、time-zone 时区

time-zone 字段也是和日期字段类型，使用不同的时区，最终日期类型字段响应的结果会不一样。

时区的表示方法有两种：

- 指定时区的名称，例如：Asia/Shanghai、Asia/Hong_Kong、America/Los_Angeles
- 通过格林威治平时`GMT`针对时分秒做`+`或者`-`自定义操作，例如：GMT+8

时区配置与代码示例如下：

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: America/Los_Angeles
```

```java
@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    private LocalDateTime localDateTime;
    private LocalDate localDate;
    private Date date;
    private Timestamp timestamp;
    private Calendar calendar;

    public static User buildOne() {
        User user = new User();
        LocalDateTime now = LocalDateTime.now();
        user.setLocalDateTime(now);
        user.setLocalDate(now.plusYears(15).toLocalDate());
        user.setDate(Date.from(now.plusYears(10).atZone(ZoneId.systemDefault()).toInstant()));
        user.setTimestamp(Timestamp.from(now.plusYears(5).atZone(ZoneId.systemDefault()).toInstant()));
        user.setCalendar(Calendar.getInstance());
        return user;
    }
}
```

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(User.buildOne());
    }
}
```

```json
{
    "name": "姓名-3",
    "age": 16,
    "localDateTime": "2022-11-23T18:31:12.1163728",
    "localDate": "2037-11-23",
    "date": "2032-11-23 02:31:12",
    "timestamp": "2027-11-23 02:31:12",
    "calendar": "2022-11-23 02:31:12"
}
```

我们在结合代码来分析，由于洛杉矶时区与上海时区相差16个小时，因此，Jackson 框架针对日期的序列化时，分别做了不同类型的处理，但我们也能看出差别

- LocalDateTime、LocalDate 类型的字段，Jackson的时区设置不会对该字段产生影响（因为这两个日期类型自带时区属性）
- Date、Timestamp、Calendar 类型的字段受Jackson序列化框架的时区设置影响

***

另外一种方式是通过格林威治平时（GMT）做加减法，主要有两种格式支持：

- GMT+HHMM或者GMT-HHMM或者GMT+H：其中HH代表的是小时数，MM代表的是分钟数，取值范围是0-9。例如我们常见的GMT+8代表东八区，也就是北京时间
- GMT+HH:MM 或 GMT-HH:MM：其中HH代表的是小时数，MM代表的是分钟数，取值范围是0-9，和上面意思差不多

SpringBoot 配置文件配置如下，配置默认是GMT0时区（与北京时间相差8小时），还是继续执行上面的代码。查看打印结果：

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT
```

```json
{
    "name": "姓名-1",
    "age": 41,
    "localDateTime": "2022-11-23T21:38:51.8442623",
    "localDate": "2037-11-23",
    "date": "2032-11-23 13:38:51",
    "timestamp": "2027-11-23 13:38:51",
    "calendar": "2022-11-23 13:38:51"
}
```

自己写测试代码进行测试，示例如下：

```java
public class TimeTest {
    public static void main(String[] args) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime.format(dateTimeFormatter));
        System.out.println(LocalDateTime.now(ZoneId.of("GMT+0901")).format(dateTimeFormatter));
        System.out.println(LocalDateTime.now(ZoneId.of("GMT+09:01")).format(dateTimeFormatter));
    }
}
```

```shell
2022-11-23 18:36:04
2022-11-23 19:37:04
2022-11-23 19:37:04
```



### 3、locale 本地化

JSON序列化时Locale的变量设置，简单说就是指定当地时区，例如：zh_HK、en_US

```yaml
spring:
  jackson:
    locale: zh_HK
```

一般locale与time-zone不会同时配置，例如我们在代码或者数据库中取出带有时区的时间，想序列化本地时区，就可以使用此配置。

下面我们来看locale配置的使用示例，设置locale为zh_HK：

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    locale: zh_HK
```

```java
@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    private LocalDateTime localDateTime = LocalDateTime.now();
    private Date date = Date.from(Instant.now().atZone(ZoneId.of("GMT")).toInstant());
    private Timestamp timestamp = Timestamp.from(Instant.now().atZone(ZoneId.of("GMT")).toInstant());
}
```

```java
@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        User user = new User();
        System.out.println(user);
        return ResponseEntity.ok(user);
    }
}
```

控制台及前端返回如下：

```java
User(name=姓名-1, age=93, 
     localDateTime=2022-11-23T22:01:10.751406300, 
     date=Thu Nov 24 06:01:10 CST 2022, 
     timestamp=2022-11-24 06:01:10.7514063)
```

```json
{
    "name": "姓名-0",
    "age": 71,
    "localDateTime": "2022-11-23T22:01:10.7614191",
    "date": "2022-11-23 22:01:10",
    "timestamp": "2022-11-23 22:01:10"
}
```

从结果可以看出，控制台输出的date和timestamp，与返回前端的JSON不一致。因为在返回前端时区被设置成了本机的了。



### 4、visibility 访问级别

Jackson支持从私有字段中读取值，但是默认情况下不这样做，如果我们的项目中存在不同的序列化反序列化需求，那么我们可以在配置文件中对visibility进行配置。修改User.java代码中的name属性的get方法修饰符从public变更为private。代码如下：

```java
@Data
public class User {
    private String name = "姓名-" + new Random().nextInt(5);
    private Integer age = new Random().nextInt(100);
    // getter方法修饰符从public修改为private
    private String getName() {
        return name;
    }
}

@RestController
public class TestController {
    @GetMapping("/queryOne")
    public ResponseEntity<User> queryOne() {
        return ResponseEntity.ok(new User());
    }
}
```

```java
{
    "age": 72
}
```

从结果中我们可以看到，由于我们将name属性的getter方法设置为了private，因此Jackson在序列化时，没有拿到该字段。

此时我们修改 application.yml 的配置，我们通过将getter设置为any级别的类型，再调用/queryOne接口，响应结果如下：

```yaml
spring:
  jackson:
    visibility:
      getter: any
    # 开启visibility私有字段或方法访问级别, 可以让私有属性或者方法被Jackson序列化或反序列化到
    # 枚举类PropertyAccessor中的枚举属性为key, 枚举类JsonAutoDetect.Visibility中的枚举属性为value
```

开启配置后重新调用/queryOne接口接口查看返回结果，可以发现私有方法中的name被序列化到了：

```json
{
    "name": "姓名-2",
    "age": 88
}
```

可以发现在设置了visibility属性后，private修改的方法也可以被序列化了。这代表即使name字段的属性和getter方法都是private，但是jackson还是获取到了该成员变量的值，并且进行了序列化处理。**通过设置visibility属性即可达到上面的效果**。开发者根据自己的需要自行进行选择。




### 5、property-naming-strategy 属性命名策略

通常比较常见的我们针对Java代码中的实体类属性一般都是驼峰命名法(Camel-Case)，但是Jackson序列化框架也提供了更多的序列化策略，而property-naming-strategy就是配置该属性的。

先来看SpringBoot框架如何配置Jackson的命名策略：JacksonAutoConfiguration.java

```java
private void configurePropertyNamingStrategyField(Jackson2ObjectMapperBuilder builder, String fieldName) {
    // Find the field (this way we automatically support new constants
    // that may be added by Jackson in the future)
    Field field = ReflectionUtils.findField(PropertyNamingStrategy.class, fieldName,
                                            PropertyNamingStrategy.class);
    Assert.notNull(field, () -> "Constant named '" + fieldName + "' not found on "
                   + PropertyNamingStrategy.class.getName());
    try {
        builder.propertyNamingStrategy((PropertyNamingStrategy) field.get(null));
    }
    catch (Exception ex) {
        throw new IllegalStateException(ex);
    }
}
```

通过反射，直接获取PropertyNamingStrategy类中的成员变量的值。PropertyNamingStrategy定义了Jackson(2.11.4)框架中的命名策略常量成员变量。

```java
package com.fasterxml.jackson.databind;

//other import

public class PropertyNamingStrategy // NOTE: was abstract until 2.7
    implements java.io.Serializable
{
    /**
     * Naming convention used in languages like C, where words are in lower-case
     * letters, separated by underscores.
     * See {@link SnakeCaseStrategy} for details.
     *
     * @since 2.7 (was formerly called {@link #CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES})
     */
    public static final PropertyNamingStrategy SNAKE_CASE = new SnakeCaseStrategy();

    /**
     * Naming convention used in languages like Pascal, where words are capitalized
     * and no separator is used between words.
     * See {@link PascalCaseStrategy} for details.
     *
     * @since 2.7 (was formerly called {@link #PASCAL_CASE_TO_CAMEL_CASE})
     */
    public static final PropertyNamingStrategy UPPER_CAMEL_CASE = new UpperCamelCaseStrategy();

    /**
     * Naming convention used in Java, where words other than first are capitalized
     * and no separator is used between words. Since this is the native Java naming convention,
     * naming strategy will not do any transformation between names in data (JSON) and
     * POJOS.
     *
     * @since 2.7 (was formerly called {@link #PASCAL_CASE_TO_CAMEL_CASE})
     */
    public static final PropertyNamingStrategy LOWER_CAMEL_CASE = new PropertyNamingStrategy();

    /**
     * Naming convention in which all words of the logical name are in lower case, and
     * no separator is used between words.
     * See {@link LowerCaseStrategy} for details.
     * 
     * @since 2.4
     */
    public static final PropertyNamingStrategy LOWER_CASE = new LowerCaseStrategy();

    /**
     * Naming convention used in languages like Lisp, where words are in lower-case
     * letters, separated by hyphens.
     * See {@link KebabCaseStrategy} for details.
     * 
     * @since 2.7
     */
    public static final PropertyNamingStrategy KEBAB_CASE = new KebabCaseStrategy();

    /**
     * Naming convention widely used as configuration properties name, where words are in
     * lower-case letters, separated by dots.
     * See {@link LowerDotCaseStrategy} for details.
     *
     * @since 2.10
     */
    public static final PropertyNamingStrategy LOWER_DOT_CASE = new LowerDotCaseStrategy();

    //others...
}
```

从源码中我们可以看到，有六种策略供我们进行配置，策略详解如下：

1、**SNAKE_CASE**：主要包含的规则，详见 SnakeCaseStrategy：

- Java属性名称中所有大写的字符都会转换为两个字符，下划线和该字符的小写形式，例如userName会转换为user_name,对于连续性的大写字符，近第一个进行下划线转换，后面的大小字符则是小写，例如theWWW会转换为the_www
- 对于首字母大写的情况，近转成小写，例如:Results会转换为results，并不会转换为_results
- 针对属性中已经包含下划线的情况，仅做小写转换处理
- 下划线出现在首位的情况下，会被去除处理，例如属性名：_user会被转换为user

```shell
Java 对象属性：userName
JSON 返回属性：user_name
```

2、**UPPER_CAMEL_CASE**：顾名思义，驼峰命名法的规则，只是首字母会转换为大写，详见 [UpperCamelCaseStrategy](https://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/PropertyNamingStrategy.UpperCamelCaseStrategy.html)

```
Java 对象属性：userName
JSON 返回属性：UserName
```

3、**LOWER_CAMEL_CASE**：效果和`UPPER_CAMEL_CASE`正好相反，其首字母会变成小写，详见 [LowerCamelCaseStrategy](https://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/PropertyNamingStrategies.LowerCamelCaseStrategy.html)

```java
Java 对象属性：userName
JSON 返回属性：userName
```

4、LOWER_CASE：从命名来看很明显，将属性名 全部转为小写，详见 [LowerCaseStrategy](https://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/PropertyNamingStrategy.LowerCaseStrategy.html)

```
Java 对象属性：userName
JSON 返回属性：username
```

5、KEBAB_CASE：KEBAB_CASE策略和SNAKE_CASE规则类似，只是下划线变成了横线`-`,详见 [KebabCaseStrategy](https://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/PropertyNamingStrategy.KebabCaseStrategy.html)

```java
Java 对象属性：userName
JSON 返回属性：user-name
```

6、LOWER_DOT_CASE：当前策略和KEBAB_CASE规则相似，只是由横线变成了点`.`，详见 [LowerDotCaseStrategy](https://www.javadoc.io/doc/com.fasterxml.jackson.core/jackson-databind/latest/com/fasterxml/jackson/databind/PropertyNamingStrategy.LowerDotCaseStrategy.html)

```java
Java 对象属性：userName
JSON 返回属性：user.name
```

**总结**：看了上面这么多属性名称的策略，其实每一种类型只是不同的场景下才需要，如果上面Jackson给定的默认策略名称无法满足，我们从源码中也能看到，通过自定义实现类，也能满足企业的个性化需求，非常方便。



### 6、mapper 通用功能开关配置

mapper属性是一个Map类型，主要是针对MapperFeature定义开关属性，是否启用这些特性。

```java
/**
 * Jackson general purpose on/off features.
 */
private final Map<MapperFeature, Boolean> mapper = new EnumMap<>(MapperFeature.class);
```

MapperFeature.java是一个枚举类型，对当前Jackson的一些特性通过枚举变量的方式来定义开关属性，也是方便使用者来使用的。

```java
public enum MapperFeature implements ConfigFeature {
    USE_ANNOTATIONS(true),
    USE_GETTERS_AS_SETTERS(true),
    PROPAGATE_TRANSIENT_MARKER(false),
    AUTO_DETECT_CREATORS(true),
    //.......
}
```

主要包含以下枚举变量：

- USE_ANNOTATIONS
- USE_GETTERS_AS_SETTERS
- PROPAGATE_TRANSIENT_MARKER
- AUTO_DETECT_CREATORS
- AUTO_DETECT_FIELDS
- AUTO_DETECT_GETTERS
- AUTO_DETECT_IS_GETTERS
- AUTO_DETECT_SETTERS
- REQUIRE_SETTERS_FOR_GETTERS
- ALLOW_FINAL_FIELDS_AS_MUTATORS
- INFER_PROPERTY_MUTATORS
- INFER_CREATOR_FROM_CONSTRUCTOR_PROPERTIES
- CAN_OVERRIDE_ACCESS_MODIFIERS
- OVERRIDE_PUBLIC_ACCESS_MODIFIERS
- USE_STATIC_TYPING
- USE_BASE_TYPE_AS_DEFAULT_IMPL
- DEFAULT_VIEW_INCLUSION
- SORT_PROPERTIES_ALPHABETICALLY
- ACCEPT_CASE_INSENSITIVE_PROPERTIES
- ACCEPT_CASE_INSENSITIVE_ENUMS
- ACCEPT_CASE_INSENSITIVE_VALUES
- USE_WRAPPER_NAME_AS_PROPERTY_NAME
- USE_STD_BEAN_NAMING
- ALLOW_EXPLICIT_PROPERTY_RENAMING
- ALLOW_COERCION_OF_SCALARS
- IGNORE_DUPLICATE_MODULE_REGISTRATIONS
- IGNORE_MERGE_FOR_UNMERGEABLE
- BLOCK_UNSAFE_POLYMORPHIC_BASE_TYPES



### 7、serialization 序列化特性开关配置

serialization 属性同mapper类似，也是一个Map类型的属性。主要是直接使用 Jackson 已经自定义好的 序列化规则。

```java
/**
 * Jackson on/off features that affect the way Java objects are serialized.
 */
private final Map<SerializationFeature, Boolean> serialization = new EnumMap<>(SerializationFeature.class);
```

枚举类SerializationFeature中的枚举属性为key，值为boolean设置jackson序列化特性，具体key请看源码：

```java
public enum SerializationFeature implements ConfigFeature {
    WRAP_ROOT_VALUE(false),
    INDENT_OUTPUT(false),
    FAIL_ON_EMPTY_BEANS(true),
    FAIL_ON_SELF_REFERENCES(true),
    WRAP_EXCEPTIONS(true),
    // ...
}
```

SpringBoot Jackson Serialization 序列化特性开关配置示例如下：

```yaml
spring:
  jackson:
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: true
      FAIL_ON_EMPTY_BEANS: true
    # 常规默认: 枚举类SerializationFeature中的枚举属性为key,值为boolean设置jackson序列化特性,具体key请看源码
    # WRITE_DATES_AS_TIMESTAMPS: true : 返回的java.util.date转换成timestamp
    # FAIL_ON_EMPTY_BEANS: true       : 对象为空时是否报错，默认true
```



### 8、deserialization 反序列化开关配置

deserialization属性同mapper类似，也是一个Map类型的属性，配置方式与serialization一样，只需配置DeserializationFeature的key即可

```java
/**
 * Jackson on/off features that affect the way Java objects are deserialized.
 */
private final Map<DeserializationFeature, Boolean> deserialization = new EnumMap<>(DeserializationFeature.class);
```

```java
public enum DeserializationFeature implements ConfigFeature {
    USE_BIG_DECIMAL_FOR_FLOATS(false),
    USE_BIG_INTEGER_FOR_INTS(false),
    USE_LONG_FOR_INTS(false),
    USE_JAVA_ARRAY_FOR_JSON_ARRAY(false),
    FAIL_ON_UNKNOWN_PROPERTIES(true),
    FAIL_ON_NULL_FOR_PRIMITIVES(false),
    FAIL_ON_NUMBERS_FOR_ENUMS(false),
}
```

SpringBoot Jackson Deserialization反序列化特性开关配置示例如下：

```yaml
spring:
  jackson:
    deserialization:
      FAIL_ON_UNKNOWN_PROPERTIES: false
    # 常用配置,json中含pojo不存在属性时是否失败报错,默认true
    # 枚举类DeserializationFeature中的枚举属性为key，值为boolean设置jackson反序列化特性,具体key请看源码
```



### 9、parser 配置

JsonParser在Jackson中负责JSON内容的读取（反序列化），具体特性请看JsonParser.Feature

```yaml
spring:
  jackson: 
    parser:
      ALLOW_SINGLE_QUOTES: true
      allow_unquoted_control_chars: true
    # 枚举类JsonParser.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonParser特性
    # JsonParser在jackson中负责json内容的读取,具体特性请看JsonParser.Feature，一般无需设置默认即可
    # ALLOW_SINGLE_QUOTES: true           :  是否允许出现单引号,默认false
    # allow_unquoted_control_chars: true  :  是否允许出现单引号,默认false
```



### 10、generator 配置

JsonGenerator在Jackson中负责JSON内容编写（序列化），具体特性请看JsonGenerator.Feature。枚举类JsonGenerator.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonGenerator特性，一般无需设置默认即可。

```yaml
spring:
  jackson:
    generator:
      write-numbers-as-strings: true
    # 枚举类JsonGenerator.Feature枚举类中的枚举属性为key，值为boolean设置JsonGenerator特性，一般无需设置默认即可
    # JsonGenerator在jackson中负责编写json内容,具体特性请看JsonGenerator.Feature
    # write-numbers-as-strings: true  :  强制将所有Java数字写为字符串的功能，默认为false
```

当前配置后序列化数字类型，接口返回接口中数字会自动被解析成字符串并带上引号。



### 11、defaultPropertyInclusion 序列化包含的属性配置

该属性是一个枚举配置，主要包含：

- ALWAYS：顾名思义，始终包含，和属性的值无关
- NON_NULL：值非空的属性才会包含属性
- NON_ABSENT：值非空的属性，或者Optional类型的属性非空
- NON_EMPTY:：空值的属性不包含
- NON_DEFAULT：不使用jackson的默认规则对该字段进行序列化,详见示例
- CUSTOM：自定义规则
- USE_DEFAULTS：配置使用该规则的属性字段，将会优先使用class上的注解规则，否则会使用全局的序列化规则，详见示例

CUSTOM 是自定义规则是需要开发者在属性字段上使用@JsonInclude注解，并且指定valueFilter属性，该属性需要传递一个Class，示例如下：

```java
// User.java
// 指定value级别是CUSTOM
@JsonInclude(value = JsonInclude.Include.CUSTOM, valueFilter = StringFilter.class)
private String name;
```

StringFilter：是判断非空的依据，该依据由开发者自己定义，返回true将会被排除，false则不会排除，示例如下：

```java
// 自定义非空判断规则
public class StringFilter {
    @Override
    public boolean equals(Object other) {
        if (other == null) {
            // Filter null's.
            return true;
        }
        // Filter "custom_string".
        return "custom_string".equals(other);
    }
}
```



## 3、SpringBoot Jackson 配置示例

```yaml
spring:
  jackson:
    # 设置当地时区
    locale: zh
    # 设置全局时区
    time-zone: GMT+8
    # 全局时间格式设置 @JsonFormat的格式pattern
    date-format: yyyy-MM-dd HH:mm:ss
    # 设置属性命名策略,对应jackson下PropertyNamingStrategy中的常量值
    # SNAKE_CASE-返回的json驼峰式转下划线，json body下划线传到后端自动转驼峰式
    property-naming-strategy: SNAKE_CASE
    # 常用: 全局设置pojo或被@JsonInclude注解的属性的序列化方式
    # 不为空的属性才会序列化,具体属性可看JsonInclude.Include
    default-property-inclusion: NON_NULL
    # 常规默认,枚举类SerializationFeature中的枚举属性为key,值为boolean设置jackson序列化特性,具体key请看源码
    # WRITE_DATES_AS_TIMESTAMPS: true : 返回的java.util.date转换成timestamp
    # FAIL_ON_EMPTY_BEANS: true       : 对象为空时是否报错，默认true
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: true
      FAIL_ON_EMPTY_BEANS: true
    # 枚举类DeserializationFeature中的枚举属性为key，值为boolean设置jackson反序列化特性,具体key请看源码
    # FAIL_ON_UNKNOWN_PROPERTIES: false   :  常用配置,json中含pojo不存在属性时是否失败报错,默认true
    deserialization:
      FAIL_ON_UNKNOWN_PROPERTIES: false
    # 枚举类MapperFeature中的枚举属性为key，值为boolean设置jackson ObjectMapper特性
    # ObjectMapper在jackson中负责json的读写、json与pojo的互转、json tree的互转,具体特性请看MapperFeature,常规默认即可
    mapper:
      # 使用getter取代setter探测属性，如类中含getName()但不包含name属性与setName()，传输的vo json格式模板中依旧含name属性
      USE_GETTERS_AS_SETTERS: true # 默认false
    # 枚举类JsonParser.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonParser特性
    # JsonParser在jackson中负责json内容的读取,具体特性请看JsonParser.Feature，一般无需设置默认即可
    parser:
      ALLOW_SINGLE_QUOTES: true # 是否允许出现单引号,默认false
    # 枚举类JsonGenerator.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonGenerator特性，一般无需设置默认即可
    # JsonGenerator在jackson中负责编写json内容,具体特性请看JsonGenerator.Feature
    # write-numbers-as-strings: true  :  强制将所有Java数字写为字符串的功能，默认为false
    generator:
      write-numbers-as-strings: true
    # 开启visibility私有字段或方法访问级别, 可以让私有属性或者方法被Jackson序列化或反序列化到
    # 枚举类PropertyAccessor中的枚举属性为key, 枚举类JsonAutoDetect.Visibility中的枚举属性为value
    visibility:
      field: any
```



## 4、SpringBoot Jackson 自动装配

在前面的文章中，我们已经详细的了解了Jackson在Spring Boot框架中的各个配置项，那么Spring Boot针对Jackson框架在约定配置时会做哪些事情呢?

在Spring Boot的spring-boot-autoconfigure-x.x.jar包中，我们可以看到Spring Boot框架针对jackson的处理源码，主要包含三个类：

- JacksonProperties:Spring Boot框架提供jackson的配置属性类，即开发者在application.yml配置文件中的配置项属性

- JacksonAutoConfiguration:Jackson的默认注入配置类
- Jackson2ObjectMapperBuilderCustomizer:自定义用于注入jackson的配置辅助接口

核心类是JacksonAutoConfiguration.java,该类是Spring Boot框架将Jackson相关实体Bean注入Spring容器的关键配置类。其主要作用：

- 注入Jackson的ObjectMapper实体Bean到Spring容器中
- 注入ParameterNamesModule实体Bean到Spring容器中
- 注入Jackson2ObjectMapperBuilder实体Bean
- 注入JsonComponentModule实体Bean
- 注入StandardJackson2ObjectMapperBuilderCustomizer实体Bean，该类是上面Jackson2ObjectMapperBuilderCustomizer的实现类，主要用于接收JacksonProperties属性，将Jackson的外部配置属性接收，然后最终执行customize方法，构建ObjectMapper所需要的Jackson2ObjectMapperBuilder属性，最终为ObjectMapper属性赋值准备

源码如下：

```java
package com.example.springbootjackson.model;

@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(ObjectMapper.class)
public class JacksonAutoConfiguration {

    private static final Map<?, Boolean> FEATURE_DEFAULTS;

    static {
        Map<Object, Boolean> featureDefaults = new HashMap<>();
        featureDefaults.put(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        featureDefaults.put(SerializationFeature.WRITE_DURATIONS_AS_TIMESTAMPS, false);
        FEATURE_DEFAULTS = Collections.unmodifiableMap(featureDefaults);
    }

    @Bean
    public JsonComponentModule jsonComponentModule() {
        return new JsonComponentModule();
    }

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(Jackson2ObjectMapperBuilder.class)
    static class JacksonObjectMapperConfiguration {

        @Bean
        @Primary
        @ConditionalOnMissingBean
        ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
            return builder.createXmlMapper(false).build();
        }

    }

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(ParameterNamesModule.class)
    static class ParameterNamesModuleConfiguration {

        @Bean
        @ConditionalOnMissingBean
        ParameterNamesModule parameterNamesModule() {
            return new ParameterNamesModule(JsonCreator.Mode.DEFAULT);
        }

    }

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(Jackson2ObjectMapperBuilder.class)
    static class JacksonObjectMapperBuilderConfiguration {

        @Bean
        @Scope("prototype")
        @ConditionalOnMissingBean
        Jackson2ObjectMapperBuilder jacksonObjectMapperBuilder(ApplicationContext applicationContext,
                                                               List<Jackson2ObjectMapperBuilderCustomizer> customizers) {
            Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
            builder.applicationContext(applicationContext);
            customize(builder, customizers);
            return builder;
        }

        private void customize(Jackson2ObjectMapperBuilder builder,
                               List<Jackson2ObjectMapperBuilderCustomizer> customizers) {
            for (Jackson2ObjectMapperBuilderCustomizer customizer : customizers) {
                customizer.customize(builder);
            }
        }

    }

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(Jackson2ObjectMapperBuilder.class)
    @EnableConfigurationProperties(JacksonProperties.class)
    static class Jackson2ObjectMapperBuilderCustomizerConfiguration {

        @Bean
        StandardJackson2ObjectMapperBuilderCustomizer standardJacksonObjectMapperBuilderCustomizer(
            ApplicationContext applicationContext, JacksonProperties jacksonProperties) {
            return new StandardJackson2ObjectMapperBuilderCustomizer(applicationContext, jacksonProperties);
        }

        static final class StandardJackson2ObjectMapperBuilderCustomizer
            implements Jackson2ObjectMapperBuilderCustomizer, Ordered {

            private final ApplicationContext applicationContext;

            private final JacksonProperties jacksonProperties;

            StandardJackson2ObjectMapperBuilderCustomizer(ApplicationContext applicationContext,
                                                          JacksonProperties jacksonProperties) {
                this.applicationContext = applicationContext;
                this.jacksonProperties = jacksonProperties;
            }

            @Override
            public int getOrder() {
                return 0;
            }

            @Override
            public void customize(Jackson2ObjectMapperBuilder builder) {

                if (this.jacksonProperties.getDefaultPropertyInclusion() != null) {
                    builder.serializationInclusion(this.jacksonProperties.getDefaultPropertyInclusion());
                }
                if (this.jacksonProperties.getTimeZone() != null) {
                    builder.timeZone(this.jacksonProperties.getTimeZone());
                }
                configureFeatures(builder, FEATURE_DEFAULTS);
                configureVisibility(builder, this.jacksonProperties.getVisibility());
                configureFeatures(builder, this.jacksonProperties.getDeserialization());
                configureFeatures(builder, this.jacksonProperties.getSerialization());
                configureFeatures(builder, this.jacksonProperties.getMapper());
                configureFeatures(builder, this.jacksonProperties.getParser());
                configureFeatures(builder, this.jacksonProperties.getGenerator());
                configureDateFormat(builder);
                configurePropertyNamingStrategy(builder);
                configureModules(builder);
                configureLocale(builder);
            }

            // more configure methods...
        }
    }
}
```

**总结**：通过一系列的方法，最终构造一个生产级别可用的`ObjectMapper`对象，供在Spring Boot框架中对Java对象实现序列化与反序列化操作。



# 十、SpringBoot 日期时间处理总结

> PS：如果你的Controller中的LocalDate类型的参数啥注解（RequestParam、PathVariable等）都没加，也是会出错的，因为默认情况下，解析这种参数是使用ModelAttributeMethodProcessor进行处理，而这个处理器要通过反射实例化一个对象出来，然后再对对象中的各个参数进行convert，但是LocalDate类没有构造函数，无法反射实例化因此会报错！！！

项目中使用LocalDateTime系列作为DTO中时间的数据类型，但是SpringMVC收到参数后总报错，为了配置全局时间类型转换，尝试了如下处理方式。



## 1、GET请求及POST表单日期时间字符串格式转换

> 这种情况要和时间作为JSON字符串时区别对待，因为前端JSON转到后端POJO底层使用的是JSON序列化Jackson工具（HttpMessgeConverter）；而时间字符串作为普通请求参数传入时，转换用的是Converter，两者在处理方式上是有区别。
>

场景：请求接口，入参转换（因为不是JSON所以不能称反序列化），非content-type=application/json情况，

- GET 请求入参：GET请求是没有 content-type 的

  ```java
  @RestController
  public class TestController {
      /**
       * GET请求:
       * ip:port/queryOne?localDateTime=20221124 111111&localDate=20221124&date=20221124&timestamp=20221124
       * response json:
       * {
       *   "localDateTime": "2022-11-25T14:49:40.5289825",
       *   "localDate": "2022-11-25",
       *   "date": "2022-11-25 06:49:40",
       *   "timestamp": "2022-11-25 06:49:40"
       * }
       * 控制台打印:
       * (localDateTime=2022-11-24T11:11:11, localDate=2022-11-24, date=Thu Nov 24 00:00:00 CST 2022, timestamp=2022-11-24 00:00:00.0)
       */
      @GetMapping("/query")
      public ResponseEntity<User> query(User user) {
          System.out.println(user);
          return ResponseEntity.ok(user);
      }
  
      @Data
      public static class User {
          private LocalDateTime localDateTime;
          private LocalDate localDate;
          private Date date;
          private Timestamp timestamp;
      }
  }
  ```

- POST 请求入参：请求类型为：content-type=application/x-www-form-urlencoded， 后台不能用@RequestBody接收

  ```java
  @RestController
  public class TestController {
      /**
       * POST http://localhost:8080/query
       * Content-Type: application/x-www-form-urlencoded
       *
       * localDateTime=20221124 111111&localDate=20221124&date=20221124&timestamp=20221124
       * 
       * response json:
       * {
       *   "localDateTime": "2022-11-24T11:11:11",
       *   "localDate": "2022-11-24",
       *   "date": "2022-11-23T16:00:00.000+00:00",
       *   "timestamp": "2022-11-23T16:00:00.000+00:00"
       * }
       * 控制台打印:
       * (localDateTime=2022-11-24T11:11:11, localDate=2022-11-24, date=Thu Nov 24 00:00:00 CST 2022, timestamp=2022-11-24 00:00:00.0)
       */
      @PostMapping("/query")
      public ResponseEntity<User> query(User user) {
          System.out.println(user);
          return ResponseEntity.ok(user);
      }
  
      @Data
      public static class User {
          private LocalDateTime localDateTime;
          private LocalDate localDate;
          private Date date;
          private Timestamp timestamp;
      }
  }
  ```



### 1、使用Spring自定义参数转换器（Converter）

在Spring中定义了3中类型转换接口，分别为：

1.  Converter 接口            ：使用最简单，最不灵活；（本次使用）
2.  ConverterFactory 接口：使用较复杂，比较灵活；
3.  GenericConverter 接口：使用最复杂，也最灵活；

实现 org.springframework.core.convert.converter.Converter，自定义参数转换器，如下：

```java
@Configuration
public class DateConverterConfig {
    @Bean
    public Converter<String, LocalDate> localDateConverter() {
        return new Converter<String, LocalDate>() {
            @Override
            public LocalDate convert(String source) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd"));
            }
        };
    }

    @Bean
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return new Converter<String, LocalDateTime>() {
            @Override
            public LocalDateTime convert(String source) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd HHmmss"));
            }
        };
    }
    
    @Bean
    public Converter<String, Date> dateConverter() {
        return new Converter<String, Date>() {
            @SneakyThrows
            @Override
            public Date convert(String source) {
                return new SimpleDateFormat("yyyyMMdd").parse(source);
            }
        };
    }

    @Bean
    public Converter<String, Timestamp> timestampConverter() {
        return new Converter<String, Timestamp>() {
            @SneakyThrows
            @Override
            public Timestamp convert(String source) {
                return new Timestamp(new SimpleDateFormat("yyyyMMdd").parse(source).getTime());
            }
        };
    }
}
```

以上两个Bean会注入到SpringMVC的参数解析器（好像叫做ParameterConversionService），当传入的字符串要转为LocalDateTime类或者LocalDate类时，Spring会调用该Converter对这个入参进行转换。



### 2、Converter中Lambda代替匿名内部类启动报错

> 注意：关于自定义的参数转换器 Converter，这里我遇到了一个坑，我再这里详细记录下，本来我的想法是为了代码精简，将上面匿名内部类的写法精简成lambda表达式，可是启动报错： does the class parameterize those types?

#### 1、场景复现

```java
/*
 * 注入自定义的LocalDateTime转换器
 */
@Bean
public Converter<String, LocalDateTime> localDateTimeConverter() {
    return source -> LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd HHmmss"));
}
```

启动项目时却出现了异常：

```shell
Caused by: java.lang.IllegalArgumentException: Unable to determine source type <S> and target type <T> for your Converter [com.example.demo126.config.MappingConverterAdapter$$Lambda$522/817994751]; does the class parameterize those types?
```



#### 2、原因分析

1. Web项目启动注册requestMappingHandlerAdapter的时候会初始化WebBindingInitializer

   ```java
   adapter.setWebBindingInitializer(getConfigurableWebBindingInitializer());
   ```

2. 而ConfigurableWebBindingInitializer需要FormattingConversionService，而FormattingConversionService会将所有的Converter添加进来,添加的时候需要获取泛型信息

   ```java
   @Override
   public void addFormatters(FormatterRegistry registry) {
       for (Converter<?, ?> converter : getBeansOfType(Converter.class)) {
           registry.addConverter(converter);
       }
       for (GenericConverter converter : getBeansOfType(GenericConverter.class)) {
           registry.addConverter(converter);
       }
       for (Formatter<?> formatter : getBeansOfType(Formatter.class)) {
           registry.addFormatter(formatter);
       }
   }
   ```

3. 添加Converter.class 一般是通过接口获取两个泛型的具体类型

   ```java
   public ResolvableType as(Class<?> type) {
       if (this == NONE) {
           return NONE;
       }
       Class<?> resolved = resolve();
       if (resolved == null || resolved == type) {
           return this;
       }
       for (ResolvableType interfaceType : getInterfaces()) {
           ResolvableType interfaceAsType = interfaceType.as(type);
           if (interfaceAsType != NONE) {
               return interfaceAsType;
           }
       }
       return getSuperType().as(type);
   }
   ```

4. Lambda表达式的接口是`Converter<?, ?>`，**不能的到具体的类型**。



#### 3、解决办法

1、方案一：等待requestMappingHandlerAdapter注册结束在注册，再添加自己的Converter就不会注册到FormattingConversionService中。网上很多人推荐此方法，实际上这是种矛盾，本人查看了源码和打了断点，既然都不会注册到FormattingConversionService中，那么我们自定义的Converter也不会被注入成功，那么实际上只是解决了启动报错的问题，根本上自定义的Converter也不胡生效。（**无效**）

```java
// 此方式实际无效
@Bean
@ConditionalOnBean(name = "requestMappingHandlerAdapter")
public Converter<String, LocalDate> localDateConverter() {
    return source -> LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd"));
}
```



2、方案二：那么真的就不能使用Lambda表达式了吗？也不是，我们实际上还可以使用SpringMVC来注册Converter。从上面原因分析可以看出根本报错原因是在：`void addConverter(Converter<?, ?> converter);`，这个调用的Lambda表达式会让产生的转换器无法解析什么是传入类型，什么是目标类型，这就导致无法生效。（**推荐**）

所以你要用Lambda的话你需要用如下方法（其中参数的意思就是传入类型和需要转换成的类型。，还有转换器定义）：

```java
<S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter);
```

使用SpringMVC的WebMvcConfigurer重写addConverter()，自动把Converter注册进去。上面的Converter是Spring容器自动注册的。

```java
@Configuration
public class DateConverterConfig {

    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        DateTimeFormatter yyyyMMdd = DateTimeFormatter.ofPattern("yyyyMMdd");
        DateTimeFormatter yyyyMMdd_hHmmss = DateTimeFormatter.ofPattern("yyyyMMdd HHmmss");

        return new WebMvcConfigurer() {
            @Override
            public void addFormatters(FormatterRegistry registry) {
                registry.addConverter(String.class, LocalDate.class, 
                                      source -> LocalDate.parse(source, yyyyMMdd));
                registry.addConverter(String.class, LocalDateTime.class, 
                                      source -> LocalDateTime.parse(source, yyyyMMdd_hHmmss));
                registry.addConverter(String.class, Date.class, source -> {
                    try {
                        return new SimpleDateFormat("yyyyMMdd").parse(source);
                    } catch (ParseException e) {
                        throw new RuntimeException(e);
                    }
                });
                registry.addConverter(String.class, Timestamp.class, source -> {
                    try {
                        return new Timestamp(new SimpleDateFormat("yyyyMMdd").parse(source).getTime());
                    } catch (ParseException e) {
                        throw new RuntimeException(e);
                    }
                });
            }
        };
    }
}
```



3、方案三：最简单的方法就是不适用Lambda表达式，还是老老实实的使用匿名内部类，这样就不会存在上述问题（**推荐**）

4、方案四：自定义接口继承Converter接口，具体类型接口，不再是泛型。这种就可以使用到Lambda表达式了（**不推荐**）

```java
@Configuration
public class DateConverterConfig {

    interface StringToLocalDateConverter extends Converter<String, LocalDate> {
    }

    interface StringToLocalDateTimeConverter extends Converter<String, LocalDateTime> {
    }

    interface StringToDateConverter extends Converter<String, Date> {
    }

    interface StringToTimestampConverter extends Converter<String, Timestamp> {
    }

    @Bean
    public StringToLocalDateConverter localDateConverter() {
        return source -> LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd"));
    }

    @Bean
    public StringToLocalDateTimeConverter localDateConverterLambda() {
        return source -> LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyyMMdd HHmmss"));
    }

    @Bean
    public StringToDateConverter dateConverter() {
        return source -> {
            try {
                return new SimpleDateFormat("yyyyMMdd").parse(source);
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        };
    }

    @Bean
    public StringToTimestampConverter timestampConverter() {
        return source -> {
            try {
                return new Timestamp(new SimpleDateFormat("yyyyMMdd").parse(source).getTime());
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        };
    }
}
```



5、方案五：自定义类实现Converter接口，重写接口方法。也是不适用Lambda表达式了。（**推荐**）

```java
@Component
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String value) {
        /**
         * 可对value进行正则匹配，支持日期、时间等多种类型转换
         * 如 yyyy-MM-dd HH:mm:ss、yyyy-MM-dd、 HH:mm:ss等，进行匹配。以适应多种场景
         * 这里我偷个懒，在匹配Date日期格式时直接使用了 hutool 为我们已经写好的解析工具类，这里就不重复造轮子了
         * cn.hutool.core.date.DateUtil
         * @param value
         * @return
         */
        return DateUtil.parse(value.trim());
    }
}
```

```xml
<!-- hutool 工具类 -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.1.3</version>
</dependency>
```



### 3、使用Spring默认自带注解@DateTimeFormat

使用Spring自带注解@DateTimeFormat(pattern = “yyyy-MM-dd”)，如下：

```java
@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date startDate;
```

**需要特别注意：**

- 如果使用了自定义参数转化器，Spring会优先使用该方式进行处理，即Spring的@DateTimeFormat注解不生效
- Spring的@DateTimeFormat 注解只支持Date 类型，Java8 时间以及 Timestamp 类型都不支持



### 4、使用@ControllerAdvice配合@initBinder

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    /**
     * @ControllerAdvice的三种用法之一：配置@InitBinder可以请求参数预处理
     */
    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalDate.parse(text, DateTimeFormatter.ofPattern("yyyyMMdd")));
            }
        });
        binder.registerCustomEditor(LocalDateTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalDateTime.parse(text, DateTimeFormatter.ofPattern("yyyyMMdd HHmmss")));
            }
        });
        binder.registerCustomEditor(LocalTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalTime.parse(text, DateTimeFormatter.ofPattern("HHmmss")));
            }
        });
    }
    
    @InitBinder
    public void initBinder2(WebDataBinder binder) {
        binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
            @SneakyThrows
            @Override
            public void setAsText(String text) {
                setValue(LocalDate.parse(text, DateTimeFormatter.ofPattern("yyyyMMdd")));
            }
        });
        binder.registerCustomEditor(LocalDateTime.class, new PropertyEditorSupport() {
            @SneakyThrows
            @Override
            public void setAsText(String text) {
                LocalDateTime yyyyMMdd = LocalDate.parse(text, DateTimeFormatter.ofPattern("yyyyMMdd")).atTime(LocalTime.MIN);
                setValue(yyyyMMdd);
            }
        });
    }
}
```



## 2、JSON入参及返回值全局处理

场景：请求类型为：post,content-type=application/json， 后台用@RequestBody接收，默认接收及返回值格式为 yyyy-MM-dd HH:mm:ss

### 1、修改 application.yml 文件

在application.propertities文件中增加如下内容：

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```

- 支持：（content-type=application/json）请求中格式为 yyyy-MM-dd HH:mm:ss的字符串，后台用@RequestBody接收，及返回值date转为yyyy-MM-dd HH:mm:ss格式string
- 不支持：（content-type=application/json）请求中yyyy-MM-dd等类型的字符串转为date
- 不支持：Java8日期API



### 2、利用Jackson的JSON序列化和反序列化

```java
@Configuration
public class JacksonConfig {

    /** 默认日期时间格式 */
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    /** 默认日期格式 */
    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    /** 默认时间格式 */
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = new ObjectMapper();

        // 忽略json字符串中不识别的属性
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 忽略无法转换的对象
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        // PrettyPrinter 格式化输出
        objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);
        // NULL不参与序列化
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        // 指定时区
        objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8:00"));
        // 日期类型字符串处理
        objectMapper.setDateFormat(new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT));

        // Java8日期日期处理
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class, 
                new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class, 
                new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class, 
                new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDateTime.class, 
                new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class, 
                new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class, 
                new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        objectMapper.registerModule(javaTimeModule);

        converter.setObjectMapper(objectMapper);
        return converter;
    }
}
```

- 支持：（content-type=application/json）请求中格式为yyyy-MM-dd HH:mm:ss的字符串，后台用@RequestBody接收，及返回值Date转为yyyy-MM-dd HH:mm:ss格式String
- 支持：Java8日期API
- 不支持：（content-type=application/json）请求中yyyy-MM-dd等类型的字符串转为Date

总结：以上两种方式为JSON入参的全局化处理，推荐使用方式二，尤其适合大型项目在基础包中全局设置。



## 3、JSON入参及返回值局部差异化处理

场景：假如全局日期时间处理格式为：yyyy-MM-dd HH:mm:ss，但是某个字段要求接收或返回日期：yyyy-MM-dd（注意接收日期：请求类型为：post,content-type=application/json， 后台用@RequestBody接收）

**方式一**：使用SpringBoot自带的注解`@JsonFormat(pattern = "")`，如下所示：

```java
@Data
public class User {
    @JsonFormat(pattern = "yyyyMMdd HHmmss")
    private LocalDateTime localDateTime = LocalDateTime.now();
    @JsonFormat(pattern = "yyyyMMdd")
    private LocalDate localDate = LocalDate.now();
    private Date date = new Date();
    private Timestamp timestamp = new Timestamp(System.currentTimeMillis());
}
```

**点评：**SpringBoot默认提供，功能强大，满足常见场景使用，并可指定时区。

**方式二**：自定义日期序列化与反序列化，如下所示：

```java
/**
 * 日期序列化
 */
class LocalDateTimeJsonSerializer extends JsonSerializer<LocalDateTime> {
    @SneakyThrows
    @Override
    public void serialize(LocalDateTime localDateTime, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) {
        jsonGenerator.writeString(DateTimeFormatter.ofPattern("yyyyMMdd HHmmss").format(localDateTime));
    }
}

/**
 * 日期反序列化
 */
class LocalDateTimeJsonDeserializer extends JsonDeserializer<LocalDateTime> {
    @SneakyThrows
    @Override
    public LocalDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) {
        return LocalDateTime.parse(jsonParser.getText(), DateTimeFormatter.ofPattern("yyyyMMdd HHmmss"));
    }
}

/**
 * 使用方式
 */
@Data
public class User {
    @JsonSerialize(using = LocalDateTimeJsonSerializer.class)
    @JsonDeserialize(using = LocalDateTimeJsonDeserializer.class)
    private LocalDateTime localDateTime = LocalDateTime.now();
    private LocalDate localDate = LocalDate.now();
    private Date date = new Date();
    private Timestamp timestamp = new Timestamp(System.currentTimeMillis());
}
```



## 4、日期时间格式化处理方式完整配置

```java
import cn.hutool.core.date.DateUtil;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalTimeSerializer;
import lombok.SneakyThrows;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.format.FormatterRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import java.sql.Timestamp;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;

/**
 * 注意: 这里偷个懒使用了hutool的日期时间解析工具类
 * <dependency>
 *     <groupId>cn.hutool</groupId>
 *     <artifactId>hutool-all</artifactId>
 *     <version>5.1.3</version>
 * </dependency>
 */
@Configuration
public class DateHandlerConfig {

    /** 默认日期时间格式 */
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    /** 默认日期格式 */
    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    /** 默认时间格式 */
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    /**
     * LocalDate转换器，用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public Converter<String, LocalDate> localDateConverter() {
        return new Converter<String, LocalDate>() {
            @Override
            public LocalDate convert(String source) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT));
            }
        };
    }

    /**
     * LocalDateTime转换器，用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return new Converter<String, LocalDateTime>() {
            @Override
            public LocalDateTime convert(String source) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT));
            }
        };
    }

    /**
     * LocalTime转换器，用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public Converter<String, LocalTime> localTimeConverter() {
        return new Converter<String, LocalTime>() {
            @Override
            public LocalTime convert(String source) {
                return LocalTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT));
            }
        };
    }

    /**
     * Date转换器，用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public Converter<String, Date> dateConverter() {
        return new Converter<String, Date>() {
            @Override
            public Date convert(String source) {
                return DateUtil.parse(source.trim());
            }
        };
    }

    /**
     * Date转换器，用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public Converter<String, Timestamp> timestampConverter() {
        return new Converter<String, Timestamp>() {
            @Override
            public Timestamp convert(String source) {
                return new Timestamp(DateUtil.parse(source.trim()).getTime());
            }
        };
    }

    /**
     * LocalDate,LocalDateTime,LocalTime,Date,Timestamp 转换器: 用于转换RequestParam（入参对象也算）和PathVariable参数
     */
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addFormatters(FormatterRegistry registry) {
                registry.addConverter(String.class, LocalDate.class,
                        source -> LocalDate.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
                registry.addConverter(String.class, LocalTime.class,
                        source -> LocalTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
                registry.addConverter(String.class, LocalDateTime.class,
                        source -> LocalDateTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
                registry.addConverter(String.class, Date.class, source -> DateUtil.parse(source.trim()));
                registry.addConverter(String.class, Timestamp.class,
                        source -> new Timestamp(DateUtil.parse(source.trim()).getTime()));
            }
        };
    }

    /**
     * Json序列化和反序列化转换器，用于转换Post请求体中的json以及将我们的对象序列化为返回响应的json
     */
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.disable(DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE);

        // LocalDateTime系列序列化和反序列化模块，继承自jsr310，我们在这里修改了日期格式
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class,
                new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class,
                new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class,
                new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDateTime.class,
                new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class,
                new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class,
                new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));

        // Date序列化和反序列化
        javaTimeModule.addSerializer(Date.class, new JsonSerializer<>() {
            @SneakyThrows
            @Override
            public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) {
                SimpleDateFormat formatter = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String formattedDate = formatter.format(date);
                jsonGenerator.writeString(formattedDate);
            }
        });
        javaTimeModule.addDeserializer(Date.class, new JsonDeserializer<>() {
            @SneakyThrows
            @Override
            public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) {
                SimpleDateFormat format = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String date = jsonParser.getText();
                return format.parse(date);
            }
        });

        // Timestamp序列化和反序列化
        javaTimeModule.addSerializer(Timestamp.class, new JsonSerializer<>() {
            @SneakyThrows
            @Override
            public void serialize(Timestamp timestamp, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) {
                SimpleDateFormat formatter = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String formattedDate = formatter.format(timestamp);
                jsonGenerator.writeString(formattedDate);
            }
        });
        javaTimeModule.addDeserializer(Timestamp.class, new JsonDeserializer<>() {
            @SneakyThrows
            @Override
            public Timestamp deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) {
                SimpleDateFormat format = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String date = jsonParser.getText();
                return new Timestamp(format.parse(date).getTime());
            }
        });

        objectMapper.registerModule(javaTimeModule);
        return objectMapper;
    }
}
```



## 5、扩充源码：深入研究数据绑定过程

> 接下来进入Debug模式，看看mvc是如何将我们request中的参数绑定到我们controller层方法入参的

1、写一个简单controller，打个断点看看方法调用栈：

```java
@GetMapping("/getDate")
public LocalDateTime getDate(@RequestParam LocalDate date,
                             @RequestParam LocalDateTime dateTime,
                             @RequestParam Date originalDate) {
    System.out.println(date);
    System.out.println(dateTime);
    System.out.println(originalDate);
    return LocalDateTime.now();
}
```

2、调用接口以后，我们看下方法调用栈中一些关键方法：

```java
// 进入DispatcherServlet
doService:942, DispatcherServlet
// 处理请求
doDispatch:1038, DispatcherServlet
// 生成调用链（前处理、实际调用方法、后处理）
handle:87, AbstractHandlerMethodAdapter
// 反射获取到实际调用方法，准备开始调用
invokeHandlerMethod:895, RequestMappingHandlerAdapter
invokeAndHandle:102, ServletInvocableHandlerMethod
// 这里是关键，参数从这里开始获取到
invokeForRequest:142, InvocableHandlerMethod
doInvoke:215, InvocableHandlerMethod
// 这个是Java reflect调用，因此一定是在这之前获取到的参数
invoke:566, Method
```

3、根据上述分析发现`invokeForRequest:142, InvocableHandlerMethod`这里的代码是用来拿到实际参数的：

```java
@Nullable
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
                               Object... providedArgs) throws Exception {
    // 这个方法是获取参数的，在这里下个断
    Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
    if (logger.isTraceEnabled()) {
        logger.trace("Arguments: " + Arrays.toString(args));
    }
    // 这里开始调用方法
    return doInvoke(args);
}
```

4、进入这个方法看看是什么操作：

```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    // 获取方法参数数组，包含了入参信息，比如类型、泛型等等
    MethodParameter[] parameters = getMethodParameters();
    // 这个用来存放一会从request parameter转换的参数
    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
        // 这里看起来没啥卵用（providedArgs为空）
        args[i] = resolveProvidedArgument(parameter, providedArgs);
        // 这里开始获取到方法实际调用的参数，步进
        if (this.argumentResolvers.supportsParameter(parameter)) {
            // 从名字就看出来：参数解析器解析参数
            args[i] = this.argumentResolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
            continue;
        }
    }
    return args;
}
```

5、进入resolveArgument看看：

```java
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
    // 根据方法入参，获取对应的解析器
    HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
    // 开始解析参数（把请求中的parameter转为方法的入参）
    return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

6、这里根据参数获取相应的参数解析器，看看内部如何获取的：

```java
// 遍历，调用supportParameter方法，跟进看看
for (HandlerMethodArgumentResolver methodArgumentResolver : this.argumentResolvers) {
    if (methodArgumentResolver.supportsParameter(parameter)) {
        result = methodArgumentResolver;
        this.argumentResolverCache.put(parameter, result);
        break;
    }
}
```

7、**这里，遍历参数解析器，查找有没有适合的解析器！那么，有哪些参数解析器呢(我测试的时候有26个)？？？我列出几个重要的看看，是不是很眼熟！！！**

```java
{RequestParamMethodArgumentResolver@7686}
{PathVariableMethodArgumentResolver@8359}
{RequestResponseBodyMethodProcessor@8366}
{RequestPartMethodArgumentResolver@8367}
```

8、我们进入最常用的一个解析器看看他的supportsParameter方法，发现就是通过参数注解来获取相应的解析器的。

```java
public boolean supportsParameter(MethodParameter parameter) {
    // 如果参数拥有注解@RequestParam，则走这个分支（知道为什么上文要对RequestParam和Json两种数据区别对待了把）
    if (parameter.hasParameterAnnotation(RequestParam.class)) {
        // 这个似乎是对Optional类型的参数进行处理的
        if (Map.class.isAssignableFrom(parameter.nestedIfOptional().getNestedParameterType())) {
            RequestParam requestParam = parameter.getParameterAnnotation(RequestParam.class);
            return (requestParam != null && StringUtils.hasText(requestParam.name()));
        }
        else {
            return true;
        }
    }
    // ......
}
```

也就是说，对于@RequestParam和@RequestBody以及@PathVariable注解的参数，SpringMVC会使用不同的参数解析器进行数据绑定！ 那么，这三种解析器分别使用什么Converter解析参数呢？我们分别进入三种解析器看一看： 首先看下RequestParamMethodArgumentResolver发现内部使用WebDataBinder进行数据绑定，底层使用的是ConversionService （也就是我们的Converter注入的地方）

```java
WebDataBinder binder = binderFactory.createBinder(webRequest, null, namedValueInfo.name);
// 通过DataBinder进行数据绑定的
arg = binder.convertIfNecessary(arg, parameter.getParameterType(), parameter);

// 跟进convertIfNecessary()
public <T> T convertIfNecessary(@Nullable Object value, @Nullable Class<T> requiredType,
                                @Nullable MethodParameter methodParam) throws TypeMismatchException {

    return getTypeConverter().convertIfNecessary(value, requiredType, methodParam);
}

// 继续跟进，看到了把
ConversionService conversionService = this.propertyEditorRegistry.getConversionService();
if (editor == null && conversionService != null && newValue != null && typeDescriptor != null) {
    TypeDescriptor sourceTypeDesc = TypeDescriptor.forObject(newValue);
    if (conversionService.canConvert(sourceTypeDesc, typeDescriptor)) {
        try {
            return (T) conversionService.convert(newValue, sourceTypeDesc, typeDescriptor);
        }
        catch (ConversionFailedException ex) {
            // fallback to default conversion logic below
            conversionAttemptEx = ex;
        }
    }
}
```

**然后看下RequestResponseBodyMethodProcessor发现使用的转换器是HttpMessageConverter类型的：**

```java
// resolveArgument方法内部调用下面进行参数解析
Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());

// step into readWithMessageConverters()，我们看到这里的Converter是HttpMessageConverter
for (HttpMessageConverter<?> converter : this.messageConverters) {
    Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
    GenericHttpMessageConverter<?> genericConverter =
        (converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
    if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
        (targetClass != null && converter.canRead(targetClass, contentType))) {
        if (message.hasBody()) {
            HttpInputMessage msgToUse =
                getAdvice().beforeBodyRead(message, parameter, targetType, converterType);
            body = (genericConverter != null ? genericConverter.read(targetType, contextClass, msgToUse) :
                    ((HttpMessageConverter<T>) converter).read(targetClass, msgToUse));
            body = getAdvice().afterBodyRead(body, msgToUse, parameter, targetType, converterType);
        }
        else {
            body = getAdvice().handleEmptyBody(null, message, parameter, targetType, converterType);
        }
        break;
    }
}
```

**最后看下PathVariableMethodArgumentResolver发现 和RequestParam走的执行路径一致（二者都是继承自AbstractNamedValueMethodArgumentResolver解析器），因此代码就不贴了。**



## 6、SpringBoot 日期时间参数使用总结

总结：如果要转换request传来的参数到我们指定的类型，根据入参注解要进行区分：

- 如果是RequestBody，那么通过配置ObjectMapper（这个玩意儿会注入到Jackson的HttpMessagConverter里面，即MappingJackson2HttpMessageConverter中）来实现JSON格式数据的序列化和反序列化；
- 如果是RequestParam（对象参数也属于这种）或者PathVariable类型的参数，通过配置Converter实现参数转换（这些Converter会注入到ConversionService中）

总结：通过POST请求和GET请求 以及 序列化和反序列化 分类：

- POST 请求序列化：
  - 全局配置除了Java8时间无效，其他都正常。
  - @JsonFormat注解配置，任何类型都生效
- POST 请求反序列化：
  - 类型为 content-type=application/x-www-form-urlencoded，后台非@RequestBody接收，全局与@JsonFormat注解配置都无效，只能使用Spring的Converter转换器，或者直接使用@DateTimeFormat进行接收参数转换
  - 类型为 content-type=application/json，后台为@RequestBody接收，注解配置生效，全局配置Java8时间不生效。
- GET 请求序列化：
  - 全局配置除了Java8时间无效，其他都正常。
  - @JsonFormat注解配置，任何类型都生效
- GET 请求反序列化：
  - 全局与@JsonFormat注解配置都无效，只能使用Spring的Converter转换器，或者直接使用@DateTimeFormat进行接收参数转换

