# Datasource

- spring.dao.exceptiontranslation.enabled
  是否开启PersistenceExceptionTranslationPostProcessor，默认为true
- spring.datasource.abandon-when-percentage-full
  设定超时被废弃的连接占到多少比例时要被关闭或上报
- spring.datasource.allow-pool-suspension
  使用Hikari pool时，是否允许连接池暂停，默认为: false
- spring.datasource.alternate-username-allowed
  是否允许替代的用户名.
- spring.datasource.auto-commit
  指定updates是否自动提交.
- spring.datasource.catalog
  指定默认的catalog.
- spring.datasource.commit-on-return
  设置当连接被归还时，是否要提交所有还未完成的事务
- spring.datasource.connection-init-sql
  指定连接被创建，再被添加到连接池之前执行的sql.
- spring.datasource.connection-init-sqls
  使用DBCP connection pool时，指定初始化时要执行的sql
- spring.datasource.connection-properties.[key]
  在使用DBCP connection pool时指定要配置的属性
- spring.datasource.connection-test-query
  指定校验连接合法性执行的sql语句
- spring.datasource.connection-timeout
  指定连接的超时时间，毫秒单位.
- spring.datasource.continue-on-error
  在初始化数据库时，遇到错误是否继续，默认false
- spring.datasource.data
  指定Data (DML)脚本
- spring.datasource.data-source-class-name
  指定数据源的全限定名.
- spring.datasource.data-source-jndi
  指定jndi的地址
- spring.datasource.data-source-properties.[key]
  使用Hikari connection pool时，指定要设置的属性
- spring.datasource.db-properties
  使用Tomcat connection pool，指定要设置的属性
- spring.datasource.default-auto-commit
  是否自动提交.
- spring.datasource.default-catalog
  指定连接默认的catalog.
- spring.datasource.default-read-only
  是否设置默认连接只读.
- spring.datasource.default-transaction-isolation
  指定连接的事务的默认隔离级别.
- spring.datasource.driver-class-name
  指定driver的类名，默认从jdbc url中自动探测.
- spring.datasource.fair-queue
  是否采用FIFO返回连接.
- spring.datasource.health-check-properties.[key]
  使用Hikari connection pool时，在心跳检查时传递的属性
- spring.datasource.idle-timeout
  指定连接多久没被使用时，被设置为空闲，默认为10ms
- spring.datasource.ignore-exception-on-pre-load
  当初始化连接池时，是否忽略异常.
- spring.datasource.init-sql
  当连接创建时，执行的sql
- spring.datasource.initial-size
  指定启动连接池时，初始建立的连接数量
- spring.datasource.initialization-fail-fast
  当创建连接池时，没法创建指定最小连接数量是否抛异常
- spring.datasource.initialize
  指定初始化数据源，是否用data.sql来初始化，默认: true
- spring.datasource.isolate-internal-queries
  指定内部查询是否要被隔离，默认为false
- spring.datasource.jdbc-interceptors
  使用Tomcat connection pool时，指定jdbc拦截器，分号分隔
- spring.datasource.jdbc-url
  指定JDBC URL.
- spring.datasource.jmx-enabled
  是否开启JMX，默认为: false
- spring.datasource.jndi-name
  指定jndi的名称.
- spring.datasource.leak-detection-threshold
  使用Hikari connection pool时，多少毫秒检测一次连接泄露.
- spring.datasource.log-abandoned
  使用DBCP connection pool，是否追踪废弃statement或连接，默认为: false
- spring.datasource.log-validation-errors
  当使用Tomcat connection pool是否打印校验错误.
- spring.datasource.login-timeout
  指定连接数据库的超时时间.
- spring.datasource.max-active
  指定连接池中最大的活跃连接数.
- spring.datasource.max-age
  指定连接池中连接的最大年龄
- spring.datasource.max-idle
  指定连接池最大的空闲连接数量.
