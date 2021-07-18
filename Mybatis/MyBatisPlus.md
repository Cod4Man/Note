# MyBatisPlus

## 1. 整合Druid

### 1.1 异常java.sql.SQLFeatureNotSupportedException

- 原因：版本冲突
- 解决方案： 
  - Druid 1.1.21修复了对MyBatisPlus3.2以上版本的支持
  - 将mybatis-plus的版本降至3.2.0或以下 
  - 修改LocalDateTime为Date

## 2. 注解们 

### 2.1 主键策略@TableId

> @TableId

设置主键映射，value 映射主键字段名

type 设置主键类型，主键的生成策略，

```java
AUTO(0),
NONE(1),
INPUT(2),
ASSIGN_ID(3),
ASSIGN_UUID(4),
/** @deprecated 过时，相当于ASSIGN_ID */
@Deprecated
ID_WORKER(3),
/** @deprecated 过时，相当于ASSIGN_ID */
@Deprecated
ID_WORKER_STR(3),
/** @deprecated 过时，相当于ASSIGN_UUID*/
@Deprecated
UUID(4);
```

| 值          | 描述                              |
| ----------- | --------------------------------- |
| AUTO        | 数据库自增                        |
| NONE        | MP set 主键，雪花算法实现         |
| INPUT       | 需要开发者手动赋值                |
| ASSIGN_ID   | MP 分配 ID，Long、Integer、String |
| ASSIGN_UUID | 分配 UUID，Strinig                |

INPUT 如果开发者没有手动赋值，则数据库通过自增的方式给主键赋值，如果开发者手动赋值，则存入该值。

AUTO 默认就是数据库自增，开发者无需赋值，就算赋值还是自增。

ASSIGN_ID MP 自动赋值，雪花算法。

ASSIGN_UUID 主键的数据类型必须是 String，自动生成 UUID 进行赋值

### 2.2 @TableField (公共字段填充MetaObjectHandler)

- 映射非主键字段，
  - value 映射字段名，
  - select 表示是否查询该字段，
  - exist 表示是否为数据库字段 false，
  - fill 表示是否自动填充，将对象存入数据库的时候，由 MyBatis Plus 自动给某些字段赋值，create_time、update_time @TableField(fill = FieldFill.INSERT_UPDATE / fill = FieldFill.INSERT)

  ```java
  // 自动填充策略
  @Component
  public class MyMetaObjectHandler implements MetaObjectHandler {
      @Override
      public void insertFill(MetaObject metaObject) {
          this.setFieldValByName("createTime",new Date(),metaObject);
          this.setFieldValByName("updateTime",new Date(),metaObject);
      }
  
      @Override
      public void updateFill(MetaObject metaObject) {
          this.setFieldValByName("updateTime",new Date(),metaObject);
      }
  }
  ```

### 2.3 乐观锁 @Version 

标记乐观锁，通过 version 字段来保证数据的安全性，当修改数据的时候，会以 version 作为条件，当条件成立的时候才会修改成功。

version = 2

线程 1:update ... set version = 2  where version = 1

线程2 ：update ... set version = 2 where version = 1

1、数据库表添加 version 字段，默认值为 1

2、实体类添加 version 成员变量，并且添加 @Version 

3    配置类

```java
@Configuration
public class MyBatisPlusConfig {
    
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInterceptor();
    }
    
}
```

### 2.4 @EnumValue

1、通用枚举类注解，将数据库字段映射成实体类的枚举类型成员变量

```java
package com.southwind.mybatisplus.enums;

import com.baomidou.mybatisplus.annotation.EnumValue;

public enum StatusEnum {
    WORK(1,"上班"),
    REST(0,"休息");

    StatusEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    @EnumValue
    private Integer code;
    private String msg;
}
```

```java
package com.southwind.mybatisplus.entity;

import com.baomidou.mybatisplus.annotation.*;
import com.southwind.mybatisplus.enums.StatusEnum;
import lombok.Data;

import java.util.Date;

@Data
@TableName(value = "user")
public class User {
    @TableId
    private String id;
    @TableField(value = "name",select = false)
    private String title;
    private Integer age;
    @TableField(exist = false)
    private String gender;
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
    @Version
    private Integer version;
    private StatusEnum status;
}
```

application.yml

```yaml
type-enums-package: 
  com.southwind.mybatisplus.enums
```

2、实现接口

```java
package com.southwind.mybatisplus.enums;

import com.baomidou.mybatisplus.core.enums.IEnum;

public enum AgeEnum implements IEnum<Integer> {
    ONE(1,"一岁"),
    TWO(2,"两岁"),
    THREE(3,"三岁");

    private Integer code;
    private String msg;

    AgeEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    @Override
    public Integer getValue() {
        return this.code;
    }
}
```

### 2.5 逻辑删除@TableLogic

1、数据表添加 deleted 字段

2、实体类添加注解@TableLogic

3    配置文件

```yml
global-config:
  db-config:
    logic-not-delete-value: 0  # 默认是0，未删除
    logic-delete-value: 1 # 删除改为1
```

## 3. 开启sql日志

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 4. 代码生成

