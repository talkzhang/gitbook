# spring 各版本特点：

- 1.x版本 只支持xml配置，确定了核心模块IOC和AOP，还不支持注解，因为注解是jdk5才有。
- 2.x版本 开始支持注解，支持了基于注解的配置。
- 3.x支持了基于Java类的配置。
- 4.x全面支持Java 8.0，支持Lambda表达式的使用，提供了对@Scheduled和@PropertySource重复注解的支持，对核心容器进行增加：支持泛型的依赖注入、Map的依赖注入、Lazy延迟依赖的注入、List注入、Condition条件注解注入、对CGLib动态代理类进行了增强。
- 5.x jdk要求最低1.8，同时junit兼容的升级，以及响应式编程的新模型webflux。

springboot1.x使用的是spring4版本，springboot2.x之后使用的是spring5版本。

# spring核心

spring核心的几个模块：spring-core、spring-beans、spring-context、spring-context-support和spring-expression、spring-aop、spring-web、spring-webmvc

- spring-core和spring-beans构成了框架最基础的部分，包括控制反转和依赖注入功能。BeanFactory是工厂模式的一个很成熟的实现
- spring-context是基于spring-core和spring-beans构建的，它提供了一种以框架风格来访问对象的方式，类似于JNDI注册。ApplicationContext接口是spring-context的焦点。
- spring-context-support为集成第三方库（如定时器Quartz）提供支持。
- spring-expression提供了一种强大的表达式语言，可以在运行时查询和操作对象。
- spring-aop提供了面向方面编程的一套成熟体系实现
- spring-web 提供了基础的 Web 开发的上下文信息，可与其他 web 进行集成。
- spring-webmvc 提供了 Web 应用的 Model-View-Controller 全功能实现。

spring核心简单来说是IOC和AOP

## 什么是IOC

IOC（Inversion of Control，控制反转），也称为DI（Dependency Injection 依赖注入）。这并不是什么具体的技术，而是一种思想，什么思想呢，松耦合的思想。

在平时的java应用开发中，我们要实现某一个功能或者说是完成某个业务逻辑时至少需要两个或以上的对象来协作完成，在没有使用Spring的时候，每个对象在需要使用他的合作对象时，自己均要使用像new object() 这样的语法来将合作对象创建出来，这个合作对象是由自己主动创建出来的，创建合作对象的主动权在自己手上，自己需要哪个合作对象，就主动去创建，创建合作对象的主动权和创建时机是由自己把控的，而这样就会使得对象间的耦合度高了，A对象需要使用合作对象B来共同完成一件事，A要使用B，那么A就对B产生了依赖，也就是A和B之间存在一种耦合关系，并且是紧密耦合在一起，而使用了Spring之后就不一样了，创建合作对象B的工作是由Spring来做的，Spring创建好B对象，然后存储到一个容器里面，当A对象需要使用B对象时，Spring就从存放对象的那个容器里面取出A要使用的那个B对象，然后交给A对象使用，至于Spring是如何创建那个对象，以及什么时候创建好对象的，A对象不需要关心这些细节问题(你是什么时候生的，怎么生出来的我可不关心，能帮我干活就行)，A得到Spring给我们的对象之后，两个人一起协作完成要完成的工作即可。

所以控制反转IoC(Inversion of Control)是说创建对象的控制权进行转移，以前创建对象的主动权和创建时机是由自己把控的，而现在这种权力转移到第三方，比如转移交给了IoC容器，它就是一个专门用来创建对象的工厂，你要什么对象，它就给你什么对象，有了 IoC容器，依赖关系就变了，原先的依赖关系就没了，它们都依赖IoC容器了，通过IoC容器来建立它们之间的关系。

## 什么是AOP

AOP（Aspect Orient Programming），一般称为面向方面（切面）编程，作为面向对象的一种补充，用于处理系统中分布于各个模块的横切关注点，比如事务管理、日志、缓存等等。可以想象下通过IOC各个模块实现解耦后，各个模块功能之间都是纵向排列互不干扰，如果这时需要在某两个或者更多模块实现日志记录功能，没有aop，怎么实现？实现起来代码非常的冗余，aop的思想可以通过横切面解决该类问题。

aop如此方便，它如何实现的？

spring aop是通过代理模式实现的，代理模式分为动态代理和静态代理，同时根据接口和类的不同，动态代理又分为jdk动态代理和cglib动态代理。

