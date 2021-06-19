#  Spring

## 1. Spring AOP

### 1.1.  pom

```xnk
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 1.2. AOP

***AOP可以拦截注解***

```java
@EnableAspectJAutoProxy // 开启
@SpringBootApplication
public class SpringannotationresourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringannotationresourceApplication.class, args);
    }

}

@Component
@Aspect // 声明这个类是个切面
public class AOPComponent {

    // AOP可以拦截注解
    @Around("@annotation(com.cod4man.fakenews.annotation.redis.RedisSelect)")
    @Pointcut(value = "execution(public int com.codeman.springaop..*.test*(..))")
    public void pointCut() {

    }
    
    // 注解作为入参
    @Around(value = "@annotation(redissionLock)")
    public Object redLock(ProceedingJoinPoint joinPoint, RedissionLock redissionLock) {
        
    }

    @Before(value = "pointCut()")
    public void before(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        Object[] args = jp.getArgs();
        System.out.println("method before, methodName=" + methodName + ". args=" + args);
    }

    @AfterThrowing(value = "pointCut()", throwing = "e")
    // 注意必须放在参数的第一位 JoinPoint jp
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

- Spring4是先after后afterThrowing/afterReturning
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

### 1.5 应用

#### 1.5.1 日志处理

#### 1.5.2 权限控制

- 比如切点为某特定注解
- 标注该注解的方法，都需要被拦截，校验权限

```java
@RestController
@RequestMapping("/area")
@Api(value = "Area", tags = "地区表")
// 整个关键Controller都会被拦截查看权限
@PreAuth(replace = "area:")
public class AreaController extends
    SuperCacheController<AreaService, Long, Area, AreaPageDTO, AreaSaveDTO, AreaUpdateDTO> {

    @ApiOperation(value = "检测地区编码是否重复", notes = "检测地区编码是否重复")
    @GetMapping("/check/{code}")
    @SysLog("检测地区编码是否重复")
    public R<Boolean> check(@RequestParam(required = false) Long id, @PathVariable String code) {
        int count = baseService.count(Wraps.<Area>lbQ().eq(Area::getCode, code).ne(Area::getId, id));
        return success(count > 0);
    }


    @Override
    // 删除方法会被拦截
    public R<Boolean> handlerDelete(List<Long> ids) {
        //TODO 重点测试递归删除
        return R.success(baseService.recursively(ids));
    }
}


// Aspect
{
    // 切PreAuth注解
    @Around("execution(public * com.zkthink.base.controller.*.*(..)) || " +
            "@annotation(com.zkthink.security.annotation.PreAuth) || " +
            "@within(com.zkthink.security.annotation.PreAuth)"
    )
    public Object preAuth(ProceedingJoinPoint point) throws Throwable {
        // 校验权限
        if (handleAuth(point)) {
            return point.proceed();
        }
        // 校验不通过，直接throw Exception
        throw BizException.wrap(ExceptionCode.UNAUTHORIZED);
    }
    
    private boolean handleAuth(ProceedingJoinPoint point) {
        MethodSignature ms = (MethodSignature) point.getSignature();
        Method method = ms.getMethod();
        // 读取权限注解，优先方法上，没有则读取类
        PreAuth preAuth = null;
        if (point.getSignature() instanceof MethodSignature) {
            method = ((MethodSignature) point.getSignature()).getMethod();
            if (method != null) {
                preAuth = method.getAnnotation(PreAuth.class);
            }
        }
        // 读取目标类上的权限注解
        PreAuth targetClass = point.getTarget().getClass().getAnnotation(PreAuth.class);
        //方法和类上 均无注解
        if (preAuth == null && targetClass == null) {
            log.debug("执行方法[{}]无需校验权限", method.getName());
            return true;
        }

        // 方法上禁用
        if (preAuth != null && !preAuth.enabled()) {
            log.debug("执行方法[{}]无需校验权限", method.getName());
            return true;
        }

        // 类上禁用
        if (targetClass != null && !targetClass.enabled()) {
            log.debug("执行方法[{}]无需校验权限", method.getName());
            return true;
        }

        String condition = null;
        if (preAuth == null) {
            condition = targetClass.value();
        } else {
            // 判断表达式
            condition = preAuth.value();
        }
        if (StrUtil.isBlank(condition)) {
            return true;
        }
        if (condition.contains("{}")) {
            if (targetClass != null && StrUtil.isNotBlank(targetClass.replace())) {
                condition = StrUtil.format(condition, targetClass.replace());
            } else {
                // 子类类上边没有 @PreAuth 注解，证明该方法不需要简要权限
                return true;
            }
        }

        Expression expression = SPEL_PARSER.parseExpression(condition);
        // 方法参数值
        Object[] args = point.getArgs();
        StandardEvaluationContext context = getEvaluationContext(method, args);
        if (expression.getValue(context, Boolean.class)) {
            return true;
        }
        throw BizException.wrap(ExceptionCode.UNAUTHORIZED.build("执行方法[%s]需要[%s]权限", method.getName(), condition));
    }
}
```

#### 1.5.3 分布式事务

```java
class {
    @RedisLock(lockName = "insertUser", key = "#appConnect.appId + ':' + #appConnect.bizUserId")
    @Caching(evict = {
			@CacheEvict(cacheNames = "yami_user", key = "#appConnect.appId + ':' + 
                        #appConnect.bizUserId"),
			@CacheEvict(cacheNames = "AppConnect", key = "#appConnect.appId + ':' + 
                        #appConnect.bizUserId")
	})
    public void insertUserIfNecessary(AppConnect appConnect) {
        
    }
}

@Aspect
@Component
public class RedisLockAspect {

	@Autowired
	private RedissonClient redissonClient;

	private static final String REDISSON_LOCK_PREFIX = "redisson_lock:";

    // 拦截自定义注解@redisLock
	@Around("@annotation(redisLock)")
	public Object around(ProceedingJoinPoint joinPoint, RedisLock redisLock) throws Throwable {
		String spel = redisLock.key();
		String lockName = redisLock.lockName();

		RLock rLock = redissonClient.getLock(getRedisKey(joinPoint,lockName,spel));

		rLock.lock(redisLock.expire(),redisLock.timeUnit());

		Object result = null;
		try {
			//执行方法
			result = joinPoint.proceed();

		} finally {
			rLock.unlock();
		}
		return result;
	}

	/**
	 * 将spel表达式转换为字符串
	 * @param joinPoint 切点
	 * @return redisKey
	 */
	private String getRedisKey(ProceedingJoinPoint joinPoint,String lockName,String spel) {
		Signature signature = joinPoint.getSignature();
		MethodSignature methodSignature = (MethodSignature) signature;
		Method targetMethod = methodSignature.getMethod();
		Object target = joinPoint.getTarget();
		Object[] arguments = joinPoint.getArgs();
		return REDISSON_LOCK_PREFIX + lockName + StrUtil.COLON + SpelUtil.parse(target,spel, targetMethod, arguments);
	}
}
```



## 2. Spring IOC过程 / 循环依赖

### 2.1 IOC

#### 2.1.1 @SpringBootApplication

**-> @EnableAutoConfiguration   -> @AutoConfigurationPackage** 

​						   	   **-> @Import(AutoConfigurationImportSelector.class)**

```java
@EnableAutoConfiguration
public @interface SpringBootApplication {}

@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}

@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {}
```

加载了META-INF/spring.factories中的信息，这样后续就可以直接从缓存中获取，而无需再去读文件。

主要文件有：ApplicationContextInitializer的实现类；ApplicationListener的实现类；EnableAutoConfiguration等

> **@Import(AutoConfigurationImportSelector.class)**

```java
class AutoConfigurationImportSelector  {
    //
    String[] selectImports() {
        // 里面会去读META-INF/spring.factories
        // spring.factories里面定义了上百个XXXXXAutoConfiguration
        // 就是SpringBoot事先整合了上百个配置，但是都是@ConditionXXX()因此都不生效（并且没有包依赖没所以底层都是报红，等到需要用时，导入相关依赖，这个事先写好的配置就可以生效了。），而且@Configuration(proxybeansMethods=false),这样Bean就只会被创建一次，提高效率
        // 而且还实现@EnableConfigurationProperties(ServerProperties.class)定义了一些配置类，程序员可以修改默认规则的properties达到定制容器的效果。
        // List<String> autoConfigurationEntry.getConfigurations() 类的完全限定名
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

> **@AutoConfigurationPackage -> @Import(AutoConfigurationPackages.Registrar.class)**

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        // new PackageImport(metadata)为获取扫描的包路径
        register(registry, new PackageImport(metadata).getPackageName());
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImport(metadata));
    }

}

class PackageImport {
    PackageImport(AnnotationMetadata metadata) {
        // (metadata.getClassName() 获取主启动类所在的包名；因此在层级之外的是扫描不到的
        this.packageName = ClassUtils.getPackageName(metadata.getClassName());
    }
}
```

#### 2.1.2  new SpringApplication()

会从文件META-INF/spring.factories实例化一些Initializer和Listener，因此可以通过配置让spring帮我们注入

```java
class SpringApplication {
  public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
      this.resourceLoader = resourceLoader;
      Assert.notNull(primarySources, "PrimarySources must not be null");
      // 记录主启动类 primarySources
      this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
      this.webApplicationType = WebApplicationType.deduceFromClasspath();
      // 实例化初始化器Initializer，从文件META-INF/spring.factories中读取
      setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
      // 实例化监听器Listener，从文件META-INF/spring.factories中读取
      setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
      this.mainApplicationClass = deduceMainApplicationClass();
	}  
}
```

#### 2.1.3 .run()

##### 2.1.3.1 ConfigurableApplicationContext.run(String... args)

```java
class SpringApplication {
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
        configureHeadlessProperty();
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting();
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // 配置环境信息：运行的计算机信息等等
            ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
            configureIgnoreBeanInfo(environment);
            // Banner
            Banner printedBanner = printBanner(environment);
            // Servelt为AnnotationConfigServletWebApplicationContext
            context = createApplicationContext();
            exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                 new Class[] { ConfigurableApplicationContext.class }, context);
            // 上下文准备
            // 加入了 environment listeners applicationArguments等等
            prepareContext(context, environment, listeners, applicationArguments, 	
                           printedBanner);
            // (重要)刷新上下文（刷新容器）  context.refresh()
            refreshContext(context);
            // 刷新后的动作，空实现，可以留给子类重写
            afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), 
                                                                            stopWatch);
            }
            listeners.started(context);
            callRunners(context, applicationArguments);
        }
        catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, listeners);
            throw new IllegalStateException(ex);
        }

        try {
            listeners.running(context);
        }
        catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, null);
            throw new IllegalStateException(ex);
        }
        return context;
    }
}
```

##### 2.1.3.2 AbstractApplicationContext.refresh()

```java
public abstract class AbstractApplicationContext {
    void refresh(){
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
            // 设置开启时间，active状态，以及验证环境必要信息等
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
            // DefaultListableBeanFactory
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
            // 给当前BeanFactory装配一些东西，包括：
            // beanFactory.setBeanClassLoader  是APPClassLoader
            // beanFactory.setBeanExpressionResolver
            // beanFactory.addBeanPostProcessor
            // beanFactory.ignoreDependencyInterface
            // beanFactory.registerResolvableDependency
            // beanFactory.registerSingleton 在这里预先注入了几个环境相关的Bean： environment/systemProperties[System.getProperties()]/systemEnvironment[System.getenv()]
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
                // 注册一些Score和Environment信息 
                // beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext));
                // WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);  注册一些serveltrRequest周期的组件：request/Session/application
				//WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext); // servlet相关的"contextParameters", "contextAttributes"
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
                 // 实例化上面步骤给的BeanFactoryPostProcessor
                // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered 按照Order顺序排
                // 这里注册了很多BeanDifinition
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                // 注册实现了BeanPostProcessor的实现类（注册进ConfigurableListableBeanFactory）
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
                // 初始化AbstractApplicationContext.MessageSource对象if (beanFactory.containsLocalBean("messageSource")){赋值} else {new 一个defalut的DelegatingMessageSource}
				initMessageSource();

				// Initialize event multicaster for this context.
                // 初始化广播器，可以广播给监听者AbstractApplicationContext.ApplicationEventMulticaster，无则创建一个SimpleApplicationEventMulticaster
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                // createWebServer() 创建Web服务器，如Tomcat，Jetty
                // WebApplicationContextUtils.initServletPropertySources(getPropertySources(), servletContext, servletConfig); 初始化一些servlet配置源（如application.yml的路径）
				onRefresh();

				// Check for listener beans and register them.
                // 注册监听器AbstractApplicationContext.ApplicationEventMulticaster
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                 // (重要) 实例化其余非懒加载的单例（大部分单例）
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
}
```

##### 2.1.3.2.5 invokeBeanFactoryPostProcessors （注册了很多beanDefinition）

```java
void invokeBeanFactoryPostProcessors() {
    String[] postProcessorNames =
        // BeanDefinitionRegistryPostProcessor的实现类都被注册beanDefinition了，后续就可以加载
        beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
            processedBeans.add(ppName);
        }
    }
    sortPostProcessors(currentRegistryProcessors, beanFactory);
    registryProcessors.addAll(currentRegistryProcessors);
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
}
```

##### 2.1.3.3 finishBeanFactoryInitialization(beanFactory);

实例化其余非懒加载的单例（大部分单例）

```java
public abstract class AbstractApplicationContext {
    protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory){
        // ...
        beanFactory.preInstantiateSingletons();
    }
}

class DefaultListableBeanFactory {
    public void preInstantiateSingletons(){

        // beanDefinitionNames可以看到，实例化Bean都是从factory的beanDefinition中拿值
        // 因此，在前面操作都实现注册了，比如说@ImportResigester
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					// .. FactoryBean的特殊操作
				}
                // ！isFactoryBean(beanName)
				else {
                    // 执行Bean的获取方法
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				// ..  SmartInitializingSingleton子类的特殊操作
			}
		}
    }
}
```

##### 2.1.3.4 getBean/doGetBean等操作

```java
abstract class AbstractBeanFactory {
   public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	} 
    
    protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

		final String beanName = transformedBeanName(name);
		Object bean;

		// Eagerly check singleton cache for manually registered singletons.
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			// 不为空，可以直接return回去了
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

			// Check if bean definition exists in this factory.
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent.
				// 。。。。
			}

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}

			try {
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// Guarantee initialization of beans that the current bean depends on.
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					// 。。。
				}

				// Create bean instance.
                // Bean定义的是单例的，走下面逻辑
                // 先从一级缓存singletonObjects中获取Bean，获取到直接返回回来，获取不到调用下面的lambda表达式，执行createBean逻辑，真正的去创建Bean
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					// It's a prototype -> create a new instance.
					// 作用域为prototype的逻辑 
                    // 直接走createBean逻辑，相比Singeleton，Singleton会先去singletonObjects中找到，找不到再执行createBean创建Bean逻辑，因此Singleton是单例的
                    prototypeInstance = createBean(beanName, mbd, args);
				}

				else {
					String scopeName = mbd.getScope();
					final Scope scope = this.scopes.get(scopeName);
					
					}
					try {
                        // 先从Scope中获取，获取不到也走createBean逻辑
						Object scopedInstance = scope.get(beanName, () -> {
							beforePrototypeCreation(beanName);
							try {
								return createBean(beanName, mbd, args);
							}
							finally {
								afterPrototypeCreation(beanName);
							}
						});
						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
						// 。。。
					}
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}

		// Check if required type matches the type of the actual bean instance.
		if (requiredType != null && !requiredType.isInstance(bean)) {
			// 。。。
		}
		return (T) bean;
	}
}


// getSingleton(String beanName, ObjectFactory<?> singletonFactory)逻辑
class DefaultSingletonBeanRegistry {
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
            // 先尝试从一级缓存singletonObjects中获取Bean，获取到直接return，因此单例
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				// 。。。
				try {
                    // 一级缓存singletonObjects中找不到Bean，则回调外层的lambda对象
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// 
				}
				catch (BeanCreationException ex) {
					// 
				}
				finally {
					// 
				}
				if (newSingleton) {
                    // Bean创建完毕，加入到singletonObjects中。
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
    
    // 直接加入一级缓存singletonObjects；移除二/三级缓存：earlySingletonObjects/singletonFactories
    protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.put(beanName, singletonObject);
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
	}
}
```

##### 2.1.3.5 找不到Bean则CreateBean

```java
abstract class AbstractBeanFactory {
    // 摸板模式，由子类实现
    protected abstract Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException;
}

abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory {
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {

		RootBeanDefinition mbdToUse = mbd;

		// Make sure bean class is actually resolved at this point, and
		// clone the bean definition in case of a dynamically resolved Class
		// which cannot be stored in the shared merged bean definition.
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			// 。。。
		}

		try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			// 。。。
		}

		try {
            // do才是真正的逻辑
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			// 。。
		}
		catch (Throwable ex) {
			// 。。。
		}
	}
    
    // 真正的创建Bean逻辑
    protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args) throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
            // 实例化Bean
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					// 。。。
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			// 将创建的Bean放入三级缓存singletonFactories中
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
            // populateBean处理的是该Bean的属性注入，循环依赖也是里面处理的
            // 普通属性直接填充，循环依赖的Bean不存在则需要继续走上面的GetBean->CreateBean逻辑
			populateBean(beanName, mbd, instanceWrapper);
            // 初始化Bean
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			// ..
		}

		if (earlySingletonExposure) {
			// ...
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			// ...
		}
		// 返回最终的Bean对象
		return exposedObject;
	}
}
```

##### 2.1.3.6 createBeanInstance：实例化Bean

有很多不同的形式，最终应该是通过反射去创建

```java
abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory{
    protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}

		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// Shortcut when re-creating the same bean...
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		if (resolved) {
			if (autowireNecessary) {
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		// No special handling: simply use no-arg constructor.
		return instantiateBean(beanName, mbd);
	}
}
```

##### 2.1.3.7  populateBean：Bean的填充属性，循环依赖就在这解决

```java
abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory{
    protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    if (bw == null) {
        if (mbd.hasPropertyValues()) {
            throw new BeanCreationException(
                    mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
        else {
            // Skip property population phase for null instance.
            return;
        }
    }

    // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
    // state of the bean before properties are set. This can be used, for example,
    // to support styles of field injection.
    // 需不需要spring来设置属性
    // 实例化之后的 bean的后置处理器
    // org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                //如果返回为 false, 则会终止属性注入
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    return;
                }
            }
        }
    }

    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
        // 这里指的是@Bean(autowire=AUTOWIRE_NO)
    //如果给一个类设置了 : AUTOWIRE_BY_NAME 和 AUTOWIRE_BY_TYPE, 那么类中的属性, 会根据规则自动注入, 而不需要@Autowired或@Resource了
    //默认情况下, 是 AUTOWIRE_NO, 所以这里默认是不执行
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        // Add property values based on autowire by name if applicable.
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        // Add property values based on autowire by type if applicable.
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }

    boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
    //深度引用检查, 引用再引用
    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

    PropertyDescriptor[] filteredPds = null;
    if (hasInstAwareBpps) {
        if (pvs == null) {
            pvs = mbd.getPropertyValues();
        }
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                // 这里也是调用的实例化后的后置处理器, 只是调用的方法不一样
                // 这里一共三个后置处理器会调用
                // 1.ConfigurationClassPostProcessor$ImportAwareBeanPostProcessor
                // 2.CommonAnnotationBeanPostProcessor
                // 3.AutowiredAnnotationBeanPostProcessor
                // 这里会进行 @Autowired 和 @Resource 的注入工作
                PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    if (filteredPds == null) {
                        filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                    }　　　　　　　　　　　　//这里的 postProcessPropertyValues 是一个过时方法
                    pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                    if (pvsToUse == null) {
                        return;
                    }
                }
                pvs = pvsToUse;
            }
        }
    }
    if (needsDepCheck) {
        if (filteredPds == null) {
            filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        }
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }

    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
	}
}
```

1. **CommonAnnotationBeanPostProcessor.postProcessProperties : 处理@Resource**

```java
class CommonAnnotationBeanPostProcessor {
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        //找出类中标注了@Resource注解的属性和方法
    	//else if (field.isAnnotationPresent(Resource.class))
		InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
		try {
            // inject内部处理
			metadata.inject(bean, beanName, pvs);
		}
		catch (Throwable ex) {
			// ...
		}
		return pvs;
	}
    
    //@Resource 在不设置 name 和 type 的情况下, 是优先使用 name 去 获取/创建 依赖的 bean , 如果name找不到, 才会使用 type. 这段代码, 就是支撑的依据.
    protected Object autowireResource(BeanFactory factory, LookupElement element, @Nullable String requestingBeanName)
			throws NoSuchBeanDefinitionException {

		Object resource;
		Set<String> autowiredBeanNames;
		String name = element.name;

		if (factory instanceof AutowireCapableBeanFactory) {
			AutowireCapableBeanFactory beanFactory = (AutowireCapableBeanFactory) factory;
			DependencyDescriptor descriptor = element.getDependencyDescriptor();
            //优先使用 根据 name 注入（!factory.containsBean(name)）, 这个 name 就是 beanName
            //当 beanName 找不到时, 才就会去根据 type 来注入（fallbackToDefaultTypeMatch）
            //依据是 !factory.containsBean(name)
			if (this.fallbackToDefaultTypeMatch && element.isDefaultName && 
                	!factory.containsBean(name)) {
				autowiredBeanNames = new LinkedHashSet<>();
				resource = beanFactory.resolveDependency(descriptor, requestingBeanName, autowiredBeanNames, null);
				if (resource == null) {
                    //如果返回时 resource , 则表明, spring容器中找不到, 也创建不了 这个依赖的bean
                    // 名字找不到，类型也找不到
					throw new NoSuchBeanDefinitionException(element.getLookupType(), "No resolvable resource object");
				}
			}
			else {
                // factory.containsBean(name) & resolveBeanByName 通过BeanName查找Bean
				resource = beanFactory.resolveBeanByName(name, descriptor);
				autowiredBeanNames = Collections.singleton(name);
			}
		}
		else {
            // 通过BeanName查找Bean
			resource = factory.getBean(name, element.lookupType);
			autowiredBeanNames = Collections.singleton(name);
		}

		if (factory instanceof ConfigurableBeanFactory) {
			ConfigurableBeanFactory beanFactory = (ConfigurableBeanFactory) factory;
			for (String autowiredBeanName : autowiredBeanNames) {
				if (requestingBeanName != null && beanFactory.containsBean(autowiredBeanName)) {
					beanFactory.registerDependentBean(autowiredBeanName, requestingBeanName);
				}
			}
		}

		return resource;
	}
    
    
    class ResourceElement{
        protected Object getResourceToInject(Object target, @Nullable String requestingBeanName) {
			return (this.lazyLookup ? buildLazyResourceProxy(this, requestingBeanName) :
					getResource(this, requestingBeanName));
		}
    }
    
    protected Object getResource(LookupElement element, @Nullable String requestingBeanName)
			throws NoSuchBeanDefinitionException {

		if (StringUtils.hasLength(element.mappedName)) {
            // jndiFactory.getBean下一个环节就是doGetBean()，然后开始循环
			return this.jndiFactory.getBean(element.mappedName, element.lookupType);
		}
		if (this.alwaysUseJndiLookup) {
			return this.jndiFactory.getBean(element.name, element.lookupType);
		}
		if (this.resourceFactory == null) {
			throw new NoSuchBeanDefinitionException(element.lookupType,
					"No resource factory configured - specify the 'resourceFactory' property");
		}
        // 处理@Resource
		return autowireResource(this.resourceFactory, element, requestingBeanName);
	}
}

class InjectionMetadata {
    class InjectedElement {
       protected void inject(Object target, @Nullable String requestingBeanName, @Nullable PropertyValues pvs) throws Throwable {
            if (this.isField) {
                Field field = (Field) this.member;
                ReflectionUtils.makeAccessible(field);
                // 在CommonAnnotationBeanPostProcessor.ResourceElement.getResourceToInject
                field.set(target, getResourceToInject(target, requestingBeanName));
            }
            else {
                if (checkPropertySkipping(pvs)) {
                    return;
                }
                try {
                    Method method = (Method) this.member;
                    ReflectionUtils.makeAccessible(method);
                    method.invoke(target, getResourceToInject(target, requestingBeanName));
                }
                catch (InvocationTargetException ex) {
                    throw ex.getTargetException();
                }
            }
    	} 
    }
}
```

- **AutowiredAnnotationBeanPostProcessor.postProcessProperties：处理@Autowire / @Value**

```java
class AutowiredAnnotationBeanPostProcessor {
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
		InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
		try {
			metadata.inject(bean, beanName, pvs);
		}
		catch (BeanCreationException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
		}
		return pvs;
	}
    
    
    class AutowiredFieldElement {
        protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
			Field field = (Field) this.member;
			Object value;
			if (this.cached) {
				value = resolvedCachedArgument(beanName, this.cachedFieldValue);
			}
			else {
				DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
				desc.setContainingClass(bean.getClass());
				Set<String> autowiredBeanNames = new LinkedHashSet<>(1);
				Assert.state(beanFactory != null, "No BeanFactory available");
                 //先根据 type 进行匹配, 当有多个type时, 在根据name进行匹配, 如果匹配不上则抛出异常
				TypeConverter typeConverter = beanFactory.getTypeConverter();
				try {
					value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
				}
				catch (BeansException ex) {
					throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
				}
				synchronized (this) {
					if (!this.cached) {
						if (value != null || this.required) {
							this.cachedFieldValue = desc;
							registerDependentBeans(beanName, autowiredBeanNames);
							if (autowiredBeanNames.size() == 1) {
								String autowiredBeanName = autowiredBeanNames.iterator().next();
								if (beanFactory.containsBean(autowiredBeanName) &&
										beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
									this.cachedFieldValue = new ShortcutDependencyDescriptor(
											desc, autowiredBeanName, field.getType());
								}
							}
						}
						else {
							this.cachedFieldValue = null;
						}
						this.cached = true;
					}
				}
			}
			if (value != null) {
				ReflectionUtils.makeAccessible(field);
				field.set(bean, value);
			}
		}
    }
}
```

##### 2.1.3.8 initializeBean：初始化Bean，包含前置增强和后置增强

applyBeanPostProcessorsBeforeInitialization 

-> invokeInitMethods 

-> applyBeanPostProcessorsAfterInitialization

```java
abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory{
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {

        // 。。。
        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            // 前置增强
            wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        }

        try {
            // 初始化
            invokeInitMethods(beanName, wrappedBean, mbd);
        }
        catch (Throwable ex) {
            // 。。。
        }
        if (mbd == null || !mbd.isSynthetic()) {
            // 后置增强
            wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        }

        return wrappedBean;
	}
}
```

### 2.2 循环依赖

- 产生原因： 相互引用/注入，形成闭环

- 解决方案：Spring官网建议用set注入，并且Bean是**Singleton(单例的Bean会存在三级缓存中)**

- 三级缓存：DefaultSingletonBeanRegistry

