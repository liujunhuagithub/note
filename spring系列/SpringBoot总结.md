# SpringBoot自动配置加载

1.  所有自动配置都是在启动的时候扫描并加载:/META-INF/spring.factories记录所有的自动配置类，只有只要导入了对应的start（对应@Condition成立），相关自动装配才会生效。
2. 自定义ViewResolver和Formatter只需要作为Bean注入即可，自动识别。设置日期格式可以用  spring.mvc.format.date
3.  通过修改server.XXX修改服务器配置
4.  开启优雅关闭server.shutdown=graceful(IDE可能不生效)



# Thymeleaf

1. 网页片段抽取   th:fragment="片段名"

2. 目标网页  th:insert="==~{视图名::片段名(参数名=值)}=="   ==（双冒号）==insert  本div插入   replace 本div替换成新的。参数一般作为标志位      #id可作为某id的片段

3. 调用(片段)方可传递参数，被调用的片段 可以接受此参数进行内部的判断`常用于进行css样式切换`

4. Thymeleaf由额外扩展，如Thymeleaf-extrasspringsrcurity,提供#authorization对象判断状态

5. th:each循环的是本元素，而非子元素

6. @{url  (  v1=${k1}  ,v2=k2  )}

7. th:selected  中设置判断条件，在option元素中，选中对应元素

8. th:with    变量赋值运算th:with="变量名={@service层.方法名}"可直接调用

9. th:object与*{}配合

   ```
   键字    　　功能介绍    　　　　案例
   th:id    　　替换id    　　　　  <input th:id="'xxx' + ${collect.id}"/>
   th:text    　文本替换   [[]] 　<p th:text="${collect.description}">description</p>
   th:utext    支持html的文本替换[()]   <p th:utext="${htmlcontent}">conten</p>
   th:object    替换对象    　　　　<div th:object="${session.user}">
   th:value    属性赋值    　　　　<input th:value="${user.name}" />
   th:with    变量赋值运算    　　　　<div th:with="isEven=${prodStat.count}%2==0"></div>
   th:style    设置样式    　　　　　　　　th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''"
   th:onclick    点击事件    　　　　　　th:onclick="'getCollect()'"
   th:each    属性赋值    　　　　　　　　tr th:each="user,userStat:${users}">
   th:if    判断条件    　　　　　　　　<a th:if="${userId == collect.userId}" >
   th:unless    和th:if判断相反    　　　　<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
   th:href    链接地址    　　　　　　　　　　<a th:href="@{/login}" th:unless=${session.user != null}>Login</a> />
   th:switch    多路选择 配合th:case 使用    <div th:switch="${user.role}">
   th:case    th:switch的一个分支    　　　　<p th:case="'admin'">User is an administrator</p>
   th:fragment    布局标签，定义一个代码片段，方便其它地方引用    <div th:fragment="alert">
   th:include    布局标签，替换内容到引入的文件    <head th:include="layout :: htmlhead" th:with="title='xx'"></head> />
   th:replace    布局标签，替换整个标签到引入的文件    <div th:replace="fragments/header :: title"></div>
   th:selected    selected选择框 选中    th:selected="(${xxx.id} == ${configObj.dd})"
   th:src    图片类地址引入    　　　　　　<img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" />
   th:inline    定义js脚本可以使用变量    <script type="text/javascript" th:inline="javascript">
   th:action    表单提交的地址    　　　　<form action="subscribe.html" th:action="@{/subscribe}">
   th:remove    删除某个属性    　　　　<tr th:remove="all"> 
   　　　　　　　　　　　　　　　　　　　　1.all:删除包含标签和所有的孩子。
   　　　　　　　　　　　　　　　　　　　　2.body:不包含标记删除,但删除其所有的孩子。
   　　　　　　　　　　　　　　　　　　　　3.tag:包含标记的删除,但不删除它的孩子。
   　　　　　　　　　　　　　　　　　　　　4.all-but-first:删除所有包含标签的孩子,除了第一个。
   　　　　　　　　　　　　　　　　　　　　5.none:什么也不做。这个值是有用的动态评估。
   th:attr    设置标签属性，多个属性可以用逗号分隔    比如 th:attr="src=@{/image/aa.jpg},title=#{logo}"，此标签不太优雅，一般用的比较少。
   ```

   

# 定制错误页面

- 默认响应错误方式：浏览器返回默认错误页面，其他客户端返回json

- 将自定义的错误页面放在thymeleaf/error文件下，使用4xx   5xx以匹配多个类型的错误，也可以精确制定

- 最后在静态资源文件夹找

- ##浏览器端页面获取的信息

  timestamp :时间戳
  status:状态码
  error:错误提示
  exception:异常对象
  message:异常消息
  errors: JSR303数据校验的错误

- ##定制Json数据（不论任何客户端都返回json）

  1. 使用==异常处理器@ControllerAdvice声明类==
  2. 其==方法声明@ExceptionHandler(异常类.class)和@ResponseBody==
  3. 返回自定义的Map

  

- ## 自适应客户端

  ​	继承DefaultErrorAttributes类，重写getErrorAttributes()方法，返回自定义的map

- 3

- 



# 容器定制

1. ## 修改服务器参数

   1. 在yml文件配置server.XXX参数
   2. spring.mvc.hiddenmethod.filter.enabled = true开启转换put/delete
   3. 编写EmbeddedServletContainerCustomizer类

