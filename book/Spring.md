#spring要点
##DI,IOC
DI(依赖注入)是spring的核心功能之一。<br/>
`Dependency Injection` 和 `Inversion of Control` 其实就是一个东西的两种不同的说法而已。本质上是一回事。`Dependency Injection` 是一个程序设计模式和架构模型， 一些时候也称作 `Inversion of Control`，尽管在技术上来讲，`Dependency Injection` 是一个 `Inversion of Control` 的特殊实现，`Dependency Injection` 是指一个对象应用另外一个对象来提供一个特殊的能力，例如：把一个数据库连接以参数的形式传到一个对象的结构方法里面而不是在那个对象内部自行创建一个连接。Inversion of Control 和 Dependency Injection 的基本思想就是把类的依赖从类内部转化到外部以减少依赖。 应用Inversion of Control，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用，传递给它。也可以说，依赖被注入到对象中。所以，Inversion of Control 是，关于一个对象如何获取他所依赖的对象的引用，这个责任的反转。IoC是通过处理对象定义依赖的方式来工作，也就是说，一起协作的对象，要么通过构造函数参数来获得，要么在构造之后给对象设置属性来获得，要么从工厂方法返回的方式来获得。容器先创建bean，然后再注入这些依赖。这个获取过程是完全反过来的，所以命名为控制反转(IoC)。<br/>
DI能够删除任何特定的依赖于别的类或第三方接口的类，并且能够在初始化构造时加载要依赖的类。DI的优点是你可以依赖类的实现而并不需要更改你的代码。你甚至可以在接口不变的条件下重写依赖的实现而不用改变你的编码，即面向接口的编程。<br/>
注入方式有两种：<br/>
*. 构造器注入：通过构造器注入，能使当前实例作为不可变对象，并且能确保所有需要的依赖都是非空的.更进一步，构造器注入返回给客户代码的是一个完全初始化状态的对象.
*. Setter方法注入：Setter方法注入作为构造器注入的补充实现.能注入可选的有默认值得依赖.否则，会随处校验依赖的非空与否.

Spring容器会在容器加载时校验依赖非空和循环依赖.在初始化Bean时，Spring会在bean真正创建之前尽可能晚的设置属性和解决依赖关系.

##Spring beans定义和依赖实现方式
*Spring的单例scope是容器级别的，即一个容器一个bean实例,spring的单例实例缓存在ConcurrentHashMap中;而GOF的单例模式是基于ClassLoader的，即一个类加载器只能有一个实例*
###XML 文本文件
通过XML文本文件构造Spring beans和依赖缺失了编译时的类型检查，任何问题比如构造器参数的类型错误，甚至是构造器错误的参数只有在application context在运行时构造时才会检查。
###使用注解
通过配置注解扫描的根包，并且在bean上使用注解@Service等标示他是一个bean
```java
@Autowired
private UserService userService;
	
@Service("userService")
public class UserServiceImpl implements UserService
```
```xml
   <!-- 开启注解配置 即Autowried -->  
    <context:annotation-config/>  
    
    <!--    使用自动注入的时候要  添加他来扫描bean之后才能在使用的时候   -->
    <context:component-scan base-package="com.wyp.module.service,com.wyp.module.dao"/>  
```
当通过`@Autowired`注入时，默认是通过类型匹配具体的实现类的，但是如果接口有多个实现类，Spring容器是没法做选择的，有两种方式解决这个问题：
1.	`@Primary`注解，指定当有多个候选实现时，首选这个实现.
2.	`@Qualifier`注解指定不同实现不同的限定符，在具体注入时，通过该注解具体限定.

*@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入*
*bean实例的初始化顺序：静态代码块(变量)-->实例代码块-->构造方法-->postConstruct-->init-->......-->preDestroy-->destroy*

