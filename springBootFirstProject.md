---
title: SpringBoot入门项目
date: 2017-03-15 12:00:00
categories: "SpringBoot"
tags: [Java,SpringBoot]
---
### SpringBoot目录结构
#### 推荐的目录结构如下(简易版本)：
```java
	com
  +- example
    +- myproject
      +- Application.java
      |
      +- model
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- web
      |  +- CustomerController.java
      |
```
### 一个简单的web项目：实现RESTful API
#### pom.xml文件添加
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```
#### Model:User类
```java
package com.liupx.model;
/**
 * Created by liupx on 2017/3/14.
 */
public class User {

    private Long id;
    private String name;
    private Integer age;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

#### 接口设计
![spring-boot-first-pj-restful-api.png](https://raw.githubusercontent.com/liupx/img/master/spring-boot-first-pj-restful-api.png)

#### web层controller
```java
package com.liupx.web;

import org.springframework.web.bind.annotation.*;
import java.util.List;

/**
 * Created by liupx on 2017/3/14.
 */
@RestController
@RequestMapping("/users")
public class UserController {

    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

    @GetMapping("/")
    public List<User> getUserList() {
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    @PostMapping("/")
    public String postUser(@ModelAttribute User user) {
        users.put(user.getId(), user);
        return "success";
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return users.get(id);
    }

    @PutMapping("/{id}")
    public String putUser(@PathVariable Long id, @ModelAttribute User user) {
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }

    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }

}
```
#### 使用Swagger2构建RESTful API文档
##### pom.xml文件添加依赖
```xml 
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```
#####创建Swagger2配置类
在Application.java同级创建Swagger2的配置类Swagger2。
```java
package com.liupx;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Created by liupx on 2017/3/15.
 */
@Configuration
@EnableSwagger2
public class Swagger2 {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.liupx.web"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("")
                .termsOfServiceUrl("https://www.liupx.com/")
                .contact("LIUPX")
                .version("1.0")
                .build();
    }
}

```
##### 修改web层 UserController
代码如下：
```java
package com.liupx.web;
import org.springframework.web.bind.annotation.*;
import java.util.List;

/**
 * Created by liupx on 2017/3/14.
 */
@RestController
@RequestMapping("/users")
public class UserController {

    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

	@ApiOperation(value="获取用户列表", notes="")
    @GetMapping("/")
    public List<User> getUserList() {
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    
    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @PostMapping("/")
    public String postUser(@ModelAttribute User user) {
        users.put(user.getId(), user);
        return "success";
    }

    
    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path")
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return users.get(id);
    }

    
    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @PutMapping("/{id}")
    public String putUser(@PathVariable Long id, @ModelAttribute User user) {
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }

    
    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path")
    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }

}
```
##### 启动Spring Boot，访问：[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
> 事实上,可以使用 Swagger2 进行 RESTful API 的测试。如下图：

![spring-boot-first-pj-swagger-ui.png](https://raw.githubusercontent.com/liupx/img/master/spring-boot-first-pj-swagger-ui.png)

#### SpringBoot中使用JPA(Java Persistence API)简化数据库访问
##### pom.xml文件添加数据源和依赖
```xml
<!-- 正式使用MYSQL数据库 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.21</version>
		</dependency>
<!--使用Spring-data-jpa简化数据库访问-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
```
##### 在 application.properties 中配置数据源
```
spring.datasource.url=jdbc:mysql://localhost:3306/spboot
spring.datasource.username=test
spring.datasource.password=test
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.properties.hibernate.hbm2ddl.auto=update
```
其中 spring.jpa.properties.hibernate.hbm2ddl.auto=update 意味着hibernate时根据model类更新表时不会删除原来的数据。

- validate	
	- 加载hibernate时，验证创建数据库表结构
- create
	- 每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。
- create-drop
	- 加载hibernate时创建，退出是删除表结构
- update
	- 加载hibernate自动更新数据库结构

**出处**
>这个是Hibernate的一个配置属性。<br>
> **在 SessionFactory 创建时，自动检查数据库结构，或者将数据库 schema 的 DDL 导出到数据库。使用 create-drop 时，在显式关闭 SessionFactory 时，将删除掉数据库 schema。
例如：validate | update | create | create-drop**<br>
详情见：[Hibernate文档(3.5)](https://docs.jboss.org/hibernate/orm/3.5/reference/zh-CN/html/session-configuration.html)

##### 创建 User 实体类
```java
package com.liupx.model;

import javax.persistence.*;

/**
 * Created by liupx on 2017/3/14.
 */
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

    public User(){}

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public User(Long id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```
##### 创建数据访问接口 
```java
package com.liupx.model;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

/**
 * Created by liupx on 2017/3/15.
 */
public interface UserRepository extends JpaRepository<User, Long> {

    User findById(Long id);

    User findByName(String name);

    User findByNameAndAge(String name, Integer age);

    @Query("from User u where u.name=:name")
    User findUser(@Param("name") String name);
    
}

```
##### 修改web层Controller
```java
package com.liupx.web;

import com.liupx.model.User;
import com.liupx.model.UserRepository;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * Created by liupx on 2017/3/14.
 */
@RestController
@RequestMapping("/users")
public class UserController {
    //static Map<Long,User> users = Collections.synchronizedMap(new HashMap<Long, User>());
    @Autowired
    private UserRepository userRepository;

    @ApiOperation(value="获取用户列表", notes="")
    @GetMapping("/")
    public List<User> getUserList(){
        List<User> ret = userRepository.findAll();
        return ret;
    }

    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @PostMapping("/")
    public String postUser(@ModelAttribute User user){

        /*
         * 当 User的id注解为 @GeneratedValue（默认为AUTO,共四种模式TABLE,SEQUENCE,IDENTITY,AUTO;）时
         *      id 自动生成 参数user中的id无效
         *      详细介绍JPA的主键生成器策略： https://my.oschina.net/u/263874/blog/529964
         * */
            userRepository.save(user);

        return "success";
    }

    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path")
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id){
        return userRepository.findById(id);
    }

    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @PutMapping("/{id}")
    public String putUser(@PathVariable Long id,@ModelAttribute User user){
        User u = userRepository.findById(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        userRepository.save(u);
        return "success";
    }

    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long",paramType = "path")
    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable Long id){
        userRepository.delete(id);
        return "success";
    }
}

```