# 第二章 开发环境搭建和基础开发

## 2.1 SpringBoot的参数配置文件优先级顺序进行加载 :

- 命令行参数 :
-  来自 java:comp/env 的JNDI 属性 ;
- Java 系统属性( System.getProperties()); 
-  操作系统环境变量 :
- RandomValuePrope1tySource 配置 的 random.*属性值 :
- jar包外部的 application-{profile}.properties 或 application.yml (带 spring.profile)配置文件:
-  jar包内部的 application-{profile}.properties 或 application.ym (带 spring.profile)配置文件 ;
-  jar包外部的 application.properties 或 application.yml (不带 spring.profile)配置文件: 
- jar包内部的 application.properties 或 application.ym (不带 spring.profile)配置文件 : 
- @Configuration 注解类上的@PropertySource;
- 通过 SpringApplication.setDefaultProperties指定的默认属性。

## 2.2 Spring MVC 的视图解析器

Spring MVC 的视 图解析器的作用主要是定位视图 ，也就是当控制器只是返回 一个 逻辑名称的时候， 是没有办法直接对应找到视图的， 这就需要视图解析器进行解析了 

````.java
spring.mvc.view.prefix=/WEB-INF/jsp/ 
springmvc.view.suffix=.jsp
````



# 第三章 全注解下的IOC

## 3.1 IOC容器介绍

Spring IoC 容器是一个管理 Bean 的容器，在 Spring 的定义中，它要求所有的 IoC 容器都需要实 现接口 BeanFactory，它是一个顶级容器接口

接口中有多个getBean方法，这也是IoC容器最重要的方法之一， 它的意义是从 IoC 容器中获取 Beano 而从多个 getBean 方法中可 以看到有按类型( by type)获取 Bean 的，也有按 名称( by name)获取 Bean 的，这就意味着在 Spring IoC 容器中 ，允许我们按类型或者名称获取 Bean,

isSingleton 方法则判断 Bean 是否在 Spring IoC 中为单例。这里需要记住的是在 Spring IoC 容器 中，默认的情况下， Bean 都是以单例存在的，也就是使用 getBean 方法返回的都是同 一个对象。与 isSingleton 方法相反的是 isPrototype 方法，如果它返回的是 true，那么当我们使用 getBean 方法获取 Bean 的时候， Spring IoC 容器就会 创 建一个新的 Bean 返 回给调用者

由于 BeanFactory 的功能还不够强大，因此 Spring在 BeanFactory 的基础上， 还设计了一个更为 高级的 接 口 ApplicationContext。 它是 BeanFactory 的子接口 之 一 ， 在 Spring 的体 系中 BeanFactory 和 ApplicationContext 是最为重要的接口设计 ，在现 实 中我们使用的大部 分 Spring IoC 容器是 ApplicationContext接口的实现类

AnnotationConfigApplicationContext，从名称就可以看出它是一个基于注解的 IoC 容器。 之所以研究 它， 是因为 SpringBoot装配和获取 Bean的方法与它如出一辙。

## 3.2 注入对象的注解

**@Configuration**代表这是一个 Java配置文件，它来生成 IoC 容器去装配 Bean; **@Bean** 代表将 initUser 方法返回的 POJO 装配到 IoC 容器中，而其 属性 name 定义这个 Bean 的名称

````.java
// 创建ApplicationContext对象,获取配置类AppConfig
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class)
// 获得对象
User user= ctx.getBean(User .class); 
````

**@Component** 是标明l哪个类被扫描进入 Spring IoC 容器，而**@ComponentScan** 则是标明采用何种策略去扫描装配 Bean,加入了@ComponentScan，意味着它会进行扫描，但是它只会扫描类 AppConfig 所在的当前 包和其子包 .