###配置解释
####`<context:annotation-config/>`
 `<context:annotation-config/>`这样一条配置，它的作用是隐式的向`Spring`容器注册
                           `AutowiredAnnotationBeanPostProcessor`,
                           `CommonAnnotationBeanPostProcessor`,
                           `PersistenceAnnotationBeanPostProcessor`,
                           `RequiredAnnotationBeanPostProcessor` 
 这4个`BeanPostProcessor`.注册这4个bean处理器主要的作用是为了你的系统能够识别相应的注解。
 如果想使用`@Autowired`,`@PersistenceContext`,`@Required`,`@Resource`,`@PostConstruct`,`@PreDestroy`,就需要按照传统声明一条一条去声明注解Bean，就会显得十分繁琐.
 因此如果在Spring的配置文件中事先加上`<context:annotation-config/>`这样一条配置的话，那么所有注解的传统声明就可以被忽略，即不用在写传统的声明，Spring会自动完成声明。
 ###`<context:component-scan base-package="com.xx" />`
 `<context:component-scan/>`的作用是让Bean定义注解工作起来,也就是上述传统声明方式.它的base-package属性指定了需要扫描的类包，类包及其递归子包中所有的类都会被处理。
 值得注意的是`<context:component-scan/>`不但启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能（即还隐式地在内部注册了`AutowiredAnnotationBeanPostProcessor`
 和`CommonAnnotationBeanPostProcessor`），因此当使用`<context:component-scan/>`后，就可以将`<context:annotation-config/>`移除了。
 ###`@Autowired`
 `@Autowired`可以对成员变量、方法和构造函数进行标注，来完成自动装配的工作。`@Autowired`的标注位置不同
 它们都会在Spring在初始化这个bean时，自动装配这个属性。注解之后就不需要`set/get`方法了。
###事务管理
Spring通过`TransactionManager`来实现事务管理，现有两种方式，一种是通过aop注入式的方式实现，另一种是通过`@Transactional`在方法上实现事务管理.
####aop注入式
```
    <bean id="txManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>

    <tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="query*" read-only="true" propagation="REQUIRED" />
			<tx:method name="get*" read-only="true" propagation="REQUIRED" />
			<tx:method name="create*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="add*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="save*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="update*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="delete*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="remove*" propagation="REQUIRED"
				rollback-for="java.lang.Exception,java.lang.RuntimeException" />
			<tx:method name="*" propagation="REQUIRED" />
		</tx:attributes>
	</tx:advice>

	<aop:config>
		<!--切入点指明了在所有方法产生事务拦截操作  -->
		<aop:pointcut id="module-pointcut"
			expression="execution(* com.wyp.module.service.*.*(..))" />
		<!--定义了将采用何种拦截操作，这里引用到 txAdvice,即在什么时候做什么事  -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="module-pointcut" />
	</aop:config>
```
####`@Transactional`
需要在配置文件中有tx相关的注解.
```
<!-- 此注解表示声明式事务，在方法上通过@Transactional控制事务 -->
    <tx:annotation-driven transaction-manager="txManager" />
```
在需要的地方通过`@Transactional`注入:
```java
@Transactional(propagation=Propagation.REQUIRED)
	public UserType registerUser(Customer customer) {
```
##spring mvc
###Configuring a servlet container
通过`org.springframework.web.servlet.DispatcherServlet`管理，在`web.xml`里面配置：
```
<servlet>
        <servlet-name>rest</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>  
	      <param-name>contextConfigLocation</param-name>  
          <param-value>/WEB-INF/rest-servlet.xml</param-value>  
	</init-param>
	<!-- 这个配置文件在容器启动的时候 就加载 -->
    <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>rest</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
如果
```
    <init-param>  
	      <param-name>contextConfigLocation</param-name>  
          <param-value>/WEB-INF/rest-servlet.xml</param-value>  
	</init-param>
```
不指定，则默认会加载该文件` WEB-INF/[servlet-name]-servlet.xml`.

通过这个`web.xml`，引出了另一个问题，即`web.xml`文件的加载顺序是什么样的？
通过查看tomcat源码，tomcat加载web.xml的顺序是:
```
context-param--->listener--->filter--->servlet
```
<p>
首先tomcat会生成一个程序应用级`ServletContext`,全局唯一，其中将`context-param`放在第一位主要是listener和filter会用到配置的初始化参数，
比如Spring配置的`contextConfigLocation`,在`ContextLoaderLister`加载时会从`ServletContext`的初始化参数中获取配置文件，进行bean的初始化操作.
上面还有一点，那就是`servlet`的加载，当`load-on-startup`大于等于0时，表示在tomcat容器启动时加载这个`Servlet,否则，在第一次使用时才加载.
</p>
###`<mvc:annotation-driven />`
`<mvc:annotation-driven />` 是一种简写形式，完全可以手动配置替代这种简写形式。`<mvc:annotation-driven />`
会自动注册`DefaultAnnotationHandlerMapping`与`AnnotationMethodHandlerAdapter` 两个bean,是`spring MVC`为`@Controllers`分发请求所必须的。

