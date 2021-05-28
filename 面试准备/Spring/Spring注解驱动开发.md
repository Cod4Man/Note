# Spring注解驱动开发（https://www.bilibili.com/video/BV1gW411W7wy）

## 1. 获取Bean （@Configuration + @Bean）

### 1.1 通过xml

- xml

  ```xml
  <bean id="person" class="com.codeman.springannotationresource.entity.Person">
      <property name="age" value="15"/>
      <property name="name" value="xiaozhang"/>
  </bean>
  ```

- ClassPathXmlApplicationContext加载xml获取Bean

  ```java
  ApplicationContext applicationContext
                  = new ClassPathXmlApplicationContext("/core/spring-bean.xml");
  
  // 查询类型为person.class的BeanNames
  String[] beanNamesForType = applicationContext.getBeanNamesForType(Person.class);
  // 查询类型为person.class的Beans
  Map<String, Person> beansOfType = applicationContext.getBeansOfType(Person.class);
  
  Person person = applicationContext.getBean(Person.class);
  System.out.println(person);
  ```

### 1.2 通过注解 @Bean

- @Configuration + @Bean(可以导入没有类注解的类，如第三方包)

  ```java
  // 生命该类为配置文件类
  @Configuration
  public class BeanConfig {
      // 注册Bean，id为xiaozhang
      @Bean(name = "xiaozhang")
      public Person personXiaozhang() {
          return new Person(25, "xiaozhang");
      }
      
      // 该方法调用Bean创建方法，并不会实际调用，再去new一个对象，而是直接去找Bean。
      // 这是因为Spring对@Configuration类有特殊处理。
      public void test2() {
          personXiaozhang();
      }
  }
  ```

- AnnotationConfigApplicationContext加载Configuration对象

  ```java
  AnnotationConfigApplicationContext annotationConfigApplicationContext
                  = new AnnotationConfigApplicationContext(BeanConfig.class);
  Object xiaozhang = annotationConfigApplicationContext.getBean("xiaozhang");
  System.out.println(xiaozhang);
  ```

- @Configuration和@Component

  Component注解也会当做配置类，但是并不会为其生成CGLIB代理Class，所以在生成Driver对象时和生成Car对象时调用car()方法执行了两次new操作，所以是不同的对象。当时Configuration注解时，生成当前对象的子类Class，并对方法拦截，第二次调用car()方法时直接从BeanFactory之中获取对象，所以得到的是同一个对象。

  

### 1.3 通过注解 @Import 三种方式

```java
@Configuration
@Import(value = 
        {Color.class, // 直接导入指定类
         CustomImportSelector.class,  // Import选择器接口实现
         CustomImportBeanDefinitionRegistrar.class // ImportBean注册接口实现
            })
public class BeanConfig {
    
}
```

- Import -> ImportSelect

```java
public void refresh() throws BeansException, IllegalStateException {
    // 注册相关的结构registered
    // Invoke factory processors registered as beans in the context.
    invokeBeanFactoryPostProcessors(beanFactory);
    // ...
}
// 底层的
processDeferredImportSelectors();
```



- Import ->  ImportBeanDefinitionRegistrar原理

```java
public void refresh() throws BeansException, IllegalStateException {
    // 注册相关的结构registered
    // Invoke factory processors registered as beans in the context.
    invokeBeanFactoryPostProcessors(beanFactory);
    // ...
}

protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, 			getBeanFactoryPostProcessors());

    // ...
}

public static void invokeBeanFactoryPostProcessors(
    ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    // ...
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
    // ...
}

private static void invokeBeanDefinitionRegistryPostProcessors(
			Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup) {

    for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {
        StartupStep postProcessBeanDefRegistry = applicationStartup.start("spring.context.beandef-registry.post-process")
            .tag("postProcessor", postProcessor::toString);
        postProcessor.postProcessBeanDefinitionRegistry(registry);
        postProcessBeanDefRegistry.end();
    }
}

public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    int registryId = System.identityHashCode(registry);
    if (this.registriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
            "postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
    }
    if (this.factoriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
            "postProcessBeanFactory already called on this post-processor against " + registry);
    }
    this.registriesPostProcessed.add(registryId);

    processConfigBeanDefinitions(registry);
}

public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    // ...
    // ConfigurationClassBeanDefinitionReader
    this.reader.loadBeanDefinitions(configClasses);
    // ..
}

public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();
    for (ConfigurationClass configClass : configurationModel) {
        loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }
}

private void loadBeanDefinitionsForConfigurationClass(
    ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {
    // ...
    // 取到所有ImportBeanDefinitionRegistrar实现类configClass.getImportBeanDefinitionRegistrars()
    loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}

private void loadBeanDefinitionsFromRegistrars(Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> registrars) {
    // registrar->AspectJAutoProxyRegistrar
    // metadata -> StandAnnotationMetadata
    // 遍历调用ImportBeanDefinitionRegistrar实现类.registerBeanDefinitions即AspectJAutoProxyRegistrar.registerBeanDefinitions
    registrars.forEach((registrar, metadata) ->
                       registrar.registerBeanDefinitions(metadata, this.registry, 
                                                         this.importBeanNameGenerator));
}
```



### 1.4 FactoryBean

- 通过FactoryBean的id获取Bean，默认得到的是FactoryBean调用getObject()方法返回的对象
- 想要获取FactoryBean工厂本身，只需在FactoryBean的id前面加个“&”即可

```java
public class BlackFactoryBean implements FactoryBean<Black> {

    @Override
    public Black getObject() throws Exception {
        return new Black();
    }

    @Override
    public Class<?> getObjectType() {
        return Black.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

@Bean
public BlackFactoryBean blackFactoryBean() {
    return new BlackFactoryBean();
}

@Test 
void getBeanFormConfiguration() {
    Object blackFactoryBean = annotationConfigApplicationContext.getBean("blackFactoryBean");
    Object blackFactoryBean2 = annotationConfigApplicationContext.getBean("&blackFactoryBean");
    // class com.codeman.springannotationresource.entity.Black
    System.out.println("bean的类型为：" + blackFactoryBean.getClass());
    // class com.codeman.springannotationresource.config.BlackFactoryBean
    System.out.println("bean2的类型为：" + blackFactoryBean2.getClass()); 
}

```



## 2. Bean扫描 @ComponentScan / @ComponentScans

### 2.1. 通过xml

```xml
<!-- 扫描包底下的所有Bean -->
<context:component-scan  base-package="com.codeman.springannotationresource"/>
```

### 2.2 注解

```java
@ComponentScan(
        value = "com.codeman",
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Service.class})},
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
                @ComponentScan.Filter(type = FilterType.CUSTOM, classes = CustomTypeFilter.class)},
        useDefaultFilters = false) // useDefaultFilters只扫描，需要禁掉默认规则
@ComponentScans(value = @ComponentScan({"com.codeman"}))
@Configuration
public class BeanConfig {
    
}

public enum FilterType {
	/**
	 * 注解
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 */
	ANNOTATION,

	/**
	 * 类型
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 */
	ASSIGNABLE_TYPE,

	/**
	 * AspectJ表达式，不常用
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 */
	ASPECTJ,

	/**
	 * 正则表达式
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 */
	REGEX,

	/** 
	 * 自定义
	 * Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM
}

public class CustomTypeFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        String className = annotationMetadata.getClassName();
        if (className != null && className.startsWith("com.codeman")) {
            System.out.println("*****************************");
            System.out.println(className);
            System.out.println("*****************************");
            // return true 说明匹配上了
            return true;
        }
        return false;
    }
}
```

## 3. Bean的作用域 @Scope  @Lazy

