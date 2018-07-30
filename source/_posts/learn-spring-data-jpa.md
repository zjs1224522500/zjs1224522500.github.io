---
title: 'SpringData-JPA'
date: 2018-05-06 20:52:13
tags: Spring
---
## SpringData-JPA

> - 此篇博文主要是结合官方文档以及对JPA的一些使用进行记录。
> - 对于相关介绍，可参考相应博客。
> - 后续会对相关实现机制和核心思想进行深入探讨。

#### 参考书目：
- **汪云飞：JavaEE开发的颠覆者 Spring Boot实战**

#### 参考博客：

- [乐百川：Spring Data JPA 介绍和使用
](https://www.jianshu.com/p/633922bb189f)
- [Javahih：Spring data jpa入门教程
](https://www.jianshu.com/p/5592369fc536)

#### 参考文档：
- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html)

<!-- more -->

### 相关使用
#### 1、引入依赖
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <!-- 版本号省略，maven相关不过多介绍 -->
</dependency>
```
#### 2、建立数据访问层

> The goal of Spring Data repository abstraction is to significantly reduce the amount of boilerplate code required to implement data access layers for various persistence stores.

- 建立抽象的 **Repository或Dao** 从而减少持久层相关代码的编写（模板化的代码，重复的代码）

- **空接口、标记接口：没有包含方法声明的接口**
```java
public interface Repository<T, ID extends Serializable> {

}
```

- 以自定义  **PersonRepository** 为例： 使得 Spring 容器能对自定义的 PersonRepository 进行管理（**两种实现**）
```java
/**
 * 继承 Repository<T, ID extends Serializable> 接口使得 Spring 容器能对自定义的 PersonRepository 进行管理
 */
public interface PersonRepository extends Repository<Person, Long> {
    //定义数据访问操作的方法
}

/**
 * 除了继承 Repository 接口以外，还可以通过注解实现 Spring 容器对自定义 repository 进行管理
 */
@RepositoryDefinition(domainClass = Person.class,idClass = Long.class)
public interface PersonRepository {
    //定义数据访问操作的方法
}
```

##### Repository 的相关子类实现
###### 自带Repository
- 1)、**CrudRepository**
```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

  // Saves the given entity.
  <S extends T> S save(S entity);      

  // Returns the entity identified by the given id.
  Optional<T> findById(ID primaryKey); 

  // Returns all entities.
  Iterable<T> findAll();               

  // Returns the number of entities
  long count();                        
  
  // Deletes the given entity.
  void delete(T entity);               

  // Indicates whether an entity with the given id exists.
  boolean existsById(ID primaryKey);   

  // … more functionality omitted.
}
```

- 2)、**PagingAndSortingRepository extends CrudRepositor**y
```java
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

- 3)、**JpaRepository extends PagingAndSortingRepository**
```java

```

###### 自定义抽象 Repository 
- 选择性地暴露相关持久层数据操作的相关方法
- Repository 中关于 **null** 的处理，使用 spring 相关注解:
    - **@NonNullApi** – to be used on the package level to declare that the default behavior for parameters and return values is to not accept or produce null values. 
    - **@NonNull** – to be used on a parameter or return value that must not be null (not needed on parameter and return value where @NonNullApi applies).
    - **@Nullable** – to be used on a parameter or return value that can be null.
```java
**************************************interface MyBaseRepository****************************
// Make sure you add that annotation to all repository interfaces 
// that Spring Data should not create instances for at runtime.
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {
  // Java 8 Optinal refering http://www.importnew.com/6675.html
  // 也可以返回其他包装类型 
  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}


**************************************package-info.java**********************************
@NonNullApi
package tech.shunzi.repository;

import org.springframework.lang.NonNullApi;

**************************************interface UserRepository****************************
package tech.shunzi.repository;                                                       

import org.springframework.lang.Nullable;

public interface UserRepository extends MyBaseRepository<User, Long> {
  // test success
  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAddress);
}
```
- 关于 **Repository** 或 **项目** 中可能出现的 **SpringData** 多模块问题，所谓的 **多模块** 指的是在 **classpath** 中同时存在多个 **Spring Data** 模块，譬如 **MongoDB** 和 **JPA** 两个模块。
    - 对于 **仓库** 中如果显式继承了模块的特定接口：JpaRepository，则不存在多模块的问题。
    - 对于 **域对象domain** 中如果显式使用了模块特定的注解，如JPA的@entity，如Spring Data MongoDB/Spring