### 静态代理

静态代理是一种冗余的实现方式，它的特点就是在`程序编译期就已经确认谁去代理谁了`，

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210909113018.png)

举例说明：

```java
publicinterface Subject {
    publicvoid operate(); 
}
 
 
/**
 * 具体主题
 */ 
publicclass RealSubject implements Subject{ 
 
   @Override 
   publicvoid operate() { 
        System.out.println("realsubject operatestarted......"); 
   } 
}
 
/**
 * 代理类
 */ 
publicclass Proxy implements Subject{ 
 
   private Subject subject;   
    
   public Proxy(Subject subject) { 
        this.subject = subject; 
   } 
   
   @Override  
   publicvoid operate() { 
   
        System.out.println("before operate......"); 
   
        subject.operate(); 
   
        System.out.println("after operate......"); 
   
   } 
   
}

 
/**
 * 客户端
 */ 
publicclass Client { 
   /**
    * @param args
    */ 
   publicstaticvoid main(String[] args) { 
        Subject subject = new RealSubject(); 
        Proxy proxy = new Proxy(subject); 
        proxy.operate(); 
   } 
}
```

### 动态代理

动态代理分为jdk动态代理和cglib动态代理。

*jdk动态代理*

它的实现需要实现InvocationHandler接口，并且实现接口中的invoke方法。在invoke方法中实现切面的各种逻辑，**该代理是通过java反射实现的，且jdk动态代理是基于接口类来的**

*cglib动态代理*

实现MethodInterceptor的intercept方法，**该代理基于asm字节码技术实现，且cglib动态代理基于类实现的**

## springMVC是什么

说springMVC之前先说MVC，MVC模式（Model-View-Controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。

控制器（Controller）：Servlet，控制器主要处理用户的请求

视图（View）：HTML, JSP, 前端框架

模型（Model）：逻辑业务程序（后台的功能程序）, Service, Dao, JavaBean

这套规范可以极大程度将视图、控制层、以及模型层进行解耦，对软件系统架构来说非常适合，扩展性极高，现在的前后端分离也是基于MVC的模式衍生出来的，因为大家各自独立之后，每层结构用什么技术栈没有强限制。

### springMVC执行流程

执行流程图如下：

![springMVC执行流程图](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20210910140851.png)

1. 前置分发器 DispatcherServlet 接收到 HTTP 请求之后，将查找适当的控制器 Controller 来处理请求，它通过解析 HTTP 请求的 URL 获得 URI，再根据该 URI 从处理器映射 HandlerMapping 当中获得该请求对应的处理器 Handler 和处理器拦截器 HandlerInterceptor，最后以 HandlerExecutionChain 形式返回。
2. 前置分发器 DispatcherServlet 根据获得的处理器 Handler 选择合适的适配器 HandlerAdapter。如果成功获得适配器 HandlerAdapter，在调用处理器 Handler 之前其拦截器的方法 preHandler() 优先执行。
3. 方法 preHandler() 提取 HTTP 请求中的数据填充到处理器 Handler 的入参当中，然后开始调用处理器 Handler（即控制器 Controller）相关方法。
4. 控制器 Controller 执行完成之后，向前置分发器 DispatcherServlet 返回一个模型与视图名对象 ModelAndView 。在填充Handler的入参过程中，根据配置，Spring将做一些额外的工作。
    - 消息转换。将请求消息（如json，xml等数据）转换成一个对象，将对象转换为指定的响应信息。
    - 数据转换。对请求消息进行数据转换，如String转换成Integer、Double等。
    - 数据格式化。对请求消息进行数据格式化，如将字符串转换成格式化数字或格式化日期等。
    - 数据验证。验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中。
5. 前置分发器 DispatchServlet 根据模型与视图名对象 ModelAndView 选择适合的视图解析器 ViewResolver，前提该视图解析器必须已经注册至 Spring IOC 容器当中。
6. 视图解析器 ViewResolver 将根据 ModelAndView 里面指定的视图名称获得特定的视图 View。
7. 前置分发器 DispatchServlet 将模型数据填充进视图当中，然后将渲染结果返回给客户端。

## spring有哪些注入方式

常用得有构造函数、setter、属性field注入。

## @resource和@autowried区别

resource是javax包下的注解，autowried是spring框架中的注解。

