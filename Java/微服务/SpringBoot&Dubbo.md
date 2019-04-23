### Spring Boot&Dubbo

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz8myl7anvj210n0g9abk.jpg)

#### 摘要

​         本文只会对Spring Boot与Dubbo环境搭建进行详细地讲解,但不会对Spring Boot与Dubbo展开详细地论述.

***

#### 1. 基本概念

​           这里主要是理清Dubbo的基本概念,下面通过一张图来理解下.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz8ndvn008j20es0bidg9.jpg)

Dubbo中最重要的三个概念可以在这张图体现出来:

1. Provider:服务提供者
2. Consumer:服务消费者
3. Registry:注册中心

想了解更新详细的过程可以去官网:

http://dubbo.apache.org/zh-cn/docs/user/preface/architecture.html



#### 2. 环境搭建

***

##### 2.1 基本环境参数

- JDK:1.8
- Maven : 3.5.3
- IDE: IntelliJ IDEA
- 操作系统: Windows 8.1

其实官方跟Alibaba在github上的仓库都有Spring Boot跟Dubbo整合的Demo.我只是记录整合的过程以及会碰到的一些问题.

apache incubator-dubbo-spring-boot:https://github.com/apache/incubator-dubbo-spring-boot-project

alibaba-dubbo-spring-boot-starter:https://github.com/alibaba/dubbo-spring-boot-starter

##### 2.2  创建父工程

- 使用Idea中的Spring Initializr创建Spring Boot 父工程.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9oodbyebj20uh0l8mxx.jpg)

- 填写GroupId.ArtifactId.然后不需要添加任何模块,一直点击Next即可.
- 创建成功后的目录结构:(请忽略红色划掉的部分,那是下面会创建的模块.把src的部分删掉.)

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9orqn061j20a00990sv.jpg)

- 查看父工程的Pom.xml文件(可以知道其实每个Spring Boot工程都默认有一个父工程)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.springboot-dubbo</groupId>
    <artifactId>parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>parent</name>
    <description>Spring Boot Dubbo</description>
    <modules>
        <module>provider</module>
    </modules>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

##### 2.3 创建子工程-Provider

- 步骤跟创建父工程的一样.点击新建模块

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9p4xueuij20eg0hgt9y.jpg)

- 创建完成后的目录结构

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9pnnboo0j209k08sjrf.jpg)

- 创建完成记得在父工程的pom.xml中添加以下的配置:

```xml
<modules>
        <module>provider</module>
</modules>
```



##### 2.4 创建子工程-Consumer

- 创建工程的步骤还是一样的.但这次在选择模块的时候加入Web.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9si0vaf1j20uh0l8jsi.jpg)

- `创建完成后记得在父工程中添加配置`

```xml
<modules>
        <module>provider</module>
        <module>consumer</module>
</modules>
```

- 这样的话基本环境就差不多搭建好了.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9t1rn0opj209o0bn0st.jpg)





#### 3. 基本使用例子

***

##### 3.1 Dubbo-Provider

- 通过以上的了解可以知道Dubbo中比较重要的几个概念:服务提供者,服务消费者,以及注册中心.

- 先在Provider工程中添加一个接口

```java
package com.springbootdubbo.provider.api;

public interface DemoService{

    String sayHello(String name);
}
```

- 然后实现该接口

```java
public class DemoServiceImpl implements DemoService {

    @Override
    public String sayHello(String name) {
        return "Provider say Hello:" + name;
    }
}
```

- 那么好了,现在可以加入关于Dubbo 的依赖了

```xml
<dependency>
            <groupId>com.alibaba.spring.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.0.0</version>
</dependency>
```

- 在启动类添加注解@EnableDubboConfiguration

```java
@SpringBootApplication
@EnableDubboConfiguration
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }

}
```

- 在实现中添加配置注解

```java
@Component
@Service(interfaceClass = DemoService.class)
public class DemoServiceImpl implements DemoService {

    @Override
    public String sayHello(String name) {
        return "Provider say Hello:" + name;
    }
}
```

- 最后在application.properties中配置关于Dubbo的配置

```
spring.application.name=dubbo-spring-boot-starter
spring.dubbo.server=true
spring.dubbo.registry=N/A
```

- 到此时服务提供者这边就准备得差不多了

##### 3.2 Dubbo-api

​     在开始之前,我们先思想一个问题.Consumer怎么引用到Provider的接口呢?如果将模块直接引入到Consumer那么RPC不就没有意义了吗?接口多起来的时候怎么办?

> 以上的这些问题,通过新建一个聚合工程.该聚合工程用来放置我们的接口.那么Consumer与Provider都能引用了.

- 新建工程api,与以上创建工程的步骤差不多.该工程不需要任何模块.

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9tmw2i9cj209c041q2r.jpg)

- 父工程的pom.xml

```xml
<modules>
        <module>provider</module>
        <module>consumer</module>
        <module>api</module>
    </modules>
```

- 添加接口

```java
public interface DemoService {

    String sayHello(String name);
}
```

- 把Provider里面的接口删掉并引入api的依赖

```xml
<dependency>
            <groupId>com.springboot-dubbo</groupId>
            <artifactId>api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```



##### 3.3 Dubbo-Consumer

- 添加Dubbo 依赖以及api依赖

```xml
<dependency>
            <groupId>com.alibaba.spring.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.springboot-dubbo</groupId>
            <artifactId>api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

- 对启动类添加注解以及application.properties配置

```java
@SpringBootApplication
@EnableDubboConfiguration
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

```
spring.application.name=dubbo-spring-boot-starter
```

到这里配置就差不多完成,那怎么测试成不成功呢?以下通过一个web示例来测试

##### 3.4 Dubbo-web(测试)

- 添加Controller(并添加@Reference注解)

```java
@Controller
public class HelloController {

    @Reference(url = "dubbo://127.0.0.1:20880")
    private DemoService demoServiceImpl;

    @ResponseBody
    @GetMapping("/hello")
    public String hello(String name){
        return demoServiceImpl.sayHello(name);
    }
}
```

- 先启动Provider,再启动Consumer.(因为在Dubbo中有安全检查机制)

  - http://localhost:8080/hello?name=jason 访问该url

    ![](https://ws1.sinaimg.cn/large/6b297ce5gy1fz9ujnbmn3j206t016a9t.jpg)

    看到以上的结果说明调用成功啦!