### 3.1 xml

```xml
<bean id="person" class="com.codeman.springannotationresource.entity.Person" scope="prototype" lazy-init="true">
    <property name="age" value="15"/>
    <property name="name" value="xiaozhang"/>
</bean>
```

### 3.2 注解

```java
@Bean(name = "xiaozhang")
@Scope(value = "prototype")
@Lazy  // 懒加载，用时才创建
public Person personXiaozhang() {
    return new Person(25, "xiaozhang");
}
```

### 3.3 类别

-  singleton (默认)： 单例（一开始就添加到Bean容器内）
- prototype ： 原型，**自带懒加载（用时才创建Bean）** 
- request ： 请求
- session ： 会话

## 4. 条件过滤Bean

```java
@Bean
@Conditional(value = WidowsCondition.class)
public Person bill() {
    return new Person(60, "bill");
}

public class WidowsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if (property != null && property.contains("Window")) {
            System.out.println("windows==" + property);
            return true;
        }
        return false;
    }
}
```

## 5. Bean的生命周期

- 值得注意的是，懒加载是不会触发destroy的，如@Lazy或者Scope=prototype
- 并且init是在实例化后才执行

### 5.1 xml

```xml
<bean id="beanLife2" class="com.codeman.springannotationresource.entity.BeanLife"
        init-method="init" destroy-method="destroy" />
```

### 5.2 注解

```java
@Bean(initMethod = "init", destroyMethod = "destroy")
@Lazy
public BeanLife beanLife() {
    return new BeanLife();
}
```

### 5.3 实现接口

```java
public class BeanLife2 implements InitializingBean, DisposableBean {

    public BeanLife2() {
        System.out.println("BeanLife2 创建");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("BeanLife2 销毁");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("BeanLife2 初始化");
    }
}
```

### 5.4 JSR250规范的注解

```java
public class BeanLife3 {
    public BeanLife3() {
        System.out.println("BeanLife3 创建");
    }

    @PostConstruct
    public void init() {
        System.out.println("BeanLife3 init");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("BeanLife3 destroy");
    }
}
```

### 5.5 全局的: 所有Bean的初始化前后

```java
public class CustomBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization " + beanName);
        return bean;
    }
}

```

- 源码

  ```
  // 1.new ClassPathXmlApplicationContext("/core/spring-bean.xml")
  public ClassPathXmlApplicationContext(
      String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
      throws BeansException {
  
      super(parent);
      setConfigLocations(configLocations);
      if (refresh) {
      refresh();
      }
  }
  // 2. refresh()
  public void refresh() throws BeansException, IllegalStateException {
  	// ...
      // Instantiate all remaining (non-lazy-init) singletons.
      finishBeanFactoryInitialization(beanFactory);
      // ...
  }
  
  //3. finishBeanFactoryInitialization(beanFactory)
  protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
      // ...
      // Instantiate all remaining (non-lazy-init) singletons.
      beanFactory.preInstantiateSingletons();
  }
  
  // 4. beanFactory.preInstantiateSingletons()
  public void preInstantiateSingletons() throws BeansException {
  	// ...
      // Trigger initialization of all non-lazy singleton beans...
      for (String beanName : beanNames) {
      	getBean(beanName);
      }
      // ...
  }
  
  // 5. doGetBean
  public Object getBean(String name) throws BeansException {
  	return doGetBean(name, null, null, false);
  }
  
  protected <T> T doGetBean(
  String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
  throws BeansException {
  	// ...
  	sharedInstance = getSingleton(beanName, () -> {
          try {
          return createBean(beanName, mbd, args);
          }
          catch (BeansException ex) {
          // Explicitly remove instance from singleton cache: It might have been put there
          // eagerly by the creation process, to allow for circular reference resolution.
          // Also remove any beans that received a temporary reference to the bean.
          destroySingleton(beanName);
          throw ex;
          }
      });
      // ...
  }
  
  // 6. createBean
  public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
  	// ...
  	singletonObject = singletonFactory.getObject()
  	// ...
  }
  
  protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
  	// ...
  	try {
          Object beanInstance = doCreateBean(beanName, mbdToUse, args);
          if (logger.isTraceEnabled()) {
          logger.trace("Finished creating instance of bean '" + beanName + "'");
          }
          return beanInstance;
      }
      // ...
  }
  
  // 7.doCreateBean
  protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
  	// ...
  	try { 
  		// populateBean给Bean装填属性
          populateBean(beanName, mbd, instanceWrapper);
          // 初始化逻辑
          exposedObject = initializeBean(beanName, exposedObject, mbd);
      }
  	// ...
  }
  
  // 8. 初始化逻辑initializeBean(beanName, exposedObject, mbd);
  protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
      Object wrappedBean = bean;
      if (mbd == null || !mbd.isSynthetic()) {
      // 初始化前逻辑
     	 wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
      }
  
      try {
      	// 初始化逻辑
      	invokeInitMethods(beanName, wrappedBean, mbd);
      }
      catch (Throwable ex) {
      	throw new BeanCreationException(
          (mbd != null ? mbd.getResourceDescription() : null),
          beanName, "Invocation of init method failed", ex);
     }
     if (mbd == null || !mbd.isSynthetic()) {
     		// 初始化后逻辑
     		wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
      }
  
      return wrappedBean;
      
  }
  
  //9. 初始化前逻辑，可以看出是循环遍历BeanPostProcessor的实现类，然后调用对应的方法
  public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName) throws BeansException {
  
      Object result = existingBean;
      for (BeanPostProcessor processor : getBeanPostProcessors()) {
          Object current = processor.postProcessBeforeInitialization(result, beanName);
          if (current == null) {
              return result;
          }
          result = current;
      }
      return result;
  }
  
  ```

- BeanPostProcessor.postProcessBeforeInitialization(result, beanName)可以点出来自定义实现类

![1620899883716](E:\SoftwareNote\面试准备\Spring\img\BeanPostProcessor_postProcessBeforeInitialization(result, beanName)可以点出来自定义实现类.png)

## 6. BeanPostProcessor.postProcessBeforeInitialization(result, beanName)所引申的注解/注入/AOP等

由5.5可以看出，在Bean初始化前，会遍历所有BeanPostProcessor的实现类，挨个调用postProcessBeforeInitialization方法。

### 6.1 ApplicationContextAwareProcessor 

```java
@Nullable
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
          bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
          bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware ||
          bean instanceof ApplicationStartupAware)) {
        return bean;
    }

    AccessControlContext acc = null;

    if (System.getSecurityManager() != null) {
        acc = this.applicationContext.getBeanFactory().getAccessControlContext();
    }

    if (acc != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            invokeAwareInterfaces(bean);
            return null;
        }, acc);
    }
    else {
        invokeAwareInterfaces(bean);
    }

    return bean;
}

private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationStartupAware) {
        ((ApplicationStartupAware) bean).setApplicationStartup(this.applicationContext.getApplicationStartup());
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```

### 6.2 InitDestroyAnnotationBeanPostProcessor 

```java

```

### 6.3 EmbeddedValueResolverAware

```java
@Override
public void setEmbeddedValueResolver(StringValueResolver resolver) {
    // 可以取到配置文件信息
    resolver.resolveStringValue("abc ${person.name}");
}
```



## 7. @Value 和 @PropertySource

- PropertySource可以加载properties配置文件中的数据，会被加载到ApplicationContext.Environment，因此可以从里面拿到数据