autowried通过bytype完成注入，当某一接口有多个实现想要指定某一个类时可以结合@Qualifier注解一起使用，同时如果想使用autowried通过byname形式注入也可以；resource默认通过byname来注入。

## spring 有哪些常见的注解

1. @Controller 、@RestController 声明是一个controller由于接受web请求的类
2. @Service、@Component 声明是spring管理的bean实例
3. @RequestMapping 声明请求路径，同时衍生出@PostMapping、@GetMapping等。
4. @Autowired 将spring管理的bean注入
5. @RequestParam 接受路径上问号之后的参数
6. @PathVariable 接收路径模板参数
7. @ControllerAdvice 标注是统一处理异常类，配合@ExceptionHandler(value = ParmsException.class)（用于方法上）实现全局异常处理

# springboot

## springboot是什么

先得从spring说起，首先，spring确定提供了了一套比较稳定的基于java的一套web应用框架，但是使用它开发最大的问题就是维护配置文件，尤其是项目较大，使用的技术栈、依赖较多时，可能会出现各种配置文件满天飞，且技术栈过多，开发过程中容易导致某些依赖版本冲突，而springboot就是用来解决这些问题的，它的核心就是“约定大于配置”，它只有一个配置文件，.properties或者.yaml，所有的配置结合springboot-starter这些集成后，基本都可以在该配置内解决，当然，也可以自己定制一些更灵活的配置，比如xml、或者通过java类等都可以实现配置效果。

Spring Boot实现了auto-configuration自动配置（另外三大神器actuator监控，cli命令行接口，starter依赖），降低了项目搭建的复杂度。它主要是为了解决使用Spring框架需要进行大量的配置太麻烦的问题，所以它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具；同时它集成了大量常用的第三方库配置(例如Jackson, JDBC, Mongo, Redis, Mail等等)，Spring Boot应用中这些第三方库几乎可以零配置的开箱即用(out-of-the-box)。

## springboot如何实现监控

springboot actuator是springboot集成的一个监控组件，该组件没有ui，提供了restful接口来完成对服务的监控功能。

springboot admin 是针对actuator做的一套提供ui的高级封装版，它通过server和client端来完成对每个服务的监控，有点像configserver和configclient，可以完成对jvm、项目配置信息、线程使用情况等作出监控查看。

## 常见的一些注解

- @Primary 多个相同的bean类时，标注该注解后再注入时，会默认注入有该注解的bean。
- @ConfigurationProperties 使用该注解可以指定配置文件的前缀部分，将后缀部分作为实体属性封装，这样该类就会成为一个纯纯的对象形式配置的类。
- @EnableConfigurationProperties 该注解配合`@ConfigurationProperties`使用，当只有@ConfigurationProperties注解的类需要注入使用时（该类没有其他注解，只有它），它还不能被spring上下文扫描到，使用`@EnableConfigurationProperties(xxx.class)`可以声明配置的类可以作为bean在当前类注入，一句话：使使用 `@ConfigurationProperties` 注解的类生效。。
- @ConditionalOnProperty prefix指定前缀，value/name指定后缀属性（是数组结构），havingValue指定拥有的配置的值，matchIfMissing声明如果不存在该配置是否加载。
- @propertysource 可以指定某个配置类读取配置的来源，可以配合Environment或者@Value实现根据配置来源注入。
- @ConditionalOnMissingBean 它是修饰bean的一个注解，主要实现的是，当你的bean被注册之后，如果再注册相同类型的bean，就不会成功，它会保证你的bean只有一个，即你的实例只有一个，当你注册多个相同的bean时，会出现异常，以此来告诉开发人员，一句话概述：如果已经注入相同bean，则不会再次注入。
- @ConditionalOnBean 当给定的在bean存在时,则实例化当前Bean。例如注入A需要操作B，则需要注入A时B先与A注入才对，这个朱姐就是用来解决这个问题。
- @ConditionalOnClass // 当给定的类名在类路径上存在，则实例化当前Bean 
- @ConditionalOnMissingClass // 当给定的类名在类路径上不存在，则实例化当前Bean
- @ImportResource 可以指定使用xml配置文件来加载bean。

## @Import 注解的作用和原理