2. ## 注册外置三大件

   ServletRegistrationBean
   FilterRegistrationBean
   ServletListenerRegistrationBean

   

   1. 创建Servlet

      ~~~java
      public class MyServlet extends HttpServlet {
          @Override
          protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
              super.doGet(req, resp);
          }
      
          @Override
          protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
              super.doPost(req, resp);
          }
      }
      ~~~

      

   2. 配置类中注入ServletRegistrationBean，它自定义的servlet映射路径。每个注册bean注册一个servlet

      ~~~java
      @Bean
      public ServletRegistrationBean servletRegistrationBean(){
       return new ServletRegistrationBean(new MyServlet(),"/test4");
           }
      ~~~

      

   

3. ## 使用外部servlet容器--tomcat

   1. 设置打包war方式，将嵌入式的Tomcat指定为provided ;

   2. modules配置项目的目录src/main/webapp及web.xml文件

   3. 配置外部的Tomcat，发布方式为exploded

   4. 创建SpringBootServletlnitializer实现类 ，重写configure

   5. SpringApplicationBuilder.source(@SpringBootApplication标注的入口类)

      ~~~java
      public class ServletInitializer extends SpringBootServletInitializer{
      protected SpringApplicationBuilder configure(SpringApplicationBuilder application){
      return application.sources(主入口类);
        }
      }
      ~~~

      

   6. 3

   7. 

4. 3

5. 3

6. 3

7. 3

8. 

# 国际化

1. 在resources下创建i18n文件夹，新建语言环境文件  XXX\_语言_国家.properties
2. 在yml中配置 spring.messages.basename===i18n.XXX==
3. 在thymeleaf中==#{ key}==   不需要写文件名



# 异步Async

1. 在配置类开启异步功能@EnableAsync
2. 在service层的方法声明@Async注解



 在@Async标注的方法，同时也适用了@Transactional进行了标注；在其调用数据库操作之时，将无法产生事务管理的控制，原因就在于其是基于异步处理的操作。

   那该如何给这些操作添加事务管理呢？可以将需要事务管理操作的方法放置到异步方法内部，在内部被调用的方法上添加@Transactional.

  例如：  方法A，使用了@Async/@Transactional来标注，但是无法产生事务控制的目的。

​     方法B，使用了@Async来标注，  B中调用了C、D，C/D分别使用@Transactional做了标注，则可实现事务控制的目的。



在子线程抛异常了主线程能回滚吗？ 答案是不能，因为主线程拿不到子线程抛的异常信息，spring事务管理的是当前线程下的，并且事务的隔离级别默认是 **PROPAGATION_REQUIRED--支持当前事务，假设当前没有事务。就新建一个事务,这涉及到ThreadLocal以及线程私有栈的概念,如果Spring 事务使用InhertableThreadLocal就可以把连接传到子线程,但是为什么Spring不那么干呢?因为这样毫无意义,如果把同一个连接传到子线程,那就是SQL操作会串行执行,那何必还多线程呢**，很显然，在另外一个线程下自然会创建一个新的事物，而不是进行事务传播，所以不能够回滚业务

这个时候，我想到了这个类Callable/Future，之前无意中有了解过它的特性，也是作为异步线程调用自己的业务的，特点就是它可以拿到子线程的返回信息。可以满足要求



1. 对于无返回值的异步任务，配置AsyncExceptionConfig类，统一处理。
2. 对于有返回值的异步任务，可以在contoller层捕获异常，进行处理。
3. 



# 定时任务

1. 在配置类开启定时功能@EnableScheduling

2. 在service层的方法声明@Scheduled注解，使用cron表达式（参考linux）

3. 秒 分 时 日 月 星期  

   1. 0和7代表sun，1-7代表周一——六

   2. ==？用于  日    和   星期    冲突匹配==

   3. L  ：最后     W：工作日     C：计算值     #：每月第几个星期

      0 0  2    ?     \*    6  L每个月的最后一个周六凌晨2点执行一次
      0 0  2    Lw  \*    ?每个月的最后一个工作日凌晨2点执行一次
      0 0  2-4  ?   \*   1#1每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次;l

# 邮件任务

1. 邮件发送引入spring-boot-starter-mail，Spring Boot自动配置MailSenderAutoConfiguration
2. 定义MailProperties内容，配置在application.yml中
   1. username  password
   2. host ：目标SMTP服务器地址
   3. 开启ssl安全连接  spring.mail.properties.maiL.smtp.ssl.enable=true
3. 自动装配JavaMailSender

# 数据库配置

## 数据源配置

1. 自定义@Configuration生成DataSource即可，和主配置文件绑定，需要@ConfigurationProperties(prefix = "spring.datasource")
2. 配置Driud监控功能 ：使用servlet注入官方提供的       。loginUsername LoginPassword分别代表账户密码作为Map。调用ServletRegistrationBean.setInitParameters( )注入。但可以在yml配置全部属性
3. 内嵌数据可可以自动执行resources/schema.sql和data.sql。但其他类型的数据库需要配置

# 参考文档

1. 相关注解大全 ：[csdn的springboot注解大全](https://blog.csdn.net/weixin_40753536/article/details/81285046)

   - @Import：用来导入其他配置类（带有@Configuration的java类），支持导入普通的java类,并将其声明成一个bean

     ​	

     ```java
     @Import(DemoService.class)//@Configuration配置类
     public class DemoConfig {
     }
     ```

     

   - @ImportResource：用来加载spring的xml配置文件。同时需要配合@Configuration注解一起使用，定义为配置类

   - @PropertySource：加载指定的属性文件（只能是properties文件）

   - @ConfigurationProperties：只针对springboot的application.yml文件的属性，prefix........自动根据pojo的属性名赋值

     

2. 