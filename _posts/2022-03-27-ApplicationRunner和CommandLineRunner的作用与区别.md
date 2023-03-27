---
title: ApplicationRunner和CommandLineRunner的作用与区别
author: Travis <Hongxu Wei>
date: 2023-03-27 12:39:52 +0800
categories: [Java Learning Space]
tags: [ApplicationRunner, CommandLineRunner]
math: false
---



## 一、应用场景

ApplicationRunner和CommandLineRunner都用于在容器启动后（也就是SpringApplication.run()执行结束）执行某些逻辑。可用于项目的一些准备工作，比如加载配置文件，加载执行流，定时任务等。

在开发过程中会有这样的场景：需要在容器启动的时候执行一些内容，如读取配置文件信息，清除缓存信息等。在Spring框架下是通过ApplicationListener监听器来实现的。在Spring Boot中，我们也可以根据下面要提到的两个接口来帮助我们实现这样的需求。这两个接口就是CommandLineRunner和ApplicationRunner，它们的执行时机是容器启动完成的时候。

## 二、共同点和区别

- 共同点：
  - 执行时机都是在容器启动完成的时候进行；
  - 这两个接口中都有一个run()方法
- 区别：
  - ApllicationRunner中run方法的参数为ApplicationArguments，而CommandLineRunner接口的run方法的参数为String数组。**这两个接口间没有很大的区别，如果想要更详细地获取命令行参数，那就使用ApplicationRunner接口。**

## 三、修改执行顺序

方案一：可以在实现类加上@Order注解指定执行的顺序

方案二：可以在实现类上实现Ordered来标识。

注意：数字越小，优先级越高，也就是@Order(1)注解的类会在@Order(2)注解的类之前执行。

## 四、事例分析

- Spring Boot启动类：

  ```java
  @SpringBootApplication
  public class Application {
      public static void main(String[] args) {
          System.out.println("Spring BOOT 启动开始");
          SpringApplication.run(Application.class,args);
          System.out.println("Spring BOOT 启动结束");
      }
  }
  ```

- 实现CommandLineRunner接口类：

  ```java
  @Component
  public class CommandLineRunnerTest implements CommandLineRunner{
      @Override
      public void run(String[] args)throws Exception{
          System.out.println("CommandLineRunner=====执行开始");
          System.out.println("hehe");
          System.out.println("CommandLineRunner=====执行完毕");
      }
  }
  ```

- 实现ApplicationRunner接口类：

  ```java
  @Component
  public class ApplicationRunerTest implements ApplicationRunner{
      @Override
      public void run(ApplicationArguments args)throws Exception{
          System.out.println("ApplicationRunner=====执行开始");
          System.out.println(args.getNonOptionArgs());
          System.out.println(args.getOptionNames());
          System.out.println("ApplicationRunner=====执行完成");
      }
  }
  ```

- 系统启动日志输出：

  ```bash
  Connected to the target VM, address: '127.0.0.1:2777', transport: 'socket'
  Spring BOOT 启动开始
   
    .   ____          _            __ _ _
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
    '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::        (v1.5.1.RELEASE)
  2019-08-11 12:33:13.941  INFO 416 --- [           main] org.spring.springboot.Application        : Starting Application on LAPTOP-CRSICKSO with PID 416 (G:\springboot-learning-example\springboot-helloworld\target\classes started by chao in G:\springboot-learning-example\springboot-helloworld)
  2019-08-11 12:33:13.947  INFO 416 --- [           main] org.spring.springboot.Application        : No active profile set, falling back to default profiles: default
  2019-08-11 12:33:14.072  INFO 416 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6892b3b6: startup date [Sun Aug 11 12:33:14 GMT+08:00 2019]; root of context hierarchy
  2019-08-11 12:33:16.253  INFO 416 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration' of type [class org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
  2019-08-11 12:33:16.412  INFO 416 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'validator' of type [class org.springframework.validation.beanvalidation.LocalValidatorFactoryBean] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
  2019-08-11 12:33:17.472  INFO 416 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
  2019-08-11 12:33:17.501  INFO 416 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
  2019-08-11 12:33:17.503  INFO 416 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.11
  2019-08-11 12:33:17.718  INFO 416 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
  2019-08-11 12:33:17.718  INFO 416 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 3656 ms
  2019-08-11 12:33:17.936  INFO 416 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
  2019-08-11 12:33:17.947  INFO 416 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
  2019-08-11 12:33:17.948  INFO 416 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
  2019-08-11 12:33:17.948  INFO 416 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
  2019-08-11 12:33:17.948  INFO 416 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
  2019-08-11 12:33:18.478  INFO 416 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6892b3b6: startup date [Sun Aug 11 12:33:14 GMT+08:00 2019]; root of context hierarchy
  2019-08-11 12:33:18.626  INFO 416 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/]}" onto public java.lang.String org.spring.springboot.web.HelloWorldController.sayHello()
  2019-08-11 12:33:18.633  INFO 416 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
  2019-08-11 12:33:18.634  INFO 416 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
  2019-08-11 12:33:18.698  INFO 416 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
  2019-08-11 12:33:18.698  INFO 416 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
  2019-08-11 12:33:18.792  INFO 416 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
  2019-08-11 12:33:19.366  INFO 416 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
  2019-08-11 12:33:19.619  INFO 416 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
  ApplicationRunner=====执行开始
  []
  []
  ApplicationRunner=====执行完成
  CommandLineRunner=====执行开始
  hehe
  CommandLineRunner=====执行完毕
  2019-08-11 12:33:19.629  INFO 416 --- [           main] org.spring.springboot.Application        : Started Application in 6.381 seconds (JVM running for 7.409)
  Spring BOOT 启动结束
  ```

- 由上述执行结果看：ApplicationRunner接口实现方法默认先于CommandLineRunner接口实现方法执行



## 附录

[Spring Boot的ApplicationRunner与CommandLineRunner接口的使用与区别](https://blog.csdn.net/chao821/article/details/99179198)