Data Elasticsearch的@document注解，也不存在多模块的问题。
- 存在多模块的问题的情况有：
    - 使用通用接口的仓库定义 **Repository** 
    - 使用有多个注解的域类型的仓库定义。 **@Entity、@Document** 同时修饰一个对象
- 用于区分仓库的方法是限制仓库的 **base packages**
```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
interface Configuration { }
```

##### 2、JPA的基本使用: 4 steps
###### 1)、建立相关接口 two steps
```java
// step 1：Declare an interface extending Repository or one of its subinterfaces and type it to the domain class and ID type that it will handle.
interface PersonRepository extends Repository<Person, Long> {

  // step 2: Declare query methods on the interface.
  List<Person> findByLastname(String lastname);
}
```
- 其中定义相应的查询方法主要有两种方式：It can derive the query from the method name directly, or by using a manually defined query.
    - **direct name**:
    - **manual define**:

###### 2)、在 Spring 中为定义好的接口创建代理实例
- step 3：Set up Spring to create proxy instances for those interfaces.
    - use xml 
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

        <jpa:repositories base-package="com.acme.repositories"/>
    
    </beans>
    ```
    - use java annotation to config
    ```java
    import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

    @EnableJpaRepositories
    class Config {}
    ```
###### 3)、获取 Repository 实例并调用相关方法
- step 4: Get the repository instance injected and use it.
```java
class SomeClient {

  private final PersonRepository repository;

  SomeClient(PersonRepository repository) {
    this.repository = repository;
  }

  void doSomething() {
    List<Person> persons = repository.findByLastname("Matthews");
  }
}
```

##### 3、定义查询方法
###### 1）、查询策略
- `CREATE`：根据查询方法名称构建，从方法名称中移除一组已知的前缀并解析该方法的其余部分。`find…By, read…By, query…By, count…By, get…By, And and Or.`
- `USE_DECLARED_QUERY`：试图找到一个声明的查询（注释或其他方式声明），如果未找到会抛出一个异常。
- `CREATE_IF_NOT_FOUND`：default.它首先查找已声明的查询，并且如果未找到已声明的查询，则会创建一个基于自定义方法名称的查询。这是默认的查找策略，因此如果不明确配置任何内容，将会使用它。它允许通过方法名称快速查询定义，还可以根据需要引入已声明的查询来自定义这些查询。

###### 2）、属性表达式
- 分辨算法会对findBy后的字段进行匹配。从右往左开始匹配。先匹配`AddressZipCode`，没有该属性，匹配失败，从右往左，匹配`Code`和`AddressZip`，匹配失败，继续匹配`Address`和`ZipCode`，匹配成功。对应的则为`Address.ZipCode`。
```Java
List<Person> findByAddressZipCode(ZipCode zipCode);
```
- **注意**：如果`Person`有`addressZip`属性，则会因为`addressZip`没有`code`属性匹配失败，此时需要使用下划线来消除歧义。**--不推荐使用**
```Java
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

###### 3）、特殊参数处理（Pageable,Sort）
```java
// A Page knows about the total number of elements and pages available. 
Page<User> findByLastname(String lastname, Pageable pageable);

// A Slice only knows about whether there’s a next Slice available 
// Slice is suitable for large result set.
Slice<User> findByLastname(String lastname, Pageable pageable);

// Can use sort parameter to sort data.
List<User> findByLastname(String lastname, Sort sort);

// List can avoid extra query for page instance,such as total amount.
List<User> findByLastname(String lastname, Pageable pageable);
```

> To find out how many pages you get for a query entirely you have to trigger an additional count query. By default this query will be derived from the query you actually trigger. 

