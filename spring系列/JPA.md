# 体系结构

1. 多个spring-data项目定义dao冲突，分别定义@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")和@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
2. JPA初始化建表策略ddl-auto配置：
   - none：什么都不做(非嵌入默认)
   - create：全删除新建，丢失原来数据(嵌入embed默认)
   - create-drop：每次程序结束会清空表
   - update：每次运行程序，没有表会新建表，数据不请客————最常用
   - validate：运行程序会校验数据与数据库的字段类型是否相同，不同会报错
3. 先执行resourcrs/schema.sql，后jpa dll。根据ddl策略覆盖(默认none)之前的结果。但是嵌入模式下，ddl默认create
4. JPA和mybatis共用时，支持事务
5. 

#Entity实体类

## 基本注解

| 注解                | 含义                                   | 示例                                                         |
| ------------------- | -------------------------------------- | ------------------------------------------------------------ |
| @MappedSupperclass  | 公共字段父类                           | 某entity继承此父类，常用于审计aduite                         |
| @Entity             | 表明此类为实体类                       |                                                              |
| @Table(name="表名") | 设置表明，默认实体类名                 |                                                              |
| @Id                 | 表明主键                               | 每个联合主键字段都要加@Id                                    |
| @GeneratedValue     | 主键策略                               | @GeneratedValue(strategy=GenerationType.IDENTITY)            |
| @IdClass            | 表明此类有联合主键                     | @IdClass(value=C.class)  //序列化接口+hash+equals            |
| @Basic              | 表明此字段为映射，默认eager            | @Basic(fetch=FetchType.Lazy)                                 |
| @Transient          | 实体映射忽略此字段                     |                                                              |
| @Column             | 设置列名与实体属性映射，支持unique约束 | @Column(name="...",unique=false)支持unique约束、字段是否增改、最大长度 |
| @Temporal           | 用于设置日期字段的格式                 | @Temporal(TemporalType.TIMESTAMP)                            |
| @Enumerated         | 枚举属性取值(ordinal或string)          | @Enumerated(EnumType.STRING)                                 |
| @Lob                | 大数据对象                             |                                                              |
| @JoinColumn         | 关系所有方(从表外键字段)               | name=外键列名,referencedColumnbName=主键列名                 |
| @Version            | 乐观锁                                 | ==乐观锁null识别为insert==                                   |

1. 主键策略

   - 默认AUTO：自动选择
   - IDENTITY：自增主键，适用于mysql
   - SEQUENCE：序列化，配合@SequenceGenerator使用，用于oracle
   - TABLE：第三方表产生主键

2. 联合主键：

   - 专设表示联合主键的类，实现Serializeable接口、无惨构造函数、Hash和equals

   - 实体类@Entity设置@IdClass(value="主键类.class")

   - 每个主键字段都标注@Id

     ```java
     public class Key implement Serializable{
         private Integer id;
         private String name;
         hash----equals
         }
     @Entity
     @IdClass(Key.class)
     public class Company {
         @Id
         private Integer id;
         @Id
         private String name;
         private String address;
         }
     ```




4. 乐观锁@Version为null，会当做insert处理。即：update时，ID存在和Version合法都要满足

4. 关系所有方(owning side) ：拥有连接列的一方，通常是从表——外键。@JoinColumn/@JoinTable

5. 



## 一对一

1. 从表外键实体注@JoinColumn和@OneToOne
2. 主表@OneToOne

## 一对多/多对一

1. 多方：@ManyToOne+@JoinColumn
2. 一方：@OneToMany(mappedBy=多方属性名)

##多对多

1. 此时主被动关系不明确，joinColumns为连接表第一个连接字段，inverseJoinColumns第二个连接字段

2. 主动方：@ManyToMany+@JoinTable

