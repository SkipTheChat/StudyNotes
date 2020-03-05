# 1 代码示例

它为 Java 开发人员提供了一种对象/关联映射工具来管理 Java 应用中的关系数据。他的出现主要是为了简化现有的持久化开发工作和整合 ORM 技术.

> 配置

```properties
spring.datasource.url=jdbc:mysql://*:*/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql=true
```



> 使用

```java
//Entity实体类
@Entity
@Data
public class User {

    @Id
    @GeneratedValue
    private long id;
    @Column(nullable = false, unique = true)
    private String userName;
    @Column(nullable = false)
    private String password;
    @Column(nullable = false)
    private int age;
}


//声明 UserRepository接口，继承JpaRepository，默认支持简单的 CRUD 操作
public interface UserRepository extends JpaRepository<User, Long> {

    User findByUserName(String userName);

}

//测试
@Slf4j
public class UserTest extends ApplicationTests {

    @Autowired
    private UserRepository userRepository;

    @Test
    @Transactional
    public void userTest() {
        User user = new User();
        user.setUserName("wyk");
        user.setAge(30);
        user.setPassword("aaabbb");
        userRepository.save(user);
        User item = userRepository.findByUserName("wyk");
        log.info(JsonUtils.toJson(item));
    }
}
```



# 2 原理

导入jpa，自动配置扫描包内所有继承了Repository接口的接口，然后遍历这些接口，针对每个接口依次创建如下几个实例 ：

被代理的类是`JpaRepository`的一个实现`SimpleJpaRespositry` 

1.  SimpleJpaRespositry——用来进行默认的DAO操作，是所有Repository的默认实现
2. JpaRepositoryFactoryBean——装配bean，装载了动态代理Proxy，会以对应的DAO的beanName为key注册到DefaultListableBeanFactory中，在需要被注入的时候从这个bean中取出对应的动态代理Proxy注入给DAO
3. JdkDynamicAopProxy——每个实现了JpaRepository接口的接口，都会生成一个动态代理，动态代理对应的InvocationHandler，负责拦截DAO接口的所有的方法调用

 

> 总结

spring 会在启动的时候扫描所有继承自 Repository 接口的 DAO  接口，然后为其实例化一个动态代理，同时根据它的方法名、参数等为其装配一系列DB操作组件，在需要注入的时候为对应的接口注入这个动态代理，在 DAO 方法被调用的时会走这个动态代理，然后经过一系列的方法拦截路由到最终的 DB 操作执行器`JpaQueryExecution`，然后拼装 sql，执行相关操作，返回结果。 

 

 

 



