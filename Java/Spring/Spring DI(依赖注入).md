#### Spring DI(依赖注入)

​     什么是Spring的依赖注入？

> 举个例子，例如Service层需要用到Dao层的对象，这时Service如果想使用Dao层的对象就需要用硬编码的方式new一个对象出来了，这样做的耦合度会很高。而Spring就能通过依赖注入来解决这个问题。 控制反转（Inverse of Control）和依赖注入（Dependency Inject）其实是同一个概念的不同角度，通过把对象交给Spring管理，Spring帮我们实例对象，并把对象中所需要的另一个对象依赖注入。

​      控制反转跟依赖注入是相辅相成的，它们目的都是达到解耦合。

[![iYgxv8.png](https://s1.ax1x.com/2018/10/10/iYgxv8.png)](https://imgchr.com/i/iYgxv8)

[![iY2NrD.png](https://s1.ax1x.com/2018/10/10/iY2NrD.png)](https://imgchr.com/i/iY2NrD)

##### Spring 依赖注入的三种方式

    1. Setter方法注入
    
       ##### XML配置
    
       ```java
       public class User{
          private String name；
          private int age;
          private Car car;
       
         public void setName(String name){
              this.name = name;
          }
         public void setAge(int age){
              this.age = age;
          }
         public void setCar(Car car){
              this.car =car;
          }
       }


​       
       <!--属性注入-->
       <bean name="user" class="com.test.pojo.User">
            <property name="name" value="tom"></property>
            <property name="age" value="18"></property>
            </bean>
       
       <!--对象注入-->
       <bean name="user" class="com.test.pojo.User">
            <property name="name" value="tom"></property>
            <property name="age" value="18"></property>
            <property name="Car" ref="car"></property>
        </bean>
            <bean name="car" class="com.test.pojo.Car">
             <property name="name" value="AE86"></property>
             <property name="color" value="白色"></property>
            </bean>
       ```
    
       注意:值类型使用value属性,引用类型使用ref属性.



       ##### 注解配置
    
       ```java
       public class User{
          private String name；
          private int age;
          private Car car;
         
         @Value("tom")
         public void setName(String name){
              this.name = name;
          }
         @Value("12")  
         public void setAge(int age){
              this.age = age;
          }
         @Autowired
         @Qualifier("car")
         public void setCar(Car car){
              this.car =car;
          }
       }
       ```

2.构造方法注入

##### XML配置

| 属性  | 详解                   |
| :---: | ---------------------- |
| name  | 对象的属性名           |
| value | 值                     |
|  ref  | 引用的对象             |
| index | 构造函数参数列表的顺序 |
| type  | 构造函数的参数类型     |

```java
<bean name="user" class="com.test.pojo.User">
        <constructor-arg name="name" value="666" index="1" type="java.lang.Integer"></constructor-arg>
        <constructor-arg name="car" ref="car" index="0"></constructor-arg>
     </bean>
     
    
    //构造方法
public User(String name, Car car) {
    this.name = name;
    this.car = car;
}

public User(Car car, Integer name) {
    this.name = name+"";
    this.car = car;
}
```

##### 注解配置

```java
private String name;
//private Integer name;

@Autowired
public User(String name, Car car) {
    this.name = name;
    this.car = car;
}

public User(Car car, Integer name) {
    this.name = name+"";
    this.car = car;
}
```

另外在Spring中如果使用注解配置，XML配置文件中需要开启扫描包的配置

```xml
<context:component-scan base-package="com.test" />
```

而在Springboot中就不需要以上的这个了，而且使用构造函数注入的情况也不一样了：

>You are free to use any of the standard Spring Framework techniques to define your beans and their injected dependencies. For simplicity, we often find that using`@ComponentScan` (to find your beans) and using `@Autowired` (to do constructor injection) works well.
>
>If you structure your code as suggested above (locating your application class in a root package), you can add `@ComponentScan` without any arguments. All of your application components (`@Component`, `@Service`, `@Repository`, `@Controller` etc.) are automatically registered as Spring Beans.
>
>The following example shows a `@Service` Bean that uses constructor injection to obtain a required `RiskAssessor` bean:

 你可以任意使用Spring的技术来定义的Bean以及他们的依赖注入。显而易见，我们经常会使用@ComponentScan来查找Bean，结合@Autowired的构造注入效果会是比较好的。

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	@Autowired
	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}
	// ...
}
```

如果这个Bean 有构造方法还可以省略@Autowired

```java
@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}
	// ...
}
```

这篇只是大概的说一下概念以及使用方法，以后会再写关于背后原理的文章。

参考资料：

 http://suwish.com/talk-about-spring-introduction.html

 Springboot Reference Manual：

https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-spring-beans-and-dependency-injection