```java
@Configuration
@PropertySource(value = {"classpath:/application.properties"})
public class BeanConfig {
    {
        ConfigurableEnvironment environment = 
            annotationConfigApplicationContext.getEnvironment();	
        String property = environment.getProperty("person.name"); 
    }

}

@Data
public class Person {
    private int age;
    @Value("${person.name}")
    private String name;

    public Person(int age, String name) {
        System.out.println("person对象创建, name=" + name);
        this.age = age;
        this.name = name;
    }

    public Person() {
        System.out.println("person对象创建2");
    }
}


```

## 8. 依赖注入 DI 

### 8.1 @Autowire ： 结合@Qualifer  @Primary 

- 先按照类型查找：XX.class

  `applicationContext.getBean(Person.class)`

- 找不到再用变量命名查找

  `applicationContext.getBean("person")`

- 可以用@Qualifer指定容器id

  `@Qualifer("person1")`

- 找不到拉倒

  `@Autowire(required=false)`

- 首选Bean : 比如多数据源场景

  `@Primary`

- 应用场景： 

  - 构造器: 单参数时，甚至可以忽略，也可以注入

    ```java
    @Autowire
    public Person(Dog dog) {
        this.dog=dog;
    }
    
    // 相当于
    @Autowire
    Dog dog;
    
    // 单参数时，甚至可以忽略，也可以注入
    ```

  - 属性: 上面的用法

  - 方法： 可用于**静态属性注入**  

    ```
    静态方法是属于类（class）的，普通方法才是属于实体对象（也就是New出来的对象）的，spring注入是在容器中实例化对象，所以不能使用静态方法。
    ```

    ```java
    // 无法静态注入
    private static Dog dog;
    
    // 供外部静态调用，内部属性必须是静态
    public static Dog getDog() {
        return dog;
    }
    
    @Autowire
    public void setDog(Dog dog) {
        this.dog = dog;
    }
    ```

  - 参数

    ```java
    public void setDog(@Autowire Dog dog) {
        this.dog = dog;
    }
    
    public Person(@Autowire Dog dog) {
        this.dog=dog;
    }
    ```

  - **不写** 

    - @Bean+单方法入参

      ```java
      @Bean
      // @Autowire 此处不写，dog依赖会注入
      public Person(Dog dog) {
          Person person = new Person (dog);
          return person;
      }
      ```

    - 构造注入

      依赖不可变: 只有使用构造函数注入才能注入final 

      在使用构造器的使用能避免注入的依赖是空的情况 

      ```java
      private Dog dog;
      private final Cat cat;
      // @Autowire 此处不写，dog/cat依赖会注入
      public Person(Dog dog, Cat cat) {
          this.dog=dog;
          this.cat=cat;
      }
      ```

### 8.2 @Resource JSR250规范

- 默认按照属性命名查找

  `@Resource Person person`

- 指定容器id

  `@Resource(name = "person1")`

- 与@Autowire的差异

  - 没有required=false
  - 没有@Primary

### 8.3 @Inject ： JSR330规范

- 需要包支持javax.inject
- 和@Autowire大体相同
- 不支持required=false的功能，但是可以有@Primary

### 8.4 BeanPostProcessor.postProcessBeforeInitialization等底层set操作的注入

见# 6

## 9. Profile 多环境切换

- 指定环境
  - 启动服务器，可以添加环境参数 `-Dspring.profile.active=test`
  - `annotationConfigApplicationContext.getEnvironment().setActiveProfiles("test");`

- 当没有指定环境信息时，有使用@Profile声明的都不会注入
- 写在配置文件类上，相当于声明整个类
- 没有标注的，任何环境都会生效

```java
@Bean
@Profile("test") // 声明所属环境test
public DataSource datasourceTest() {
    return null;
}

@Bean
@Profile("dev")
public DataSource datasourceDev() {
    return null;
}

@Bean
@Profile("dev")
public Person person02() {
    return null;
}

@Bean // 永远生效
public Person person01() {
    return null;
}


```

## 10. Spring AOP

***AOP可以拦截注解***

### 10.1 使用见Spring.md#1

### 10.2 原理

#### 10.2.1 开启切面支持 @EnableAspectJAutoProxy

1. Enable注解导入的其他Bean @Import（AspectJAutoProxyRegistrar.class）

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
    
}
```

2. Import注册器实现类： Import原理见# 1.3

   注册了Bean ： AnnotationAwareAspectJAutoProxyCreator

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    /**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
        AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		// 注册一些信息 org.springframework.aop.config.internalAutoProxyCreator
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

        AnnotationAttributes enableAspectJAutoProxy =
            AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        if (enableAspectJAutoProxy != null) {
            if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
                AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
            }
            if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
                AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
            }
        }
	}
}

public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry) {
    return registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry, null);
}

public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}

private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            if (currentPriority < requiredPriority) {
                apcDefinition.setBeanClassName(cls.getName());
            }
        }
        return null;
    }
    // cls = AnnotationAwareAspectJAutoProxyCreator.class
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    // 注册Bean internalAutoProxyCreator
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}
```

3. AnnotationAwareAspectJAutoProxyCreator的继承关系： 顶层实现了BeanPostProcessor和BeanFactoryAware

   ![1620967407930](E:\SoftwareNote\面试准备\Spring\img\AnnotationAwareAspectJAutoProxyCreator继承关系图.png)

4. 由# 6可知，实现了BeanPostProcessor，在bean初始化前后会做一些set操作。当有被切入的bean创建时，就会调用以下方法构建bean对应的代理类。

   ```java
   postProcessBeforeInitialization
   postProcessAfterInitialization
   
   public abstract class AbstractAutoProxyCreator {
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
               if (bean != null) {
                   Object cacheKey = getCacheKey(bean.getClass(), beanName);
                   if (!this.earlyProxyReferences.contains(cacheKey)) {
                       return wrapIfNecessary(bean, beanName, cacheKey);
                   }
               }
               return bean;
           }
           
       public Object postProcessBeforeInitialization(Object bean, String beanName) {
   		return bean;
   	}
   	
   	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   		if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
   			return bean;
   		}
   		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
   			return bean;
   		}
   		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
   			this.advisedBeans.put(cacheKey, Boolean.FALSE);
   			return bean;
   		}
   
   		// Create proxy if we have advice.
   		// 返回所有的切面方法，如果该bean有方法作为切点的话
   		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
   		// 如果不为空，则进行代理类注册
   		if (specificInterceptors != DO_NOT_PROXY) {
   			this.advisedBeans.put(cacheKey, Boolean.TRUE);
   			// 创建代理类
   			Object proxy = createProxy(
   					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
   			this.proxyTypes.put(cacheKey, proxy.getClass());
   			return proxy;
   		}
   
   		this.advisedBeans.put(cacheKey, Boolean.FALSE);
   		return bean;
   	}
   	
   	protected Object createProxy(
   			Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
   
   		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
   			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
   		}
   		// 创建代理工厂
   		ProxyFactory proxyFactory = new ProxyFactory();
   		proxyFactory.copyFrom(this);
   
   		if (!proxyFactory.isProxyTargetClass()) {
   			if (shouldProxyTargetClass(beanClass, beanName)) {
   				proxyFactory.setProxyTargetClass(true);
   			}
   			else {
   				evaluateProxyInterfaces(beanClass, proxyFactory);
   			}
   		}
   		// 切面方法
   		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
   		proxyFactory.addAdvisors(advisors);
   		proxyFactory.setTargetSource(targetSource);
   		customizeProxyFactory(proxyFactory);
   
   		proxyFactory.setFrozen(this.freezeProxy);
   		if (advisorsPreFiltered()) {
   			proxyFactory.setPreFiltered(true);
   		}
   		// 创建代理类
   		return proxyFactory.getProxy(getProxyClassLoader());
   	}
   }
   public class ProxyFactory extends ProxyCreatorSupport {
   	public Object getProxy(ClassLoader classLoader) {	
   		return createAopProxy().getProxy(classLoader);
   	}
   	
   	protected final synchronized AopProxy createAopProxy() {
   		if (!this.active) {
   			activate();
   		}
   		return getAopProxyFactory().createAopProxy(this);
   	}
   }
   
   
   public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
   
   	@Override
   	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
   		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
   			Class<?> targetClass = config.getTargetClass();
   			if (targetClass == null) {
   				throw new AopConfigException("TargetSource cannot determine target class: " + "Either an interface or a target is required for proxy creation.");
   			}
   			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
   				// 有接口，用JDK静态代理
   				return new JdkDynamicAopProxy(config);
   			}
   			// CGLib代理模式，默认都是
   			return new ObjenesisCglibAopProxy(config);
   		}
   		else {
   			// 用JDK静态代理
   			return new JdkDynamicAopProxy(config);
   		}
   	}
   }
   
   	
   ```


