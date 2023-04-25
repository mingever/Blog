---
title: jpa
date: 2023-04-25 20:41:40
categories:
tags:
---

## 介绍

ORM(Object-Relational Mapping, 对象关系映射)，是一种面向对象编程语言中的对象和数据库中的数据之间的映射。使用ORM工具、框架可以让应用程序操作数据库。

Java Persistence API，即java持久化api，是SUN公司推出的一套基于`ORM`的规范。JPA 通过注解描述对象-关系表的映射关系，并将运行期的实体对象持久化到数据库中 。

总的来说JPA是ORM规范，Hibernate、TopLink等是JPA规范的具体实现，开发者可以面向JPA规范进行持久层的开发，而底层的实现可以自由切换。

Spring Data JPA是Spring Data家族的一部分，可以轻松实现基于JPA的存储库，此模块处理对基于JPA的数据访问层的增强支持。其是在JPA之上添加另一层抽象（Repository层的实现），极大地简化持久层开发及ORM框架切换的成本。

JPQL查询语言：通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。
如：from Student s where s.name = ?

<img src="https://oss.mingever.com/note/springBoot/jpaIntroduction.png" style="zoom: 80%;" />

## 快速开始

**POM**

```xaml
<!--spring data jpa依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

**yaml**

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/jpaDemo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    username: root
    password: 169736
  #jpa配置
  jpa:
    hibernate:
      #自动更新
      ddl-auto: update
    #日志显示sql语句
    show-sql: true
```

## entity

```java
@Data
@AllArgsConstructor
@NoArgsConstructor

//使用jap注解配置映射关系
@Entity//告诉jps这是一个实体类(和数据表映射的类) 注意这是javax的包！不是hibernate包里的那个
@Table(name = "tbl_user") //@Table来指定和哪个数据库表对象；如果省略默认表名就是user
public class User {

    @Id //声明一个实体类的属性映射为数据库的主键列
    @GeneratedValue(strategy = GenerationType.IDENTITY) //自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //用来标识实体类中属性与数据表中字段的对应关系
    private String lastName;

    @Column
    private String email;
}
```

@Entity、@Table是javax包中 Java Persistence API中定义的注解，不是jpa、hibernate、Spring的
优先级 @Table > @Entity

如果没有@Entity、@Id这两个注解的话，该类就是一个典型的POJO类，加上这两个注解之后，就可以作为一个实体类与数据库中的表相对应

==@Table、@Column和数据库表名、字段名的映射问题和mybatis-plus基本一致，遵循驼峰==

- **@Entity**

	==不可省略==

	告诉jps这是一个实体类（和数据表映射的类）

- **@Table** 

	可以省略

	来指定和哪个数据库表对应

	当实体类名和数据库表名不同时，使用`name`属性来指定对应的数据库表

- **@Id**

	==不可省略==

	声明一个实体类的属性映射为数据库的主键列

	如果属性名和字段名不一致，可以加上==@Column==，使用其name属性来指定

	> @Entity、@Table是javax包中 Java Persistence API中定义的注解，不是jpa、hibernate、Spring的
	> 优先级 @Table > @Entity
	>
	> 如果没有@Entity、@Id这两个注解的话，该类就是一个典型的POJO类，加上这两个注解之后，就可以作为一个实体类与数据库中的表相对应

- **@GeneratedValue**

	用于标注主键的生成策略，通过`strategy`属性指定。

	默认情况下，JPA自动选择一个最适合底层数据库的主键生成策略：SqlServer对应identity，MySQL对应auto increment。

	在javax.persistence.GenerationType中定义了以下几种可供选择的策略：

	- IDENTITY：采用数据库ID自增长的方式来自增主键字段，Oracle 不支持这种方式；
	- AUTO： JPA自动选择合适的策略，是默认选项；

		* SEQUENCE：通过序列产生主键，通过@SequenceGenerator 注解指定序列名，MySql不支持这种方式
		* TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。

- **@Column**

	可以省略

	用来标识实体类中属性与数据表中字段的对应关系，使用`name`属性来解决实体类的字段名与数据库中的字段名不匹配的问题

	- `name`：映射的列名
	- `unique`：是否唯一
	- `nullable`：是否允许为空；
	- `length`：对于字符型列，length属性指定列的最大字符长度；
	- `insertable`：是否允许插入；
	- `updatetable`：是否允许更新；
	- columnDefinition：定义建表时创建此列的DDL；
	- secondaryTable：从表名。如果此列不建在主表上（默认是主表），该属性定义该列所在从表的名字

