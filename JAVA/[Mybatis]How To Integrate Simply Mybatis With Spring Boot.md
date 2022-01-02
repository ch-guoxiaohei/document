- [How To Integrate Simply Mybatis With Spring Boot](#how-to-integrate-simply-mybatis-with-spring-boot)
  - [add mybatis-starter](#add-mybatis-starter)
  - [Add Below Config To Your YML File](#add-below-config-to-your-yml-file)
  - [Add Mapper Scan Config](#add-mapper-scan-config)
  - [Add Pojo , Mapper Class And Mapper Xml](#add-pojo--mapper-class-and-mapper-xml)
  - [Test Query](#test-query)

# How To Integrate Simply Mybatis With Spring Boot

## add mybatis-starter

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.2.0</version>
    </dependency>
```

[mybatis-starter-repo](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/2.2.0 "mybatis-starter-repo")

## Add Below Config To Your YML File

```yaml
mybatis:
  # xml path 
  mapper-locations: classpath*:mybatis/*.xml
  configuration:
    # if you didn't set your log lever to debug, you can set below config to print mybatis log
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    log-prefix: mybatis-
```

## Add Mapper Scan Config

Create a Class and marked it with `@Configuration`. You should be specify mappe scan package names.

```java
package com.guoxiaohei.markdown.config;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = {"com.guoxiaohei.markdown.mybatis"})
public class MybatisConfig {

}

```

## Add Pojo , Mapper Class And Mapper Xml

For this post , we have a pojo class named `Article`.

- Article

```java
package com.guoxiaohei.markdown.model.projo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Article {

  private String id;
}

```

- Article Mapper Class

We used `Mapper` marked this class.

```java
package com.guoxiaohei.markdown.mybatis;

import com.guoxiaohei.markdown.model.projo.Article;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface ArticleMapper {

  List<Article> selectAll();
}

```

- Create a Mapper xml in `classpath*:mybatis`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <!-- this namespace is above Class -->
<mapper namespace="com.guoxiaohei.markdown.mybatis.ArticleMapper">

  <select id="selectAll" resultType="com.guoxiaohei.markdown.model.projo.Article">
    select c_id as id from t_post;
  </select>
</mapper>
```

## Test Query

I created a rest api for testing .

```java
package com.guoxiaohei.markdown.api;

import com.guoxiaohei.markdown.model.projo.Article;
import com.guoxiaohei.markdown.mybatis.ArticleMapper;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("api/mybatis")
public class MybatisTest {

  @Autowired
  private ArticleMapper articleMapper;

  @GetMapping
  public List<Article> articles(){
    return articleMapper.selectAll();
  }

}

```

Get Result as blow

```json
[  {
    "id": "f75e0e42c89d415a8cc27d508ecd478a",
    "title": null,
    "overview": null,
    "content": null,
    "originContent": null,
    "categoryId": null,
    "userId": null,
    "createTime": null,
    "updateTime": null
  }
]
```

Print Sql Log As Below

```text
JDBC Connection [HikariProxyConnection@1038839819 wrapping conn0: url=jdbc:h2:~/h2-markdown user=SA] will not be managed by Spring
==>  Preparing: select c_id as id from t_post;
==> Parameters: 
<==    Columns: ID
<==        Row: f75e0e42c89d415a8cc27d508ecd478a
<==      Total: 1
```