###### 4）、限制查询结果
- **First/Top** 关键字，可以追加数字进行结果集中结果数量的限制，默认为 1。
- **Distinct** 关键字
- 结合 **Sort** 和 **First/Top** 可实现寻找 **第K大/小**。
```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

###### 5）、流式查询结果
- 利用 **Java8** 的 **Stream<T>** 作为返回结果，使用完对应的 **stream** 后需要 **close**
```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);

try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```

###### 6）、异步查询结果
- 方法调用后立即返回结果，
```Java
// java.util.concurrent.Future
@Async
Future<User> findByFirstname(String firstname);               

// Java 8 java.util.concurrent.CompletableFuture
@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

// org.springframework.util.concurrent.ListenableFuture
@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

##### 4、创建 Repo 实例
###### 1、XML 配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.springframework.org/schema/data/jpa"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/jpa
    http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

  <!-- Use filter to exclude some repo to from being instantiated. -->
  <repositories base-package="com.acme.repositories">
     <context:exclude-filter type="regex" expression=".*SomeRepository" />
  </repositories>

</beans:beans>
```

###### 2、Java Config (Recommend)
- Annotation **@Enable${store}Repositories**
```
@Configuration
@EnableJpaRepositories("com.acme.repositories")
class ApplicationConfiguration {

  @Bean
  EntityManagerFactory entityManagerFactory() {
    // …
  }
}
```

###### 3、独立使用
```java
RepositoryFactorySupport factory = … // Instantiate factory here
UserRepository repository = factory.getRepository(UserRepository.class);
```

##### Repo 的自定义实现
###### 定制个人Repo
- 1、定义接口和实现
```java
interface CustomizedUserRepository {
  void someCustomMethod(User user);
}

class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  public void someCustomMethod(User user) {
    // Your custom implementation
  }
  
  // you can add some beans according to your demand
  @Autowired
  private JdbcTemplate jdbcTemplate;
  
}

// Doing so combines the CRUD and custom functionality 
class UserRepository extends CrudRepository<User, Long>, CustomizedUserRepository {
    // Declare query methods here
}
```

> 注意： 当 `CustomRepo` 中包含和 `BaseRepo`（JPA自带Repo）相同的方法时，譬如如下例子的 `save`，`CustomRepo` 有更高的优先级，所以可以实现 自定义`Repo` 对原生的 `Repo` 的相关方法的重写。与此同时，可以结合 **泛型** 使得自定义 `Repo` 被更多的 `RepoImpl` 类继承实现。

```Java
interface CustomizedSave<T> {
  <S extends T> S save(S entity);
}

class CustomizedSaveImpl<T> implements CustomizedSave<T> {

  public <S extends T> S save(S entity) {
    // Your custom implementation
  }
}

interface UserRepository extends CrudRepository<User, Long>, CustomizedSave<User> {
}

interface PersonRepository extends CrudRepository<Person, Long>, CustomizedSave<Person> {
}
```
- 对于自定义`Repo`的相关配置，如果使用 `namespace` 相关配置，会自动检测扫描指定包下 指定格式的自定义 `Repo`，默认后缀为 `Impl`，可以通过修改相关配置进行定制。
```Java
<repositories base-package="com.acme.repository" />

<repositories base-package="com.acme.repository" repository-impl-postfix="FooBar" />
```
- 当一个 `Repo` 接口有多个实现，且位于不同的包下时， Spring 将基于 `beanName` 对 `RepoImpl`进行扫描.接口未指定对应的 `beanName` 时，默认为 **接口名 + Impl**，指定了`Component` 时，则对应扫描 `Component` 对应的 `Repo`.
> 注意：也可以在配置中自定义实例化相关 Bean 

```Java
package com.acme.impl.one;

class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}



package com.acme.impl.two;

@Component("specialCustomImpl")
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}


interface UserRepository implements CustomizedUserRepository {
    // default matches CustomizedUserRepositoryImpl
    // if @Component("specialCustomImpl"),matches CustomizedUserRepositoryImpl
}
```

###### 2、自定义基础Repo
----
###### 未完待续


#### 实体类 Entity

> 结合文档官方文档 Version 2.0.4.RELEASE,
2018-02-19

##### 1、@Entity 注解修饰实体类
