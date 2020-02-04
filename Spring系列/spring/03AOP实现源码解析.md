[Spring AOP 源码分析系列文章导读](http://www.coolblog.xyz/2018/06/17/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/)

[Spring AOP 源码分析 - 筛选合适的通知器](http://www.tianxiaobo.com/2018/06/20/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E7%AD%9B%E9%80%89%E5%90%88%E9%80%82%E7%9A%84%E9%80%9A%E7%9F%A5%E5%99%A8/)

[Spring AOP 源码分析 - 创建代理对象](http://www.tianxiaobo.com/2018/06/20/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%88%9B%E5%BB%BA%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1/)

[Spring AOP 源码分析 - 拦截器链的执行过程](http://www.coolblog.xyz/2018/06/22/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E6%8B%A6%E6%88%AA%E5%99%A8%E9%93%BE%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B/)



# 1 AOP实现原理剖析

先放一个AOP开发部分代码：

```java
//用于增强的类需要配Conponent和Aspect注解
@Component
@Aspect
public class TransactionManager {
	
	//表示给谁做增强
	@Pointcut("execution(* cn.wolfcode.service.*Service.*(..))")
	public void txPoint() {
		
	}
	
	//各种何时增强的标签
	@Before("txPoint()")
	public void begin(JoinPoint jp) {
		System.out.println("开启事务");
	}

	@AfterReturning("txPoint()")
	public void commit(JoinPoint jp) {
		System.out.println("提交事务");
	}

	@AfterThrowing(value="txPoint()",throwing="ex")
	public void rollback(JoinPoint jp, Throwable ex) {
		//ex和App-context.xml里面的ex一致
		System.out.println("回滚事务,异常信息:" + ex.getMessage());
	}

	@After("txPoint()")
	public void close(JoinPoint jp) {
		System.out.println("释放资源");
	}

	@Around("txPoint()")
	public Object aroundMethod(ProceedingJoinPoint pjp) {
		Object ret = null;
		System.out.println("开启事务");
		try {
			ret = pjp.proceed();//调用真实对象的方法
			System.out.println("提交事务");
		} catch (Throwable e) {
			System.out.println("回滚事务,异常信息=" + e.getMessage());
		} finally {
			System.out.println("释放资源");
		}
		return ret;
	}
}
```



```xml
	<!-- DI注解解析器 -->
	<context:annotation-config />
	<!-- IoC注解解析器 -->
	<context:component-scan base-package="cn.wolfcode" />
	<!-- AOP注解解析器 -->
	<!-- proxy-target-class="true"表示使用JDK，JDK动态代理需要实现接口 -->
	<aop:aspectj-autoproxy proxy-target-class="false"/>
```



[博客1](https://www.jianshu.com/p/8af3cdf3b499)

[博客2](https://blog.csdn.net/fighterandknight/article/details/51209822)

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/Spring%E7%B3%BB%E5%88%97/spring/assets/1.4.png)



**流程说明：**

1. AOP标签的定义解析刘彻骨肯定是从NamespaceHandlerSupport的实现类开始解析的，这个实现类就是AopNamespaceHandler。至于为什么会是从NamespaceHandlerSupport的实现类开始解析的，这个的话我想读者可以去在回去看看Spring自定义标签的解析流程，里面说的比较详细。
2. 要启用AOP，我们一般会在Spring里面配置<aop:aspectj-autoproxy/>  ，所以在配置文件中在遇到aspectj-autoproxy标签的时候我们会采用AspectJAutoProxyBeanDefinitionParser解析器
3. 进入AspectJAutoProxyBeanDefinitionParser解析器后，调用AspectJAutoProxyBeanDefinitionParser已覆盖BeanDefinitionParser的parser方法，然后parser方法把请求转交给了AopNamespaceUtils的registerAspectJAnnotationAutoProxyCreatorIfNecessary去处理
4. 进入AopNamespaceUtils的registerAspectJAnnotationAutoProxyCreatorIfNecessary方法后，先调用AopConfigUtils的registerAspectJAnnotationAutoProxyCreatorIfNecessary方法，里面在转发调用给registerOrEscalateApcAsRequired，注册或者升级AnnotationAwareAspectJAutoProxyCreator类。对于AOP的实现，基本是靠AnnotationAwareAspectJAutoProxyCreator去完成的，它可以根据@point注解定义的切点来代理相匹配的bean。
5. AopConfigUtils的registerAspectJAnnotationAutoProxyCreatorIfNecessary方法处理完成之后，接下来会调用useClassProxyingIfNecessary() 处理proxy-target-class以及expose-proxy属性。如果将proxy-target-class设置为true的话，那么会强制使用CGLIB代理，否则使用jdk动态代理，expose-proxy属性是为了解决有时候目标对象内部的自我调用无法实现切面增强。
6. 最后的调用registerComponentIfNecessary 方法，注册组建并且通知便于监听器做进一步处理。







![](./assets/1.6.png)



AnnotationAwareAspectJAutoProxyCreator是用来处理 当前项目当中 所有 AspectJ 注解的切面类。以及所有的 Spring Advisor ，同时 也是一个 BeanPostProcessor 。 

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/Spring%E7%B3%BB%E5%88%97/spring/assets/1.5.png)

我们可以看到这个类实现了BeanPostProcessor接口，那就意味着这个类在spring加载实例化前会调用postProcessAfterInitialization方法，对于AOP的逻辑也是由此开始的。 



## 1 创建代理类过程

AbstractAutoProxyCreator类

**postProcessAfterInitialization():**spring 容器启动，每个bean的实例化之前都会先经过这个方法

**wrapIfNecessary：**主要作用就是获得当前bean的增强器，然后将增强器的数组和bean作为参数，调用createProxy封装成一个代理对象。

```java
@Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    //bean 就是需要 被增强的类。
        if (bean != null) {
            Object cacheKey = getCacheKey(bean.getClass(), beanName);
            //防止 bean 多次被增强。
            if (!this.earlyProxyReferences.contains(cacheKey)) {
            //如果有必要 就进行封装，有没有必要主要取决于 bean 是否需要被增强。
                return wrapIfNecessary(bean, beanName, cacheKey);
            }
        }
        return bean;
    }

//======================wrapIfNecessary()===============================

/**
     *如果有必要 封装 bean
     */
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		//如果已经处理过
        if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
            return bean;
        }
		//如果不需要增强 就直接返回
        if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
            return bean;
        }
        //如果是一个 组件类，组件类包括Advice，Pointcut,Advisor,AopInfrastructureBean实现类，和Aspect注解注释的类，那么就标记为不需要增强的bean。所以 切面类 自己不会被AOP 增强。
        if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return bean;
        }
        // getAdvicesAndAdvisorsForBean：这个方法会提取当前bean的所有增强方法，然后获取到适合的当前bean的增强方法，然后对增强方法进行排序，最后返回。 
        Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
        //如果匹配到增强器 那么就创建增强代理类。
        if (specificInterceptors != DO_NOT_PROXY) {
            this.advisedBeans.put(cacheKey, Boolean.TRUE);
            //创建 增强代理类
            Object proxy = createProxy(
                    bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
            this.proxyTypes.put(cacheKey, proxy.getClass());
            return proxy;
        }
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }
```

 



### 1.1 获取增强器

##### 1.1.1 获取合适的增强器并排序返回

**getAdvicesAndAdvisorsForBean()：**方法获取到适合的当前bean的增强方法，然后对增强方法进行排序，最后返回。 

```java
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, TargetSource targetSource) {
    // 查找合适的通知器
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}

protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    
    //获取所有的增强器
        List<Advisor> candidateAdvisors = findCandidateAdvisors();
    //匹配合适的增强器
        List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
        extendAdvisors(eligibleAdvisors);
        if (!eligibleAdvisors.isEmpty()) {
            //对增强器排序
            eligibleAdvisors = sortAdvisors(eligibleAdvisors);
        }
    	//返回增强器
        return eligibleAdvisors;
    }

```



##### 1.1.2 获取所有增强器的代码

`findEligibleAdvisors.findCandidateAdvisors()：`获取所有增强器代码

```java
public class AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator {

    //...

    @Override
    protected List<Advisor> findCandidateAdvisors() {
        // 调用父类方法从容器中查找所有的通知器
        List<Advisor> advisors = super.findCandidateAdvisors();
        // 解析 @Aspect 注解，并构建通知器
        advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
        return advisors;
    }

    //...
}
```

- aspectJAdvisorsBuilder.buildAspectJAdvisors()
   代码比较多，就不粘出来了，大致逻辑就是

1. 获取容器中已经注册的BeanNames

2. 遍历 所有已经注册的Bean,查找 @AspectJ

3. 通过AspectJAdvisorFactory获取Advisor

   * 从容器中查找所有类型为 Advisor 的 bean 对应的名称
   * 遍历 advisorNames，并从容器中获取对应的 bean

   

 

##### 1.1.3 解析 @Aspect 注解并构建通知器

`buildAspectJAdvisors()`

```java
public List<Advisor> buildAspectJAdvisors() {
    List<String> aspectNames = this.aspectBeanNames;

    if (aspectNames == null) {
        synchronized (this) {
            aspectNames = this.aspectBeanNames;
            if (aspectNames == null) {
                List<Advisor> advisors = new LinkedList<Advisor>();
                aspectNames = new LinkedList<String>();
                // 从容器中获取所有 bean 的名称
                String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
                        this.beanFactory, Object.class, true, false);
                // 遍历 beanNames
                for (String beanName : beanNames) {
                    if (!isEligibleBean(beanName)) {
                        continue;
                    }
                    
                    // 根据 beanName 获取 bean 的类型
                    Class<?> beanType = this.beanFactory.getType(beanName);
                    if (beanType == null) {
                        continue;
                    }

                    // 检测 beanType 是否包含 Aspect 注解
                    if (this.advisorFactory.isAspect(beanType)) {
                        aspectNames.add(beanName);
                        AspectMetadata amd = new AspectMetadata(beanType, beanName);
                        if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
                            MetadataAwareAspectInstanceFactory factory =
                                    new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);

                            // 获取通知器
                            List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
                            if (this.beanFactory.isSingleton(beanName)) {
                                this.advisorsCache.put(beanName, classAdvisors);
                            }
                            else {
                                this.aspectFactoryCache.put(beanName, factory);
                            }
                            advisors.addAll(classAdvisors);
                        }
                        else {
                            if (this.beanFactory.isSingleton(beanName)) {
                                throw new IllegalArgumentException("Bean with name '" + beanName +
                                        "' is a singleton, but aspect instantiation model is not singleton");
                            }
                            MetadataAwareAspectInstanceFactory factory =
                                    new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
                            this.aspectFactoryCache.put(beanName, factory);
                            advisors.addAll(this.advisorFactory.getAdvisors(factory));
                        }
                    }
                }
                this.aspectBeanNames = aspectNames;
                return advisors;
            }
        }
    }

    if (aspectNames.isEmpty()) {
        return Collections.emptyList();
    }
    List<Advisor> advisors = new LinkedList<Advisor>();
    for (String aspectName : aspectNames) {
        List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
        if (cachedAdvisors != null) {
            advisors.addAll(cachedAdvisors);
        }
        else {
            MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
            advisors.addAll(this.advisorFactory.getAdvisors(factory));
        }
    }
    return advisors;
}

public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory,
        int declarationOrderInAspect, String aspectName) {

    validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());

    // 获取切点实现类
    AspectJExpressionPointcut expressionPointcut = getPointcut(
            candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
    if (expressionPointcut == null) {
        return null;
    }

    // 创建 Advisor 实现类
    return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
            this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}
```



如上getAdvisor 方法包含两个主要步骤，一个是获取 AspectJ 表达式切点，另一个是创建 Advisor  实现类。在第二个步骤中，包含一个隐藏步骤 – 创建 Advice。下面我将按顺序依次分析这两个步骤，先看获取 AspectJ  表达式切点的过程，如下： 



。。。。。。。。。。。。。。





### 1.2 创建代理类

获取到当前bean的增强方法后，便调用createProxy方法，创建代理。先创建代理工厂proxyFactory，然后获取当前bean 的增强器advisors，把当前获取到的增强器添加到代理工厂proxyFactory，然后设置当前的代理工的代理目标对象为当前bean，最后根据配置创建JDK的动态代理工厂，或者CGLIB的动态代理工厂，然后返回proxyFactory 

```java
/**
	 * @param specificInterceptors 此bean的拦截器集合(may be empty, but not null)
	 * @param targetSource the TargetSource for the proxy,
	 * already pre-configured to access the bean
	 * @return the AOP proxy for the bean
	 * @see #buildAdvisors
	 */

	protected Object createProxy(
			Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
 
		ProxyFactory proxyFactory = new ProxyFactory();
		// Copy our properties (proxyTargetClass etc) inherited from ProxyConfig.
        //this应该是当前正在实例化的Bean对象
		proxyFactory.copyFrom(this);
 
		if (!shouldProxyTargetClass(beanClass, beanName)) {
			
			Class<?>[] targetInterfaces = ClassUtils.getAllInterfacesForClass(beanClass, this.proxyClassLoader);
			for (Class<?> targetInterface : targetInterfaces) {
				proxyFactory.addInterface(targetInterface);
			}
		}
 		
        //获取当前bean的增强器advisors
		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
        //把当前获取到的增强器添加到代理工厂proxyFactory
		for (Advisor advisor : advisors) {
			proxyFactory.addAdvisor(advisor);
		}
 		//设置当前的代理工厂的代理目标对象为当前bean,这个对象是单例处理的。
		proxyFactory.setTargetSource(targetSource);
		customizeProxyFactory(proxyFactory);
 
		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}
 		//获取 AopProxy,AopProxy有JDK实现和 Cglib实现两种。
		return proxyFactory.getProxy(this.proxyClassLoader);
	}
```



调用proxyFactory.getProxy(this.proxyClassLoader)：

```java
public Object getProxy(ClassLoader classLoader) {
    //先获取 AopProxy,AopProxy有JDK实现和 Cglib实现两种。
        return createAopProxy().getProxy(classLoader);
}

```



判断采取JDK代理 或Cglib代理。
 判断条件：

1. Optimize： 是否采用了 激进的优化策略，该优化仅支持 Cglib代理
2. ProxyTargetClass： 代理目标类，代理目标类 就是采用 子类继承的方式创建代理，所以也是Cglib代理，可以通过。
3. hasNoUserSuppliedProxyInterfaces：判断是否是实现了接口，如果没有必须采用Cglib代理。
    所以如果我们想在项目中 采用Cglib代理的话 application.properties中配置：
    spring.aop.proxy-target-class=false，或者使用注解配置proxyTargetClass = true .

 createAopProxy()：

```java
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    //判断采用什么代理类型。
    //Optimize 是否采用了 激进的优化策略，该优化仅支持 Cglib代理
    //ProxyTargetClass 代理目标类，代理目标类 就是采用 子类继承的方式创建代理，所以也是Cglib代理，可以通过
    // 判断是否是实现了接口，如果没有必须采用Cglib代理。
        if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: " +
                        "Either an interface or a target is required for proxy creation.");
            }
           // 如果目标对象是接口 采用JDK代理。
            if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
                return new JdkDynamicAopProxy(config);
            }
           
            return new ObjenesisCglibAopProxy(config);
        }
        else {
            return new JdkDynamicAopProxy(config);
        }
    }
```

 

拿JdkDynamicAopProxy来说。 

获取代理类：

JDK动态代理的关键是 InvocationHandler，JdkDynamicAopProxy实现了InvocationHandler接口。

getProxy：

```java
@Override
    public Object getProxy(ClassLoader classLoader) {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        //获取代理类，其中InvocationHanler 是 this,就是JdkDynamicAopProxy。
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }

```





## 2 增强方法的执行

增强方法的执行 是AOP的核心所在，理解Spring Aop 是如何 协调 各种通知 的执行，是理解的关键。

通知 分为前置 后置 异常 返回 环绕通知，在一个方法中每种通知的执行时机不同，协调他们之间执行顺序很重要。

但是Spring AOP 采用了很聪明的方法让各种各样的通知准确有序的工作。

 由于invoker 方法很大，删除了大部分的代码，保留几处关键代码： 

```java
//method是要增强的方法，proxy是代理类
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    //省略了 对 hashCode equals 等方法的处理....
    //exposeProxy属性的支持。
            if (this.advised.exposeProxy) {
                // Make invocation available if necessary.
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
            }
            // 获取目标对象类
            target = targetSource.getTarget();
            if (target != null) {
                targetClass = target.getClass();
            }
      // 获取拦截器链 这里this.advised 就是AnnotationAwareAspectJAutoProxyCreator
      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
     //如果 没有拦截器链 则直接执行。
      if (chain.isEmpty()) {
         Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
         retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
      }
      else {    
          //开始执行拦截器链
         invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
         retVal = invocation.proceed();
      }
    //返回执行结果。
      return retVal;
  
}

```

 

###  2.1 创建拦截器链

拦截器链InterceptorAndDynamicMethodMatcher 类 和 MethodInterceptor 类型的集合。

InterceptorAndDynamicMethodMatcher  封装了 MethodInterceptor 和 MethodMather ，作用就是在执行时进行再次匹配。

创建拦截器的过程就是 对所有 适合 目标类的 Advisor进行再一次筛选。对匹配的Advisor封装成 MethodInterceptor。通过MethodInterceptor 统一 增强方法的调用。

 

###  2.2 执行拦截器链

使用ReflectiveMethodInvocation 对拦截器链进行封装。通过process 方法触发拦截器开始执行。

```java
public Object proceed() throws Throwable {
        //  判断拦截器链 是否执行结束，如果执行结束执行目标方法。
        if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
            return invokeJoinpoint();
        }
        //获取下一个 需要执行的 拦截器
        Object interceptorOrInterceptionAdvice =
        this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
        //如果需要执行时 进行匹配
        if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
            InterceptorAndDynamicMethodMatcher dm =
                    (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
            //如果匹配，并不清楚 为什么还要在这里 进行再次匹配。
            if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) 
            //注意这里 将ReflectiveMethodInvocation 自己当参数 ，传入调用。
                return dm.interceptor.invoke(this);
            }
            else {
            //递归的调用
                return proceed();
            }
        }
        else {
            //如果 MethodInterceptor 
            return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
        }
    }


```

 

查看代码并没有发现任何的关于协调前置，后置各种通知的代码。其实所有的协调工作都是由MethodInterceptor 自己维护。 

前置MethodInterceptor（MethodBeforeAdviceInterceptor）的 `invoke`：

```java
    @Override
  public Object invoke(MethodInvocation mi) throws Throwable {
        //激活 前置增强方法
      this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis() );
        //继续调用下一个 拦截器。
      return mi.proceed();
  }


```

后置MethodInterceptor（AspectJAfterAdvice） ：

```java
@Override
  public Object invoke(MethodInvocation mi) throws Throwable {
      try {
            //继续调用一下拦截器。
          return mi.proceed();
      }
      finally {
            //在finally 里面激活 后置增强方法
          invokeAdviceMethod(getJoinPointMatch(), null, null);
      }
  }

```



## 3 模拟拦截器

为了更好理解 拦截器调用，自己实现了一个简单的拦截器链调用过程。 

接口：

```java
//方法调用   
interface MethodInvocation{
         Object process() throws Throwable;
    }
//方法拦截器
    interface MethodInterceptor{
        Object invoke(MethodInvocation mi) throws Throwable;
    }

```

实现 ：

```java
//后置增强方法的 拦截器

class AfterMethodInterceptor implements MethodInterceptor{
        String identification;
        public AfterMethodInterceptor(String identification){
            this.identification = identification;
        }
        @Override
        public Object invoke(MethodInvocation mi) throws Throwable {
            try {
                return mi.process();
            }finally {
                System.out.println("执行后置通知"+identification);
            }
        }
    }
//前置 的 方法拦截器 
    class BeforMethodInterceptor implements MethodInterceptor{
        String identification;
        public BeforMethodInterceptor(String identification){
            this.identification = identification;
        }
        @Override
        public Object invoke(MethodInvocation mi) throws Throwable {
            System.out.println("执行前置通知"+identification);
            return mi.process();
        }
    }
// 默认的 方法调用实现
 class DefaultMethodInvacation implements MethodInvocation {
        List<MethodInterceptor> chian;
        Object target; //目标对象
        Method method; //目标方法
        Object[] args; //方法参数
        int currentChianIndex; //记录拦截器链当前执行位置
        public  DefaultMethodInvacation(List<MethodInterceptor> chian,Object target,Method method,Object[] args){
            this.chian = chian;
            this.method = method;
            this.target = target;
            this.args = args;
        }
        @Override
        public Object process() throws Throwable{
            Object value ;
            //如果 拦截器 执行完毕 执行 目标方法
            if(currentChianIndex == chian.size()){
                value = method.invoke(target, args);
                return value;
            }
            //获取下一个 拦截器
            MethodInterceptor methodInterceptor = chian.get(currentChianIndex++);
            return methodInterceptor.invoke(this);
        }
    }
//目标对象
class TargetObj{
    //目标方法
      public void targetMethod(){
           System.out.println("-----目标方法执行----");
       }
    }

```



测试代码：

```java
@Test
    public void aopchain() throws Throwable {
        List<MethodInterceptor> chain = Lists.newArrayList();
        //拦截器链，穿插这 放入 前置 和 后置 拦截器 。
        chain.add(new AfterMethodInterceptor("1"));
        chain.add(new BeforMethodInterceptor("1"));
        chain.add(new AfterMethodInterceptor("2"));
        chain.add(new BeforMethodInterceptor("2"));
        chain.add(new AfterMethodInterceptor("3"));
        chain.add(new BeforMethodInterceptor("3"));
        //目标对象
        TargetObj targetObj = new TargetObj();
        //目标方法
        Method method = TargetObj.class.getMethod("targetMethod");
        DefaultMethodInvacation defaultMethodInvacation = new DefaultMethodInvacation(chain, targetObj, method, null);
        //执行拦截器链
        defaultMethodInvacation.process();
    }

```



执行结果：虽然 前置 和 后置 都是 无序的 存放在 拦截器链中，但是 前置 和 后置 都在各自的位置执行。 

```
执行前置通知1
执行前置通知2
执行前置通知3
-----目标方法执行----
执行后置通知3
执行后置通知2
执行后置通知1
```



## 4 总结

**1.postProcessAfterInitialization(Object bean, String beanName)：**

> 主要作用是在bean实例化的时候调用此方法，判断bean是否需要被包装，如果需要就调用wrapIfNecessary方法。

容器启动，bean实例化之前经过此方法。使用cacheKey判断bean是否被增强过，以防止bean多次被增强。若没有被增强过，`return wrapIfNecessary(bean, beanName, cacheKey)`;进行代理包装。



**2.wrapIfNecessary(Object bean, String beanName, Object cacheKey)：**

> 主要作用就是获得当前bean的增强器，然后将增强器的数组和bean作为参数，调用createProxy封装成一个代理对象。

1. 通过`Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);`方法获取到适合的当前bean的增强方法，然后对增强方法进行排序，最后返回。 

   `getAdvicesAndAdvisorsForBean()`具体实现：里面的`findEligibleAdvisors方法`

   分三步：

   - 获取所有的增强器：`this.aspectJAdvisorsBuilder.buildAspectJAdvisors()`方法，获取容器中已经注册的BeanNames，遍历 所有已经注册的Bean,找出所有 @Aspecj 注解申明的切面类，构建增强器
   - 匹配合适的增强器
   - 对增强器排序
   - 返回增强器

   

   

1. 如果匹配到了增强器，创建增强代理类：

```java
  if (specificInterceptors != DO_NOT_PROXY) {
            this.advisedBeans.put(cacheKey, Boolean.TRUE);
            //创建 增强代理类
            Object proxy = createProxy(bean.getClass(),
                                       beanName, 
                                       specificInterceptors, //传入增强器
                                       new SingletonTargetSource(bean));//单例
            this.proxyTypes.put(cacheKey, proxy.getClass());
            return proxy;
        }
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
```



**3.调用createProxy方法，创建代理**

> 主要作用就是将增强器数组，代理目标对象等信息添加进proxyFactory，然后调用return proxyFactory.getProxy(this.proxyClassLoader)

先创建代理工厂proxyFactory，beanClass和beanName用于判断当前Bean是否需要代理，然后获取当前bean 的增强器advisors，把当前获取到的增强器添加到代理工厂proxyFactory，然后设置当前的代理工厂的代理目标对象为当前bean。

最后`return proxyFactory.getProxy(this.proxyClassLoader);`调用getProxy



**4.proxyFactory.getProxy()**

> 主要作用就是调用其他两个方法，一个是createAopProxy，一个是getProxy

通过此方法调用`return createAopProxy().getProxy(classLoader);`



**5.createAopProxy()**

> 主要用于判断采取JDK代理 或Cglib代理,根据情况创建合适的代理。

如果符合jdk动态代理的条件： `return new JdkDynamicAopProxy(config);`



**6.new JdkDynamicAopProxy(config)**

> JdkDynamicAopProxy 是final类并且实现了InvocationHandler 接口，`invoke`该接口中唯一一个定义的方法。



**7.getProxy**

获取代理类。

 `return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);`

其中InvocationHanler 是 this,就是JdkDynamicAopProxy。



**8.invoke**

> 具体实现AOP的核心方法。增强器包括了before，after，around等等。

JdkDynamicAopProxy类的唯一方法，用于实现增强方法的执行

执行拦截器链：

```java
 invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);

 retVal = invocation.proceed();

return retVal；

```



**9.ReflectiveMethodInvocation**

使用ReflectiveMethodInvocation 对拦截器链进行封装。通过`proceed()`方法触发拦截器开始执行。而且是递归执行。









