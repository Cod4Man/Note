# SpringBoot

## 1. 启动自动加载配置类原理@SpringBootApplication

### 1.1@SpringBootApplication包含了@EnableAutoConfiguration 

```java
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

### 1.2 @EnableAutoConfiguration

```
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

### 1.3 @Import(AutoConfigurationImportSelector.class) 

```java
class AutoConfigurationImportSelector  {
    //
    selectImports() {
        // 里面会去读META-INF/spring.factories
        // spring.factories里面定义了上百个XXXXXAutoConfiguration
        // 就是SpringBoot事先整合了上百个配置，但是都是@ConditionXXX()因此都不生效（并且没有包依赖没所以底层都是报红，等到需要用时，导入相关依赖，这个事先写好的配置就可以生效了。），而且@Configuration(proxybeansMethods=false),这样Bean就只会被创建一次，提高效率
        // 而且还实现@EnableConfigurationProperties(ServerProperties.class)定义了一些配置类，程序员可以修改默认规则的properties达到定制容器的效果。
    }
}
```

![1622477596631](C:\Users\ASUS\AppData\Local\Temp\1622477596631.png)