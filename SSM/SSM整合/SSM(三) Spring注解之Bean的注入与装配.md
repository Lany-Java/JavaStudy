**目录：**
- [Spring与Bean注入和装配相关的注解详解](#spring与bean注入和装配相关的注解详解)
  * [bean的注入](#bean的注入)
    + [@Component](#component)
    + [@Repository](#repository)
    + [@Service](#service)
    + [@Controller](#controller)
    + [@Configuration](#configuration)
    + [@Bean](#bean)
    + [@Bean和@Component](#bean和component)
    + [@ComponentScan](#componentscan)
    + [@Qualifier](#qualifier)
    + [@Scope](#scope)
  * [bean的装配](#bean的装配)
    + [@Autowired](#autowired)
    + [@Resource](#resource)
    + [@Autowired和@Resource](#autowired和resource)
    + [@Primary](#primary)
    + [@Priority](#priority)


# Spring与Bean注入和装配相关的注解详解

## bean的注入

**当使用了SpringBean注入的注解后，我们需要在配置文件中添加<context:component-scan/>来扫描添加了注解的类，这样子声明注解的类才能起作用**

注解|注解|
:---:|:---:|
@Qualifier|@Component|
@Scope|@Repository|
@Configuration|@Service|
@Bean|@Controller|
@ComponentScan|




### @Component

作用：**标注此类是一个普通的Bean，并注入到Spring框架中进行管理**

- @Component主要用于将一个Java类注入到Spring框架中，它相当于XML配置文件中的< bean id=”xxx” class=”xxx”/ >
- Bean实例的名称默认是Bean类的首字母小写，其他部分不变
- 也可以通过设置注解的value属性来自主设定bean的名称

### @Repository

作用：**标注此类是一个Dao组件类，并注入到Spring框架中进行管理**

- 与@Component的用法相同，只是标注的类表示的是Dao层组件

### @Service

作用：**标注此类是一个Service组件类，并注入到Spring框架中进行管理**

- 与@Component的用法相同，只是标注的类表示的是Service层组件

### @Controller

作用：**标注此类是一个Controller组件类，并注入到Spring框架中进行管理**

- 与@Component的用法相同，只是标注的类表示的是Controller层组件

### @Configuration

作用：**配置spring容器(应用上下文)**
- @Configuration标注在类上，相当于把该类作为spring的xml配置文件中的\<beans\>,即将此类作为Spring容器
- 如果将一个类标注为@Configuration注解，那么通常也就意味着这个class将会作为创建各种bean的工厂（类似于一个新的容器）
- 使用@Configuration之后，因为类被作为了Spring的容器，所以在加载的时候，**用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext**

  ```
  package com.test.spring.support.configuration;

  @Configuration
  public class TestConfiguration {
      public TestConfiguration(){
          System.out.println("spring容器启动初始化。。。");
      }
  }
  ```
  相当于：
  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context" 
      xmlns:jdbc="http://www.springframework.org/schema/jdbc"  
      xmlns:jee="http://www.springframework.org/schema/jee" 
      xmlns:tx="http://www.springframework.org/schema/tx"
      xmlns:util="http://www.springframework.org/schema/util" 
      xmlns:task="http://www.springframework.org/schema/task" 
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
          http://www.springframework.org/schema/context 
          http://www.springframework.org/schema/context/spring-context-4.0.xsd
          http://www.springframework.org/schema/jdbc 
          http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
          http://www.springframework.org/schema/jee 
          http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
          http://www.springframework.org/schema/tx 
          http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
          http://www.springframework.org/schema/util 
          http://www.springframework.org/schema/util/spring-util-4.0.xsd
          http://www.springframework.org/schema/task 
          http://www.springframework.org/schema/task/spring-task-4.0.xsd" default-lazy-init="false">


  </beans>
  ```
  主方法进行测试：
  ```
  package com.test.spring.support.configuration;

  public class TestMain {
      public static void main(String[] args) {

          //@Configuration注解的spring容器加载方式，用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext
          ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);

          //如果加载spring-context.xml文件：
          //ApplicationContext context = new ClassPathXmlApplicationContext("spring-context.xml");
      }
  }
  ```

### @Bean

作用：**注册bean对象**
- @Bean注解在**返回实例**的方法上，如果未通过@Bean指定bean的名称，则默认与标注的方法名相同
- @Bean将方法返回的实例对象注册为Bean，交由Spring管理，由此它通常和@Configuration结合使用，创建工厂类
- 使用@Bean注解的好处就是能够**动态获取一个Bean对象，能够根据环境不同得到不同的Bean对象**，所以易与@Configuration结合使用
- @Bean注解默认作用域为单例singleton作用域，可通过@Scope(“prototype”)设置为原型作用域
- @Bean注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理。产生这个Bean对象的方法Spring只会调用一次，随后这个Spring将会将这个Bean对象放在自己的IOC容器中

  bean类：
  ```
  package com.test.spring.support.configuration;

  public class TestBean {

      public void sayHello(){
          System.out.println("TestBean sayHello...");
      }

      public String toString(){
          return "username:"+this.username+",url:"+this.url+",password:"+this.password;
      }

      public void start(){
          System.out.println("TestBean 初始化。。。");
      }

      public void cleanUp(){
          System.out.println("TestBean 销毁。。。");
      }
  }
  ```
  配置类：
  ```
  package com.test.spring.support.configuration;

  @Configuration
  public class TestConfiguration {
          public TestConfiguration(){
              System.out.println("spring容器启动初始化。。。");
          }

      //@Bean注解注册bean,同时可以指定初始化和销毁方法
      //@Bean(name="testNean",initMethod="start",destroyMethod="cleanUp")
      @Bean
      @Scope("prototype")
      public TestBean testBean() {
          return new TestBean();
      }
  }
  ```
  主方法测试类：
  ```
  package com.test.spring.support.configuration;

  public class TestMain {
      public static void main(String[] args) {
          ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
          //获取bean
          TestBean tb = context.getBean("testBean");
          tb.sayHello();
      }
  }
  ```
  
### @Bean和@Component

- @Component用在类上，@Bean用在返回实例的方法上
- @Component可以在任意类上使用，@Bean通常在配置类中使用，也就是类上需要加上@Configuration注解
- 二者都可以用过@Autowired装配

那为什么有了@Component还需要@Bean呢？</br>
如果你想讲第三方库中的组件装配到你的应用之中，在这种情况下，是没有办法在它的类上添加@Component注解的，因此就不能使用@Component了，但是我们可以使用
@Bean来将它装配到我们的应用之中



### @ComponentScan

作用：**自动扫描注解**
- @ComponentScan(basePackages = "com.test.spring.support.configuration")，basePackages为包路径

### @Qualifier

作用：**指定注入Bean的名称和为方法指定注入Bean名称**
- 如果容器中有一个以上匹配的Bean时，则可以通过@Qualifier注解限定注入的Bean名称
  ```
  package com.zltedu.bean;


  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.stereotype.Component;


  @Component
  public class Computer {
      @Autowired
      @Qualifier("storage01")
      private Storage storage;
      public void m01() {
          storage.show();
      }
  }
  ```

  ```
  package com.zltedu.bean;


  import org.springframework.stereotype.Component;


  @Component("storage01")
  public class Storage { //此类是在com.zltedu.bean下的Storage
      public void show() {
          System.out.println("东芝硬盘");
      }
  }
  ```

  ```
  package com.zltedu.test;


  import org.springframework.stereotype.Component;


  @Component
  public class Storage { //此类是在com.zltedu.test下的Storage

  }
  ```
- @Qualifier也可以为方法指定注入Bean名称
  ```
  package com.zlt.spring.day02;


  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.stereotype.Component;


  @Component
  public class StudentAnnotation {

      private CourseAnnotation courseAnnotation;

      /**
       * @Autowired来自动装备构造方法里面的对象
       * @Qualifier来指定依赖对象的bean
       * @paramcourseAnnotation
       */
      @Autowired
      public StudentAnnotation(@Qualifier("course") CourseAnnotation courseAnnotation) {
          this.courseAnnotation = courseAnnotation;
      }

      public void showCourse() {
          System.out.println(courseAnnotation.hashCode());
      }
  }
  ```
### @Scope

作用：**显式指定Bean作用范围**
```
@Scope("prototype")
@Component
public class Car {

}
```

----

## bean的装配

注解|
:--:|
@Autowired|
@Resource|
@Inject|

### @Autowired

作用：**可以为类的属性，构造器，方法注值**

装配策略：**先进行byType匹配，如果有多个同类型bean，再根据byName匹配，如果byName也无法确定，那么通过主和优先级（Primary和Priority）来确定**

- @Autowired默认按照类型装配，如果容器中包含多个同一类型的Bean，那么启动容器时会找不到指定类型的异常（可以结合@Qualified注解进行bean限定，指定注入bean的名称）
- 如果容器中包含多个同一类型的bean，那么接下来根据注入参数的Name，来进行匹配
  - 如果Name匹配，则成功
  - 如果Name不匹配，那么按主和优先级判断，看是否有Bean包含@Primary注解
    - 包含@Primary，则注入此Bean
    - 不包含，看是否有@Priority注解
      - 如果有，根据@Priority注解中value的大小，按优先级，数字越小的越优先匹配
      - 如果没有，匹配失败
- 默认情况下，注入的对象必须存在（bean可用），如果需要改变这种默认方式，可以设置其required属性为false


### @Resource

作用：**也是和@Autowired相似的依赖注入**

装配策略：**先进行byName匹配，如果name属性为空，然后进行byType匹配**

- 在启动Spring容器时，会默认寻找容器扫描范围内的可加载bean，然后查找哪些bean上的属性和方法上有@Resource注解
- 找到@Resourse注解后，判断@Resource注解括号中name属性是否为空
- 如果name为空，看Spring容器中的bean的id与@Resource要注解的那个变量属性名是否相同
  - 如果相同，则匹配成功
  - 如果不相同，看Spring容器中bean的id对应的类型是否与@Resource要注解的变量属性对应的类型相等
    - 如果相等，匹配成功
    - 如果不相等，匹配失败
- 如果name不为空，看name的属性值和容器中的bean的id名是否相等
  - 如果相等，匹配成功
  - 如果不相等，匹配失败

### @Autowired和@Resource

同：
- @Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上

异：
- @Autowired默认按类型装配，@Resource默认按照名称进行装配
- @Autowired是Spring提供的，@Resource是JDK提供的

### @Primary

作用：**此注解标记的bean在多选择的情况下，会优先注入**

- 如果在@Autowired自动装配的时候，有两个符合条件的bean，则有@Primary标注的bean被注入

### @Priority

作用：**配置bean的优先级**
- 如果在@Autowired自动装配的时候，有两个符合条件的bean，又没有@Primary，则按@Priority标注的优先级大小进行优先匹配，数字越小，越优先匹配

> 参考文章 [spring 常用注解](http://www.voidcn.com/article/p-vnuuxhnq-bcq.html)</br>
> 参考文章 [Spring @Bean注解使用](https://juejin.im/entry/597834286fb9a06bad65749f)</br>
> 参考文章 [Spring整理系列(11)——@Configuration注解、@Bean注解以及配置自动扫描、bean作用域](https://blog.csdn.net/javaloveiphone/article/details/52182899)</br>
> 参考文章 [精进Spring—Spring常用注解【经典总结】](https://blog.csdn.net/u010648555/article/details/76299467)</br>
> 参考文章 [Spring常用注解介绍](https://zhuanlan.zhihu.com/p/43235193)</br>
> 参考文章 [@Resource注解用法](https://blog.csdn.net/u010502101/article/details/78950045)</br>
> 参考文章 [彻底搞明白Spring中的自动装配和Autowired](https://juejin.im/post/5c84b5285188257c5b477177)</br>