该注解从字面上可以看出，使用该注解可以指定注入的bean类，除了这个，它另一个主要作用是，当注入类实现了ImportBeanDefinitionRegistrar、ImportSelector接口时，会进行单独处理，比如实现ImportSelector接口，会调用selectImports方法去里面做一些扩展集成，实现ImportBeanDefinitionRegistrar，则将该实现的类进行实例化之后放入configclass，之后一起当做bean注入到spring容器中，如果没有实现任何接口，那这个类就当做普通bean注入。

详细可参考连接：[https://blog.csdn.net/gongsenlin341/article/details/113281596](https://blog.csdn.net/gongsenlin341/article/details/113281596)

## springboot 启动流程

运行一个springboot项目，只需要启动类上有注解，调用springboot内的一个静态run方法即可完成一个普通项目搭建。

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

既然如此，springboot启动大致包含两方面：

1. @SpringBootApplication注解。
2. SpringApplication.run方法。

如图：

![springboot启动简图](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/springboot%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E7%AE%80%E5%9B%BE.png)

### @SpringBootApplication注解

点进去该注解源码，可以发现该注解其实包含三个注解，分别是@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
     // ....
}
```

### @SpringBootConfiguration

该注解其实就是@Configuration，即声明是一个spring配置类，spring会将该配置类下的bean示例通过ioc注入到上下文当中。

### @ComponentScan

该注解声明spring需要扫描哪个包下的文件去管理。

### @EnableAutoConfiguration

springboot神级注解，该注解表示开启自动配置管理，先看下代码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    
}

```
这是一个复合注解，关键在@Import注解，它会加载AutoConfigurationImportSelector类，然后就会触发这个类的selectImports()方法，（上面有关于@Import注解的介绍）根据返回的String数组(配置类的Class的名称)加载配置类。然后看该类的selectImports方法：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
          // 重点看这里 加载后它是返回值
		List<String> configurations = getCandidateConfigurations(annotationMetadata,
				attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return StringUtils.toStringArray(configurations);
	}

     // 调用该方法获取加载bean路径
     protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

     // 该方法声明要获取factories文件中具体的配置
     protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}
```

该方法主要是从`spring.factories`文件获取spring配置类信息，根据这些配置类加载所需要加载的bean实例，按如上代码，会最终找到如下配置：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20220120182320.png)

现在终于明白，为什么springboot官方starter和自定义starter的区别了，所谓官方，就是springboot通过@EnableAutoConfiguration默认加载的那些配置类，可以随便看一下，里面有redis等这些项目中常用的starter的配置类，都在这里配置后加载了，而自定义starter，需要自己写一个`spring.factories`文件，并在该文件中声明配置类路径即可完成自动配置。

看到这里思考一个问题，如果你的项目中什么都没有用，那这里配置的这么多配置类找不到文件，不会报错吗？其实这里就用到spring中@Condition*相关的注解的，比如我们项目中没有用到rabbismq，但是根据如上看到通过自动加载，有该类的相关配置，看看spring是怎么做的：

```java
@Configuration
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
     //...
}
```

通过@ConditionalOnClass来判断你的项目中是否有依赖了Rabbit，如果没有那当前配置类就不会去加载任何东西，只有满足条件才会加载。

关于注解就先告一段落，接着说下run方法

### run方法

虽然在启动时直接通过该方式（`SpringApplication.run(DemoApplication.class, args);`）调用完成，但其实run方法内部还是实例化SpringApplication，然后通过new出来的实例去调用内部的run方法。

```java
//启动类的main方法
public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
}

//启动类调的run方法
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    //调的是下面的，参数是数组的run方法
    return run(new Class<?>[] { primarySource }, args);
}

//和上面的方法区别在于第一个参数是一个数组
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    //实际上new一个SpringApplication实例，调的是一个实例方法run()
    return new SpringApplication(primarySources).run(args);
}
```

看下构造器干了些什么：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    //断言primarySources不能为null，如果为null，抛出异常提示
    Assert.notNull(primarySources, "PrimarySources must not be null");
    //启动类传入的Class
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //判断当前项目类型，有三种：NONE、SERVLET、REACTIVE
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //设置ApplicationContextInitializer
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    //设置监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //判断主类，初始化入口类
    this.mainApplicationClass = deduceMainApplicationClass();
}

//判断主类
private Class<?> deduceMainApplicationClass() {
    try {
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) {
            if ("main".equals(stackTraceElement.getMethodName())) {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    }
    catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
    return null;
}
``` 