## 5. MP启动时SQL生成原理(springboot-start)

> 1. spring.factories

```tex
# Auto Configure
org.springframework.boot.env.EnvironmentPostProcessor=\
  com.baomidou.mybatisplus.autoconfigure.SafetyEncryptProcessor
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.baomidou.mybatisplus.autoconfigure.MybatisPlusLanguageDriverAutoConfiguration,\
  com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration
```

> 2. MybatisPlusAutoConfiguration

```java
// 因此，springboot启动时，就会加载该配置类
@Configuration
public class MybatisPlusAutoConfiguration implements InitializingBean {
    
    // 该配置类创建了org.apache.ibatis.session.SqlSessionFactory对象
    @Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        // TODO 使用 MybatisSqlSessionFactoryBean 而不是 SqlSessionFactoryBean
        MybatisSqlSessionFactoryBean factory = new MybatisSqlSessionFactoryBean();
        // ..
        return factory.getObject();
    }
}

public class MybatisSqlSessionFactoryBean implements FactoryBean<SqlSessionFactory> {
    @Override
    public SqlSessionFactory getObject() throws Exception {
        if (this.sqlSessionFactory == null) {
            afterPropertiesSet();
        }

        return this.sqlSessionFactory;
    }
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // ..
        this.sqlSessionFactory = buildSqlSessionFactory();
    }
    
    protected SqlSessionFactory buildSqlSessionFactory() {
        // ...
        XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
                            targetConfiguration, mapperLocation.toString(), targetConfiguration.getSqlFragments());
                        xmlMapperBuilder.parse();
        // ...
    }
}

class XMLMapperBuilder {
    public void parse() {
        if (!configuration.isResourceLoaded(resource)) {
          configurationElement(parser.evalNode("/mapper"));
          configuration.addLoadedResource(resource);
            // 这里
          bindMapperForNamespace();
        }

        parsePendingResultMaps();
        parsePendingCacheRefs();
        parsePendingStatements();
  }
    
    private void bindMapperForNamespace() {
        String namespace = builderAssistant.getCurrentNamespace();
        if (namespace != null) {
          Class<?> boundType = null;
          try {
            boundType = Resources.classForName(namespace);
          } catch (ClassNotFoundException e) {
            // ignore, bound type is not required
          }
          if (boundType != null && !configuration.hasMapper(boundType)) {
            // Spring may not know the real resource name so we set a flag
            // to prevent loading again this resource from the mapper interface
            // look at MapperAnnotationBuilder#loadXmlResource
            configuration.addLoadedResource("namespace:" + namespace);
             // 这里做的就是往configuration增加该Mapper接口组装的SQL 
            configuration.addMapper(boundType);
      }
    }
  }
}

```

> 3. MybatisMapperAnnotationBuilder获取SQL注入器

```java
class MybatisMapperAnnotationBuilder {
    public void parse() {
        if (GlobalConfigUtils.isSupperMapperChildren(configuration, type)) {
                        parserInjector();
        }
    }
    
    void parserInjector() {
        // 获取SQL注入器
        GlobalConfigUtils.getSqlInjector(configuration).inspectInject(assistant, type);
    }
}

class AbstractSqlInjector {
    
    void inspectInject() {
        if (!mapperRegistryCache.contains(className)) {
            	// 获取该Mapper接口 mapperClass的所有抽象方法（即接口定义的方法）
                List<AbstractMethod> methodList = this.getMethodList(mapperClass);
                if (CollectionUtils.isNotEmpty(methodList)) {
                    TableInfo tableInfo = TableInfoHelper.initTableInfo(builderAssistant, modelClass);
                    // 
                    // 循环注入自定义方法
                    methodList.forEach(m -> m.inject(builderAssistant, mapperClass, modelClass, tableInfo));
                } else {
                    logger.debug(mapperClass.toString() + ", No effective injection method was found.");
                }
                mapperRegistryCache.add(className);
            }
    }
}

// SQL默认注入器定义了17个方法
public class DefaultSqlInjector extends AbstractSqlInjector {

    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        return Stream.of(
            new Insert(),
            new Delete(),
            new DeleteByMap(),
            new DeleteById(),
            new DeleteBatchByIds(),
            new Update(),
            new UpdateById(),
            new SelectById(),
            new SelectBatchByIds(),
            new SelectByMap(),
            new SelectOne(),
            new SelectCount(),
            new SelectMaps(),
            new SelectMapsPage(),
            new SelectObjs(),
            new SelectList(),
            new SelectPage()
        ).collect(toList());
    }
}
```

> 4. 为接口方法注入SQL

