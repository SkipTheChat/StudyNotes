[Spring AOP 源码分析系列文章导读](http://www.coolblog.xyz/2018/06/17/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/)

[Spring AOP 源码分析 - 筛选合适的通知器](http://www.tianxiaobo.com/2018/06/20/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E7%AD%9B%E9%80%89%E5%90%88%E9%80%82%E7%9A%84%E9%80%9A%E7%9F%A5%E5%99%A8/)

[Spring AOP 源码分析 - 创建代理对象](http://www.tianxiaobo.com/2018/06/20/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%88%9B%E5%BB%BA%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1/)

[Spring AOP 源码分析 - 拦截器链的执行过程](http://www.coolblog.xyz/2018/06/22/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E6%8B%A6%E6%88%AA%E5%99%A8%E9%93%BE%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B/)

> 先放一个AOP开发部分代码

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





# 🎉 正式源码解析

# 1 入口方法

1.AOP时机：Spring AOP 抽象代理创建器实现了  BeanPostProcessor 接口，并在 bean 初始化后置处理过程中（postProcessAfterInitialization）向 bean 中织入通知。 



下面是 Spring AOP 创建代理对象的入口方法：

1. 若 bean 是 AOP 基础设施类型，则直接返回
2. 为 bean 查找合适的通知器
3. 如果通知器数组不为空，则为 bean 生成代理对象，并返回该对象
4. 若数组为空，则返回原始 bean

```java
  @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    //bean 就是需要被增强的类。
        if (bean != null) {
            //1.从缓存中找到bean的cacheKey
            Object cacheKey = getCacheKey(bean.getClass(), beanName);
            
            //2.如果还没有被增强过才调用wrapIfNecessary，防止bean多次被增强。
            if (!this.earlyProxyReferences.contains(cacheKey)) {
            
                //3.wrapIfNecessary主要作用就是获得当前bean的增强器，然后将
                //增强器的数组和bean作为参    数，调用createProxy封装成一个代理对象。
                return wrapIfNecessary(bean, beanName, cacheKey);
            }
        }
        return bean;
    }

//======================wrapIfNecessary()===============================
//wrapIfNecessary：主要作用就是获得当前bean的增强器，然后将增强器的数组和bean作为参数，调用createProxy封装成一个代理对象。

/**
     *如果有必要 封装 bean
     */
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		//1.如果已经处理过，直接return bean
        if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
            return bean;
        }
		//2.如果不需要增强，就直接返回
        if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
            return bean;
        }
        //如果是一个组件类，组件类包括Advice，Pointcut,Advisor实现类和Aspect注解注释的类，那么就标记为不需要增强的bean。所以切面类自己不会被AOP增强。
        if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return bean;
        }
    
    //正式进入逻辑
    
        //这个方法会获取到适合的当前bean的增强方法，对增强方法进行排序，最后返回。 
  		Object[] specificInterceptors 
      		= getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    
        //如果匹配到增强器 那么就创建增强代理类。
        if (specificInterceptors != DO_NOT_PROXY) {
            this.advisedBeans.put(cacheKey, Boolean.TRUE);  //标记为需要增强
            //创建增强代理类
            Object proxy = createProxy(
                    bean.getClass(), 
                	beanName, 
                	specificInterceptors, 
                	new SingletonTargetSource(bean));
            		this.proxyTypes.put(cacheKey, proxy.getClass());
            return proxy;
        }
    
        this.advisedBeans.put(cacheKey, Boolean.FALSE); //增强完毕，标记为不需要增强
        return bean;
    }

```





# 2 查找通知器

Spring 先查询出所有的通知器，然后再调用 findAdvisorsThatCanApply 对通知器进行筛选。

在下面几节中，我将分别对 findCandidateAdvisors（查找所有的通知器） 和 findAdvisorsThatCanApply（筛选可应用在 beanClass 上的 Advisor） 两个方法进行分析。

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
    // 查找所有的通知器
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    /*
     * 筛选可应用在 beanClass 上的 Advisor，通过 ClassFilter 和 MethodMatcher
     * 对目标类和方法进行匹配
     */
    
    //筛选合适的通知器
    List<Advisor> eligibleAdvisors = 
        findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    
    // 拓展操作
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```



## 2.1 查找所有通知器

> Spring 提供了两种配置 AOP 的方式，一种是通过 XML 进行配置，另一种是注解。对于两种配置方式，Spring 的处理逻辑是不同的。

`findCandidateAdvisors`：AnnotationAwareAspectJAutoProxyCreator 覆写了父类的方法 findCandidateAdvisors，**并增加了一步操作，即解析 @Aspect 注解**，并构建成通知器。 

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





#### 2.1.1 通过xml查找通知器

看`findAdvisorBeans`方法，从容器中查找 Advisor 类型的 bean，主要做了两件事情：

1. 从容器中查找所有类型为 Advisor 的 bean 对应的名称
2. 遍历 advisorNames，并从容器中获取对应的 bean

```java
public abstract class AbstractAdvisorAutoProxyCreator extends AbstractAutoProxyCreator {

    private BeanFactoryAdvisorRetrievalHelper advisorRetrievalHelper;
    
    //这是父类的findCandidateAdvisors方法，本质就在调用findAdvisorBeans
    protected List<Advisor> findCandidateAdvisors() {
        return this.advisorRetrievalHelper.findAdvisorBeans();
    }

    //这个方法做了两件事：
    //1.通过BeanFactoryUtils工具类查找到所有Advisor类型的bean（String[]）
    //2.遍历这个beanName的字符串数组，调用this.beanFactory.getBean(name, Advisor.class)获取Advisor，然后加入到通知器的List集合，最后返回。
    public List<Advisor> findAdvisorBeans() {
    String[] advisorNames = null;
    synchronized (this) {
        // cachedAdvisorBeanNames 是 advisor 名称的缓存
        advisorNames = this.cachedAdvisorBeanNames;
        /*
         * 如果 cachedAdvisorBeanNames 为空，这里到容器中查找，
         * 并设置缓存，后续直接使用缓存即可
         */ 
        if (advisorNames == null) {
            // 从容器中查找 Advisor 类型 bean 的名称
            advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
                    this.beanFactory, Advisor.class, true, false);
            // 设置缓存
            this.cachedAdvisorBeanNames = advisorNames;
        }
    }
    if (advisorNames.length == 0) {
        return new LinkedList<Advisor>();
    }

    List<Advisor> advisors = new LinkedList<Advisor>();
    // 遍历 advisorNames
    for (String name : advisorNames) {
        if (isEligibleBean(name)) {
            // 忽略正在创建中的 advisor bean
            if (this.beanFactory.isCurrentlyInCreation(name)) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Skipping currently created advisor '" + name + "'");
                }
            }
            else {
                try {
                    /*
                     * 调用 getBean 方法从容器中获取名称为 name 的 bean，
                     * 并将 bean 添加到 advisors 中
                     */ 
                    advisors.add(this.beanFactory.getBean(name, Advisor.class));
                }
                catch (BeanCreationException ex) {
                    Throwable rootCause = ex.getMostSpecificCause();
                    if (rootCause instanceof BeanCurrentlyInCreationException) {
                        BeanCreationException bce = (BeanCreationException) rootCause;
                        if (this.beanFactory.isCurrentlyInCreation(bce.getBeanName())) {
                            if (logger.isDebugEnabled()) {
                                logger.debug("Skipping advisor '" + name +
                                        "' with dependency on currently created bean: " + ex.getMessage());
                            }
                            continue;
                        }
                    }
                    throw ex;
                }
            }
        }
    }

    return advisors;
}
}
```





##### 2.1.2 通过注解查找通知器

`buildAspectJAdvisors` ：

1. 获取容器中所有 bean 的名称（beanName）
2. 遍历上一步获取到的 bean 名称数组，并获取当前 beanName 对应的 bean 类型（beanType）
3. 根据 beanType 判断当前 bean 是否是一个的 Aspect 注解类，若不是则不做任何处理
4. 否则的话则调用 advisorFactory.getAdvisors 获取通知器

```java
//注解方法要麻烦一些，除了获取beanName，还需要获取beanName的beanType
//根据beanType判断当前bean是否是一个的 Aspect注解类，
//若不是则不做处理，否则调用advisorFactory.getAdvisors获取通知器
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
```





`getAdvisors`：

1. for循环为每个方法调用getAdvisor 方法获取通知器（切点），然后存进LinkedList类型的advisors 集合。
2. 完了以后调用getPointcut方法获取切点实现类（`@Pointcut("execution(* cn.wolfcode.service.*Service.*(..))")`）
3. 最后按照注解类型生成相应的 Advice 实现类。用一个switch来判断注解类型，比如case AtBefore， case AtAfter，case AtAfterReturning，case AtAfterThrowing...

```java
//两件事，一个是获取 AspectJ 表达式切点，另一个是创建 Advisor 实现类
public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
    // 获取 aspectClass 和 aspectName
    Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
    validate(aspectClass);

    MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
            new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

    List<Advisor> advisors = new LinkedList<Advisor>();

    // getAdvisorMethods 用于返回不包含 @Pointcut 注解的方法
    for (Method method : getAdvisorMethods(aspectClass)) {
        // 为每个方法分别调用 getAdvisor 方法
        Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, advisors.size(), aspectName);
        if (advisor != null) {
            advisors.add(advisor);
        }
    }

    // If it's a per target aspect, emit the dummy instantiating aspect.
    if (!advisors.isEmpty() && lazySingletonAspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
        Advisor instantiationAdvisor = new SyntheticInstantiationAdvisor(lazySingletonAspectInstanceFactory);
        advisors.add(0, instantiationAdvisor);
    }

    // Find introduction fields.
    for (Field field : aspectClass.getDeclaredFields()) {
        Advisor advisor = getDeclareParentsAdvisor(field);
        if (advisor != null) {
            advisors.add(advisor);
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



给一个生成的 Advice 实现类，这里以before方法为例：

这个方法调用了父类中的 invokeAdviceMethod，然后 invokeAdviceMethod 在调用  invokeAdviceMethodWithGivenArgs，最后在 invokeAdviceMethodWithGivenArgs  通过反射执行通知方法 

```java
public class AspectJMethodBeforeAdvice extends AbstractAspectJAdvice implements MethodBeforeAdvice {

    public AspectJMethodBeforeAdvice(
            Method aspectJBeforeAdviceMethod, AspectJExpressionPointcut pointcut, AspectInstanceFactory aif) {

        super(aspectJBeforeAdviceMethod, pointcut, aif);
    }


    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        // 调用通知方法
        invokeAdviceMethod(getJoinPointMatch(), null, null);
    }

    @Override
    public boolean isBeforeAdvice() {
        return true;
    }

    @Override
    public boolean isAfterAdvice() {
        return false;
    }

}

protected Object invokeAdviceMethod(JoinPointMatch jpMatch, Object returnValue, Throwable ex) throws Throwable {
    // 调用通知方法，并向其传递参数
    return invokeAdviceMethodWithGivenArgs(argBinding(getJoinPoint(), jpMatch, returnValue, ex));
}

protected Object invokeAdviceMethodWithGivenArgs(Object[] args) throws Throwable {
    Object[] actualArgs = args;
    if (this.aspectJAdviceMethod.getParameterTypes().length == 0) {
        actualArgs = null;
    }
    try {
        ReflectionUtils.makeAccessible(this.aspectJAdviceMethod);
        // 通过反射调用通知方法
        return this.aspectJAdviceMethod.invoke(this.aspectInstanceFactory.getAspectInstance(), actualArgs);
    }
    catch (IllegalArgumentException ex) {
        throw new AopInvocationException("Mismatch on arguments to advice method [" +
                this.aspectJAdviceMethod + "]; pointcut expression [" +
                this.pointcut.getPointcutExpression() + "]", ex);
    }
    catch (InvocationTargetException ex) {
        throw ex.getTargetException();
    }
}
```



## 2.2 筛选合适的通知器

以下是通知器筛选的过程，筛选的工作主要由 ClassFilter 和 MethodMatcher 完成。关于 ClassFilter 和 MethodMatcher 我在[导读](http://www.coolblog.xyz/2018/06/17/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/)一文中已经说过了，这里再说一遍吧。在 AOP 中，切点 Pointcut 是用来匹配连接点的，以 AspectJExpressionPointcut  类型的切点为例。该类型切点实现了ClassFilter 和 MethodMatcher 接口，匹配的工作则是由 AspectJ  表达式解析器复杂。除了使用 AspectJ 表达式进行匹配，Spring  还提供了基于正则表达式的切点类，以及更简单的根据方法名进行匹配的切点类。大家有兴趣的话，可以自己去了解一下，这里就不多说了。 

在完成通知器的查找和筛选过程后，还需要进行最后一步处理 – 对通知器列表进行拓展。怎么拓展呢？我们一起到下一节中一探究竟吧。 

```java
//从通知器中获取类型过滤器 ClassFilter，并调用 matchers 方法进行匹配。
//如果匹配成功就会通过反射getAllDeclaredMethods拿到类的方法数组，
//遍历调用方法判断方法是否匹配
protected List<Advisor> findAdvisorsThatCanApply(
        List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

    ProxyCreationContext.setCurrentProxiedBeanName(beanName);
    try {
        // 调用重载方法
        return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
    }
    finally {
        ProxyCreationContext.setCurrentProxiedBeanName(null);
    }
}

public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
    if (candidateAdvisors.isEmpty()) {
        return candidateAdvisors;
    }
    List<Advisor> eligibleAdvisors = new LinkedList<Advisor>();
    for (Advisor candidate : candidateAdvisors) {
        // 筛选 IntroductionAdvisor 类型的通知器
        if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
            eligibleAdvisors.add(candidate);
        }
    }
    boolean hasIntroductions = !eligibleAdvisors.isEmpty();
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor) {
            continue;
        }

        // 筛选普通类型的通知器
        if (canApply(candidate, clazz, hasIntroductions)) {
            eligibleAdvisors.add(candidate);
        }
    }
    return eligibleAdvisors;
}

public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
    if (advisor instanceof IntroductionAdvisor) {
        /*
         * 从通知器中获取类型过滤器 ClassFilter，并调用 matchers 方法进行匹配。
         * ClassFilter 接口的实现类 AspectJExpressionPointcut 为例，该类的
         * 匹配工作由 AspectJ 表达式解析器负责，具体匹配细节这个就没法分析了，我
         * AspectJ 表达式的工作流程不是很熟
         */
        return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
    }
    else if (advisor instanceof PointcutAdvisor) {
        PointcutAdvisor pca = (PointcutAdvisor) advisor;
        // 对于普通类型的通知器，这里继续调用重载方法进行筛选
        return canApply(pca.getPointcut(), targetClass, hasIntroductions);
    }
    else {
        return true;
    }
}

public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
    Assert.notNull(pc, "Pointcut must not be null");
    // 使用 ClassFilter 匹配 class
    if (!pc.getClassFilter().matches(targetClass)) {
        return false;
    }

    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) {
        return true;
    }

    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
    }

    /*
     * 查找当前类及其父类（以及父类的父类等等）所实现的接口，由于接口中的方法是 public，
     * 所以当前类可以继承其父类，和父类的父类中所有的接口方法
     */ 
    Set<Class<?>> classes = new LinkedHashSet<Class<?>>(ClassUtils.getAllInterfacesForClassAsSet(targetClass));
    classes.add(targetClass);
    for (Class<?> clazz : classes) {
        // 获取当前类的方法列表，包括从父类中继承的方法
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            // 使用 methodMatcher 匹配方法，匹配成功即可立即返回
            if ((introductionAwareMethodMatcher != null &&
                    introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions)) ||
                    methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }

    return false;
}
```



# 3 创建代理对象

为目标 bean 创建代理对象前，需要先创建 AopProxy 对象，然后再调用该对象的 getProxy 方法创建实际的代理类。我们先来看看 AopProxy 这个接口的定义，如下： 

```java
public interface AopProxy {

    /** 创建代理对象 */
    Object getProxy();
    
    Object getProxy(ClassLoader classLoader);
}
```

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/Spring%E7%B3%BB%E5%88%97/spring/assets/3.4.jpg)



Spring 在为目标 bean 创建代理的过程中，要根据 bean 是否实现接口，以及一些其他配置来决定使用 AopProxy 何种实现类为目标 bean 创建代理对象。 



### 3.1 createProxy

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
			Class<?> beanClass, 
        	String beanName, 
        	Object[] specificInterceptors, 
        	TargetSource targetSource) {
 
        //创建代理工厂
		ProxyFactory proxyFactory = new ProxyFactory();
		// Copy our properties (proxyTargetClass etc) inherited from ProxyConfig.
        //this应该是当前正在实例化的Bean对象
		proxyFactory.copyFrom(this);
 
          /*
     * 默认配置下，或用户显式配置 proxy-target-class = "false" 时，（false表示jdk）
     * proxyFactory.isProxyTargetClass()为false表示配置为jdk。
     */
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            /*
             * 检测 beanClass 是否实现了接口，若未实现，则将 
             * proxyFactory 的成员变量 proxyTargetClass 设为 true
             */ 
            evaluateProxyInterfaces(beanClass, proxyFactory);
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



`调用getProxy(ClassLoader classLoader);实现动态代理：`

判断采取JDK代理 或Cglib代理。
 判断条件：

1. Optimize： 是否采用了 激进的优化策略，该优化仅支持 Cglib代理
2. ProxyTargetClass： 代理目标类，代理目标类 就是采用 子类继承的方式创建代理，所以也是Cglib代理，可以通过。
3. hasNoUserSuppliedProxyInterfaces：判断是否是实现了接口，如果没有必须采用Cglib代理。
   所以如果我们想在项目中 采用Cglib代理的话 application.properties中配置：
   spring.aop.proxy-target-class=false，或者使用注解配置proxyTargetClass = true .

```java
public Object getProxy(ClassLoader classLoader) {
    
    	//先获取 AopProxy,AopProxy有JDK实现和 Cglib实现两种。
        return createAopProxy().getProxy(classLoader);
    
}

//主要通过判断是否采用了激进的优化策略，是否实现接口等判断采用jdk还是cdlib
//如果满足jdk条件就 return new JdkDynamicAopProxy(config);
//否则return new ObjenesisCglibAopProxy(config);
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    //判断采用什么代理类型。
    //1.Optimize 是否采用了激进的优化策略，该优化仅支持Cglib代理
    //2.ProxyTargetClass代理目标类，就是采用子类继承的方式创建代理，所以也是Cglib代理
    //3.没有实现接口
    //都不满足就用jdk代理
        if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: " +
                        "Either an interface or a target is required for proxy creation.");
            }
           // 如果目标对象是接口采用JDK代理
            if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
                return new JdkDynamicAopProxy(config);
            }
           
            return new ObjenesisCglibAopProxy(config);
        }
        else {
            return new JdkDynamicAopProxy(config); // 都不满足用jdk代理
        }
    }

```





# 3 拦截器链的执行过程

现在我们的得到了 bean 的代理对象，且通知也以合适的方式插在了目标方法的前后。接下来要做的事情，就是执行通知逻辑了 .当目标方法被多个通知匹配到时，Spring 通过引入拦截器链来保证每个通知的正常执行。 

> 插一个基础知识-expose-proxy 

Spring 引入 expose-proxy 特性是为了解决**目标方法调用同对象中其他方法时，其他方法的切面逻辑无法执行的问题**。 



### 3.1 以JDK 动态代理为例

增强方法的执行 是AOP的核心所在，理解Spring Aop 是如何 协调 各种通知 的执行，是理解的关键。

通知 分为前置 后置 异常 返回 环绕通知，在一个方法中每种通知的执行时机不同，协调他们之间执行顺序很重要。

流程：

1. 检测 expose-proxy 是否为 true，若为 true，则暴露代理对象（expose-proxy是为了解决**目标方法调用同对象中其他方法时，其他方法的切面逻辑无法执行的问题**。 ）
2. 获取适合当前方法的拦截器
3. 如果拦截器链为空，则直接通过反射执行目标方法
4. 若拦截器链不为空，则创建方法调用 ReflectiveMethodInvocation 对象
5. 调用 ReflectiveMethodInvocation 对象的 proceed() 方法启动拦截器链
6. 处理返回值，并返回该值

在以上6步中，我们重点关注第2步和第5步中的逻辑。第2步用于获取拦截器链，第5步则是启动拦截器链。下面先来分析获取拦截器链的过程。

> invoke: JdkDynamicAopProxy类的唯一方法，用于实现增强方法的执行

```java
//method是要增强的方法，proxy是代理类
//JdkDynamicAopProxy的唯一方法invoke
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    //省略了 对 hashCode equals 等方法的处理....
    
    //1.检测 expose-proxy 是否为 true，若为 true，则暴露代理对象
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
      // 2.获取适合当前方法的拦截器链
      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
     //3.如果拦截器链为空，则直接通过反射执行目标方法
      if (chain.isEmpty()) {
         Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
         retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
      }
      else {    
          //4.若拦截器链不为空，则创建方法调用 ReflectiveMethodInvocation 对象
         invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
          //5.调用 ReflectiveMethodInvocation 对象的 proceed() 方法启动拦截器链
         retVal = invocation.proceed();
      }
    //6.处理返回值，并返回该值
      return retVal;
  
}

```

 

```java
//method是要增强的方法，proxy是代理类
//JdkDynamicAopProxy的唯一方法invoke
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   
            // 获取目标对象类targetClass
            target = targetSource.getTarget();
            if (target != null) {
                targetClass = target.getClass();
            }
      // 2.获取适合当前方法的拦截器链
      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
     //3.如果拦截器链为空，则直接通过反射执行目标方法
      if (chain.isEmpty()) {
         Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
         retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
      }
      else {    
          //4.若拦截器链不为空，则创建方法调用 ReflectiveMethodInvocation 对象
         invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
          //5.调用 ReflectiveMethodInvocation 对象的 proceed() 方法启动拦截器链
         retVal = invocation.proceed();
      }
    //6.处理返回值，并返回该值
      return retVal;
}
```



> 这里看一下上面代码的第五步触发拦截器

proceed 根据 currentInterceptorIndex 来确定当前应执行哪个拦截器

查看代码并没有发现任何的关于协调前置，后置各种通知的代码。其实所有的协调工作都是由MethodInterceptor 自己维护。 而proceed只是起到了一个递归调用的功能，不断地递归调用invoke，传入传输，invoke由传入的参数决定是触发前置还是后置方法等。

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
              //调用拦截器逻辑，并传递 ReflectiveMethodInvocation 对象
            return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
        }
    }


```



> 前置后置环绕各种代码的实现-invoke

这里以前置MethodInterceptor（MethodBeforeAdviceInterceptor）的 为例：

```java
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, Serializable {
    
    /** 前置通知 */
    private MethodBeforeAdvice advice;

    public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
        Assert.notNull(advice, "Advice must not be null");
        this.advice = advice;
    }

    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        // 执行前置通知逻辑
        this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
        // 通过 MethodInvocation 调用下一个拦截器，若所有拦截器均执行完，则调用目标方法
        return mi.proceed();
    }
}
```

后置MethodInterceptor（AspectJAfterAdvice） ：又去调用proceed递归方法，由finally调用最后的后置方法

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



> 来看一下调用通知方法的逻辑，proceed很明显是递归调用，等到currentInterceptorIndex == 拦截器

![](D:/Jessica(note)/Marie(2019)/programming/08%E6%80%BB%E7%AC%94%E8%AE%B0/Spring%E7%B3%BB%E5%88%97/spring/assets/3.5.jpg)