#### 10.2.2 代理被切类:在创建Bean的时候

见# 10.2.1 -4  

#### 10.2.3 代理方法调用

```java
class CglibAopProxy implements AopProxy, Serializable {
    private static class DynamicAdvisedInterceptor implements MethodInterceptor, Serializable {
        // 1. 调用方法时，用的是代理类调用 SpirngAOPDemo$$EnhancerBySpringCGLIB$$acd54dcc@4550
        public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
            // ...
            // 获取该方法的所有切面方法，按顺序，并且会存再缓存里面，key为方法全名，value为链路
            List<Object> chain = 
                this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
            // ...
            // 执行CglibMethodInvocation.proceed()方法做增强动作以及原始调用
            retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, 
                                               methodProxy).proceed();
            
        }
    }
}

public class ReflectiveMethodInvocation implements ProxyMethodInvocation, Cloneable {
    public Object proceed() throws Throwable {
        //	We start with an index of -1 and increment early.
        // 遍历interceptorsAndDynamicMethodMatchers，即外层时chain，切面调用链，interceptorsAndDynamicMethodMatchers里面是有序的，顺序正好是AOP不同切入方式的顺序。
        // 注意Spring4和5的区别，5修改了4的顺序。
        // 4.after->先afterThrowing/afterReturning
        // 5.先afterThrowing/afterReturning->after  明显5更符合逻辑
        if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size()-1) 		{ 
            // 遍历结束，调用方法
            return invokeJoinpoint(); 
        }
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// Evaluate dynamic method matcher here: static part will already have
			// been evaluated and found to match.
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			else {
				// Dynamic matching failed.
				// Skip this interceptor and invoke the next in the chain.
				return proceed();
			}
		}
		else {
			// It's an interceptor, so we just invoke it: The pointcut will have
			// been evaluated statically before this object was constructed.
            // 遍历出来的元素即切面，每次调用invoke
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
    }
}

// @Before
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, Serializable {
	public Object invoke(MethodInvocation mi) throws Throwable {
		this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis() );
		return mi.proceed();
	}
}

// @AfterThrowing
public class AspectJAfterThrowingAdvice extends AbstractAspectJAdvice
		implements MethodInterceptor, AfterAdvice, Serializable {
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		catch (Throwable ex) {
			if (shouldInvokeOnThrowing(ex)) {
				invokeAdviceMethod(getJoinPointMatch(), null, ex);
			}
			throw ex;
		}
	}
}

class CglibAopProxy implements AopProxy, Serializable {
    private static class CglibMethodInvocation extends ReflectiveMethodInvocation {
    	protected Object invokeJoinpoint() throws Throwable {
			if (this.publicMethod) {
                // 执行原始方法
				return this.methodProxy.invoke(this.target, this.arguments);
			}
			else {
				return super.invokeJoinpoint();
			}
		}
    }
}

// 最终是通过这里跳进切面执行方法，就是反射而已
public abstract class AbstractAspectJAdvice implements Advice, AspectJPrecedenceInformation, Serializable {
	protected Object invokeAdviceMethodWithGivenArgs(Object[] args) throws Throwable {
		Object[] actualArgs = args;
		if (this.aspectJAdviceMethod.getParameterTypes().length == 0) {
			actualArgs = null;
		}
		try {
			ReflectionUtils.makeAccessible(this.aspectJAdviceMethod);
			// TODO AopUtils.invokeJoinpointUsingReflection
			// aspectJAdviceMethod = “public void com.codeman.springaop.demo.AOPComponent.before(org.aspectj.lang.JoinPoint)”
			// this.aspectInstanceFactory.getAspectInstance()=“AOPComponent”切面类
			return 	
			this.aspectJAdviceMethod.invoke(this.aspectInstanceFactory.getAspectInstance(), 
				actualArgs);
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

- MethodInterceptor的实现，各种增强，@After @Before等等的底层

![1620978423232](E:\SoftwareNote\面试准备\Spring\img\MethodInterceptor的实现.png)

- Spring4的AOP增强调用顺序

![1620955408293](E:\SoftwareNote\面试准备\Spring\img\Spring4的AOP增强调用顺序.png)

### 10.3 总结

1. 注解@EnableAspectJAutoProxy会开启AOP公告
2. 在注解@EnableAsepctJAutoProxy内部，有@Import(AsepctJAutoProxyRegistrar.class) ，即开启AOP的同时，会导入另外一个Bean -> AsepctJAutoProxyRegistrar
3. 而这个Bean是实现了ImportBeanDefinitionRegistrar，即重写registerBeanDefinitions方法，可以在Bean创建时，手动添加注册Bean，AsepctJAutoProxyRegistrat内部注册了AnnotationAwareAspectJAutoProxyCreator
4. AnnotationAwareAspectJAutoProxyCreator实现了BeanPostProcessor，意味着Spring所有的Bean在初始化前后，都会调用该实现类的postProcessBeforeInitialization和postProcessAfterInitialization方法。
5. AnnotationAwareAspectJAutoProxyCreator的父类AbstractAutoProxyCreator重写了后置处理器方法postProcessAfterInitialization，该方法内部对所有Bean进行判断，如果有被增强（切点为该bean的方法），那么就会创建该bean的代理Bean（proxyFactory.getProxy默认采用ObjenesisCglibAopProxy，即CGLIB代理模式，而非JdkDynamicAopProxy），并且带有所有的增强方法。
6. 以上，完成了代理Bean的创建。接下来就是代理Bean来执行方法调用（增强原方法）
7. 执行原方法时，就会被代理类接管，跳到CglibAopProxy.DynamicAdvisedInterceptor.intercept。方法内部是利用拦截器的链式戒指，依次进入每个拦截器进行执行（类似CP的bizRuleCheck）。
8. 注意Spring4和5的区别，5修改了4的顺序。

- Spring4 ： before -> 目标方法 -> after->先afterThrowing/afterReturning
- Spring5 ： before -> 目标方法 -> afterThrowing/afterReturning->after  明显5更符合逻辑

## 11. 事务

- pom

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  ```

- 启动开关

  `@EnableTransactionManagement`

- 方法注解

  `@Transactional`

### 11.1 源码解析 ： 和AOP思路相当，@Import采用的是实现ImportSelector

#### 11.1.1 @Import注册了AutoProxyRegistrar/ProxyTransactionManagementConfiguration