创建了SpringApplication实例之后，就完成了SpringApplication类的初始化工作，这个实例里包括监听器、初始化器，项目应用类型，启动类集合，类加载器。

得到SpringApplication实例后，接下来就调用实例方法run()。继续看代码：

```java
public ConfigurableApplicationContext run(String... args) {
    //创建计时器
    StopWatch stopWatch = new StopWatch();
    //开始计时
    stopWatch.start();
    //定义上下文对象
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    //Headless模式设置
    configureHeadlessProperty();
    //加载SpringApplicationRunListeners监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    //发送ApplicationStartingEvent事件
    listeners.starting();
    try {
        //封装ApplicationArguments对象
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        //配置环境模块
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        //根据环境信息配置要忽略的bean信息
        configureIgnoreBeanInfo(environment);
        //打印Banner标志
        Banner printedBanner = printBanner(environment);
        //创建ApplicationContext应用上下文
        context = createApplicationContext();
        //加载SpringBootExceptionReporter
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                                                         new Class[] { ConfigurableApplicationContext.class }, context);
        //ApplicationContext基本属性配置
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        //刷新上下文
        refreshContext(context);
        //刷新后的操作，由子类去扩展
        afterRefresh(context, applicationArguments);
        //计时结束
        stopWatch.stop();
        //打印日志
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        //发送ApplicationStartedEvent事件，标志spring容器已经刷新，此时所有的bean实例都已经加载完毕
        listeners.started(context);
        //查找容器中注册有CommandLineRunner或者ApplicationRunner的bean，遍历并执行run方法
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        //发送ApplicationFailedEvent事件，标志SpringBoot启动失败
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        //发送ApplicationReadyEvent事件，标志SpringApplication已经正在运行，即已经成功启动，可以接收服务请求。
        listeners.running(context);
    }
    catch (Throwable ex) {
        //报告异常，但是不发送任何事件
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

run方法流程如图：

![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/20220120185054.png)

其实不需要特别细致，吧springboot启动流程内用到的解耦思路，以及编码过程中用到的设计模式学到手，那真是你的了。

说一下我的理解，run方法是springboot启动的核心流程，一个服务要想启动需要具备很多东西，比如监听器、ioc容器的创建、tomcat容器的加载等，springboot通过观察者模式，以事件发布的形式通知，降低耦合，易于扩展，思考一个问题，springboot是如何实现事件发布进行一系列动作的？

参考连接：https://juejin.cn/post/6895341123816914958#heading-8

## springboot如何实现事件发布的

springboot事件是通过监听器模式实现的，监听器并不是23中设计模式中的，它是基于观察者模式衍生的一种编码模式。

既然监听器模式是这样实现，我觉得应该先从观察者模式说起，观察者模式顾名思义，观察目标某种现象，如果发生某些变化，观察者需要根据变化做点什么，最简单的生活例子，比如你过马路，红灯了就要等一等，绿灯了可以通过。观察者模式用于建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应作出反应。在观察者模式中，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间可以没有任何相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，关于观察者模式的细节代码实现，可参考`csdn作者刘伟`关于观察者模式的例子，[链接](https://blog.csdn.net/LoveLion/article/details/7720232)

这时如果回忆起或者刚了解观察者模式后，再来看监听器模式，该模式内有三个角色，事件、监听器、广播器，事件就是用来维护某个事件的状态，监听器用于针对事件做一些动作（个人理解就是观察者模式中的观察者角色），广播器是什么呢，一个事件可能需要多个监听器去分别完成所需的工作，那这些监听器可以放在广播器里面维护，当某个事件状态发生改变，触发这个广播器去广播通知所有监听器干该干的活就ok了。这里思考一个问题，既然一个事件可以使多个监听器有兴趣，那在执行时如果需要多个监听器顺序执行怎么办？还记得spring中的@Order注解或者Ordered接口吗，它就是用来干这个的，这样执行起来有点像责任链的意思了。

废话不多讲，通过案例了解监听器的运行方式，我们以天气事件为例，在下雪时我们希望通知人们注意添衣，下雨时我们希望提醒人们带伞，show me the code：

首先定义天气事件：

```java
/**
 * 抽象天气事件
 *
 * @author pk.zhang
 * @date 2022/1/25 14:22
 */
public abstract class WeatherEvent {

