# Spring Data JPA简介
- Spring Data JPA 是 Spring Data 项目家族中的一员，它为基于Spring框架应用程序提供了更加便捷和强大的数据操作方式。
- Spring Data JPA 支持多种数据存储技术，包括MySQL、PostgreSQL、Oracle 等数据库。
- Spring Data JPA 提供了简单、一致且易于使用的API来访问和操作数据存储，其中包括基本的CRUD操作、分页、排序、自定义查询方法、动态查询、MongoDB 等功能。

## 使用 Spring Data JPA

### 1、引入pring Data JPA 和 对应版本的数据库类型相关驱动依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
~~~

### 2、配置文件添加配置
~~~yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/appdb?serverTimezone=Asia/Shanghai
    username: appuser
    password: 123456
    driver-class-name: org.postgresql.Driver
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      # 该配置比较常用，当服务首次启动会在数据库中生成相应表，后续启动服务时如果实体类有增加属性会在数据中添加相应字段，
      # 原来数据仍在，该配置除了 update 还有如下其他配置值
      # create ：该值慎用，每次重启项目的时候都会删除表结构，重新生成，原来数据会丢失不见。
      # create-drop ：慎用，当项目关闭，数据库中的表会被删掉。
      # validate ： 验证数据库和实体类的属性是否匹配，不匹配将会报错。
      ddl-auto: update
    # 该配置当在执行数据库操作的时候会在控制台打印 sql 语句，方便进行检查排错等
    show-sql: true
    # properties:
    #   hibernate:
    #     format_sql: true  # 可选：格式化 SQL，便于阅读
    #     use_sql_comments: true  # 可选：添加 SQL 注释，说明查询来源
    #   show-sql: false  # 关闭简易打印，避免重复输出
    # # 配置日志级别：让 Hibernate 打印参数值
    # logging:
    #   level:
    #     # 打印 SQL 语句和参数值（核心）
    #     org.hibernate.SQL: DEBUG
    #     # 打印参数的具体值（关键）
    #     org.hibernate.type.descriptor.sql.BasicBinder: TRACE
~~~

### 3、创建实体类
- @Entity：该注解标记这个类是JPA 实体类，表示该类对应数据库中的一张表，JPA 框架会识别并管理这个类与数据库表的映射关系，首次启动项目的时候，默认会在数据中生成一个同实体类相同名字的表（table），也可以通过注解中的 name 属性来修改表（table）名称， 如@Entity(name=“stu”) , 这样数据库中表的名称则是stu。当然也可以通过@Table进行指定数据表名称，不够该注解仍然十分重要，如果没有该注解首次启动项目的时候你会发现数据库没有生成对应的表。
- @Table：该注解指定实体类对应的数据库表名，解决类名与表名不一致的问题，同时对于一些数据库（postgreSQL）有schema模式需要指定，例如@Table(name="users", schema="public")
- @Id：该注解表明该属性字段是数据表主键，该属性必须具备，不可缺少。
- @GeneratedValue：该注解通常和 @Id 主键注解一起使用，用来定义主键的呈现形式，该注解通常有多种使用策略详细策略如下，generator = "menuSeq"：是指定 序列生成器的名称，需和后面的 @SequenceGenerator 注解的 name 一致
    - @GeneratedValue(strategy= GenerationType.IDENTITY) 该注解由数据库自动生成，主键自增型，推荐 mysql 数据库中使用，postgreSQL和oracle 不支持。
    - @GeneratedValue(strategy= GenerationType.AUTO) 主键由程序插入的数据控制，默认的主键生成策略，oracle 默认是序列化的方式，mysql 默认是主键自增的方式。由 JPA 自动选择策略（不推荐，跨数据库易出问题）
    - @GeneratedValue(strategy= GenerationType.SEQUENCE) 表示使用数据库序列（Sequence）生成主键，条件是数据库需要支持序列，推荐PostgreSQL和Oracle使用，Mysql不支持。
    - @GeneratedValue(strategy= GenerationType.TABLE) 使用一个特定的数据库表格来保存主键，较少使用。
- @SequenceGenerator：配置 序列生成器 的具体参数（配合 @GeneratedValue的 GenerationType.SEQUENCE 策略使用），@SequenceGenerator(name = "menuSeq", initialValue = 100000, allocationSize = 1, sequenceName = "MENU_SEQUENCE") 的详细说明如下
    - name = "menuSeq"：生成器名称，必须和 @GeneratedValue 的 generator 参数一致
    - initialValue = 100000：序列的起始值（即第一个生成的主键是 100000）
    - allocationSize = 1：序列的步长（每次生成的主键值 + 1，PostgreSQL 推荐设为 1，避免主键不连续）
    - sequenceName = "MENU_SEQUENCE"：指定数据库中序列的名称（JPA 会在数据库中创建名为 MENU_SEQUENCE 的序列，用于生成主键）。