```java
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
    boolean proxyTargetClass() default false;
    AdviceMode mode() default AdviceMode.PROXY;
    int order() default Ordered.LOWEST_PRECEDENCE;
}

public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

    @Override
    protected String[] selectImports(AdviceMode adviceMode) {
        switch (adviceMode) {
            case PROXY:
                // 注册了AutoProxyRegistrar/ProxyTransactionManagementConfiguration
                return new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
            case ASPECTJ:
                return new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
            default:
                return null;
        }
    }
}

public abstract class AdviceModeImportSelector<A extends Annotation> implements ImportSelector {
    protected abstract String[] selectImports(AdviceMode adviceMode);
    
	public final String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 拿泛型，EnableTransactionManagement.class
		Class<?> annoType  = GenericTypeResolver.resolveTypeArgument(getClass(), 		AdviceModeImportSelector.class);
        // 获取注解所带的属性以及值order/mode/proxyTargetClass
		AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
		if (attributes == null) {
			throw new IllegalArgumentException(String.format(
				"@%s is not present on importing class '%s' as expected",
				annoType.getSimpleName(), importingClassMetadata.getClassName()));
		}
		// 获取@EnableTransactionManagement.mode，默认为PROXY
		AdviceMode adviceMode = attributes.getEnum(this.getAdviceModeAttributeName());
        // abstract selectImports()需要子类重写
		String[] imports = selectImports(adviceMode);
		if (imports == null) {
			throw new IllegalArgumentException(String.format("Unknown AdviceMode: '%s'", adviceMode));
		}
		return imports;
	}
}
```

#### 11.1.2 AutoProxyRegistrar/ProxyTransactionManagementConfiguration的作用

1. AutoProxyRegistrar.class 实现了ImportBeanDefinitionRegistrar，

   注册了InfrastructureAdvisorAutoProxyCreator

```java
public class AutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    // importingClassMetadata为创建上下文的类信息
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean candidateFound = false;
        // 获取该类头上的注解
        Set<String> annoTypes = importingClassMetadata.getAnnotationTypes();
        for (String annoType : annoTypes) {
            AnnotationAttributes candidate = 
                AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
            if (candidate == null) {
                continue;
            }
            Object mode = candidate.get("mode");
            Object proxyTargetClass = candidate.get("proxyTargetClass");
            if (mode != null && proxyTargetClass != null && AdviceMode.class == mode.getClass() 
                && Boolean.class == proxyTargetClass.getClass()) {
                candidateFound = true;
                if (mode == AdviceMode.PROXY) {
                    AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
                    if ((Boolean) proxyTargetClass) {
                        AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
                        return;
                    }
                }
            }
		}
		// ... 
	}
}

// 和AOP一个套路，注册了InfrastructureAdvisorAutoProxyCreator.class
public abstract class AopConfigUtils {
    public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry
                                                                     registry) {
        return registerAutoProxyCreatorIfNecessary(registry, null);
    }

    public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry 
                                                                     registry, Object source) {
        return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, 
                                               registry, source);
    }
}
```

​	1.1 InfrastructureAdvisorAutoProxyCreator和AspectJAwareAdvisorAutoProxyCreator（AOP）同级，共用AbstractAutoProxyCreator的postProcessAfterInitialization后置通知，创建代理类。

```java

```



2. ProxyTransactionManagementConfiguration : 简单粗暴，该类是个@Configuration。创建了三个Bean

   BeanFactoryTransactionAttributeSourceAdvisor
   TransactionAttributeSource
   TransactionInterceptor

```java
@Configuration
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {
    @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
		BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
		advisor.setTransactionAttributeSource(transactionAttributeSource());
		advisor.setAdvice(transactionInterceptor());
		advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
		return advisor;
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionAttributeSource transactionAttributeSource() {
        // 这里
		return new AnnotationTransactionAttributeSource();
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionInterceptor transactionInterceptor() {
		TransactionInterceptor interceptor = new TransactionInterceptor();
		interceptor.setTransactionAttributeSource(transactionAttributeSource());
		if (this.txManager != null) {
			interceptor.setTransactionManager(this.txManager);
		}
		return interceptor;
	}
}
```

2. 1 AnnotationTransactionAttributeSource.class

```java
public class AnnotationTransactionAttributeSource extends 
    AbstractFallbackTransactionAttributeSource
		implements Serializable {
		
    public AnnotationTransactionAttributeSource() {
		this(true);
	}
	
	public AnnotationTransactionAttributeSource(boolean publicMethodsOnly) {
		this.publicMethodsOnly = publicMethodsOnly;
		this.annotationParsers = new LinkedHashSet<TransactionAnnotationParser>(2);
		// 这里，事务解析器
		this.annotationParsers.add(new SpringTransactionAnnotationParser());
		if (jta12Present) {
			this.annotationParsers.add(new JtaTransactionAnnotationParser());
		}
		if (ejb3Present) {
			this.annotationParsers.add(new Ejb3TransactionAnnotationParser());
		}
	}
}

public class SpringTransactionAnnotationParser implements TransactionAnnotationParser, Serializable {
	// 解析事务属性
	public TransactionAttribute parseTransactionAnnotation(AnnotatedElement ae) {
		AnnotationAttributes attributes = 
			AnnotatedElementUtils.getMergedAnnotationAttributes(ae, Transactional.class);
		if (attributes != null) {
			// 这里
			return parseTransactionAnnotation(attributes);
		}
		else {
			return null;
		}
	}
	// 解析@Transactional()所标注的属性
	protected TransactionAttribute parseTransactionAnnotation(AnnotationAttributes attributes) {
		RuleBasedTransactionAttribute rbta = new RuleBasedTransactionAttribute();
		Propagation propagation = attributes.getEnum("propagation");
		rbta.setPropagationBehavior(propagation.value());
		Isolation isolation = attributes.getEnum("isolation");
		rbta.setIsolationLevel(isolation.value());
		rbta.setTimeout(attributes.getNumber("timeout").intValue());
		rbta.setReadOnly(attributes.getBoolean("readOnly"));
		rbta.setQualifier(attributes.getString("value"));
		ArrayList<RollbackRuleAttribute> rollBackRules = new ArrayList<RollbackRuleAttribute>();
		Class<?>[] rbf = attributes.getClassArray("rollbackFor");
		for (Class<?> rbRule : rbf) {
			RollbackRuleAttribute rule = new RollbackRuleAttribute(rbRule);
			rollBackRules.add(rule);
		}
		String[] rbfc = attributes.getStringArray("rollbackForClassName");
		for (String rbRule : rbfc) {
			RollbackRuleAttribute rule = new RollbackRuleAttribute(rbRule);
			rollBackRules.add(rule);
		}
		Class<?>[] nrbf = attributes.getClassArray("noRollbackFor");
		for (Class<?> rbRule : nrbf) {
			NoRollbackRuleAttribute rule = new NoRollbackRuleAttribute(rbRule);
			rollBackRules.add(rule);
		}
		String[] nrbfc = attributes.getStringArray("noRollbackForClassName");
		for (String rbRule : nrbfc) {
			NoRollbackRuleAttribute rule = new NoRollbackRuleAttribute(rbRule);
			rollBackRules.add(rule);
		}
		rbta.getRollbackRules().addAll(rollBackRules);
		return rbta;
	}
}
```

2. 2 TransactionInterceptor.class