    protected abstract String getWeather();
}

/**
 * 下雪事件
 *
 * @author pk.zhang
 * @date 2022/1/25 14:23
 */
public class SnowEvent extends WeatherEvent {

    @Override
    protected String getWeather() {
        return "下雪了";
    }
}

/**
 * 下雨事件
 *
 * @author pk.zhang
 * @date 2022/1/25 14:23
 */
public class RainEvent extends WeatherEvent {

    @Override
    protected String getWeather() {
        return "下雨了";
    }
}
```

定义天气监听器：

```java
/**
 * 天气监听器接口
 *
 * @author pk.zhang
 * @date 2022/1/25 14:25
 */
public interface WeatherListener<E extends WeatherEvent> {

    void onWeatherEvent(E event);
}

/**
 * 下雪监听器
 *
 * @author pk.zhang
 * @date 2022/1/25 14:27
 */
public class SnowListener implements WeatherListener {

    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof SnowEvent) {
            event.getWeather();
            System.out.println("今天下雪！请注意适当增加衣物，做好保暖措施。");
        }
    }
}

/**
 * 下雨事件监听器
 *
 * @author pk.zhang
 * @date 2022/1/25 14:38
 */
public class RainListener implements WeatherListener {

    @Override
    public void onWeatherEvent(WeatherEvent event) {
        if (event instanceof RainEvent) {
            event.getWeather();
            System.out.println("今天下雨！请注意雨天路滑，准备好雨具设备。");
        }
    }
}
```

这个事件监听器定义好了，需要通过广播器实现对应天气执行对应监听器逻辑：

```java
/**
 * 实现天气事件的广播 典型的模板方法模式
 *
 * @author pk.zhang
 * @date 2022/1/25 14:47
 */
public class WeatherEventMulticaster extends AbstractMulticaster {

    @Override
    void doStart() {
        System.out.println("开始广播天气预报！");
    }

    @Override
    void doEnd() {
        System.out.println("广播结束！Over！");
    }
}

/**
 * 抽象类实现广播接口
 *
 * @author pk.zhang
 * @date 2022/1/25 14:41
 */
public abstract class AbstractMulticaster implements EventMulticaster {

    //存放监听器的集合，所有需要监听的事件都存在这里
    private List<WeatherListener> listenerList = new ArrayList<>();

    @Override
    public void multicastEvent(WeatherEvent event) {
        //采用模板方法，子类可以实现的doStart和doEnd，在调用监听器之前和之后分别作出扩展
        //SpringBoot中有着大量相似的操作
        //SpringBoot中的前置处理器和后置处理器，就是这样实现的
        doStart();
        listenerList.forEach(lt -> lt.onWeatherEvent(event));
        doEnd();
    }

    @Override
    public void addListener(WeatherListener weatherListener) {
        listenerList.add(weatherListener);
    }

    @Override
    public void removeListener(WeatherListener weatherListener) {
        listenerList.remove(weatherListener);
    }

    abstract void doStart();

    abstract void doEnd();
}

/**
 * 实现天气事件的广播 典型的模板方法模式
 *
 * @author pk.zhang
 * @date 2022/1/25 14:47
 */
public class WeatherEventMulticaster extends AbstractMulticaster {

    @Override
    void doStart() {
        System.out.println("开始广播天气预报！");
    }

    @Override
    void doEnd() {
        System.out.println("广播结束！Over！");
    }
}
```

通过main方法执行一下：

```java
public static void main(String[] args) {
        // 创建广播器
        WeatherEventMulticaster weatherEventMulticaster = new WeatherEventMulticaster();
        // 创建监听器
        RainListener rainListener = new RainListener();
        SnowListener snowListener = new SnowListener();
        // 向广播器内添加监听器
        weatherEventMulticaster.addListener(rainListener);
        weatherEventMulticaster.addListener(snowListener);

        // 创建事件
        RainEvent rainEvent = new RainEvent();
        SnowEvent snowEvent = new SnowEvent();

        // 广播事件
        weatherEventMulticaster.multicastEvent(rainEvent);
        weatherEventMulticaster.multicastEvent(snowEvent);
    }
