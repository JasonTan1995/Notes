## Spring Boot 自定义处理错误页面

### 1. 前言

***

大多数的网站中,当你输入一条不存在的路径时,返回的页面大多数都是自定义的错误页面.譬如:`Spring`的官网路径找不到的页面是这样的:

![](https://spring.io/img/404-icon.png) Page Not Found

还有这样的:

![](https://pic1.zhimg.com/v2-4ea8efd8c998a4c74ef44573ab1f242d_r.jpg)

类似这样的提示就会显得对用户很友好,因为普通用户不一定是一个程序员,不一定能看得懂状态码(404,500).因此这样友好提示的页面就显得非常必要了.

而`Spring Boot`默认的错误页面是:

![](https://pic2.zhimg.com/80/v2-2c58ee116868ea42e73d93442d7dff55_hd.jpg)

相比之下就显得十分不友好了

那么`Spring Boot`中想要出错时跳转到自定义的页面,需要怎么做呢?


### 2. 解决方法

***

网上流传着非常多的方法,下面也会逐一介绍,在这之前我觉得要先看`Spring Boot`的文档是怎么解决的:

#### 2.1 `Spring Boot`文档的解决方案
[![ZeU7Xq.png](https://s2.ax1x.com/2019/06/26/ZeU7Xq.png)](https://imgchr.com/i/ZeU7Xq)

- 大概意思就是说:我们自定义的错误页面必须要放在`resources`文件夹下的`error`文件夹中,`Spring Boot`会根据状态码去查找`error`文件夹中对应的html文件.而html的名字也应该要跟状态码一致

- 例如:想要映射到404的html文件,你的文件夹结构应该如下:

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 404.html
             +- <other public assets>
```
- 而如果你使用的模板引擎是`FreeMarker`的话,你的文件夹结构应该为:
```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftl
             +- <other templates>
```
- 如果你还有制定更多复杂的映射的话,可以实现`ErrorViewResolver`接口:
```java
public class MyErrorViewResolver implements ErrorViewResolver {

	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request,
			HttpStatus status, Map<String, Object> model) {
      // Use the request or status to optionally return a ModelAndView
      Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
      if (statusCode == 404) {
         return new ModelAndView("/error/404");
      } else if (statusCode == 405) {
         return new ModelAndView("/error/405");
      } else if (statusCode == 500) {
         return new ModelAndView("/error/500");
      }
      return null;
	 }
}
```
- 而对于不使用`Sprint MVC`的情况,`Spring Boot`也提供了一个接口`ErrorPageRegistrar`来让用户来映射错误页面.
```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
	return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}
}
```

#### 2.2 实现ErrorController接口

其实该种方法是根据官方文档中的一段话得出来的.

[![ZeKNes.png](https://s2.ax1x.com/2019/06/26/ZeKNes.png)](https://imgchr.com/i/ZeKNes)

大概意思就是:可以参考`BasicErrorController`来做扩展,扩展自定义的错误处理.根据类型的不同来选择不同的返回值(视图,或者是JSON数据).

- 然后根据源码中可以看出`BasicErrorController`是继承自`AbstractErrorController`,`AbstractErrorController`实现了`ErrorController`接口.

```java
@FunctionalInterface
public interface ErrorController {
    String getErrorPath();
}
```

- 自定义的实现:
```java
@Controller
public class MyErrorController implements ErrorController {

    @RequestMapping("/error")
    public String handleError(HttpServletRequest request){
        //获取statusCode:401,404,500
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if(statusCode == 401){
            return "/401";
        }else if(statusCode == 404){
            return "/404";
        }else if(statusCode == 403){
            return "/403";
        }else{
            return "/500";
        }

    }
    
    @Override
    public String getErrorPath() {
        return "/error";
    }
}
```

#### 2.3 `Spring Boot` 2.0 之前的处理方法
 
这种方法是参考网上的,是关于`Spring Boot2.0`之前处理自定义错误页面的解决方法.

```java
//统一页码处理配置
	@Bean
	public EmbeddedServletContainerCustomizer containerCustomizer() {
	    return new EmbeddedServletContainerCustomizer() {
	        @Override
	        public void customize(ConfigurableEmbeddedServletContainer container) {
	            //ErrorPage error401Page = new ErrorPage(HttpStatus.UNAUTHORIZED, "/401.html");
	            ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/Err404.html");
	            ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/Err500.html");
 
	            container.addErrorPages( error404Page, error500Page);
	        }
	    };
	}
```

### 总结

一般来说,只需要把页面的名字改成相对应的状态码放在`resources\templates\error`下,当错误出现时`Spring Boot`就会默认去该路径下查找对应的页面.
而如果有更多的页面需要处理(映射),也可自行实现`Spring Boot`所提供的一些接口.总的来说`Spring Boot`对于开发者想自定义的一些东西还是很友好的.

#### 参考文章

[SpringBoot2.0如何自定义处理/error、404](https://zhuanlan.zhihu.com/p/43352579)

[Developing Web Applications-Custom Error Page](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-template-engines)


