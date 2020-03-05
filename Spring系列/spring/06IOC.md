> 基于`Spring 4.3.17.RELEASE` 

[Spring IOC 容器源码分析系列文章导读](http://www.coolblog.xyz/2018/05/30/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/)

[Spring IOC 容器源码分析 - 获取单例 bean](http://www.coolblog.xyz/2018/06/01/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E8%8E%B7%E5%8F%96%E5%8D%95%E4%BE%8B-bean/)

[Spring IOC 容器源码分析 - 创建单例 bean 的过程](http://www.coolblog.xyz/2018/06/04/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%88%9B%E5%BB%BA%E5%8D%95%E4%BE%8B-bean/)

[Spring IOC 容器源码分析 - 创建原始 bean 对象](http://www.coolblog.xyz/2018/06/06/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%88%9B%E5%BB%BA%E5%8E%9F%E5%A7%8B-bean-%E5%AF%B9%E8%B1%A1/)

[Spring IOC 容器源码分析 - 循环依赖的解决办法](http://www.coolblog.xyz/2018/06/08/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E7%9A%84%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95/) 

[Spring IOC 容器源码分析 - 填充属性到 bean 原始对象](http://www.coolblog.xyz/2018/06/11/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%A1%AB%E5%85%85%E5%B1%9E%E6%80%A7%E5%88%B0-bean-%E5%8E%9F%E5%A7%8B%E5%AF%B9%E8%B1%A1/) 

[Spring IOC 容器源码分析 - 余下的初始化工作](http://www.coolblog.xyz/2018/06/11/Spring-IOC-%E5%AE%B9%E5%99%A8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%BD%99%E4%B8%8B%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B7%A5%E4%BD%9C/) 

> 以上内容暂时没读



# 1 简介

> 反控：调用者只管负责从Spring 容器中获取需要使用的对象，不关心对象的创建过程，也不关心该对象依
> 赖对象的创建以及依赖关系的组装，也就是把创建对象的控制权反转给了Spring 框架。

##### IOC管理bean原理

1. 通过Resource 对象加载配置文件

2. 解析配置文件，得到指定名称的bean

3. 解析bean 元素，id 作为bean 的名字，class 用于反射得到bean 的实例：

   注意：此时，bean 类必须存在一个无参数构造器(和访问权限无关)；

4. 调用getBean 方法的时候，从容器中返回对象实例；





# 2

[博客](https://javadoop.com/post/spring-ioc)

```java
@Test
void testBeanApplicationContext() throws Exception {
    ApplicationContext ctx = new
ClassPathXmlApplicationContext("classpath:applicationfile.xml");//已创建好所有bean
ctx.getBean("someBean", SomeBean.class);
}
```





