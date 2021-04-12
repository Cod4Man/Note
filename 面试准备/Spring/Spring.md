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

