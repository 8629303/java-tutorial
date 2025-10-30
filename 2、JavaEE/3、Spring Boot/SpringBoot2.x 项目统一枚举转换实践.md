# SpringBoot 项目统一枚举转换实践

> SpringBoot 项目统一枚举转换实践：https://blog.csdn.net/qq_40161813/article/details/123037862

## 1、现有问题背景

目前的项目中，有些枚举字段，在传递的时候，需要经常对枚举进行对应的转换，有如下场景：

1. 存储进数据库的时候，需要存储为 int；
2. 查询出来的时候，需要对该数值进行转换；
3. 接收前端参数的时候，需要将数字转换为我们系统的枚举；
4. 响应的参数包含枚举的时候，需要将枚举转换成 int；
5. 发送或接收 MQ 消息时，又得对枚举进行转换。

可以看到，我们在系统中需要做大量的枚举转换工作，那么是不是有什么方法对枚举转换进行简化呢？



## 2、数据库转换枚举

我们这边的系统使用的持久层框架是 Spring Data jpa，底层实现是 Hibernate，对于数据库枚举的转化提供了 AttributeConverter 接口，可以实现枚举和 int 自动转换。

1、例如我们有一个枚举类

```java
@Getter
@RequiredArgsConstructor
public enum SexEnum {
    /**
     * 1 男
     */
    MALE(1, "男"),
    /**
     * 2 女
     */
    FEMALE(2, "女"),
    ;
    private final Integer number;
    private final String value;
}
```

2、那么，我们就可以实现 AttributeConverter 接口，来实现枚举的自动转换。

```java
@Converter(autoApply = true)
public class SexEnumConverter implements AttributeConverter<SexEnum, Integer> {

    @Override
    public Integer convertToDatabaseColumn(SexEnum attribute) {
        return attribute == null ? null : attribute.getValue();
    }

    @Override
    public SexEnum convertToEntityAttribute(Integer dbData) {
        return Arrays.stream(SexEnum.class)
            .filter(sexEnum -> sexEnum.getValue().equals(dbData))
            .findFirst()
            .orElse(null);
    }
}
```

但是这样子每一个枚举类又必须写对应的枚举转换的代码，很麻烦，因此，接下来进行优化与增强。

3、我们可以先定义一个父类的枚举接口

```java
public interface IEnum {
    Integer getNumber();
}
```

4、然后优化我们的枚举转换类

```java
import javax.persistence.AttributeConverter;
import java.lang.reflect.ParameterizedType;
import java.util.Arrays;
import java.util.Optional;

public class EnumConverter<E extends IEnum> implements AttributeConverter<E, Integer> {

    private final Class<E> clazz;

    @SuppressWarnings("unchecked")
    public EnumConverter() {
        this.clazz = (Class<E>) (((ParameterizedType) this.getClass().getGenericSuperclass()).getActualTypeArguments())[0];
    }

    @Override
    public Integer convertToDatabaseColumn(E attribute) {
        return Optional.ofNullable(attribute)
                .map(IEnum::getValue)
                .orElse(null);
    }

    @Override
    public E convertToEntityAttribute(Integer dbData) {
        return Arrays.stream(clazz.getEnumConstants())
                .filter(iEnum -> iEnum.getNumber().equals(dbData))
                .findFirst()
                .orElse(null);
    }
}
```

5、然后枚举类继承我们的父类枚举接口，再定义一个子类实现，就可以实现枚举的自动转换

```java
import lombok.Getter;
import lombok.RequiredArgsConstructor;
import javax.persistence.Converter;

@Getter
@RequiredArgsConstructor
public enum SexEnum implements IEnum {
    /**
     * 1 男
     */
    MALE(1, "男"),
    /**
     * 2 女
     */
    FEMALE(2, "女"),
    ;
    private final Integer number;
    private final String value;

    @Converter(autoApply = true)
    public static class SexEnumConverter extends EnumConverter<SexEnum> {
    }
}
```

这样子的话，对于数据库的查询或者保存，就可以直接使用枚举进行操作，而不用再去手动转换。



## 3、请求参数转换枚举

### 1、@Param 参数转换成枚举

对于请求参数，我们分为两种，先看第一种，携带在 URL 后面的参数。

