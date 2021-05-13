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
  }
  ```

-  AnnotationConfigApplicationContext加载Configuration对象

  ```java
  AnnotationConfigApplicationContext annotationConfigApplicationContext
                  = new AnnotationConfigApplicationContext(BeanConfig.class);
  Object xiaozhang = annotationConfigApplicationContext.getBean("xiaozhang");
  System.out.println(xiaozhang);
  ```

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
        useDefaultFilters = false)
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

    - 构造注入+单入参

      ```java
      // @Autowire 此处不写，dog依赖会注入
      public Person(Dog dog) {
          this.dog=dog;
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

### 10.1 使用见Spring.md#1

### 10.2 原理