- spring.datasource.max-lifetime
  指定连接池中连接的最大生存时间，毫秒单位.
- spring.datasource.max-open-prepared-statements
  指定最大的打开的prepared statements数量.
- spring.datasource.max-wait
  指定连接池等待连接返回的最大等待时间，毫秒单位.
- spring.datasource.maximum-pool-size
  指定连接池最大的连接数，包括使用中的和空闲的连接.
- spring.datasource.min-evictable-idle-time-millis
  指定一个空闲连接最少空闲多久后可被清除.
- spring.datasource.min-idle
  指定必须保持连接的最小值(For DBCP and Tomcat connection pools)
- spring.datasource.minimum-idle
  指定连接维护的最小空闲连接数，当使用HikariCP时指定.
- spring.datasource.name
  指定数据源名.
- spring.datasource.num-tests-per-eviction-run
  指定运行每个idle object evictor线程时的对象数量
- spring.datasource.password
  指定数据库密码.
- spring.datasource.platform
  指定schema要使用的Platform(schema-${platform}.sql)，默认为: all
- spring.datasource.pool-name
  指定连接池名字.
- spring.datasource.pool-prepared-statements
  指定是否池化statements.
- spring.datasource.propagate-interrupt-state
  在等待连接时，如果线程被中断，是否传播中断状态.
- spring.datasource.read-only
  当使用Hikari connection pool时，是否标记数据源只读
- spring.datasource.register-mbeans
  指定Hikari connection pool是否注册JMX MBeans.
- spring.datasource.remove-abandoned
  指定当连接超过废弃超时时间时，是否立刻删除该连接.
- spring.datasource.remove-abandoned-timeout
  指定连接应该被废弃的时间.
- spring.datasource.rollback-on-return
  在归还连接时，是否回滚等待中的事务.
- spring.datasource.schema
  指定Schema (DDL)脚本.
- spring.datasource.separator
  指定初始化脚本的语句分隔符，默认: ;
- spring.datasource.sql-script-encoding
  指定SQL scripts编码.
- spring.datasource.suspect-timeout
  指定打印废弃连接前的超时时间.
- spring.datasource.test-on-borrow
  当从连接池借用连接时，是否测试该连接.
- spring.datasource.test-on-connect
  创建时，是否测试连接
- spring.datasource.test-on-return
  在连接归还到连接池时是否测试该连接.
- spring.datasource.test-while-idle
  当连接空闲时，是否执行连接测试.
- spring.datasource.time-between-eviction-runs-millis
  指定空闲连接检查、废弃连接清理、空闲连接池大小调整之间的操作时间间隔
- spring.datasource.transaction-isolation
  指定事务隔离级别，使用Hikari connection pool时指定
- spring.datasource.url
  指定JDBC URL.
- spring.datasource.use-disposable-connection-facade
  是否对连接进行包装，防止连接关闭之后被使用.
- spring.datasource.use-equals
  比较方法名时是否使用String.equals()替换==.
- spring.datasource.use-lock
  是否对连接操作加锁
- spring.datasource.username
  指定数据库名.
- spring.datasource.validation-interval
  指定多少ms执行一次连接校验.
- spring.datasource.validation-query
  指定获取连接时连接校验的sql查询语句.
- spring.datasource.validation-query-timeout
  指定连接校验查询的超时时间.
- spring.datasource.validation-timeout
  设定连接校验的超时时间，当使用Hikari connection pool时指定
- spring.datasource.validator-class-name
  用来测试查询的validator全限定名.
- spring.datasource.xa.data-source-class-name
  指定数据源的全限定名.
- spring.datasource.xa.properties
  指定传递给XA data source的属性

# JPA

- spring.jpa.database
  指定目标数据库.
- spring.jpa.database-platform
  指定目标数据库的类型.
- spring.jpa.generate-ddl
  是否在启动时初始化schema，默认为false
