# 乐优商城 - 微服务架构 

# 1 项目背景

近年来中国的电子商务发展的很快，双十一的时候全天成交额高达2600亿，每秒的成交量可以高达50万，这样的高并发量如果项目架构不够完善，有一点细节上的漏洞都会导致错误，甚至会发生系统奔溃。

乐优商城是一个全品类的电商购物网站（B2C，商家对个人，如：亚马逊、当当等）

分为前台用户界面和后台商家管理界面。

前台用户界面目前实现的功能有

- 根据关键词搜索、根据分类过滤搜索、点击商品进入详情页
- 注册、登录、点击商品属性加入购物车
- 下单、支付



后台管理界面实现的功能有：

- 商品管理的分类管理、品牌管理、商品列表、规格参数四个模块的增删改查，还有分级列表和搜索

![TIM截图20191209214657](.\assets\TIM截图20191209214657.png)

![TIM截图20191209214804](.\assets\TIM截图20191209214804.png)

![TIM截图20191209214851](.\assets\TIM截图20191209214851.png)



目标是希望未来3到5年可以支持千万用户的使用



# 2 使用技术

Springboot+springcloud开发。

- SpringBoot
- SpringCloud
- SpringJPA
- Mybatis
- Elasticsearch
- RabbitMQ
- Thymeleaf
- mysql 5.6
- Redis
- FastDFS
- nginx

JDK：统一使用JDK1.8，maven3.3.9以上版本



一级域名：www.leyou.com，leyou.com leyou.cn 

二级域名：manage.leyou.com/item , api.leyou.com







# 3 项目架构

![1525703759035](./assets/1525703759035.png)





springcloud：

- Eureka：服务治理组件，包含服务注册中心，服务注册与发现机制的实现。（服务治理，服务注册/发现） 
- Zuul：网关组件，提供智能路由，访问过滤功能 
- Ribbon：客户端负载均衡的服务调用组件（客户端负载） 
- Feign：服务调用，给予Ribbon和Hystrix的声明式服务调用组件 （声明式服务调用） 
- Hystrix：容错管理组件，实现断路器模式，帮助服务依赖中出现的延迟和为故障提供强大的容错能力。(熔断、断路器，容错) 

![1525575656796](./assets/1525575656796.png)



![1525597885059](./assets/1525597885059.png)



- Eureka：就是服务注册中心（可以是一个集群），对外暴露自己的地址
- 提供者：启动后向Eureka注册自己信息（地址，提供什么服务）
- 消费者：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新
- 心跳(续约)：提供者定期通过http方式向Eureka刷新自己的状态





### 3.1 初步搭建

**leyou-common(不需要启动类)**

**leyou-interface(不需要启动类)**

**leyou-item-service：**

启动类：

```java
@SpringBootApplication
@EnableDiscoveryClient //来开启Eureka客户端功能
public class LeyouItemApplication {

    public static void main(String[] args) {
        SpringApplication.run(LeyouItemApplication.class);
    }
}
```

> 这里有个注意点

@SpringBootApplication其实是一个springboot的组合注解，用于启动springboot相关配置，这里重点的注解有3个：

- @SpringBootConfiguration
- @EnableAutoConfiguration：开启自动配置
- @ComponentScan：开启注解扫描



resources下的yml配置：

```yaml
server:
  port: 9081 # 端口
spring:
  application:
    name: item-service  # 应用名称，会在Eureka中显示
  datasource:
    url: jdbc:mysql:///hm49
    username: root
    password: admin
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10086/eureka  
  instance:
    lease-renewal-interval-in-seconds: 5 #心跳时间
    lease-expiration-duration-in-seconds: 15 #过期时间
mybatis:
  type-aliases-package: com.leyou.item.pojo #mybatis扫描包
```

> 参数说明

默认情况下每个30秒服务会向注册中心发送一次心跳，证明自己还活着。如果超过90秒没有发送心跳，EurekaServer就会认为该服务宕机，会从服务列表中移除，这两个值在生产环境不要修改，默认即可。



**leyouregistry注册中心：**

```java
@SpringBootApplication
@EnableEurekaServer  // 声明当前springboot应用是一个eureka服务中心
public class LeyouResgistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(LeyouResgistryApplication.class);
    }
}
```



yml配置：

```yaml
server:
  port: 10086
spring:
  application:
    name: leyou-registry
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10086/eureka
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 10000
```



**leyougateway网关：**

```java
@SpringBootApplication
@EnableDiscoveryClient //开启ereka
@EnableZuulProxy //开启网关
public class LeyouGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(LeyouGatewayApplication.class);
    }
}
```



yml配置文件：

