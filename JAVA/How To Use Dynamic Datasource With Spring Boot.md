- [How To Use Dynamic Datasource With Spring Boot](#how-to-use-dynamic-datasource-with-spring-boot)
  - [Create Different DataSource](#create-different-datasource)
  - [Create A DataSourceHolder For Manager DataSource](#create-a-datasourceholder-for-manager-datasource)
  - [Create DynamicDatasource Extends AbstractRoutingDataSource](#create-dynamicdatasource-extends-abstractroutingdatasource)
  - [Use AOP Swtich DataSource](#use-aop-swtich-datasource)
    - [Create A Annotation](#create-a-annotation)
    - [Create Aspect](#create-aspect)
  - [Test Class](#test-class)
  - [Refer To Doc](#refer-to-doc)

# How To Use Dynamic Datasource With Spring Boot

> Last blog we know how to use mulitpart datasource with mybatis. [mulitpart datasource](/content.html?articleId=53adfa7688184d58bbcfdae25a0316bd).But sometimes we can use dynamic datasource to query same data stucture table.

## Create Different DataSource

```java
  @Bean(name = "masterTransactionManager")
  @Primary
  public DataSourceTransactionManager masterTransactionManager() {
    return new DataSourceTransactionManager(masterDataSource());
  }
  
  @Bean(name = "secondTransactionManager")
  @Primary
  public DataSourceTransactionManager masterTransactionManager() {
    return new DataSourceTransactionManager(masterDataSource());
  }
```

## Create A DataSourceHolder For Manager DataSource

Here We used `threadlocal` to manage datasource.

```java
package com.guoxiaohei.mybatis.config.datasource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DataSourceHolder {

  private static final Logger LOGGER = LoggerFactory.getLogger(DataSourceHolder.class);

  private static ThreadLocal<String> holder = new ThreadLocal<>();


  public static void setDbType(String type) {
    holder.set(type);
  }

  public static String getDb() {
    return holder.get();
  }

  public static void remove() {
    holder.remove();
  }
}

```

## Create DynamicDatasource Extends AbstractRoutingDataSource

```java
package com.guoxiaohei.mybatis.config.datasource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

public class DynamicDatasource extends AbstractRoutingDataSource {

  private Logger logger = LoggerFactory.getLogger(this.getClass());

  @Override
  protected Object determineCurrentLookupKey() {
    String db = DataSourceHolder.getDb();
    logger.info("switch db type -> {}", db);
    return db;
  }
}

```

Registry Spring Bean. Here I created mybatis sql session for using.

```java
package com.guoxiaohei.mybatis.config.datasource;

import java.util.Map;
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages = {
    "com.guoxiaohei.mybatis.dynamic"}, sqlSessionFactoryRef = "dynamicSqlSessionFactory")
public class DynamicDataSourceConfig {


  @Bean(value = "dynamicDatasource")
  public DynamicDatasource dynamicDatasource(
      @Autowired @Qualifier("masterDataSource") DataSource masterDataSource,
      @Autowired @Qualifier("secondDataSource") DataSource secondDataSource) {
    DynamicDatasource dynamicDatasource = new DynamicDatasource();
    dynamicDatasource.setDefaultTargetDataSource(masterDataSource);
    dynamicDatasource
        .setTargetDataSources(Map.of("master", masterDataSource, "second", secondDataSource));
    return dynamicDatasource;
  }


  @Bean(name = "dynamicTransactionManager")
  @Primary
  public DataSourceTransactionManager masterTransactionManager(
      @Autowired @Qualifier("masterDataSource") DataSource masterDataSource,
      @Autowired @Qualifier("secondDataSource") DataSource secondDataSource) {
    return new DataSourceTransactionManager(dynamicDatasource(masterDataSource, secondDataSource));
  }

  @Bean(name = "dynamicSqlSessionFactory")
  @Primary
  public SqlSessionFactory masterSqlSessionFactory(
      @Qualifier("dynamicDatasource") DynamicDatasource dynamicDatasource)
      throws Exception {
    final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(dynamicDatasource);
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
        .getResources("classpath:mybatis/dynamic/*.xml"));
    return sessionFactory.getObject();
  }
}

```

Look AbstractRoutingDataSource. It implements InitializingBean and override `afterPropertiesSet` method.

```java
 @Override
 public void afterPropertiesSet() {
  if (this.targetDataSources == null) {
   throw new IllegalArgumentException("Property 'targetDataSources' is required");
  }
  this.resolvedDataSources = CollectionUtils.newHashMap(this.targetDataSources.size());
  this.targetDataSources.forEach((key, value) -> {
   Object lookupKey = resolveSpecifiedLookupKey(key);
   DataSource dataSource = resolveSpecifiedDataSource(value);
   this.resolvedDataSources.put(lookupKey, dataSource);
  });
  if (this.defaultTargetDataSource != null) {
   this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
  }
 }
```

Target DataSources will be look up via resolveSpecifiedDataSource.This Abstract class override  `getConnection` method.

```java
 @Override
 public Connection getConnection() throws SQLException {
  return determineTargetDataSource().getConnection();
 }
```

`determineTargetDataSource` called determineTargetDataSource to get connection.

```java
 protected DataSource determineTargetDataSource() {
  Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
  Object lookupKey = determineCurrentLookupKey();
  DataSource dataSource = this.resolvedDataSources.get(lookupKey);
  if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
   dataSource = this.resolvedDefaultDataSource;
  }
  if (dataSource == null) {
   throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
  }
  return dataSource;
 }
```

The key was determineCurrentLookupKey. It is a abstract method.

```java
 @Nullable
 protected abstract Object determineCurrentLookupKey();
```

So We just create a DynamicDatasource . Expend AbstractRoutingDataSource and override `determineCurrentLookupKey`

```java
package com.guoxiaohei.mybatis.config.datasource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

public class DynamicDatasource extends AbstractRoutingDataSource {

  private Logger logger = LoggerFactory.getLogger(this.getClass());

  @Override
  protected Object determineCurrentLookupKey() {
    String db = DataSourceHolder.getDb();
    logger.info("switch db type -> {}", db);
    return db;
  }
}

```

Flollow We used aop to switch different datasource.

## Use AOP Swtich DataSource

### Create A Annotation

```java
package com.guoxiaohei.mybatis.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface DataSourceSwitch {

  String name() default "master";
}
```

### Create Aspect

```java
package com.guoxiaohei.mybatis.aop;

import com.guoxiaohei.mybatis.annotation.DataSourceSwitch;
import com.guoxiaohei.mybatis.config.datasource.DataSourceHolder;
import java.lang.reflect.Method;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class DatasourceAop {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Before("@annotation(com.guoxiaohei.mybatis.annotation.DataSourceSwitch)")
  public void beforeSwitchDb(JoinPoint joinPoint) {
    Signature signature = joinPoint.getSignature();
    String dataSource = "master";
    Method method = ((MethodSignature) signature).getMethod();
    if (method.isAnnotationPresent(DataSourceSwitch.class)) {
      DataSourceSwitch annotation = method.getAnnotation(DataSourceSwitch.class);
      dataSource = annotation.name();
    }
 // Before method was invoked, the datasource name will set to holder
 // Then AbstractRoutingDataSource#getConnection will be invoked and lookup datasource we are specify.
    DataSourceHolder.setDbType(dataSource);
  }

  @After("@annotation(com.guoxiaohei.mybatis.annotation.DataSourceSwitch)")
  public void afterSwitchDS(JoinPoint point) {
    DataSourceHolder.remove();
  }
}
```

## Test Class

```java
package com.guoxiaohei.mybatis.api;

import com.guoxiaohei.mybatis.annotation.DataSourceSwitch;
import com.guoxiaohei.mybatis.dynamic.DynamicArticleMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("api")
public class MybatisDynamicDatasourceApi {

  @Autowired
  private DynamicArticleMapper articleMapper;


 //12
  @GetMapping("dynamicMaster")
  @DataSourceSwitch()
  public int articles() {
    return articleMapper.count();
  }


  // 10
  @GetMapping("dynamicSecond")
  @DataSourceSwitch(name = "second")
  public int secondArticles() {
    return articleMapper.count();
  }
}

```

## Refer To Doc

[Spring Boot + Mybatis多数据源和动态数据源配置](https://zhuanlan.zhihu.com/p/47204637)