```java
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable {
    public Object invoke(final MethodInvocation invocation) throws Throwable {
		// ...
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

		// Adapt to TransactionAspectSupport's invokeWithinTransaction...
		return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
			@Override
			public Object proceedWithInvocation() throws Throwable {
                 // 这里也是类似AOP的链式调用，底层是同一个接口，实现类不同
				return invocation.proceed();
			}
		});
	}
    protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation) throws Throwable {
        
        // If the transaction attribute is null, the method is non-transactional.
		final TransactionAttribute txAttr = 
            getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
        // 获取事务管理器
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

		if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
			// Standard transaction demarcation with getTransaction and commit/rollback calls.
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
                // 执行原方法
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
                // 异常回滚rollback
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}
            // 正常提交commit
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}
        
        // ...
    }
}
public abstract class TransactionAspectSupport implements BeanFactoryAware, InitializingBean {
    
    // 获取事务管理器
    protected PlatformTransactionManager determineTransactionManager(TransactionAttribute txAttr) {
		// Do not attempt to lookup tx manager if no tx attributes are set
		if (txAttr == null || this.beanFactory == null) {
			return getTransactionManager();
		}
		String qualifier = txAttr.getQualifier();
        // 看看@Transactional(transactionManager="")有无指定事务管理器transactionManager
		if (StringUtils.hasText(qualifier)) {
            // 有则直接返回
			return determineQualifiedTransactionManager(qualifier);
		}
		else if (StringUtils.hasText(this.transactionManagerBeanName)) {
			return determineQualifiedTransactionManager(this.transactionManagerBeanName);
		}
		else {
			PlatformTransactionManager defaultTransactionManager = getTransactionManager();
			if (defaultTransactionManager == null) {
				defaultTransactionManager = 
                    this.transactionManagerCache.get(DEFAULT_TRANSACTION_MANAGER_KEY);
				if (defaultTransactionManager == null) {
                    // 没有指定事务管理器，则从IOC容器中获取PlatformTransactionManager类型的Bean
                    // 因此，我们想要指定某个事务处理器，可以直接@Bean导入一个即可,不需额外配置
					defaultTransactionManager = 
                        this.beanFactory.getBean(PlatformTransactionManager.class);
					this.transactionManagerCache.putIfAbsent(
							DEFAULT_TRANSACTION_MANAGER_KEY, defaultTransactionManager);
				}
			}
			return defaultTransactionManager;
		}
	}
    
    // 异常回滚事务
    protected void completeTransactionAfterThrowing(TransactionInfo txInfo, Throwable ex) {
        if (txInfo != null && txInfo.hasTransaction()) {
            // @Transactional(rollbackOn=true)
            if (txInfo.transactionAttribute.rollbackOn(ex)) {
                // ...
                txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
            } else {
                // @Transactional(rollbackOn=false) 没开事务的就不进行回滚
                // ... 
                txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
            }
        }
    }
    // 正常提交事务
    protected void commitTransactionAfterReturning(TransactionInfo txInfo) {
		if (txInfo != null && txInfo.hasTransaction()) {
			txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
		}
	}
}
```

### 11.2 手动提交事务

```java
@Autowired private TransactionTemplate transactionTemplate;

public String decrById(@PathVariable Integer id, HttpSession session) { 
    //设置传播行为：总是新启一个事务，如果存在原事务，就挂起原事务PROPAGATION_REQUIRES_NEW
    transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
    transactionTemplate.execute(status -> {
                // 扣除商品库存
                mallGoodsService.decrById(id);
                // 从缓存中获取userid
                Object uid = session.getAttribute("uid");
                uid = uid == null ? "1" : uid;
                try {
                     shopcartFeignService.updateById(Integer.parseInt((String) uid), id);
                }catch(Exception e) {
                    //手动回滚事务
                    status.setRollbackOnly(); 
                }
               
                return null;
            });

            return "扣除成功";
}
```



## 12 . 拓展

### 12.1 BeanFactoryPostProcessor 

- 类似BeanPostProcessor, 是BeanFactory的后置处理器.
- 在BeanFactory标准初始化之后调用,来指定和修改BeanFactroy内容.
- 所有的Bean定义已经保存在BeanFactroy,但此时Bean还没创建
- fresh的invokeBeanFactoryPostProcessors(beanFactory)方法内部,找到所有BeanFactoryPostProcessor的实现,调用postProcessBeanFactory方法.在其他组件初始化前执行

### 12.2 BeanDefinitionRegistryPostProcessor

- BeanDefinition是Bean的定义信息，里面包含了class/scope/abstract/lazyInit等等

```tex
Root bean: class [com.yami.shop.mp.config.WxMpConfiguration]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in file [E:\Java_Code\IDEA_CODE\mall4j\yami-shop-mp\target\classes\com\yami\shop\mp\config\WxMpConfiguration.class]
```

- BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor
- 实现postProcessBeanDefinitionRegistry方法
- 在所有Bean定义信息将要被加载,Bean实例还未创建.优先于BeanFactoryPostProcessor执行
- 利用该实现,可以z再给容器增加一些组件

### 12.3 监听器ApplicationListener

- 自定义监听器

```java
@Component
public class MyApplicationListener implements ApplicationListener {

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("收到收到：" + event);
    }
}

方式二:
@EventListener(classes={ApplicatinEvent.class})
public void test(ApplicationEvent event) {
    
}
```

- 发布事件

```
applicationContext.publishEvent(new ApplicationEvent("你好啊") {});

```

- 源码解析

```java
public void refresh() {
 	finishRefresh();   
}

protected void finishRefresh() {
	// Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));
}
```

## 13. Servlet3.0

### 13.1 @WebServlet 替换web.xml (tomcat7+)

同理还有@WebFilter  @WebListener

```xml
<servlet>
    <servlet-name>hello2</servlet-name>
</servlet>
<servlet-mapping>
    <servlet-name>hello2</servlet-name>
    <url-pattern>/index.jsp</url-pattern>
</servlet-mapping>
```

```java
@WebServlet("/hello")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello  world");
//        super.doGet(req, resp);
    }
}
```

### 13.2 @HandlesTypes + ServletContainerInitializer

```java
// 感兴趣的type
@HandlesTypes(value = {Object.class})
public class MyServeltContainerInitialier implements ServletContainerInitializer {

    @Override
    // 启动时，可以拿到HandlesTypes里面的所有子类集合set
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        System.out.println("---------------");
        System.out.println(set);
        System.out.println("---------------");
    }
}

// 创建文件
META-INF/services/javax.servlet.ServletContainerInitializerjavax.servlet.ServletContainerInitializer
// 里面放ServletContainerInitializer的实现类全类名
com.codemna.springservlet.servlet.MyServeltContainerInitialier
// 这样启动web时，就会调用实现类的onStartup方法
```

### 13.3 使用ServletContext注册三大组件，替换web.xml，实现SpringMVC

```java
@HandlesTypes(value = {Object.class})
public class MyServeltContainerInitialier implements ServletContainerInitializer {

    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
//        System.out.println("---------------");
//        System.out.println(set);
//        System.out.println("---------------");

        System.out.println("ServletContainerInitializer.onStartup......");

        // servlet
        ServletRegistration.Dynamic servlet1 = servletContext.addServlet("servlet1", new MyServlet());
        servlet1.addMapping("/my");

        // listener
        servletContext.addListener(MyListener.class);

        // filter
        FilterRegistration.Dynamic filter1 = servletContext.addFilter("filter1", new MyFilter());
        // 配置filter映射信息
        // 拦截所有请求 /*
        filter1.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*" );

    }
}

//@WebServlet("/hello")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello  world");
//        System.out.println("MyServlet doGet...");
//        super.doGet(req, resp);
    }
}

public class MyFilter extends HttpFilter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        System.out.println("MyFilter  doFilter...");
        chain.doFilter(req, res);
    }

    @Override
    protected void doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain) throws IOException, ServletException {
        System.out.println("MyFilter  doFilter2...");
        chain.doFilter(req, res);
    }
}

public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("MyListener.contextInitialized");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("MyListener.contextDestroyed");
    }
}

// 创建文件
META-INF/services/javax.servlet.ServletContainerInitializerjavax.servlet.ServletContainerInitializer
// 里面放ServletContainerInitializer的实现类全类名
com.codemna.springservlet.servlet.MyServeltContainerInitialier
// 这样启动web时，就会调用实现类的onStartup方法
```