- 首先可以通过配置项 basePackages 定义 扫描的包名，在没有 定义的情 况下，它只会扫描当前包和其子包下的路径
- 还可以通过 basePackageClasses 定义扫描的类;
- 其中还有 includeFilters 和 excludeFilters, includeFilters 是定义满 足过滤器( Filter)条件的 Bean 才去扫描， excludeFilters 则是排除过滤器条件的 Bean，它们都需要 通过一个注解@Filter去定义，它有一个 type类型，这里可以定义为注解或者正则式等类型 。 
- classes 定义注解类， pattern 定义正则式类。

@SpringBootApplication 也注入了@ComponentScan,但是这里需要特别注意的是，它提供的 exclude和 excludeName 两个方法是对于其内部的自动配置类才会生效的。为了能够排除其他类，还可以再加入@ComponentScan 以达到我们的目的 。`@ComponentScan(basePackages = { ” com . springboot . chapter3 ” } ,excludeFilters = {@Filter(classes = Serv工ce . class) ))`

注解**@Value** 则是指定具体的值，使得 Spring IoC 给予对应的 属性注入对应的值



**@Autowired**

同样 ， 它除了可以标注属性外，还可以标注方法 ， 如 setAnimal 方法，如下所示 

````

@Override
@Autowired
public void setAnimal (Animal animal) {
this.animal = animal;
}
这样它也会使用 setAnimal 方法从 IoC 容器中找到对应的动物进行注入，甚至我们还可以使用在 方法的参数上，后面会再谈到它 。
````

首先是一个注解**@Primary**，它是一个修改优先权的注解，当我们有猫有狗的时候，假设这次 需 要使用猫 ， 那么只需要在猫类的定义上加入@Primarγ 就可以了，类似下面这样 :

````
@Component
@Primary
public class Cat 工mplements Animal {

}

// 这里的@Primary 的含义告诉 Spring IoC 容器 ，当发现有多个同样类型的 Bean时,请优先使用该Bean进行注入
````

## 3.3 带有参数的构造方法类的装配

有些类只有带有参数的构造方法，于是上述的方法都不能再使用了。为 了满足这个功能，我们可以 使用@Autowired 注解对构造方法的参数进行注入，

