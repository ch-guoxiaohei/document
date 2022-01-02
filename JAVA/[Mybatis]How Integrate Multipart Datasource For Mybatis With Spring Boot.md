
- [How Integrate Multipart Datasource  For Mybatis With Spring Boot](#how-integrate-multipart-datasource--for-mybatis-with-spring-boot)
  - [Config Two Datasource In Your Properties/YML](#config-two-datasource-in-your-propertiesyml)
  - [Add Configration For SqlSession](#add-configration-for-sqlsession)
  - [Test](#test)
  - [refer post](#refer-post)

# How Integrate Multipart Datasource  For Mybatis With Spring Boot

> This Article will share use mybatis with mulitpart datasource in springboot . But I didn't have different database. So I will copy h2 datasource to another to simulation it.

## Config Two Datasource In Your Properties/YML

You can config different datasource in your config file as below. Here I will use YML.

```yaml
spring:
  master:
    driver-class-name: org.h2.Driver
    username: sa
    password: sa
    jdbc-url: jdbc:h2:~/h2-markdown;SCHEMA=markdowneditor
  second:
    driver-class-name: org.h2.Driver
    username: sa
    password: sa
    jdbc-url: jdbc:h2:~/h2-markdown2;SCHEMA=markdowneditor
```

## Add Configration For SqlSession

Datasource One.

```java
package com.guoxiaohei.mybatis.config;

import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages = {
    "com.guoxiaohei.mybatis.master"}, sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MybatisMasterConfig {

  @Bean(name = "masterDataSource")
  @Primary
  @ConfigurationProperties(prefix = "spring.master")
  public DataSource masterDataSource() {
    // ①
    return DataSourceBuilder.create().build();
  }


  @Bean(name = "masterTransactionManager")
  @Primary
  public DataSourceTransactionManager masterTransactionManager() {
    return new DataSourceTransactionManager(masterDataSource());
  }

  @Bean(name = "masterSqlSessionFactory")
  @Primary
  public SqlSessionFactory masterSqlSessionFactory(
      @Qualifier("masterDataSource") DataSource masterDataSource)
      throws Exception {
    final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(masterDataSource);
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
        .getResources("classpath:mybatis/master/*.xml"));
    return sessionFactory.getObject();
  }

}

```

For Above Code ①. Because of we default use HikariDataSource pool , So Spring will default use HikariDataSourceProperties to read config.

- Build DataSource

```java
public T build() {
  DataSourceProperties<T> properties = DataSourceProperties.forType(this.classLoader, this.type);
  DataSourceProperties<DataSource> deriveFromProperties = getDeriveFromProperties();
  ...
  return dataSource;
 }
```

- Get DataSource Properties For Type
`DataSourceProperties<T> properties = DataSourceProperties.forType(this.classLoader, this.type);`

```java
static <T extends DataSource> DataSourceProperties<T> forType(ClassLoader classLoader, Class<T> type) {
   MappedDataSourceProperties<T> mapped = MappedDataSourceProperties.forType(classLoader, type);
   return (mapped != null) ? mapped : new ReflectionDataSourceProperties<>(type);
  }

 static <T extends DataSource> MappedDataSourceProperties<T> forType(ClassLoader classLoader, Class<T> type) {
   MappedDataSourceProperties<T> pooled = lookupPooled(classLoader, type);
   if (type == null || pooled != null) {
    return pooled;
   }
   return lookupBasic(classLoader, type);
  }

private static <T extends DataSource> MappedDataSourceProperties<T> lookupPooled(ClassLoader classLoader,
    Class<T> type) {
   MappedDataSourceProperties<T> result = null;
   result = lookup(classLoader, type, result, "com.zaxxer.hikari.HikariDataSource",
     HikariDataSourceProperties::new);
   result = lookup(classLoader, type, result, "org.apache.tomcat.jdbc.pool.DataSource",
     TomcatPoolDataSourceProperties::new);
   result = lookup(classLoader, type, result, "org.apache.commons.dbcp2.BasicDataSource",
     MappedDbcp2DataSource::new);
   result = lookup(classLoader, type, result, "oracle.ucp.jdbc.PoolDataSourceImpl",
     OraclePoolDataSourceProperties::new, "oracle.jdbc.OracleConnection");
   return result;
  }
```

We can get HikariDataSourceProperties When type is HikariDataSource.So when we use below config , we just build datasource with `DataSourceBuilder.create().build();`

```java
HikariDataSourceProperties() {
   add(DataSourceProperty.URL, HikariDataSource::getJdbcUrl, HikariDataSource::setJdbcUrl);
   add(DataSourceProperty.DRIVER_CLASS_NAME, HikariDataSource::getDriverClassName,
     HikariDataSource::setDriverClassName);
   add(DataSourceProperty.USERNAME, HikariDataSource::getUsername, HikariDataSource::setUsername);
   add(DataSourceProperty.PASSWORD, HikariDataSource::getPassword, HikariDataSource::setPassword);
  }
```

Here is another config.

```java
package com.guoxiaohei.mybatis.config;

import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages = {
    "com.guoxiaohei.mybatis.second"}, sqlSessionFactoryRef = "secondSqlSessionFactory")
public class MybatisSecondConfig {

  @Bean(name = "secondDataSource")
  @Primary
  @ConfigurationProperties(prefix = "spring.second")
  public DataSource masterDataSource() {
    return DataSourceBuilder.create().build();
  }


  @Bean(name = "secondTransactionManager")
  @Primary
  public DataSourceTransactionManager masterTransactionManager() {
    return new DataSourceTransactionManager(masterDataSource());
  }

  @Bean(name = "secondSqlSessionFactory")
  @Primary
  public SqlSessionFactory masterSqlSessionFactory(
      @Qualifier("secondDataSource") DataSource masterDataSource)
      throws Exception {
    final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(masterDataSource);
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
        .getResources("classpath:mybatis/second/*.xml"));
    return sessionFactory.getObject();
  }
}

```

The Rest , we just create mybatis mapper java and mybatis mapper xml.

```java
package com.guoxiaohei.mybatis.master;

import com.guoxiaohei.mybatis.pojo.Article;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface MasterArticleMapper {

  List<Article> selectAll();
}


package com.guoxiaohei.mybatis.second;

import com.guoxiaohei.mybatis.pojo.Article;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface SecondArticleMapper {

  List<Article> selectAll();
}


```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.guoxiaohei.mybatis.master.MasterArticleMapper">

  <select id="selectAll" resultType="com.guoxiaohei.mybatis.pojo.Article">
    select c_id as id from t_post;
  </select>
</mapper>


<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.guoxiaohei.mybatis.second.SecondArticleMapper">

  <select id="selectAll" resultType="com.guoxiaohei.mybatis.pojo.Article">
    select c_id as id from t_post;
  </select>
</mapper>
```

## Test

Create a restful api to test.

```java
package com.guoxiaohei.mybatis.api;

import com.guoxiaohei.mybatis.master.MasterArticleMapper;
import com.guoxiaohei.mybatis.pojo.Article;
import com.guoxiaohei.mybatis.second.SecondArticleMapper;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("api")
public class MybatisTest {

  @Autowired
  private MasterArticleMapper articleMapper;

  @Autowired
  private SecondArticleMapper secondArticleMapper;

  @GetMapping("master")
  public List<Article> articles() {
    return articleMapper.selectAll();
  }


  @GetMapping("second")
  public List<Article> secondArticles() {
    return secondArticleMapper.selectAll();
  }

}

```

## refer post

[Spring Boot + Mybatis多数据源和动态数据源配置](https://zhuanlan.zhihu.com/p/47204637)