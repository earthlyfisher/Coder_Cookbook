#spring要点
##DI,IOC
DI(依赖注入)是spring的核心功能之一。DI能够删除任何特定的依赖于别的类或第三方接口的类，并且能够在初始化构造时加载要依赖的类。DI的优点是你可以依赖类的实现而并不需要更改你的代码。你甚至可以在接口不变的条件下重写依赖的实现而不用改变你的编码，即面向接口的编程。
##Spring beans定义和依赖实现方式
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