- @Column：该注解可以定义一个字段映射到数据表的字段属性，比如字段长度，映射到数据表时字段属性的具体名字等。
- @Transient：该注解标注的字段不会被应射到数据表当中。
- @OneToOne、@OneToMany、@ManyToMany 注解来标注关系映射。这些注解通常与 @JoinColumn 注解一起使用，用于指定关联的外键列，当然这些配置会更加的复杂一些
~~~java
import javax.persistence.*;
import java.io.Serializable;

@Entity
@Table(name="users", schema="public")
public class User implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "menuSeq")
    @SequenceGenerator(name = "menuSeq", initialValue = 100000, allocationSize = 1, sequenceName = "MENU_SEQUENCE")
    @Column(name="id")//数据表字段名
    private int id;

    // age 和 name 没有加 @Column 注解，JPA 会采用默认映射规则：直接使用字段名称进行映射
    // 并且字段类型自动匹配：int → integer，String → varchar(255) 
    // 不过如果数据库中已经存在表的情况下需要注意字段类型手动设置以免字段类型不匹配
    // 例如 @Column(length = 8, nullable = false) 设置字段长度是8(字段长度设置仅对 String 类型生效)并且不能为null
    private int age;
    private String name;


    public User() {
    }
    public User(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", age=" + age + ", name=" + name + "]";
    }
}
~~~


### 4、创建DAO层Repository
~~~java
import com.entity.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface UserRepository extends JpaRepository<User,Integer> {


    // JpaRepository 中有一部分的save、saveAll、findById、findAllById、deleteById、deleteAllById等可以进行常规的CRUD操作
    // 上述方法 SQL 会被缓存在 JPA 持久化上下文，事务提交时才执行
    // 但是对于需要即时看到更新结果的场景（如后续代码依赖更新后的数据）可以使用 flush、saveAndFlush 等方法立即执行缓存的SQL




    // 可以使用 JPA 提供的 find 、 get 、delete 、count 关键字与字段拼接作为方法名，JPA底层会生成sql的方式来实现与数据库的交互，其关键词的搭配又很多方式，基本能覆盖表查询的所有情况
    // 需要注意的是 JpaRepository 自带的新增修改删除操作框架已经内置了事务支持，无需手动加@Transactional注解
    // 但自定义的删除、新增、更新方法（如 deleteByName），框架不会自动开启事务，必须由开发者手动声明事务，在调用 Repository 方法的 Service 方法上或者更上层的Controller 添加@Transactional注解
    // 如下是一些使用案例

    // 查询数据库中指定名字的用户
    List<User> findByName(String name);

    // 根据名字和年龄查询用户
    List<User> getByNameAndAge(String name, Integer age);

    // 删除指定名字的用户,删除时需要在Service层调用是添加事务注解@Transactional
    Long deleteByName(String name);

    // 统计指定名字学生的数量
    Long countByName(String name);



    // JPA同样支持写sql语句来操作数据，sql 有如下两种呈现形式，不过直接编写insert、update、delete的sql语句的需要使用 @Modifying 注解标注，
    // insert、update、delete的sql语句框架不会自动开启事务，必须由开发者手动声明事务，在调用 Repository 方法的 Service 方法上或者更上层的Controller 添加@Transactional注解
    // 1、JPQL 形式的 sql 语句，from 后面是以类名呈现的。
    // 2、原生的 sql 语句，需要使用 nativeQuery = true 指定使用原生 sql

    // 另外 sql 中的参数传递也有两种形式：
    // 1、使用问号 ? 紧跟数字序列，数字序列从1 开始，如 ?1 接收第一个方法参数的值。
    // 2、使用冒号 : 紧跟参数名，参数名是通过@Param 注解来确定。

    // 使用的 JPQL 的 sql 形式 from 后面是类名
    // ?1 代表是的是方法中的第一个参数
    @Query("select s from User s where s.name =?1")
    List<User> findClassRoom1(String name);

    // 这是使用原生的 sql 语句去删除数据，需要注意的是使用update 和 delete 语句需要使用 @Modifying 注解标注
    // :name 是通过 @Param 注解去确定的
    @Modifying
    @Query(nativeQuery = true, value = "DELETE FROM users u WHERE u.age >= :age")
    void deleteByAgeGreaterThanEqual(@Param("age")Integer age);


    // 排序正常使用，只是多加了一个 sort 参数而已
    @Query(value = "select t from User t where t.name =:name")
    List<User> getUsers(String name, Sort sort);


    // 分页正常使用，只是多加了一个 Pageable 参数而已
    @Query(nativeQuery = true,
            value = "select * from users t where t.name =:name",
            countQuery = "select count(*) from users t WHERE t.name = :name" // 总条数查询 SQL（必须补充,否则会按照limit、offset查询来计算总条数）
    )
    Page<User> getPage(@Param("name") String name, Pageable pageable);


    // 使用 Specification 进行动态查询
    // 需要根据传入的参数动态生成查询条件并查询用户列表，可能传入的用户属性参数有10项也可能只有1项
    // 其实就是动态生成sql中的where查询条件
    List<User> findAll(Specification<User> spec);

}
~~~