````
@Component
public class BussinessPerson implements Perso口{
private Animal an工mal = null;
public Buss工nessPerson( @Autowired @Qualifier( "dog”) Ani皿al an丰皿al) {
this.animal = animal ;
}

代码中取消了@Autowired 对属性和方法的标注。注意加粗的代码， 在参数上加入了 @Autowired 和 @Qualifier 注解，使得它能够注入进来。这里使用@ Qualifier 是 为 了避免歧义性
````

## 3.4 Bean的生命周期

Bean 的生命周期的过程 ， 它大致分为 Bean 定义、 Bean的初始化、 Bean的生存期和Bean的销毁4个部分

1. Bean定义过程大致如下。

   - Spring通过我们的配置，如@ComponentScan 定义的扫描路径去找到带有@Component 的类， 这个过程就是一个资源定位的过程 。
   -  一旦找到了资源，那么它就开始解析，并且将定义的信息保存起来 。注意 ，此时还没有初始 化 Bean，也就没有 Bean 的实例，它有的仅仅是 Bean 的定义。
   - 然后就会把 Bean 定义发布到 Spring IoC 容器中 。 此时， IoC 容器也只有 Bean 的定义，还是 没有 Bean 的实例生成。
   - 完成了这 3 步只是 一 个资源定位并将 Bean 的定义发布到 IoC 容器的过程，还没有 Bean 实例的 生成，更没有完成依赖注入。

   在默认的情况下， Spring 会继续去完成 Bean 的 实例化和依赖注入，这 样从 IoC 容器中就可以得到 一个依赖注入完成的 Bean。 但是，有些 Bean 会受到变化因 素 的影响，这 时我们倒希望是取出 Bean 的时候完成初始化和依赖注入，换句话说就是让那些 Bean 只是将定义发 布到 IoC 容器而不做实例化和依赖注入， 当我们取出来的时候才做初始化和依赖注入等操作 。

   **@ComponentScan** 中还有一个配置项 lazyInit，只可以配置 Boolean 值，且默认值为 false，也就是 默认不进行延迟初始化，因此在默认的情况下 Spring会对 Bean进行实例化和依赖注入对应的属性值。

**@PostConstruct** 定义了初始化方法

**Bean的具体初始化方法为详细深入,有空可以详细深入,P40**

## 3.5 使用属性文件()

但是有时候我们会觉得如果把所有的内容都配置到 application.properties，显然这个文件将有很多内 容。为了更好地配置，我们可以选择使用新的属性文件 。 例如， 数据库 的属性可以配置在 jdbc.properties 中， 然后使用 **@PropertySource**去定义对应的属性文件，把它加载到 Spring 的上下文中

````

@PropertySource(value={”classpath:jclbc.properties”} ，ignoreResourceNotFound=true)
public class Chapter3Application {
public static void main (String[] args ) {
SpringApplication.run(Chapter3Application.class , args ) ;


````

## 3.6 条件装配Bean

有时候某些客观的因素会使一些 Bean 无法进行初始化，例如，在数据库连接池的配置中漏掉一 些配置会造成数据源不能连接上。 在这样的情况下， IoC容器如果还进行数据源的装配， 则系统将会 抛出异常，导致应用无法继续。这时倒是希望 IoC 容器不去装配数据源。
为了处理这样的场景， Spring 提供了@Conditional 注解帮助我们，而它需要配合另外一个接口 Condition ( org.springframework.context.annotation.Condition)来完成对应的功能 。 例如，下面把代码 清单 3-12 中的代码修改为代码 清单 3-26 中的代码 。

````

@Bean (name = ”dataSource”, destroyMethod = ”close”)
@Conditional(DatabaseConditional . class)
public DataSource getDataSource (
@Value ( ”${ database .driverName ) ” )
String driver, 
@Value(”${database.url)") 
String url,
@Value ("${database .username ) ” )
String username , 
@Value(”♀{database.password}”) 
String password ){
Properties props= new Properties (); props .setProperty (”driver”, driver); props setProperty( "url”, url);
props .setProperty (”username”, username); props setProperty (”password", password); DataSource dataSource = null ;
try {
dataSource = BasicDataSourceFactory.createDataSource(props) ;
) catch (Exception e) { e .printStackTrace() ;
return dataSource ;
}
````

这里可以看到 ，加入了@Conditional 注解 ， 井且配置了类 DatabaseConditional，那么这个类就必须 实现 Condition接口。 对于 Condition接口则要求实现 matches方法，它的内容如代码清单 3-27所示。

````

public class DatabaseConditional implements Condition {
/**数据库装配条件
*@param context 条件上下文
*@param metadata 注释类型的元数据
*@return true装配Bean，否则不装配
*/
 @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
  //取出环境配置
  Environment env = context .getEnvironment(); //判断属性文件是否存在对应的数据库配置
  return env .containsProperty (” database.driverName” )
  && env.containsProperty (” database.url”)
  && env . containsProperty (” database.username ”)
  && env.containsProperty (” database .password");
  	}
  }
````

matches 方法首先读取其上下文环境 ， 然后 判 定是否已经配置了对应的数据库信息。这样，当这 些都己经配置好后则返回 true。这个时候 Spring会装配数据库连接池的 Bean，否则是不装配的。

## 3.7 Bean 的作用域

 @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)指定类的作用域,prototype

这里 的 ConfigurableBeanFactory 只能提供 单 例 ( SCOPE_SINGLETON )和 原 型 ( SCOPE_ PROTOTYPE)两种作用域供选择， 如果是在 SpringMVC环境中，还可以使用 WebApplicationContext 去定义其他作用域， 如请求( SCOPE REQUEST)、 会话 (SCOPE_SESSION) 和应用 (SCOPE APPLICATION)。 

`@Scope(W ebApplicationContext.SCOPE REQUEST)`

这样同一个请求范围 内去获取这个 Bean 的时候只会共用 同一个 Bean，，第二次请求就会产生新的 Bean。因此两个不同的请求将获得不同的实例的 Bean， 这一点是需要注意的。

## 3.8 环境的切换

**Spring 是 先判 定是否 存 在 spring.profiles.active 配 置后 ， 再去查 找 spring.profiles.default 配置 的，所以 spring.profiles.active 的优先级要大于 spring.profiles.default,**

## 3.9 引人XML配置Bean

我们也可以在 Spring Boot 中使用 XML 对 Bean 进行配置。 这里需要使用的是注解 **@ImportResource**，通过它可以引入对应的 XML 文件，用 以加载 Bean。有时候有些框架(如 Dubbo) 是基于 Spring 的 XML 方式进行开发的，这个时候需要引入 XML 的方式来实现配置。

````
@ImportResource (value = {'’classpath: spring-other.xml"}) 
public class AppConfig {
		...
}
````

这样就可以引入对应的 XML，从而将 XML 定义的 Bean 也装配到 IoC 容器中。

## 3.10 使用 SpringEL

待深入学习内容...



# 第三章 AOP

## 4.3.6 引入

某个服务并不是自己所提供，而是别人提供的，我们不能修改它，这时 Spring 还允许增强这个接口 的功能，我们可以为这个接口引入新的接口 

````java
@Aspect
public class MyAspect {
@DeclareParents(
value= "com.springboot.chapter4.aspect.service.impl.UserService工mpl ＋”，
defaultimpl=UserValidator工mpl.class)
public UserValidator userValidator;
    ......
}
````

注解```＠DeclareParents` ， 它的作用是引入新的类来增强服务 ， 它有两个必须
配置的属性 `value` 和 `defaultlmpl` 。
• value：指向你要增强功能的目标对象,(全限定名) 这里是要增强 UserServicelmpl 对象 ， 因此可以看到
配置为 com .springboot.chapter4.aspect. se凹ice.impl.UserServicelmpl**＋**。
• defaultlmpl ： 引入增强功能的类这里配置为 UserValidatorlmpl ，用来提供校验用户是否为
空的功能 。 

Spring会把 UserService 和 UserValidator 两个接口传递进去 ，让代理对象下挂到这两个接 口下， 这样这个代理对象就能够相互转换并且使用它们的方法了 

## 4.3.7 通知获取参数

有时候我们希望能够传递参数给通知，这也是允许的，我们只需要在切点处加入对应的正则式就可以了。当然 ，对于非环绕通知还可以使用一个连接点 (JoinPoint）类型的参数 ， 通过它也可以获取参数。 

 ```java
 @Before (” pointCut () && args(user)” )
public void beforeParam (Join Point point , User user) {
Object[] args = point.getArgs ( ) ;
System . out.println (”before ..... . ” ) ;
    ...
}
 ```

正则式 pointCut() && args(user） 中 ， pointCut（）表示启用原来定义切点 的规则 ，并且约定将连接点（目标对象方法）名称为 user 的参数传递进来。 这里要注意 ， JoinPoint 类型 的参数对于非环绕通知而
言， Spring AOP 会自动地把它传递到通知中：对于环绕通知而言，可以使用 ProceedingJoinPoint 类型的参数。之前我们讨论过它的结构，使用它将允许进行目标对象的回调 



## 4.4多个切面

切面的执行顺序是混乱的，而在我做的测试中也没有找到多个切面执行顺序的规律。 但是，在很多时候，开发者需要确定切面的执行顺序，来决定哪些切面先执行，哪些切面后执行。为此,Spring 提供了一个注解``＠Order`` 和一个接口 Ordered，我们可以使用它们的任意一个指定切面的顺序。 下面我们先展示＠Order，例如，我们指定 MyAspectl 的顺序为 1  

```.java


@Aspect
@Order (1)
public class MyAspectl {
	...
}
```

对于前置通知（ before ）都是从小到大运行的，而对于后置通知和返回通知都是从大到小运行的，这就是一个典型的责任链模式的顺序 。 



# 第五章 访问数据库