3. 被动方：@ManyToMany(mappedBy=主动方属性名)

   ```java
   @ManyToMany(targetEntity ="对方实体类名",cascade = CascadeType.ALL)
   @JoinTable(//主动方                
    joinColumns = {@JoinColumn(name = "主动方从表外键名", referencedColumnName = "主动方主表主键名")}, 
            //被动方
          inverseJoinColumns = @JoinColumn(name = "被动方从表外键名", referencedColumnName = "被动方主表主键名"))
   ```


## JSon双向依赖的循环嵌套

1. 转成DTO忽略一方关系

2. @JsonIgnore忽略本实体的关系属性

3. 一方关系属性注@JsonIgnoreProperties("另一方的关系属性")：忽略本实体 对应另一方实体的某些属性

   ```java
   //通常@OneToMany的List忽略对方属性，即mappedBy的属性
   @JsonIgnoreProperties("user")
       @OneToMany(mappedBy="user")
       private List<Role> roles; //忽略此rolesList所有元素的user属性
   ```
```
   
   

##审计功能

1. 开启@EnableJpaAuditing，实体类==@EntityListeners({AuditingEntityListener.class,自定义监听器})==，可与@MappedSupper结合构造公共部分，然后被Entity继承

2. 提供的AuditingEntityListener，支持@Create@ModifiedBy/Date，填充Auditor需注入AuditorAware@bean

3. 自定义EntityListener无需实现接口，其方法注解@Pre/PostXXX。只需@EntityListeners配置。

4. 自定义EntityListener拦截机制：@@Pre/Post-Persist/update/remove

5. ==EntityListener是同步机制，一旦拦截处理错误，整体报错，可利用线程池或MQ==

   ```java
       @Bean
       public AuditorAware<String> auditorAware() {  //注入AuditorAware获取当前用户
           return new AuditorAware<String>(){
               @Override
               public Optional<String> getCurrentAuditor() { //针对--By
                   return Optional.ofNullable();//从Security/RequestContext获取
               }
           };
       }