### 5、创建Service层进行使用
~~~java
import com.dao.UserRepository;
import com.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.StringUtils;

import javax.persistence.criteria.Predicate;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;


@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // 如下是JpaRepository默认的CRUD方法调用
    public List<User> getUser() {
        return userRepository.findAll();
    }

    public User getUser(int id) {
        Optional<User> res = userRepository.findById(id);
        return res.orElseThrow(() -> new RuntimeException("没有找到对应ID的user"));
    }

    public String saveUser(User user) {
        userRepository.save(user);
        return "success to add user";
    }

    public String updateUser(User user) {
        User up = userRepository.findById(user.getId()).orElseThrow(() -> new RuntimeException("没有找到对应ID的user"));
        up.setName(user.getName());
        up.setAge(user.getAge());
        userRepository.save(up);
        return "success to update user";
    }

    public String deleteUser(int id) {
        userRepository.deleteById(id);
        return "success to delete user";
    }




    // 如下是使用JPA通过关键字自定义方法名称做到自动生成sql进行执行
    public List<User> findByName(String name) {
        return userRepository.findByName(name);
    }

    public List<User> getByNameAndAge(String name, Integer age) {
        return userRepository.getByNameAndAge(name, age);
    }

    @Transactional
    public String deleteByName(String name) {
        userRepository.deleteByName(name);
        return "success to delete user name：" + name;
    }

    public Long countByName(String name) {
        return userRepository.countByName(name);
    }




    // 如下是调用手动编写的sql执行
    public List<User> getByName1(String name) {
        return userRepository.findClassRoom1(name);
    }

    @Transactional
    public void deleteByAgeGreaterThanEqual(Integer age) {
        userRepository.deleteByAgeGreaterThanEqual(age);
    }


    // 排序操作
    public List<User> getUsers(String name) {
        // 第一种方法实例化出 Sort 类，根据年龄进行升序排列
        Sort sort1 = Sort.by(Sort.Direction.ASC, "age");

        //定义多个字段的排序
        Sort sort2 = Sort.by(Sort.Direction.DESC, "id", "age");

        // 通过实例化 Sort.Order 来排序多个字段
        List<Sort.Order> orders = new ArrayList<>();
        Sort.Order order1 = new Sort.Order(Sort.Direction.DESC, "id");
        Sort.Order order2 = new Sort.Order(Sort.Direction.DESC, "age");
        orders.add(order1);
        orders.add(order2);
        Sort sort3 = Sort.by(orders);

        //可以传不同的 sort1,2,3 去测试效果
        return userRepository.getUsers(name, sort1);
    }

    // 分页操作
    public Page<User> getUserPages(String name, int offset, int size) {
        // 分页
        PageRequest pageRequest1 = PageRequest.of(offset, size);

        // 分页同时对分页后的数据进行排序
        Sort sort1 = Sort.by(Sort.Direction.ASC, "age");
        PageRequest pageRequest2 = PageRequest.of(offset, size, sort1);

        //可以传不同的 pageRequest1,2,3 去测试效果
        return userRepository.getPage(name, pageRequest1);
    }


    // 使用 Specification 进行动态查询
    public List<User> findUsersByCondition(String name, Integer age) {
        return userRepository.findAll((root, query, criteriaBuilder) -> {
            List<Predicate> list = new ArrayList<>();
            if (StringUtils.hasLength(name)) {
                list.add(criteriaBuilder.equal(root.get("name"), name));
            }
            if (age != null) {
                list.add(criteriaBuilder.greaterThanOrEqualTo(root.get("age"), age));
            }
            Predicate[] predicates = list.toArray(new Predicate[list.size()]);
            return criteriaBuilder.and(predicates);
        });
    }

}
~~~

