- **@OneToMany**   [关系映射 具体文档 ](https://www.jianshu.com/p/22c422694d7e)

	建立一对多的关系映射

- **@ManyToOne**

	建立多对一的关系

- **@JoinColumn**

	用于定义主键字段和外键字段的对应关系

- **@ManyToMany**

	用于映射多对多关系

- **@JoinTable**

	针对中间表的配置

## repository

dao(数据访问对象 data access object)在JPA规范中叫做`repository`

使用时：自定义接口要继承==JpaRepository==，传入泛型，第一个参数为要操作的==实体类==，第二个参数为==该实体类的主键类型==

```java
//继承JpaRepository来完成对数据库的操作
@Repository
public interface UserRepository extends JpaRepository<User,Integer>{

    User findByEmail(String email);

    List<User> findByIdAfter(Integer id);

    //JPQL不能写select * 要写select 别名
    //如果有查询条件，也只能用别名，如where u.id =1，而不是User.id =1
    @Query("select u from User u ")
    List<User> queryFindUser();

    //JPQL ? 占位符
    @Transactional
    @Modifying
    @Query("update User u set u.lastName = ?1,u.email = ?2 where u.id = ?3")
    int updateUser(String lastName, String email, Integer id);

    //JPQL ":" 赋值
    @Transactional
    @Modifying
    @Query("update User set lastName = :lastName,email = :email where id = :id")
    int updateUser1(String lastName, String email, Integer id);

    //sql原生语句
    @Transactional
    @Modifying
    @Query(value = "update tbl_user set last_name = ?1,email = ?2 where id = ?3",nativeQuery = true)
    int updateUser2(String lastName, String email, Integer id);

    /** 结果集转换：
     * 无论是原生sql还是JPQL，如果查询所有字段，无论是一个一个查还是原生sql的select * 还是JPQL的查对象
     * 结果都可以转化为Entity，也只能使用Entity，不能使用VO。如直接List<User>
     * ======================================================================================
     * 但是无论原生sql还是JPQL，如果只查部分属性的话，结果集就不能转为对象。其返回形式是，如果结果集是：
     * 单个结果，Object Map
     * 多个结果,Object[]，list<Map>
     * ===========================================================
     * 如果用List<Map>
     * 原生sql
     *  Map的key，为查出来的字段名或者是as取的别名。
     *  如果给Map添加泛型,如List<Map<String,Vo>，其Vo也不起作用，依然是原来的Object。
     * JPQL
     *  Map的key，为null 需要用as为添加别名
     * ===============================================================
     * 因此，无论是原生sql还是JPQL都建议使用as为每一个字段取别名，返回值使用List<Map<String,Object>>来接收
     * 然后使用fastJson来将List<Map<String,Object>>转换为List<Vo>，如
     *       JSONArray jsonArray = new JSONArray();
     *       jsonArray.addAll(resultList);
     *       List<UserTeacherVO> userTeacherVOList = jsonArray.toJavaList(UserTeacherVO.class);
     *       userTeacherVOList.stream().forEach(System.out::println);
     */

    @Query("select id as id,email as email,lastName as lastName from User ")
    List<Map<String,Object>> queryUser3();
    @Query(value = "select id,email,last_name from tbl_user",nativeQuery = true)
    List<Map<String,Object>> queryUser4();


    //sql原生查询，也可以用":" 赋值
    @Query(value = " select user.id as userId,user.email as userEmail,user.last_name as lastName," +
            " teacher.id as teacherId,teacher.name as teacherName" +
            " from tbl_user user,tbl_teacher teacher where user.teacher_id = teacher.id " +
            " and user.id = :id",nativeQuery = true)
    List<Map<String,Object>> queryUserTeacherVo(Integer id);

}
```

### 继承关系

- Repository

	提供了 findBy+属性 方法

- CrudRepository

	继承了Repository，添加了一组对数据的增删改查的方法

	<img src="https://oss.mingever.com/note/springBoot/CrudRepository.png" style="zoom:67%;" />

- PagingAndSortRepository

	继承了CrudRepository，添加了一组分页排序相关的方法，缺点是只能对所有的数据进行分页或者排序，不能做条件判断

	<img src="https://oss.mingever.com/note/springBoot/PagingAndSortingRepository.png" style="zoom: 80%;" />

- **JpaRepository**

	主要继承了PagingAndSortRepository，开发中经常使用的接口，添加了一组JPA规范相关的方法，

	对返回值类型做了适配，因为在父类接口中通常都返回迭代器，需要我们自己进行强制类型转化。而在JpaRepository中，直接返回了List。

	<img src="https://oss.mingever.com/note/springBoot/JpaRepositoryDepend.png" style="zoom: 50%;" />

	<img src="https://oss.mingever.com/note/springBoot/JpaRepository.png" style="zoom:67%;" />

- JpaSpecificationExecutor

	提供了多条件查询

	其单独存在，没有继承以上接口。主要提供了多条件查询的支持，并且可以在查询中添加分页和排序。因为这个接口单独存在，因此需要配合以上说的接口使用。

	<img src="https://oss.mingever.com/note/springBoot/JpaSpecificationExecutor.png" style="zoom:67%;" />

### 常用方法

**save(S): S**  相当于saveOrUpdate

- 当传入的时候不带自增主键时
	- 无论每次传入的数据是否一样，都会当做一条新的数据插入
	- 即只有一条insert语句，即直接插入一条
	- id为数据库中最后一条ID+1
- 当传入的时候带自增主键时
	- 若数据不存在或者主键不存在，则先select再insert语句
	- 此主键有数据存在，则先select再update

### Jpa方法命名规则

JpaRepository支持接口规范方法名查询，如果在接口中定义的查询方法符合它的命名规则，就可以不用写实现，目前支持的条件关键字如下

|    **Keyword**    | **Sample**                     | **JPQL**                   |
| :---------------: | ------------------------------ | -------------------------- |
|        And        | findByNameAndPwd               | where name= ? and pwd =?   |
|        Or         | findByNameOrSex                | where name= ? or sex=?     |
|     Is,Equals     | findById,findByIdEquals        | where id= ?                |
|      Between      | findByIdBetween                | where id between ? and ?   |
|     LessThan      | findByIdLessThan               | where id < ?               |
|  LessThanEquals   | findByIdLessThanEquals         | where id <= ?              |
|    GreaterThan    | findByIdGreaterThanEquals      | where id > = ?             |
|       After       | findByIdAfter                  | where id > ?               |
|      Before       | findByIdBefore                 | where id < ?               |
|      IsNull       | findByNameIsNull               | where name is null         |
| isNotNull,NotNull | findByNameNotNull              | where name is not null     |
|       Like        | findByNameLike                 | where name like ?          |
|      NotLike      | findByNameNotLike              | where name not like ?      |
|   StartingWith    | findByNameStartingWith         | where name like ‘?%’       |
|    EndingWith     | findByNameEndingWith           | where name like ‘%?’       |
|    Containing     | findByNameContaining           | where name like ‘%?%’      |
|      OrderBy      | findByIdOrderByXDesc           | where id=? order by x desc |
|        Not        | findByNameNot                  | where name <> ?            |
|        In         | findByIdIn(Collection<?> c)    | where id in (?)            |
|       NotIn       | findByIdNotIn(Collection<?> c) | where id not in (?)        |
|       True        | findByAaaTue                   | where aaa = true           |
|       False       | findByAaaFalse                 | where aaa = false          |
|    IgnoreCase     | findByNameIgnoreCase           | where UPPER(name)=UPPER(?) |
|        top        | findTop10                      | top 10/where ROWNUM <=10   |

### @Querry

在方法上标注@Query来指定本地查询

参数nativeQuery默认为false，nativeQuery=false时，value参数写的是JPQL，JPQL是用来操作model对象的
nativeQuery=true时，value参数写的是原生sql

```java
//JPQL
@Query(value="select s from Spu s where s.title like %?1" )
public List<Spu> findByTitle(String title);

//SQL
@Query(value="select * from spu s where s.title like %?1",nativeQuery=true )
public List<Spu> findByTitle(String title);

//使用@Param命名化参数
@Query(value = "select s from Spu s where s.title in (:titles)")
List<Spu> findByTitle(@Param("titles") List<String> titles);
```

### JPQL

Java Persistence Query Language是一种可移植的查询语言，是JPA的一部分，是一个平台无关的面向对象查询语言，通过类似 SQL 的语句进行 JPA 查询，这在构建动态查询时非常有用

旨在以面向对象表达式语言的表达式，将 SQL 语法和简单查询语义绑定在一起使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上的 SQL。

其特征与原生SQL语句类似，并且完全面向对象，通过类名和属性访问，而不是表名和表的属性。

HQl：Hibernate 的面向对象查询语言，它是 JPQL 的超集，一句 HQL 不可能一定是 JPQL，但一句 JPQL 一定是 HQL。它超出 JPQL 的部分，比如 额外的内置函数，各数据库方言等。

- 语法

	select 实体别名.属性名，
	from 实体名 as 实体别名 
	where 实体别名.实体属性 op 比较值

- 规则

	- 里面不能出现表名,列名,只能出现java的类名,属性名，区分大小写
	- 出现的sql关键字是一样的意思,关键字不区分大小写
	- 3.不能写select * 要写select 别名

```sql
#语法
select 实体别名.属性名，
from 实体名 as 实体别名 
where 实体别名.实体属性 op 比较值

@Transactional
@Modifying
@Query("update Customer c set c.custName = ?1 where c.custId = ?2") # ?后跟变量序号
int updateCustomer(@Param("custName") String custName, @Param("custId") Long custId);

@Transactional
@Modifying
@Query("delete from Customer c where c.custId = :custId and c.custIndustry = :custIndustry")# :后跟变量名
int deleteCustomer(@Param("custId") Long custId, @Param("custIndustry") String custIndustry);
```



## 分页

```java

```



### Pageable

- 概述

	Pageable是Spring Data库中的一个接口，主要用于构造分页查询语句。
	Pageable接口包含了分页的相关信息，使用`getPageNumber()`和`getPageSize()`，可以获取当前是第几页和每页的的数据量大小。
	在查询语句中加入Pageable对象作为参数，JPA就能够通过该Pageable对象的信息，分析生成一个带分页查询的SQL语句。

- 获取

	使用PageRequest类中的静态方法of来获取Pageable对象。
	PageRequest类继承了AbstractPageRequest类，而AbstractPageRequest类有实现了Pageable接口

	```java
	//不带排序的Pageable对象
	//public static PageRequest of(int page, int size)，
	//page为当前页数，从0开始，size为每一页元素的个数，对应limit(page,size)
	Pageable pageable = PageRequest.of(0,3);
	
	//page.next()为下一页，如(0,3) next后为下一页，为(1,3)
	Pageable pageable2 = pageable.next();
	
	//带有排序的Pageable对象，根据实体中的id属性进行倒序排序
	//public static PageRequest of(int page, int size, Sort sort)
	Pageable pageable3 = PageRequest.of(0,3, Sort.by("id").descending());
	```

### Page

Page也是Spring Data库中的一个接口，主要用于存储JPA查询数据库的结果集。
当使用Pageable对象作为参数进行查询的时候，JPA返回的是一个Page对象

```java
Page<User> page = userRepository.findAll(PageRequest.of(0,3));

//Page接口中有许多的方法可以获取，查询的结果信息
int totalPages = page.getTotalPages();             //获取总页数
long totalElements = page.getTotalElements();      //获取记录的总条数
int number = page.getNumber();			           //获取当前是第几页 等于page.getPageable().getPageNumber();
int size = page.getSize();			               //获取每页的记录条数 等于page.getPageable().getPageSize();
int  numberOfElements= page.getNumberOfElements(); //获取当前Content记录的条数
List<User> userList = page.getContent();           //获取记录的集合列表
Pageable pageable4 = page.getPageable();           //获取当前的Pageable

/**
 * PageImpl自定义分页
 * public PageImpl(List<T> content, Pageable pageable, long total)
 * content：当前页的记录。
 * total：所有记录数
 * 也就是用PageImpl实现Page接口的时候，不能把数据全取回来放入content，得按分页的size放入。而最后一个参数需要记录的是所有数据的计数。
 */
Page<User> page1 = new PageImpl<User>(userList);
Page<User> page2 = new PageImpl<User>(userList,page.getPageable(),totalElements);
int i1 = page2.getNumber();
int i2 = page2.getSize();
```

### Demo

UserRepository

```java
Page<User> findByIdAfter(Pageable pageable,Integer id);

//原生sql分页
//自带的count可能会出现问题，因此用自己写的countQuery来代替
@Query(nativeQuery = true,value =
        " select user.id as userId,user.email as userEmail,user.last_name as lastName, " +
        " teacher.id as teacherId,teacher.name as teacherName " +
        " from tbl_user user,tbl_teacher teacher" +
        " where " +
        " user.teacher_id = teacher.id " +
        " and if(:lastName is not null and :lastName != '', user.last_name = :lastName , 1=1) " +
        " and if(:teacherName is not null and :teacherName !='', teacher.name = :teacherName , 1=1) ",
        countQuery =
        " select count(*) " +
        " from tbl_user user,tbl_teacher teacher" +
        " where " +
        " user.teacher_id = teacher.id " +
        " and if(:lastName is not null and :lastName != '', user.last_name = :lastName , 1=1) " +
        " and if(:teacherName is not null and :teacherName !='', teacher.name = :teacherName , 1=1) "
)
Page<List<Map<String,Object>>> nativePage(Pageable pageable,String lastName,String teacherName);
```

Test

```java
Pageable pageable = PageRequest.of(0,3);

//默认提供的
Page<User> page1 = userRepository.findAll(pageable);

//自己在Repository里拼接的
Page<User> page2 = userRepository.findByIdAfter(pageable,2);

//Repository里@Query的
Page<List<Map<String,Object>>> nativePage = userRepository.nativePage(pageable,null,"张老师");

JSONArray jsonArray = new JSONArray();
jsonArray.addAll(nativePage.getContent());
List<UserTeacherVo> list = jsonArray.toJavaList(UserTeacherVo.class);

//demo
int totalPages;
int i=0;
List<UserTeacherVo> userTeacherVoList = new ArrayList<>();
do{
    Page<List<Map<String,Object>>> allPage = userRepository.nativePage(pageable,null,null);
    totalPages = allPage.getTotalPages();

    JSONArray jsonArray1 = new JSONArray();
    jsonArray1.addAll(allPage.getContent());
    List<UserTeacherVo> list1 = jsonArray1.toJavaList(UserTeacherVo.class);

    userTeacherVoList.addAll(list1);

    pageable = pageable.next();
    i++;
}
while (i<totalPages);

userTeacherVoList.stream().forEach(System.out::println);
```



## CURD

[增删改查具体文档](https://blog.csdn.net/fly910905/article/details/78557110)

### 查

#### 简单查询

- 继承Repository\<T,ID>即可使用jpa默认已经实现的增删改查等方法

	如`findById(ID)`，`findAll()`，`findAllById()`等

- 自定义简单查询：根据方法名来自动生成SQL

	按照jpa规范，查询方法以find、read、get开头，涉及查询条件时，条件的属性用条件关键字连接

	```java
	findByLastNameAndFirstName(String lastName,String firstName);
	//注意：条件的属性名称与个数要与参数的位置与个数一一对应，且条件属性以首字母大写
	```

- 使用@Query来指定本地查询


#### 动态查询

#### 多表查询

### 增

- 使用jpa默认已经实现的方法，`save()`、`saveAll()`等
- 使用@Query来指定本地查询

### 删

- sql方式删除

	==执行delete和update语句一样，需要添加@Modifying注解==，使用时==在Repository或者更上层需要@Transactional注解==

	```java
	Query(value = "delete from r_upa where user_id= ?1 and point_indecs_id in (?2)", nativeQuery = true)
	@Modifying
	void deleteByUserAndPointIndecs(Long uid, List<Long> hids);
	```

- 函数方式删除

	- 直接可以使用jpa已经默认实现的deleteById(id)，依据id来删除一条数据
		还有`delete(T)`，`deleteAllById()`，`deleteAll()`等
	- 也可以使用`deleteByXxx(String xxx)`根据方法名生成的函数来删除，
		但要注意：==和update一样，需要添加@Transactional注解，才能使用==，
		其是先select，在整个Transaction完了之后才执行delete

### 改

- 通过@Query

	添加@Modifying即可

	==方法的返回值应该是int，表示更新语句所影响的行数==

	==在调用的地方必须加事务，没有事务不能正常执行==

	```java
	@Modifying
	@Query(value="update UserModel o set o.name=:newName where o.name like %:nn")
	public int findByUuidOrAge(@Param("nn") String name,@Param("newName") String newName);
	```

- ==通过save()==

	save()传入的数据如果主键存在的话就执行update操作

	