Spring 提供了一个 ConverterFactory 接口，我们通过实现他，便可以实现对枚举的自动转换，这里同样要使用到我们的父类接口 IEnum。

```java
@Component
public class IEnumConverterFactory implements ConverterFactory<String, IEnum> {
    @Override
    public <T extends IEnum> Converter<String, T> getConverter(Class<T> targetType) {
        return source ->
                Arrays.stream(targetType.getEnumConstants())
                        .filter(x -> x.getValue().toString().equals(source))
                        .findFirst()
                        .orElse(null);
    }
}
```

然后实现 WebMvcConfigurer，添加该类型转换器

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {
    private final IEnumConverterFactory iEnumConverterFactory;
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverterFactory(iEnumConverterFactory);
    }
}
```

然后我们便可以在在方法上使用枚举，MVC 框架会自动帮我们把数值转成对应的枚举。



### 2、@RequestBody 参数转换成枚举

使用 @RequestBody 接收请求参数，底层其实是使用了 Jackson 的反序列化功能。

我们可以继承 Jackson 的 JsonDeserializer 抽象类，自定义我们的反序列化策略。

```java
public class IEnumDeserializer extends JsonDeserializer<IEnum> implements ContextualDeserializer {
    private Class<? extends IEnum> clazz;

    public IEnumDeserializer() {
    }

    public IEnumDeserializer(Class<? extends IEnum> clazz) {
        this.clazz = clazz;
    }

    @SneakyThrows
    @Override
    public IEnum deserialize(JsonParser jsonParser, DeserializationContext ctxt) {
        final String param = jsonParser.getText();
        return Arrays.stream(clazz.getEnumConstants())
                .filter(x -> x.getValue().toString().equals(param))
                .findFirst()
                .orElse(null);
    }

    @SuppressWarnings({"unchecked"})
    @Override
    public JsonDeserializer<?> createContextual(DeserializationContext ctxt, BeanProperty property) throws JsonMappingException {
        JavaType type = property.getType();
        // 如果是容器，则返回容器内部枚举类型
        while (type.isContainerType()) {
            type = type.getContentType();
        }
        return new IEnumDeserializer((Class<? extends IEnum>) type.getRawClass());
    }
}
```

然后在我们的 IEnum 类上，添加 @JsonDeserialize 注解，指定反序列化。

```java
@JsonDeserialize(using = IEnumDeserializer.class)
public interface IEnum {
    Integer getValue();
}
```

这样子，就可以在 @RequestBody 注解的实体类里面使用枚举。



## 4、响应参数转换枚举

我们返回响应参数时，经常需要把枚举值转成对应的 int，因为Spring boot框架响应参数的时候，采用的也是 Jackson 序列化，因此我们可以采用类似上面的做法，继承 JsonSerializer 抽象类，实现自定义转换。

```java
public class IEnumSerializer extends JsonSerializer<IEnum> {
    @Override
    public void serialize(IEnum value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        if (Objects.isNull(value)) {
            gen.writeNull();
            return;
        }
        gen.writeNumber(value.getValue());
    }
}
```

然后在我们的 IEnum 类上，添加 @JsonDeserialize 注解，指定序列化。

```java
@JsonSerialize(using = IEnumSerializer.class)
@JsonDeserialize(using = IEnumDeserializer.class)
public interface IEnum {
    Integer getValue();
}
```

这样子，我们在给前端返回枚举的时候，就可以不用自己手动进行转换，让框架自动帮我们进行转换。



## 5、消息参数转换枚举

其实通过我们上面的配置，已经自定义了枚举的序列化和反序列化，只需要在我们的消息接收和发送的时候，采用 Jackson 序列化的形式，就可以实现枚举的自动转换。



## 6、总结

通过以上的配置，基本上可以实现我们整个项目的枚举统一，消除我们在项目内频繁需要对枚举和数值互相转换的操作。

如果是前端页面有相关的枚举下拉框选择查询，我们还可以根据顶层的枚举接口，对所有枚举进行扫描并存储到内存中，然后提供枚举查询的接口给前端，这样子的话，后续新增枚举参数的话，可以直接就从接口获取到更新的值。