```java
abstract class AbstractMethod {
    
    public void inject(MapperBuilderAssistant builderAssistant, Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        this.configuration = builderAssistant.getConfiguration();
        this.builderAssistant = builderAssistant;
        this.languageDriver = configuration.getDefaultScriptingLanguageInstance();
        /* 注入自定义方法 */
        // getMethodList里面的各自实现
        injectMappedStatement(mapperClass, modelClass, tableInfo);
    }
    
    protected MappedStatement addSelectMappedStatementForTable(Class<?> mapperClass, String id, SqlSource sqlSource,
                                                               TableInfo table) {
        String resultMap = table.getResultMap();
        if (null != resultMap) {
            /* 返回 resultMap 映射结果集 */
            return addMappedStatement(mapperClass, id, sqlSource, SqlCommandType.SELECT, null,
                resultMap, null, new NoKeyGenerator(), null, null);
        } else {
            /* 普通查询 */
            return addSelectMappedStatementForOther(mapperClass, id, sqlSource, table.getEntityType());
        }
    }
    
    protected MappedStatement addMappedStatement(Class<?> mapperClass, String id, SqlSource sqlSource,
                                                 SqlCommandType sqlCommandType, Class<?> parameterType,
                                                 String resultMap, Class<?> resultType, KeyGenerator keyGenerator,
                                                 String keyProperty, String keyColumn) {
        String statementName = mapperClass.getName() + DOT + id;
        if (hasMappedStatement(statementName)) {
            logger.warn(LEFT_SQ_BRACKET + statementName + "] Has been loaded by XML or SqlProvider or Mybatis's Annotation, so ignoring this injection for [" + getClass() + RIGHT_SQ_BRACKET);
            return null;
        }
        /* 缓存逻辑处理 */
        boolean isSelect = false;
        if (sqlCommandType == SqlCommandType.SELECT) {
            isSelect = true;
        }
        // 最底层调用的是ibatis的方法，
        return builderAssistant.addMappedStatement(id, sqlSource, StatementType.PREPARED, sqlCommandType,
            null, null, null, parameterType, resultMap, resultType,
            null, !isSelect, isSelect, false, keyGenerator, keyProperty, keyColumn,
            configuration.getDatabaseId(), languageDriver, null);
    }
}

public class SelectById extends AbstractMethod {

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        SqlMethod sqlMethod = SqlMethod.SELECT_BY_ID;
        SqlSource sqlSource = new RawSqlSource(configuration, String.format(sqlMethod.getSql(),
            sqlSelectColumns(tableInfo, false),
            tableInfo.getTableName(), tableInfo.getKeyColumn(), tableInfo.getKeyProperty(),
            tableInfo.getLogicDeleteSql(true, true)), Object.class);
        return this.addSelectMappedStatementForTable(mapperClass, getMethod(sqlMethod), sqlSource, tableInfo);
    }
}
```

## 6. mapper-locations

```yaml
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
#mybatis:
#  mapperLocations: classpath:/mapper/*.xml
```

## 7. 重写SqlSessionFactory

注意将configuration改为MyBatisPlus的configuration **MybatisConfiguration** ,否则启动时基础方法注入就全部失效

```java
@Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setDefaultScriptingLanguage(MybatisXMLLanguageDriver.class);
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        sqlSessionFactoryBean.setConfiguration(configuration);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }
```

## 8. 最大分页限制

```java
public class PaginationInterceptor extends AbstractSqlParserHandler implements Interceptor {

    /**
     * 单页限制 500 条，小于 0 如 -1 不受限制
     */
    protected long limit = 500L;
    
    public Object intercept(Invocation invocation) throws Throwable {
        if (this.limit > 0 && this.limit <= page.getSize()) {
            //处理单页条数限制
            handlerLimit(page);
        }
    }
    
    // 超过500条，就会执行这个方法，limit默认是500，所以可以修改limit的值-1/或者是改page.setSize(-1)
    protected void handlerLimit(IPage<?> page) {
        page.setSize(this.limit);
    }
}
```

## 9. 逻辑删除

### 9.1 模型字段

```java
@TableField(fill = FieldFill.INSERT)
@TableLogic
private Boolean useabled;
```

### 9.2 自动填充字段MetaObjectHandler

```java
// MP公共字段自动填充策略
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("createTime", LocalDateTime.now(), metaObject);
        this.setFieldValByName("updateTime", LocalDateTime.now(), metaObject);
        //添加乐观锁默认值是1
        // this.setFieldValByName("version",1,metaObject);
        //添加逻辑删除的默认值0
        this.setFieldValByName("useabled", false, metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime", LocalDateTime.now(), metaObject);
    }
}
```

### 9.3 配置文件 yml

```yaml
mybatis-plus.global-config: 
  db-config.logic-delete-value: true
  db-config.logic-not-delete-value: false
```

### 9.4 逻辑删除插件config, （高版本无需配置插件）

```java
	/**
     * 逻辑删除的插件
     */
@Bean
public ISqlInjector sqlInjector(){
    return new LogicSqlInjector();
}
```

- 查询结果自动拼接逻辑删除条件

```tex
 ==>  Preparing: SELECT user_id,name,nickname,password,useabled,phone_num,gender,avatar_path,create_time,update_time FROM user WHERE user_id=? AND useabled=false
2021-07-18 10:22:58.433 DEBUG 3952 --- [    Test worker] c.c.b.mapper.UserMapper.selectById       : ==> Parameters: 1(Integer)
2021-07-18 10:22:58.534 DEBUG 3952 --- [    Test worker] c.c.b.mapper.UserMapper.selectById       : <==      Total: 1
```