  - 第一级：singletonObjects存放已经经历了完整生命周期的Bean对象

    ```
    Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256)
    ```

  - 第二级：earlySingletonObjects，存放早期暴露出来的bean对象，Bean的生命周期未结束(属性还未填充完成)

    ```
    Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
    ```

  - 第三级：singletonFactories，存放可以生成bean的工厂

    ```
    Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
    ```

  三级缓存： new A(new B)

  1. new A() OK，将A放在三级缓存③中，填充A属性时，发现A依赖B,因此需要实例化B（getBean(B)）
  2. new B() OK，将B放在三级缓存③中，填充B属性时，发现B依赖A，因此又需要实例化A（getBean(A)）
  3. new A()时，发现getSingletion()可拿到三级缓存③中的A(顺便把A从③调整为②)，直接return A给B填充属性
  4. B填充的A属性后，B初始化结束，B直接addSingleton，即添加到一级缓存①，从②/③中抹去B
  5. B初始化完毕，return B给A填充属性
  6. A填充的B属性后，A初始化结束，A直接addSingleton，即添加到一级缓存①，从②/③中抹去B

- 执行过程

  - A创建前，先去一级缓存(**getSingleton()->singletonObjects**)中找A是否已创建，否则再创建(**doCreateBean**)过程中需要B(**填充属性populateBean()->applyPropertyValues()->valueResolver.resolveValueIfNecessary()->beanFactory.getBean()**)，于是先将A放在三级缓存singletonFactories里面(**addSingletonFactory()**)，然后去实例化B valueResolver.resolveValueIfNecessary
  - B实例化的时候发现需要A，然后就是上面一样的逻辑。然后A和B都在三级缓存了
  - 然后A又需要走CreateBean逻辑，一开始直接getSingleton()去查找，发现A已经在一级缓存，直接删除A然后put进二级缓存(earlySingletonObjects)，因为马上找到，所以不走第一步的逻辑，而是直接return回去，BeanB.setBeanA(return Obj)(**bw.setPropertyValues(new MutablePropertyValues(deepCopy))**。
  - B创建后，将B直接从三级缓存删除，添加到一级缓存(**addSingleton()->singletonObjects**)
  - 同理A也setB，然后从二级缓存删除，添加到一级缓存(**addSingleton()->singletonObjects**)
  - 然后配置文件读到B的创建，getSingleton发现B已经在一级缓存，直接Return

- Spring循环依赖三级缓存和四大方法

![1618498970531](E:\SoftwareNote\面试准备\Spring\img\Spring循环依赖三级缓存和四大方法.png)

- Spring循环依赖三级缓存

  ![1618919458103](E:\SoftwareNote\面试准备\Spring\img\Spring循环依赖三级缓存.png)


## 3. Bean的作用域

- singleton：单例
- prototype：原型。lazy，调用时创建。
- request：每次Http请求都会创建一个
- session：同一个Http Session共享

## 4. Spring事务的传播属性和隔离级别 

### 4.1 传播属性（@Transactional(propagation=Propagation.REQUIRES_NEW)）

|      传播属性      |                             描述                             |
| :----------------: | :----------------------------------------------------------: |
| **REQUIRED(默认)** | 如果有事务在运行，当前方法就加入这个事务运行；否则，另起一个新事务运行。 |
|  **REQUIRES_NEW**  | 当前的事务**必须启动新的事务**，如果有事务正常运行(外层调用)，**则挂起原事务** |
|      SUPPORTS      |           如果有事务则加入运行；**否则就不起事务**           |
|   NOT_SUPPROTED    |        **当前事务不可在事务中运行**；有事务则挂起事务        |
|     MANDATORY      |        当前**必须事务内运行**，没有事务**则抛出异常**        |
|       NEVER        | 当前**不可**在事务内运行，**有事务则抛出异常**。(MANDATORY反例) |
|       NESTED       | 如果有事务，则在**事务的嵌套事务运行**；否则起一个新事物运行。 |

### 4.2 隔离级别

**一个事务与其他事务的隔离程度成为隔离级别。**

SQL标准中定义不同隔离级别，**隔离级别越高，数据一致性越好，但并发性越弱**。

- **读未提交（READ UnCommitted）**

  事务1可读取事务2**未提交**的修改。 

- **读已提交（READ Committed）**（**Oracle**的默认隔离级别，**开始时常用**）

  事务1**只能**读事务2**已提交**的修改

- **可重复读（RePeatable READ）** （**MySQL**的默认隔离级别）

  **事务1执行期间，禁止其他事务修改这个字段。**

- **串行化（Serializable）**

  （类似表锁）在事务1执行期间，禁止其他事务修改这个表。这样可避免任何并发问题，然鹅性能十分底下。

> **隔离级别能力大赏**
>
> |                  | 脏读 | 不可重复读 | 幻读 |
> | :--------------: | :--: | :--------: | :--: |
> | READ uncommitted |  有  |     有     |  有  |
> |  READ committed  |  无  |     有     |  有  |
> | RePeatable READ  |  无  |     无     |  有  |
> |   Serializable   |  无  |     无     |  无  |

> **MySQL和Oracle对隔离级别的支持**
>
> |                  | Oracle  |  MySQL  |
> | :--------------: | :-----: | :-----: |
> | READ uncommitted |    ×    |    √    |
> |  READ committed  | √(默认) |    √    |
> | RePeatable READ  |    ×    | √(默认) |
> |   Serializable   |    √    |    √    |

### 4.3 数据库事务并发问题

- 脏读

  事务1还未提交(或者回滚)，事务2读到的是错误值

- 不可重复读

  事务1修改并提交，事务2前后2次读到的数据不一致(数据修改)

- 幻读

  事务1两次读取，事务2在两次中间插入的数据，事务1的两次读取不一致(多数据).

## 5. 乱码问题

### 5.1 SpringMVC

- Post请求乱码

  配置上Filter拦截器 extends CharacterEncodingFilter, encoding=UTF-8,forceRequestEncoding=true

- Get请求乱码

  Tomcat配置文件server.xml

  \<Connector URIEncoding="UTF-8" />

## 6. SpringMVC

### 6.1 SpringMVC工作流程

1. 客户端发送请求，请求到**达中央处理器（DispatcherServlet）**--ModelAndView的处理

   `DispatcherServlet.doDispatch(HttpServletRequest, HttpServletResponse)`

2. DispatcherServlet调用**处理器映射器**找到处理器**HandlerExcutionChain(url/controller映射关系)**

   ```java
   // Determine handler for the current request.
   // 根据当前请求 @RequestMapping("/getAll"),获得对应的MappingHandler(就是url和controller的映射关系，注解或者xml配置)
   // getAll -> com.codeman.mall4springcloud.controller.MallGoodsController#getAll()
   HandlerExcutionChain mappedHandler = getHandler(processedRequest);
   ```

3. 通过HandlerExcutionChain**找到处理器适配器 HandlerAdapter(做controller逻辑处理和返回M&V)**

   ```java
   // Determine handler adapter for the current request.
   // 
   HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
   ```

4. 通过处理器适配器**HandlerAdapter处理请求，也就是走controller逻辑，返回ModelAndView**

   ```java
   ModelAndView ha.handle(HttpServletRequest , HttpServletResponse , HandlerMethod) {
       // 通过反射调用controller逻辑
       invocableMethod.invokeAndHandle(webRequest, mavContainer);
       // 加工返回ModelAndView
       return getModelAndView(mavContainer, modelFactory, webRequest);
   }
   ```

5. DispatcherServle**t调用视图解析器**，将M&V加成成View（将model添加到request.attribute,返回视图名）

   ```java
   for (String attrName : attrsToCheck) {
       Object attrValue = attributesSnapshot.get(attrName);
       if (attrValue == null) {
           request.removeAttribute(attrName);
       }
       else if (attrValue != request.getAttribute(attrName)) {
           request.setAttribute(attrName, attrValue);
       }
   }
   ```

   6. **渲染view视图响应**给客户端。

![1621988799741](C:\Users\ASUS\AppData\Local\Temp\1621988799741.png)