### 13.4 Servlet3支持异步处理请求

- 在Servlet3之前，**Servlet采用Thread-Per-Request方式处理请求**。即每次Http请求都由某一个线程从头到尾负责处理。

  在等待业务逻辑处理时，请求线程不能及时放回线程池供其他请求调用，并发越来越大就会出现严重性能问题。而**Spring/Struts这种高层矿机都是建立在Servlet的基础上，因此也有相同的性能问题**。

  于是在**Servelt3.0(2009.12)引入了异步处理**。**Servlet3.1(2013.5)加入了非阻塞IO**来进一步增强异步处理的性能！

- Servlet3.0之前，一个线程处理请求

![1621070790084](E:\SoftwareNote\面试准备\Spring\img\Servlet3.0之前，一个线程处理请求.png)

- Servlet3.0开始，Http请求线程和工作异步线程分开。（可以快速回收请求线程，请求线程只是个带路的）

![1621071150381](E:\SoftwareNote\面试准备\Spring\img\Servlet3.0开始，Http请求线程和工作异步线程分开.png)

- 代码HttpServletRequest.startAsync().start(runnable)
  - 方式一：@WebServlet(value = "/hello", asyncSupported = true)，asyncSupported 异步支持打开
  - 方式二：ServletContainerInitializer的onStartup方法中开启servlet1.setAsyncSupported(true);

```java
//@WebServlet(value = "/hello", asyncSupported = true)
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        System.out.println("Main thread " + Thread.currentThread().getName() + "start " + System.currentTimeMillis());
        // 业务处理
//        BizMethod();
        // 如果当前请求不在异步模式下，则调用此方法是非法的（即isAsyncStarted（）返回false）
        AsyncContext asyncContext = req.startAsync();
        asyncContext.start(() -> {
            BizMethod();
            asyncContext.complete();
        });

        resp.getWriter().write("hello  world");
        System.out.println("Main thread " + Thread.currentThread().getName() + "end " + System.currentTimeMillis());
//        System.out.println("MyServlet doGet...");
//        super.doGet(req, resp);
    }
}

@HandlesTypes(value = {Object.class})
public class MyServeltContainerInitialier implements ServletContainerInitializer {

    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
//        System.out.println("---------------");
//        System.out.println(set);
//        System.out.println("---------------");

        System.out.println("ServletContainerInitializer.onStartup......");

        // servlet
        //@WebServlet(value="/servlet1", asyncSupported=true)
        ServletRegistration.Dynamic servlet1 = servletContext.addServlet("servlet1", new MyServlet());
        servlet1.setAsyncSupported(true);
        servlet1.addMapping("/my");
    }
}
```



## 14. SpringMVC整合Servlet3.0

### 14.1 SpringMVC是根据#13.2的servlet特性基础上开发的 

-  javax.servlet.ServletContainerInitializer，servlet中的接口，会找项目中的META-INF/services/javax.servlet.ServletContainerInitializerjavax.servlet.ServletContainerInitializer文件（里面必须是ServletContainerInitializer接口的实现类全类名）然后执行onStartup方法。

  其中方法的入参之一Set<Class<?>> webAppInitializerClasses为实现类的注解@HandlesTypes中带的class的子类且非抽象类集合

  这样就可以用HandlesTypes定义接口的实现类来做一些mvc启动事情

  ```java
  public interface javax.servlet.ServletContainerInitializer {
      void onStartup(Set<Class<?>> var1, ServletContext var2) throws ServletException;
  }
  
  @HandlesTypes(WebApplicationInitializer.class)
  public class org.springframework.web.SpringServletContainerInitializer implements ServletContainerInitializer {
      @Override
  	public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
  			throws ServletException {
  
  		List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();
  
  		if (webAppInitializerClasses != null) {
  			for (Class<?> waiClass : webAppInitializerClasses) {
  				// Be defensive: Some servlet containers provide us with invalid classes,
  				// no matter what @HandlesTypes says...
                  // WebApplicationInitializer的子类且非抽象类
  				if (!waiClass.isInterface() && 
                      !Modifier.isAbstract(waiClass.getModifiers()) &&
  						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
  					try {
  						initializers.add((WebApplicationInitializer) 
                                           waiClass.newInstance());
  					}
  					catch (Throwable ex) {
  						throw new ServletException("Failed to instantiate 
                                                     WebApplicationInitializer class", ex);
  					}
  				}
  			}
  		}
  
  		if (initializers.isEmpty()) {
  			servletContext.log("No Spring WebApplicationInitializer types detected on 
                                 classpath");
  			return;
  		}
  
  		servletContext.log(initializers.size() + " Spring WebApplicationInitializers 
                             detected on classpath");
  		AnnotationAwareOrderComparator.sort(initializers);
  		for (WebApplicationInitializer initializer : initializers) {
              // 调用WebApplicationInitializer.onStartup方法
              // 还故意把方法名字写成一样
  			initializer.onStartup(servletContext);
  		}
  	}
  }
  ```