- spring.jpa.hibernate.ddl-auto
  指定DDL mode (none, validate, update, create, create-drop). 当使用内嵌数据库时，默认是create-drop，否则为none.
- spring.jpa.hibernate.naming-strategy
  指定命名策略.
- spring.jpa.open-in-view
  是否注册OpenEntityManagerInViewInterceptor，绑定JPA EntityManager到请求线程中，默认为: true
- spring.jpa.properties
  添加额外的属性到JPA provider.
- spring.jpa.show-sql
  是否开启sql的log，默认为: false

# Jooq

- spring.jooq.sql-dialect
  指定JOOQ使用的SQLDialect，比如POSTGRES.

# H2

- spring.h2.console.enabled
  是否开启控制台，默认为false
- spring.h2.console.path
  指定控制台路径，默认为: /h2-console

# JTA

- - spring.jta.allow-multiple-lrc
    是否允许 multiple LRC，默认为: false
  - spring.jta.asynchronous2-pc
    指定两阶段提交是否可以异步，默认为: false
  - spring.jta.background-recovery-interval
    指定多少分钟跑一次recovery process，默认为: 1
  - spring.jta.background-recovery-interval-seconds
    指定多久跑一次recovery process，默认: 60
  - spring.jta.current-node-only-recovery
    是否过滤掉其他非本JVM的recovery，默认为: true
  - spring.jta.debug-zero-resource-transaction
    是否追踪没有使用指定资源的事务，默认为: false
  - spring.jta.default-transaction-timeout
    设定默认的事务超时时间，默认为60
  - spring.jta.disable-jmx
    是否禁用jmx，默认为false
  - spring.jta.enabled
    是否开启JTA support，默认为: true
  - spring.jta.exception-analyzer
    设置指定的异常分析类
  - spring.jta.filter-log-status
    使用Bitronix Transaction Manager时，是否写mandatory logs，开启的话，可以节省磁盘空间，但是调试会复杂写，默认为false
  - spring.jta.force-batching-enabled
    使用Bitronix Transaction Manager时，是否批量写磁盘，默认为true.
  - spring.jta.forced-write-enabled
    使用Bitronix Transaction Manager时，是否强制写日志到磁盘，默认为true
  - spring.jta.graceful-shutdown-interval
    当使用Bitronix Transaction Manager，指定shutdown时等待事务结束的时间，超过则中断，默认为60
  - spring.jta.jndi-transaction-synchronization-registry-name
    当使用Bitronix Transaction Manager时，在JNDI下得事务同步registry，默认为: java:comp/TransactionSynchronizationRegistry
  - spring.jta.jndi-user-transaction-name
    指定在JNDI使用Bitronix Transaction Manager的名称，默认:java:comp/UserTransaction
  - spring.jta.journal
    当使用Bitronix Transaction Manager，指定The journal是否disk还是null还是一个类的全限定名，默认disk
  - spring.jta.log-dir
    Transaction logs directory.
  - spring.jta.log-part1-filename
    指定The journal fragment文件1的名字，默认: btm1.tlog
  - spring.jta.log-part2-filename
    指定The journal fragment文件2的名字，默认: btm2.tlog
  - spring.jta.max-log-size-in-mb
    指定journal fragments大小的最大值. 默认: 2M
  - spring.jta.resource-configuration-filename
    指定Bitronix Transaction Manager配置文件名.
  - spring.jta.server-id
    指定Bitronix Transaction Manager实例的id.
  - spring.jta.skip-corrupted-logs
    是否忽略corrupted log files文件，默认为false.
  - spring.jta.transaction-manager-id
    指定Transaction manager的唯一标识.
  - spring.jta.warn-about-zero-resource-transaction
    当使用Bitronix Transaction Manager时，是否对没有使用指定资源的事务进行警告，默认为: true



# 参考资料 & 鸣谢

1. 转载：https://www.cnblogs.com/austinspark-jessylu/p/8065227.html