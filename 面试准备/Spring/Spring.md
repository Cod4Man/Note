# Spring

## 1. Spring AOP

### 1.1.  pom

```xnk
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 1.2. AOP

```java
@Component
@Aspect
public class AOPComponent {

    @Pointcut(value = "execution(public int com.codeman.springaop..*.test*(..))")
    public void pointCut() {

    }

    @Before(value = "pointCut()")
    public void before(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        Object[] args = jp.getArgs();
        System.out.println("method before, methodName=" + methodName + ". args=" + args);
    }

    @AfterThrowing(value = "pointCut()", throwing = "e")
    public void afterThrowing(JoinPoint jp, MyException e) {
        System.out.println("method afterThrowing.. e" + e);
    }

    @AfterReturning(value = "pointCut()", returning = "result")
    public void afterReturning(JoinPoint jp, Object result) {
        System.out.println("method afterReturning .. result=" + result);
    }

    @After(value = "pointCut()")
    public void after(JoinPoint jp) {
        System.out.println("method after");
    }


    @Around(value = "pointCut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("around method...before");
        Object[] args = proceedingJoinPoint.getArgs();
        String methodName = proceedingJoinPoint.getSignature().getName();
        Object proceed = proceedingJoinPoint.proceed();
        System.out.println("around method...after");
        return proceed;
    }


}
```

 ### 1.3. Spring4和Spring5的AOP执行顺序 

- Spring4是先after后afterThrowing
- Spring5和Spring4相反

![1617634454281](E:\SoftwareNote\面试准备\Spring\img\Spring4和Spring5的AOP执行顺序.png)

### 1.4 SpringTest

```java
@SpringBootTest
// Spring4 Test要加这个，Spring5不需要
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringaopApplicationTests {

    public SpringaopApplicationTests() {
    }

    @Resource(name = "demo")
    private SpirngAOPDemo spirngAOPDemo;

    // Test包也不是spring的，而是Junit的
    // spring5用的是Spring的Test
    @Test
    public void contextLoads() throws Exception {
//        int i = 1/0;
        spirngAOPDemo.test01("param01");
    }

}
```



## 2. Spring循环依赖

- 产生原因： 相互引用/注入，形成闭环

- 解决方案：Spring官网建议用set注入，并且Bean是**Singleton(单例的Bean会存在三级缓存中)**

- 三级缓存：DefaultSingletonBeanRegistry

  - 第一级：singletonObjects存放已经经历了完整生命周期的Bean对象

    ```
    Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256)
    ```

  - 第二级：earlySingletonObjects，存放早期暴露出来的bean对象，Bean的生命周期未结束(属性还未填充完成)

    ```
    Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
    ```

  - 第三级：singletonFactories，存放可以生成bean的工厂

    ```
    Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
    ```

- 执行过程

  - A创建前，先去一级缓存(**getSingleton()->singletonObjects**)中找A是否已创建，否则再创建(**doCreateBean**)过程中需要B(**填充属性populateBean()->applyPropertyValues()->valueResolver.resolveValueIfNecessary()->beanFactory.getBean()**)，于是先将A放在三级缓存singletonFactories里面(**addSingletonFactory()**)，然后去实例化B valueResolver.resolveValueIfNecessary
  - B实例化的时候发现需要A，然后就是上面一样的逻辑。然后A和B都在三级缓存了
  - 然后A又需要走CreateBean逻辑，一开始直接getSingleton()去查找，发现A已经在一级缓存，直接删除A然后put进二级缓存(earlySingletonObjects)，因为马上找到，所以不走第一步的逻辑，而是直接return回去，BeanB.setBeanA(return Obj)(**bw.setPropertyValues(new MutablePropertyValues(deepCopy))**。
  - B创建后，将B直接从三级缓存删除，添加到一级缓存(**addSingleton()->singletonObjects**)
  - 同理A也setB，然后从二级缓存删除，添加到一级缓存(**addSingleton()->singletonObjects**)
  - 然后配置文件读到B的创建，getSingleton发现B已经在一级缓存，直接Return

![1618498970531](E:\SoftwareNote\面试准备\Spring\img\Spring循环依赖三级缓存和四大方法.png)

- 源码
  - getBean / createBean

```java

