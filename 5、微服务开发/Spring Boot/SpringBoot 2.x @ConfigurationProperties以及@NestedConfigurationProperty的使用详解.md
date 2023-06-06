> 来源：@ConfigurationProperties以及@NestedConfigurationProperty详解：https://blog.csdn.net/csdn_cai/article/details/117949600



# 1、创建一个SpringBoot应用

这里就不演示创建项目了

# 2、application.properties 配置

```properties
spring.datasources.test.username=admin
spring.datasources.test.password=123456
spring.datasources.test.url=url
spring.datasources.test.driver=com.mysql
spring.datasources.duration=5
```

# 3、@NestedConfigurationProperty 属性绑定

@ConfigurationProperties以及@NestedConfigurationProperty使用进行属性绑定

```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.NestedConfigurationProperty;
import org.springframework.boot.convert.DurationUnit;
import org.springframework.stereotype.Component;

import java.time.Duration;
import java.time.temporal.ChronoUnit;

/**
 * @Description 第一种绑定配置文件的方式
 * @NestedConfigurationProperty 嵌套配置属性
 * 如果在绑定属性文件的类中 的属性是外部的类(不是在本类中)
 * 那么就可以使用这个嵌套属性类解决
 * 需要注意的是
 * 配置文件是 :
 * spring.datasources.test.username=admin
 * spring.datasources.test.password=123456
 * spring.datasources.test.url=url
 * spring.datasources.test.driver=com.mysql
 * 那么 @ConfigurationProperties 前缀是spring.datasource
 * @NestedConfigurationProperty 标注的属性上的属性名称 必须是 test
 * <p>
 * 详解:
 * @ConfigurationProperties(value = "spring.datasources", ignoreInvalidFields = true, ignoreUnknownFields = false)
 * value 代表的是配置文件中的前缀
 * ignoreInvalidFields 表示忽略无效的字段  当我们为属性配置错误时，我们不希望springboot程序启动失败  故此设置为false
 * ignoreUnknownFields 表示忽略未知的字段 默认是true  当我们配置的属性没有绑定到 @ConfigurationProperties这个类时，我们希望程序报错
 **/
@Data
@Component
@ConfigurationProperties(value = "spring.datasources", ignoreInvalidFields = true, ignoreUnknownFields = false)
public class DataSourceProperties {
    @NestedConfigurationProperty
    private DataSource test;
    /**
     * 不写单位默认按照毫秒
     */
    @DurationUnit(ChronoUnit.DAYS)
    private Duration duration;
}
```