```

   

# Dao接口构造

## 查询方法构造

| 关键字                       | 方法名                        | JPQL表达式                  |
| ---------------------------- | :---------------------------- | --------------------------- |
| And   Or                     |                               |                             |
| Is  Equals                   | findByNameAndAge              | e.name=?1 and e.age=?2      |
| Not                          |                               |                             |
| Between                      | findByAgeBetween              | e.age between ?1 and ?2     |
| LessThan    GreaterThanEqual |                               |                             |
| After  Before  日期比较      | findBySdateAfter              | e.sdate > ?1                |
| IsNull  IsNotNull            |                               |                             |
| Like  NotLike 手动%          |                               |                             |
| StartingWith  前%            |                               |                             |
| EndingWith  后%              |                               |                             |
| Containing   前后%%          |                               |                             |
| In   NotIn                   | findByAgeIn(list)             |                             |
| IgnoreCase                   | findByNameIgnoreCase          |                             |
| OrderBy                      | findByAgeOrderByIdDesc        | e.age=?1 order by e.id desc |
| First                        | findFirst10ByAge              |                             |
| Distinct                     | findDistinctByAge             |                             |
| @Param                       | findByAgeIn(@param("dd")list) | e.age in :dd                |

- 字段歧义：常用于关联属性和本实体属性驼峰冲突，用_分隔即可

## @Query语句

| 参数        | 含义                | 取值                                |
| ----------- | ------------------- | ----------------------------------- |
| nativeQuery | 是否原生SQL。       | 默认false为JPQL                     |
| countQuery  | 返回Page的count语句 | nativeQuery为true指定，一般无需设置 |

| 形参/返回值              | 含义                 | 示例                                         |
| ------------------------ | -------------------- | -------------------------------------------- |
| Pageable                 | 分页配置             | PageRequest.of(页码0..n, 页大小，[排序参数]) |
| Sort                     | 排序参数             |                                              |
| Slice                    | 每页分片，无count数  |                                              |
| List                     | 仅数据               |                                              |
| Page                     | 分片+count总数       |                                              |
| Stream/Optional          | 流式查询(==需关闭==) |                                              |
| Future/CompletableFuture | 异步查询             |                                              |
|                          |                      |                                              |

1. 支持find/count/existes/delete

2. 原生sql不支持Sort参数，但是==支持Pageable分页==

3. save(entity)先检查是否存在，后insert/update（两条sql语句）；delete同理

4. Stream需手动关闭，或者写在try-with-resources中

5. JPQL语法

   ```sql
   select c from Company c where c.id >?1 order by c.id desc
   ```

   ```java
   companyDao.findByIdGreaterThanOrderByIdDesc(5,PageRequest.of(0, 3)).getContent();
   //排序Sort参数写法
   TypedSort<Person> person = Sort.sort(Person.class);
   Sort sort = person.by(Person::getFirstname).ascending()
     .and(person.by(Person::getLastname).descending());
   
   Sort.by("firstname")
   ```

### 结果集映射Projections（返回部分字段或复合字段）

1. 新建VO interface接口，实现相关属性的getter方法。可以是dto对象，但是接口更方便

2. Dao方法返回该接口。(实际是代理方法)

3. 支持spel表达式  target为原始entity

4. ```java
   @Entity
   public class Company {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
       private String name;
       private String address;
   }
       Page<CV> findByIdGreaterThanOrderByIdDesc(Integer id,Pageable pageable);
   
       interface CV{
           String getName();
           @Value("#{target.name+''+target.address}")
           String getInfo();
       }
   ```

###@Modifying：删改操作@Query写语句并标注@Modifying(可配置刷新一级缓存)

## 动态Specification<Entity>条件构造器——实现JpaSpecificationExecutor接口

```java
(root, query, criteriaBuilder) -> {
    //root获取entity属性名，criteriaBuilder条件拼接    query负责select，where，having等结构拼接
            Predicate p1 = criteriaBuilder.greaterThan(root.get("id"), 0);
            Predicate p2 = criteriaBuilder.like(root.get("username"), "%%");
            Predicate p3 = criteriaBuilder.between(root.get("id"), 0, 300);
            Predicate and = criteriaBuilder.and(p1, criteriaBuilder.or(p2, p3));
            return query.where(and).getRestriction();
        }, PageRequest.of(0, 3))
```



1. 基本概念：
   - Root：根实体entity     构造Predicate属性
   - CriteriaQuery：语句结构(select from groupby where distince等) 连接Predicate
   - CriteriaBuilder：字段表达式组合(criteria)，返回Predicate谓词对象
2. 局限：
   - 效率低
   - 适用于where多条件动态拼接，join、复杂查询建议手写sql
   - 多表查询是，建议拆分sql或==引入视图View简化操作==
3. 3
4. 

#Web支持

##参数解析@EnableSpringDataWebSupport

1. 开启@EnableSpringDataWebSupport，它注入了DomainClassConverter路径变量转换器和HandlerMethodArhuementResolver分页排序参数转换器

2. 路径变量PathVariable转换：ID自动转换为对应Entity

3. 分页排序参数转换：

   - 默认page0,size=20，sort升序asc各属性分开写。函数形参Pageable

   - spring.data.web.pageable修改默认值

   - 案例：?page=1&size=10&sort=name&sort=age,desc

   - ```java
     @getMapping("/{id}")
     public User fingById(@PathVariable("id") User user){
     return user;
     }
     public Page<User> find(Pageable pageable){
     return userDao.findAll(pageable);
     }
     ```

     

4. 3

5. 3

6. 

## SpringDataRest

1. 默认所有CRUDReposiroty开启支持，映射为entityname实体名，自定义方法映射为方法名

2. @RepositoryRestResource设置此实体映射名、是否开启访问
3. @RestResource设置方法映射名、是否暴露      //path无需斜杠
4. query参数规则和@EnableSpringDataWebSupport一致，findByXXX方法支持XXX属性名直接查询