- WebApplicationInitializer的实现类们：都是抽象类，因此需要自己实现

  ```java
  abstract class AbstractAnnotationConfigDispatcherServletInitializer
        extends abstract AbstractDispatcherServletInitializer
        	extends abstract AbstractContextLoaderInitializer
        		implements WebApplicationInitializer
        		
        		
  /* 总的来说：
  1. WebApplicationInitializer(0)暴露onStartup()给子类重写
  2. AbstractContextLoaderInitializer(1)重写了onStartup，暴露了createRootApplicationContext()方法给子类重写
  3. AbstractDispatcherServletInitializer(2)增强了(1)的重写了onStartup，暴露了createServletApplicationContext()给子类重写
  4. AbstractAnnotationConfigDispatcherServletInitializer(3)重写(1)和(2)暴露的createRootApplicationContext()和createServletApplicationContext()方法，但是又对外暴露了getRootConfigClasses()和getServletConfigClasses()方法，用来隔离App容器和Root容器
  5. 于是，子类只需要继承AbstractAnnotationConfigDispatcherServletInitializer(3)，重写getRootConfigClasses()和getServletConfigClasses()方法即可。
  */
  ```

  - AbstractContextLoaderInitializer

    留createRootApplicationContext()方法给子类重写，该方法就是注册Root容器

  ```java
  public abstract class AbstractContextLoaderInitializer implements WebApplicationInitializer {
      @Override
      // 重写接口onStartup方法
      public void onStartup(ServletContext servletContext) throws ServletException {
  		registerContextLoaderListener(servletContext);
  	}
      
      protected void registerContextLoaderListener(ServletContext servletContext) {
          // createRootApplicationContext()方法为实现，这是等实现类重写
          // 这是注册Root容器
  		WebApplicationContext rootAppContext = createRootApplicationContext();
  		if (rootAppContext != null) {
  			ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
  			listener.setContextInitializers(getRootApplicationContextInitializers());
              // 添加到servlet上下文（容器？）中
  			servletContext.addListener(listener);
  		}
  		else {
  			logger.debug("")
  		}
  	}
      
      protected abstract WebApplicationContext createRootApplicationContext();
      
      protected ApplicationContextInitializer<?>[] getRootApplicationContextInitializers() {
  		return null;
  	}
  }
  ```

  - AbstractDispatcherServletInitializer

    留createServletApplicationContext()给子类重写，该抽象类做的就是注册App容器

  ```java
  public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
      @Override
      // 重写接口onStartup方法
  	public void onStartup(ServletContext servletContext) throws ServletException {
          // 即AbstractContextLoaderInitializer.onStartup
          // 不完全重写，是增强父类功能，在父类方法基础上拓展
  		super.onStartup(servletContext);
  		registerDispatcherServlet(servletContext);
  	}
  }
  
  protected void registerDispatcherServlet(ServletContext servletContext) {
  		String servletName = getServletName();
  		Assert.hasLength(servletName, "getServletName() must not return empty or null");
  		// createServletApplicationContext()留给子类重写
  		WebApplicationContext servletAppContext = createServletApplicationContext();
  		Assert.notNull(servletAppContext,
  				"createServletApplicationContext() did not return an application " +
  				"context for servlet [" + servletName + "]");
  
  		FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
      // getServletApplicationContextInitializers()留给子类重写
      dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());
  
  		ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
  		Assert.notNull(registration,
  				"Failed to register servlet with name '" + servletName + "'." +
  				"Check if there is another servlet registered under the same name.");
  		// servlet容器的一些属性设置
  		registration.setLoadOnStartup(1);
  		registration.addMapping(getServletMappings());
  		registration.setAsyncSupported(isAsyncSupported()); //是否开启请求异步处理功能
  
  		Filter[] filters = getServletFilters();
  		if (!ObjectUtils.isEmpty(filters)) {
  			for (Filter filter : filters) {
  				registerServletFilter(servletContext, filter);
  			}
  		}
  		// 留给子类重写
  		customizeRegistration(registration);
  	}
  
  	protected ApplicationContextInitializer<?>[] getServletApplicationContextInitializers() 	{ return null; }
  	protected void customizeRegistration(ServletRegistration.Dynamic registration) { }
  }
  ```

  - AbstractAnnotationConfigDispatcherServletInitializer

  ```java
  public abstract class AbstractAnnotationConfigDispatcherServletInitializer
      extends AbstractDispatcherServletInitializer {
      @Override
      // 重写了AbstractContextLoaderInitializer的createRootApplicationContext方法
  	protected WebApplicationContext createRootApplicationContext() {
          // getServletConfigClasses()留给子类重写getRootConfigClasses()
  		Class<?>[] configClasses = getRootConfigClasses();
  		if (!ObjectUtils.isEmpty(configClasses)) {
  			AnnotationConfigWebApplicationContext rootAppContext = new 
                  	AnnotationConfigWebApplicationContext();
  			rootAppContext.register(configClasses);
  			return rootAppContext;
  		}
  		else {
  			return null;
  		}
  	}
      
      @Override
      // 重写了AbstractDispatcherServletInitializer.createServletApplicationContext
  	protected WebApplicationContext createServletApplicationContext() {
  		AnnotationConfigWebApplicationContext servletAppContext = new 
              AnnotationConfigWebApplicationContext();
          // getServletConfigClasses()留给子类重写
  		Class<?>[] configClasses = getServletConfigClasses();
  		if (!ObjectUtils.isEmpty(configClasses)) {
  			servletAppContext.register(configClasses);
  		}
  		return servletAppContext;
  	}
      
      protected abstract Class<?>[] getRootConfigClasses();
      
      protected abstract Class<?>[] getServletConfigClasses();
  }
  ```

### 14.2 重写WebApplicationInitializerd的子类AbstractAnnotationConfigDispatcherServletInitializer

```java
public class CustomSpringMVC extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{AppConfig.class};
    }

    @Override
    // 拦截请求
    // “/” 拦截所有的请求（*.js/*.css），但不包含*.jsp
    // “/*” 拦截所有请求
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}

@ComponentScan(value = "com.codeman.springmvcannotation", useDefaultFilters = false,
        // 和AppConfig互补
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class})
        }
)
public class RootConfig {
}

@ComponentScan(value = "com.codeman.springmvcannotation", useDefaultFilters = false,
        // 和RootConfig互补
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class})
        }
)
public class AppConfig extends WebMvcConfigurerAdapter {
}
```

### 14.3  定制SpringMVC（接管MVC。即web.xml中所有mvc相关的配置）

- 开启注解@EnableWebMvc，该注解相当于web.xml的`<mvc:annotation-driven/>`
- 继承WebMvcConfigurerAdapter（实现WebMvcConfigurer），实现内部方法(配置组件：视图解析器/视图映射/静态资源映射/拦截器等等)
- 具体可以参考官方文档[https://docs.spring.io/spring-framework/docs/5.0.2.RELEASE/spring-framework-reference/web.html#mvc-config]

```java
@ComponentScan(value = "com.codeman.springmvcannotation", useDefaultFilters = false,
        // 和RootConfig互补
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class})
        }
)
// 接管MVC。即web.xml中所有mvc相关的配置，都可以移至这里
@EnableWebMvc
public class AppConfig extends WebMvcConfigurerAdapter {

    @Override
    // 配置视图解析器
    /*
    <mvc:view-resolvers>
        <mvc:content-negotiation>
            <mvc:default-views>
                <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
            </mvc:default-views>
        </mvc:content-negotiation>
        <mvc:jsp/>
    </mvc:view-resolvers>
    */
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // 配置View&Controller的映射
        // jsp("/WEB-INF/", ".jsp");
//        registry.jsp();
        // /WEB-INF/view/底下的所有.jsp文件，都通过Controller层的返回值映射找到相应的页面
        registry.jsp("/WEB-INF/view/", ".jsp");
    }

    @Override
    // 配置静态资源访问
    /*
    <mvc:default-servlet-handler/>
    */
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    // TODO 还有很多，可以看官方文档，是全的
    // https://docs.spring.io/spring-framework/docs/5.0.2.RELEASE/spring-framework-reference/web.html#mvc-config
}
```

### 14.4 SpringMVC的异步处理：在controller返回特殊的返回值Callable/DeferredResult

```
// 方式一：
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}

// 方式二
@RequestMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    // 先返回，把对象deferredResult存在某个地方，比如消息队列
    return deferredResult;
}

// In some other thread...
// 在其他地方处理完接口，在setResult，请求就能响应
deferredResult.setResult(data);
```

- Callable的形式
  1.  请求线程请求，控制器**直接返回Callable**
  2.  Spring启动**起步处理** ，讲Callable提交到TaskExecutor，使用一个隔离的线程处理请求任务
  3. DispatcherServelt和所有的Filter退出web容器的线程，但是**response保持打开状态** 
  4. Callable有返回结果，**SpringMVC将请求重新派发（因此又请求了一次，只是本次不走业务逻辑，而是之前返回callable结果）**给容器，恢复之前的处理
  5. 根据Callable的返回结果，SpringMVC继续进行视图渲染等工作

- 请求异步处理的应用场景：不同应用之间的异步

![1621077793188](E:\SoftwareNote\面试准备\Spring\img\请求异步处理的应用场景.png)

## 15. Spring的容器的创建refresh()

![1621091759169](E:\SoftwareNote\面试准备\Spring\img\refresh_01.png)

![1621086059338](E:\SoftwareNote\面试准备\Spring\img\refresh_02.png)

![1621092703195](E:\SoftwareNote\面试准备\Spring\img\refresh_03.png)



## 16. @PostConstruct: java自带 

修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。PostConstruct在构造函数之后执行，init（）方法之前执行。

通常我们会是在Spring框架中使用到@PostConstruct注解 该注解的方法在整个Bean初始化中的执行顺序：

Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)



## 17. @Async 

注释在方法上，则该方法可以异步执行。

启动类注解@EnableAsync

~~可重写AsyncTaskExecutor~~

















