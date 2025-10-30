## SpringBoot启动初始化数据库

> [SpringBoot启动时初始化数据库及spring.jpa.generate-dll与spring.jpa.hibernate.ddl-auto之间的困惑](https://www.cnblogs.com/StarkBrothers/p/11804554.html)
>
> spring.jpa.generate-dll 与 spring.jpa.hibernate.ddl-auto之间的困惑

参考文章：[Springboot2.x 自动创建表并且执行初始化数据](https://www.cnblogs.com/klyjb/p/11002778.html)

1、SpringBoot根据脚本初始化

　　结构初始化脚本文件由spring.datasource.schema属性指定，数据初始化脚本由文件spring.datasource.data属性指定。

　　这两个脚本是否执行的开关由spring.datasource.initialization-mode决定：always：一定执行，embedded：只对内存数据库执行(默认)，never：不执行



2、JPA根据类结构初始化

　　@Entity注解的类将会被初始化一张数据库表。

　　是否执行的开关由spring.jpa.hibernate.ddl-auto属性控制，可为create/update/create-drop/none/validate，其中none和validate为不执行。



3、初始化时对于另一个属性spring.jpa.generate-dll在理解上的困惑

　　可以参考这篇文章---[用属性控制Hibernate生成DDL](https://fookwood.com/hibernate-generate-ddl)。

　　按照它的描述，spring.jpa.generate-dll是jpa层面对数据表生成策略的控制，而spring.jpa.hibernate.ddl-auto是jpa实现hibernate层面上对数据表生成策略的控制。

　　实践也验证了文章里所说的，spring.jpa.generate-dll比spring.jpa.hibernate.ddl-auto有更强的控制力度，即使spring.jpa.hibernate.ddl-auto为none，只要spring.jpa.generate-dll为true，也会根据@Entity注解的实体类生成对应数据表。

　　按照文章的建议，为了避免混淆和不好理解，这两者最好不要混用，只对JPA实现层面的控制属性spring.jpa.hibernate.ddl-auto进行设置即可。



4、让SpringBoot根据脚本和让JPA根据实体类进行初始化，两者之中选择一个即可。