```

控制台输出：

```bash
开始广播天气预报！
今天下雨！请注意雨天路滑，准备好雨具设备。
广播结束！Over！
开始广播天气预报！
今天下雪！请注意适当增加衣物，做好保暖措施。
广播结束！Over！
```

springboot监听器实现大同小异，以上是硬编码方式调用的，在springboot中做了一些细节上的复杂操作，比如每次广播时，会首先找到对该事件感兴趣的监听器后再去遍历，而以上代码作为demo，只是暴力遍历所有监听器去完成动作。从以上可以看出，是广播器把事件推送给所有的监听器，每个监听器都对事件做出判断和处理。

springboot监听器使用，从源码来看，是先注册后触发，注册流程通过实例化SpringApplication时完成，代码如下：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
       //这里是ApplicationContextInitializer接口注册
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
       //这里就是ApplicationListener注册的位置，可以看出主要区别就是查询的接口类不同
       //setListeners是找到的对象存到容器中，存到一个list属性中，方便以后使用
       //这个存放对象的list，对应的是小demo的AbstractEventMulticaster类中list，作用是一样一样的
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
}
```

实例化完`SpringApplication`之后，运行run方法，可以看到run方法内的listener的一些方法去实现事件广播，和上面的例子如出一辙，只不过spring支持同意监听器可以监听多个事件，这块在spring内部做了相对多的处理工作，不做赘述。

有关详情可参考连接：[https://juejin.cn/post/6844904106847371271](https://juejin.cn/post/6844904106847371271)

## beanfacroty和applicationContext区别

BeanFactory和ApplicationContext都是接口，并且ApplicationContext间接继承了BeanFactory。

BeanFactory是Spring中最底层的接口，提供了最简单的容器的功能，只提供了实例化对象和获取对象的功能，而ApplicationContext是Spring的一个更高级的容器，提供了更多的有用的功能。  

ApplicationContext提供的额外的功能：获取Bean的详细信息(如定义、类型)、国际化的功能、统一加载资源的功能、强大的事件机制、对Web应用的支持等等。

加载方式的区别：BeanFactory采用的是延迟加载的形式来注入Bean；ApplicationContext则相反的，它是在Ioc启动时就一次性创建所有的Bean,好处是可以马上发现Spring配置文件中的错误，坏处是造成浪费。

## springboot中的tomcat是如何运行的

SpringBoot的启动是通过new SpringApplication()实例来启动的，启动过程主要做如下几件事情： > 1. 配置属性 > 2. 获取监听器，发布应用开始启动事件 > 3. 初始化输入参数 > 4. 配置环境，输出banner > 5. 创建上下文 > 6. 预处理上下文 > 7. 刷新上下文 > 8. 再刷新上下文 > 9. 发布应用已经启动事件 > 10. 发布应用启动完成事件

而启动Tomcat就是在第7步中“刷新上下文”；Tomcat的启动主要是初始化2个核心组件，连接器(Connector)和容器（Container），一个Tomcat实例就是一个Server，一个Server包含多个Service，也就是多个应用程序，每个Service包含多个连接器（Connetor）和一个容器（Container),而容器下又有多个子容器，按照父子关系分别为：Engine,Host,Context,Wrapper，其中除了Engine外，其余的容器都是可以有多个。

## @Autowried和@Resource区别

1. @Autowried是spring框架内的注解，@Resource是jdk内自带的注解
2. @Autowired默认按byType装配Bean，如果发现多个类型相同的Bean，再根据byName（属性字段名）装配Bean，如果找到了则装配成功，找不到则装配失败，所以说在一个接口多个实现类的bean，除了使用@Qualifier指定bean名称之外，也可以通过属性名定义要使用的具体的bean名称；@Resource默认按byName装配Bean，如果byName没有找到对应的Bean，再根据byType装配Bean，如果找到了则装配成功，找不到则装配失败。

@Resource装配顺序
1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；

# bean的作用域



a、Singleton （缺省作用域、单例类型）
在容器启动（实例化）时Bean就实例化和初始化，并且只实例化一次。

b、Prototype （原型类型）
容器启动时并没有实例化Bean，只有获取Bean时才会被创建，并且每一次都是新建一个对象。

c、request（web的Spring ApplicationContext下）
每个HTTP 都会有自己的Bean，当处理结束时，Bean销毁。

d、session（web的Spring ApplicationContext下）
每一个Http session有自己的Bean

e、global session（web的Spring ApplicationContext下）
global session作用域类似于标准的HTTP Session作用域，不过仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内，也不怎么用，随便说说就行。