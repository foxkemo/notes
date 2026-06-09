下面给你把 **Spring Data（JPA） + MyBatis-Plus（MP）在同一项目中做多库配置** 从结构到代码一次讲清楚。这个属于**企业级常见架构**（比如：老模块用 JPA，新模块用 MP，或者不同库用不同技术栈）。

---

# 一、总体思路（先建立全局）

👉 核心其实是：**多个 DataSource + 分别绑定各自技术栈**

```text
应用
 ├── 数据源 db1 → Spring Data JPA
 └── 数据源 db2 → MyBatis-Plus
```

---

# 二、配置文件（application.yml）

```yaml
spring:
  datasource:
    db1:
      url: jdbc:mysql://localhost:3306/db1
      username: root
      password: root
      driver-class-name: com.mysql.cj.jdbc.Driver

    db2:
      url: jdbc:mysql://localhost:3306/db2
      username: root
      password: root
      driver-class-name: com.mysql.cj.jdbc.Driver
```

---

# 三、第一步：配置两个 DataSource（核心）

```java
@Configuration
public class DataSourceConfig {

    @Bean(name = "db1DataSource")
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    public DataSource db1DataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "db2DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSource db2DataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

---

# 四、JPA（Spring Data）绑定 db1

---

## 1️⃣ EntityManagerFactory

```java
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.db1.repository",
    entityManagerFactoryRef = "db1EntityManagerFactory",
    transactionManagerRef = "db1TransactionManager"
)
public class JpaConfig {

    @Bean
    public LocalContainerEntityManagerFactoryBean db1EntityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("db1DataSource") DataSource dataSource) {

        return builder
                .dataSource(dataSource)
                .packages("com.example.db1.entity")
                .build();
    }
}
```

---

## 2️⃣ 事务管理

```java
@Bean
public PlatformTransactionManager db1TransactionManager(
        @Qualifier("db1EntityManagerFactory") EntityManagerFactory factory) {
    return new JpaTransactionManager(factory);
}
```

---

# 五、MyBatis-Plus 绑定 db2（重点）

---

## 1️⃣ SqlSessionFactory

```java
@Configuration
@MapperScan(
    basePackages = "com.example.db2.mapper",
    sqlSessionFactoryRef = "db2SqlSessionFactory"
)
public class MybatisPlusConfig {

    @Bean
    public SqlSessionFactory db2SqlSessionFactory(
            @Qualifier("db2DataSource") DataSource dataSource) throws Exception {

        MybatisSqlSessionFactoryBean factory = new MybatisSqlSessionFactoryBean();
        factory.setDataSource(dataSource);

        return factory.getObject();
    }
}
```

---

## 2️⃣ 事务管理

```java
@Bean
public DataSourceTransactionManager db2TransactionManager(
        @Qualifier("db2DataSource") DataSource dataSource) {

    return new DataSourceTransactionManager(dataSource);
}
```

---

# 六、代码结构划分（非常关键）

```text
com.example
 ├── db1（JPA）
 │     ├── entity
 │     └── repository
 │
 └── db2（MyBatis-Plus）
       ├── entity
       └── mapper
```

---

# 七、使用方式

---

## ✔ JPA

```java
@Autowired
private UserRepository userRepository;
```

---

## ✔ MyBatis-Plus

```java
@Autowired
private UserMapper userMapper;
```

---

# 八、事务使用（注意）

---

## ✔ 指定事务管理器

```java
@Transactional("db1TransactionManager")
public void jpaMethod() {}
```

```java
@Transactional("db2TransactionManager")
public void mpMethod() {}
```

---

# 九、进阶：动态数据源（统一入口）

如果你不想写两套：

👉 可以用：

```text
AbstractRoutingDataSource
```

- AOP：
    

```java
@DS("db1")
```

---

# 十、常见坑（非常重要）

---

## ❗1. @Primary 必须有一个

```java
@Primary
```

否则 Spring 不知道默认用哪个 DataSource

---

## ❗2. 包扫描必须分开

```text
JPA 和 MP 不能混在一起
```

---

## ❗3. 事务不共享

👉 JPA 和 MP：

```text
不是同一个事务管理器 ❗
```

---

## ❗4. SqlSessionFactory 必须用 MP 的

```java
MybatisSqlSessionFactoryBean
```

不是：

```java
SqlSessionFactoryBean
```

---

# 十一、本质理解（非常重要）

👉 这套配置本质是：

|层|技术|
|---|---|
|数据源|DataSource|
|ORM 1|JPA|
|ORM 2|MyBatis-Plus|
|路由|Bean + 注解|
|执行|Mapper / Repository|

---

# 十二、终极一句话总结

👉 **Spring 多库 + JPA + MyBatis-Plus，本质是将不同 DataSource 分别绑定到不同 ORM 框架，通过 Bean 配置和包隔离实现解耦与扩展**

---

# 十三、如果你想再进阶（强烈推荐）

我可以帮你讲一个企业级更高级方案：

👉 **动态数据源 + MyBatis-Plus + 注解切换（@DS）+ AOP 实现统一访问层**

这个是大厂常见架构 👍