protected <T> T doGetBean(
			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
			throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    // Eagerly check singleton cache for manually registered singletons.
    // 创建Bean前，都先看看一二三级缓存中是否有该Bean
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        // ...
        // 已有Bean，则直接Return。
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
	// 首次创建，肯定没有Bean，因此走else逻辑
    else {
        // ...

            // Create bean instance.
            // 判断是否是单例
            if (mbd.isSingleton()) {
                // 调用重载方法getSingleton，有则返回，无则用重写的getObject方法，即createBean(beanName, mbd, args)，底层为AbstractAutowireCapableBeanFactory.createBean()
                sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                    @Override
                    public Object getObject() throws BeansException {
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        catch (BeansException ex) {
                            // ...
                        }
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }

            else if (mbd.isPrototype()) {
                // ...
            }
            else {
                // ...
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    // Check if required type matches the type of the actual bean instance.
    // ...
	// 可能是populateBean，也可能是doCreateBean
    return (T) bean;
}


/*
* 首次创建Bean走的逻辑(即一二三缓存都没有bean)
*/
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
			throws BeanCreationException {

    // Instantiate the bean.
    // ...

    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isDebugEnabled()) {
            logger.debug("Eagerly caching bean '" + beanName +
                         "' to allow for resolving potential circular references");
        }
        // 添加到三级缓存
        addSingletonFactory(beanName, new ObjectFactory<Object>() {
            @Override
            public Object getObject() throws BeansException {
                return getEarlyBeanReference(beanName, mbd, bean);
            }
        });
    }

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        // 填充该Bean的属性property
        populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
    }
    catch (Throwable ex) {
        // ...
    }

   // ...

    return exposedObject;
}

```

 	- getSingleton / addSingleton

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "'beanName' must not be null");
    synchronized (this.singletonObjects) {
        // 先从一级缓存中获取，
        Object singletonObject = this.singletonObjects.get(beanName);
        // 获取到了，直接return。否则走方法入参singletonFactory，即外层lambda表达式的实现。底层为AbstractAutowireCapableBeanFactory.createBean()
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName,
                                                          "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                                                          "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
            }
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<Exception>();
            }
            try {
                // 不再一级缓存，走方法入参singletonFactory，即外层lambda表达式的实现。底层为AbstractAutowireCapableBeanFactory.createBean()
                singletonObject = singletonFactory.getObject();
                // 获取到Bean直接，将newSingleton标记置为true，下面有用
                newSingleton = true;
            }
            // ...
            if (newSingleton) {
                // 直接放入一级缓存, 这里走完才算实例化完成。属性有了，一级缓存也有了。
                addSingleton(beanName, singletonObject);
            }
        }
        return (singletonObject != NULL_OBJECT ? singletonObject : null);
    }
}

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}

protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}


protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 先看一级缓存
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 再看二级缓存
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 再看三级缓存
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    // 三级缓存有，说明已经在创建中了，只是property未加载
                    // 因此，直接将该Bean加入到二级缓存，并删除三级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

​	- populateBean

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
    // ...
    applyPropertyValues(beanName, mbd, bw, pvs);
    // ...
}

protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    // ...

    // Create a deep copy, resolving any references for values.
    List<PropertyValue> deepCopy = new ArrayList<PropertyValue>(original.size());
    boolean resolveNecessary = false;
    // 遍历该Bean需要注入的Bean属性
    for (PropertyValue pv : original) {
        if (pv.isConverted()) {
            deepCopy.add(pv);
        }
        else {
            String propertyName = pv.getName();
            Object originalValue = pv.getValue();
            // valueResolver.resolveValueIfNecessary底层还是走doCreateBean()
            Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
            Object convertedValue = resolvedValue;
            // ...
        }
    }
    // ...

    // Set our (possibly massaged) deep copy.
    try {
        // 属性创建或者获取成功，则set给该Bean
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    }
    catch (BeansException ex) {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Error setting property values", ex);
    }
}
```