```yaml
server:
  port: 10010
spring:
  application:
    name: leyou-gateway
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10086/eureka
    registry-fetch-interval-seconds: 5
zuul:
  prefix: /api
#  配置路由
  routes:
    item-service: /item/** # 路由到商品的微服务
```











架构设计比如网关啊哪个服务调用哪个服务啦消息中间件啦等等  
设计理念里面比如为什么我用redis为什么我要做分布式事务咋做的   
数据库分表  缓存  配置  监控  报警  监控报警怎么处理  
挑一个具体实现讲一讲

如何做到高性能高可用高扩展  消息异步  限流 集群 请求无状态  





# 3 数据库表设计





# 4 











# 1.项目用到的技术

项目采用前后端分离开发，前端代码已给，为leyou-manage-web和leyou-portal。

核心的技术栈用的是SpringBoot+SpringCloud。







# 2.项目模块说明

| 作用                                                        | 模块名          | 端口     |
| :---------------------------------------------------------- | :-------------- | :------- |
| Springcloud的eruka注册中心，注册了所有的模块业务            | leyou-registry  | 10086    |
| 网关                                                        | leyou-gateway   | 10010    |
| 商家后台的管理商品页面                                      | leyou-item      | 9081     |
| 专门的图片上传模块                                          | leyou-upload    | 9082     |
| 搜索功能(elasticsearch)                                     | leyou-search    | 9083     |
| 点击商品打开后的商品详情页面模块                            | leyou-goods-web | 9084     |
| 短信验证工具模块，并使用了rabbitMQ接收消息                  | leyou-sms       | 9086     |
| 用户登录注册校验身份模块，使用了rabbitMQ发送消息给leyou-sms | leyou-user      | 9085     |
| 授权中心（生成与解析token）                                 | leyou-auth      | 9087     |
| 购物车模块                                                  | leyou-cart      | 9088     |
| 订单模块                                                    | leyou-order     | 9089     |
| 公共模块，存放工具类等。（无需启动）                        | leyou-common    | 无需启动 |



# 3.项目开发进度













# 4.项目部署

>部署前请先看注意事项修改相关配置





### 4.1 开启商家管理界面

cmd进入leyou-manage-web项目路径，启动命令：

```
npm start
```



>[访问网址：manage.leyou.com](manage.leyou.com)





### 4.2 开启用户搜索界面

cmd进入leyou-portal项目路径，启动命令：

```
live-server --port=9002
```



> [访问网址：www.leyou.com](www.leyou.com)



### 4.3 开启后端项目

用编译器打开项目，分别用tomcat启动各个模块。

**模块启动顺序与项目模块说明的顺序一致**。若不一致，可能导致启动报错。





# 5.注意事项



### 5.1依赖

项目启动之前需启动**elasticsearch，redis以及nginx**，相关虚拟机地址在各yml文件中修改。



### 5.2公钥私钥生成

 验证用户登录需要使用公私钥，自动生成的代码在leyou项目目录下：

	leyou-auth/leyou-auth-common/src/test/com/leyou/auth/test/JwtTest.java



使用方法：

- 先注释下面代码块代码
- 然后运行testRsa方法生成密钥

```java
@Before
public void testGetRsa() throws Exception {
    this.publicKey = RsaUtils.getPublicKey(pubKeyPath);
    this.privateKey = RsaUtils.getPrivateKey(priKeyPath);
}
```

- 生好了记得修改leyou-gateway，leyou-auth-service,leyou-cart,leyou-order这四个模块的application.yml文件公私钥地址为你自己的公私钥生成地址





### 5.3 阿里云验证码短信

在leyou-ssm模块application.yml中配置了阿里云短信，这里改成你自己的账号。具体申请方式自发百度。

```yaml
#自己去阿里云短信申请账号填一下这里就行了
  sms:
    accessKeyId:  # 你自己的accessKeyId
    accessKeySecret:  # 你自己的AccessKeySecret
    signName:  # 签名名称
    verifyCodeTemplate:  # 模板名称
```





### 5.4 微信支付

在leyou-order模块的微信支付配置（application.yml）需要自己去申请。

我没申请到所以用的是别人的。

申请方式自行百度。





### 5.5 sql文件

> 注意：只能MySQL5.6环境引入sql文件，其他的不行哦。





### 5.6 nginx地址&host地址配置参考

- nginx配置参考当前目录下nginx.conf

- host配置如下

  ```yaml
  # SwitchHosts!
  
  # My hosts
  
  # leyou
  127.0.0.1 manage.leyou.com
  127.0.0.1 api.leyou.com
  192.168.153.129 image.leyou.com
  127.0.0.1 www.leyou.com
  ```

  



