> 自定义校验注解ConstraintValidator：https://mp.weixin.qq.com/s/5NtKsFO179pOflNYT1JPwA
>
> 一个很常见的需求，某表中的数据不能重复，我闲每次都要多加几行代码去查数据库，所以使用自定义校验注解ConstraintValidator

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.1.7.Final</version>
</dependency>
```

```java
package com.example.demo.annotation;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.util.StringUtils;
import javax.validation.Constraint;
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import javax.validation.Payload;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.Map;

/**
 * <p>
 * 数据库字段唯一值
 * </p>
 *
 * @author duguotao
 * @version 1.0.0
 * @since Created in 2022/1/26
 */
// 指定我们自定义的校验类
@Constraint(validatedBy = TableDataUnique.TableDataUniqueConstraintValidator.class)
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TableDataUnique {

    // 表明
    String table();

    // 字段
    String column();

    /**
     * 校验不通过时的报错信息
     */
    String message() default "参数错误";

    /**
     * 将validator进行分类，不同的类group中会执行不同的validator操作
     * @return validator的分类类型
     */
    Class<?>[] groups() default {};

    /**
     * 主要是针对bean，很少使用
     * @return 负载
     */
    Class<? extends Payload>[] payload() default {};

    class TableDataUniqueConstraintValidator implements ConstraintValidator<TableDataUnique, String>, ApplicationContextAware {

        String table;
        String column;
        static ApplicationContext application;
        
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            application = applicationContext;
        }

        @Override
        public void initialize(TableDataUnique tableDataUnique) {
            table = tableDataUnique.table();
            column = tableDataUnique.column();
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) {
            if (StringUtils.isEmpty(table) || StringUtils.isEmpty(column)) {
                return false;
            }

            String solitary = "'";
            String sql = "select count(1) 'count' from " + table + " where " + column + " = " + solitary + value + solitary;

            JdbcTemplate jdbcTemplate = application.getBean("jdbcTemplate", JdbcTemplate.class);
            Map<String, Object> map = jdbcTemplate.queryForMap(sql);

            return map.get("count").equals(0L);
        }

    }

}
 
 
```

```java
@Data
public class SaveRoleDTO {
    // 角色表中 名称不允许重复
    @TableDataUnique(table = "ipm_role", cloumn = "name", message = "角色名已存在")
    private String name;
}
```

