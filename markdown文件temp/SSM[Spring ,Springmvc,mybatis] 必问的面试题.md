#### SSM[Spring ,Springmvc,mybatis] 必问的面试题

###### 1. 开发中主要使用 Spring 的什么技术 ?

    . IOC 容器管理各层的组件
    . 使用 AOP 配置声明式事务
    . 整合其他框架
###### 2. 简述 AOP 和 IOC 概念

1. AOP： Aspect Oriented Program， 面向(方面)切面的编程；Filter(过滤器) 也是一种 AOP。

	 **AOP 是一种新的方法论**， 是对传统 OOP(Object-Oriented Programming， 面向对象编程) 的补充。 AOP 的主要编程对象是切面(aspect)， 而切面模块化横切关注点，可以举例通过事务说明。

2. IOC： Invert Of Control， **控制反转**。 也称为 DI(依赖注入)，其思想是**反转资源获取的方向**。

	>  **传统的资源查找方式**要求组件向容器发起请求查找资源。作为 回应， 容器适时的返回资源。
	>
	>  而应用了 IOC 之后， 则是**容器主动地将资源推送给它所管理的组件**，组件所要做的仅是选择一种合适的方式来接受资源。这种行 为也被称为查找的被动形式。

###### 3. 在 Spring 中如何配置 Bean ？

Bean 的配置方式：

> 通过全类名（反射）
> 通过工厂方法（静态工厂方法 & 实例工厂方法）
> FactoryBean

FactoryBean定义

```java
package org.springframework.beans.factory;
 
public interface FactoryBean<T> {
    T getObject() throws Exception;
 
    Class<?> getObjectType();
 
    boolean isSingleton();
}
```

FactoryBean 通常是用来创建比较复杂的bean，一般的bean 直接用xml配置即可， 但如果一个bean的创建过程中涉及到很多其他的bean 和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。

###### 4. IOC 容器对 Bean 的生命周期：

```
. 通过构造器或工厂方法创建 Bean 实例
. 为 Bean 的属性设置值和对其他 Bean 的引用
. 将 Bean 实例传递给 Bean 前置处理器的postProcessBeforeInitialization 方法
. 调用 Bean 的初始化方法(init-method)
. 将 Bean 实例传递给 Bean 后置处理器的postProcessAfterInitialization 方法
. Bean 可以使用了
. 当容器关闭时， 调用 Bean 的销毁方法(destroy-method)
```

##### 5. Spring MVC 的运行流程

1. 在整个 Spring MVC 框架中， **DispatcherServlet 处于核心位置**，负 责协调和组织不同组件以完成请求处理并返回响应的工作。
2. SpringMVC 处理请求过程：
	- 1、若一个请求匹配 DispatcherServlet 的请求映射路径(在 web.xml 中指定)， WEB 容器将该请求转交给 DispatcherServlet 处理。
	- 2、DispatcherServlet 接收到请求后， 将根据请求信息(包括 URL、HTTP 方法、请求头、请求参数、Cookie 等)及 HandlerMapping 的配置找到处理请求的处理器(Handler)。可将 HandlerMapping 看成路由控制器，将 Handler 看成目标主机。
	- 3、当 DispatcherServlet 根据 HandlerMapping 得到对应当前请求的Handler 后，通过 HandlerAdapter 对 Handler 进行封装，再以统一的适配器接口调用 Handler。
	- 4、处理器完成业务逻辑的处理后将返回一个 ModelAndView 给DispatcherServlet， ModelAndView 包含了视图逻辑名和模型数据信息。
	- 5、DispatcherServlet 借助 ViewResoler 完成逻辑视图名到真实视图对象的解析， 得到真实视图对象 View 后， DispatcherServlet 使用这个 View 对ModelAndView 中的模型数据进行视图渲染。

###### 6. 说出 Spring MVC 常用的 5 个注解

```
. @RequestMapping 、@PathVariable 、@RequestParam 、@RequestBoy 、
. @ResponseBody
```

###### 7. 如何使用 SpringMVC 完成 JSON 操作

> . 配置MappingJacksonHttpMessageConverter
> . 使用 @ResponseBody 注解或 ResponseEntity 作为返回值
> . ResponseEntity 可以定义返回的HttpHeaders和HttpStatus
> . 使用@RestController修饰控制器（Spring MVC 4）

###### 8. 相关注解

**@RequestBody**

> **处理HttpEntity传递过来的数据，**
>
> 一般用来处理非Content-Type： application/x-www-form-urlencoded编码格式的数据。
>
>  **GET请求中，因为没有HttpEntity，所以@RequestBody并不适用。** 
>
> POST请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型Content-Type，SpringMVC通过使用HandlerAdapter 配置的HttpMessageConverters来解析HttpEntity中的数据，然后绑定到相应的bean上。

**@Autowired与@Resource、@Qualifier  @Resource装配顺序**

> 1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
> 2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
> 3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
> 4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配

**主要区别**

> 1. @Autowired与@Resource都可以用来装配bean. 都可以写在字段上，或写在setter方法上。
> 2. @Autowired默认按类型装配（这个注解是属于spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false。
> 3. @Resource（这个注解属于J2EE的，是JSR规范定义的注解），默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在setter方法上默认取属性名进行装配。 当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
> 4. 推荐使用：@Resource注解在字段上，这样就不用写setter方法了，并且这个注解是属于J2EE的，减少了与spring的耦合。这样代码看起就比较优雅。
> 5. @Qualifier一般作为@Autowired()的修饰用，当使用@Autowired注入出现多个Bean的注入异常时，则需要指定注入的Bean的名称。

### 12. Spring AOP什么时候会失效？

在对象内部的方法中调用该对象的其他使用AOP机制的方法，被调用方法的AOP注解失效。

在一个类的A方法中调用同类的B方法，实际上使用的是使用实例调用方式this.B()，而代理对象是$开头的一个对象，此时的调用并不会走代理，注解会无效。 因为在被代理对象的方法中调用被代理对象的其他方法时。其实是没有用代理调用，是用了被代理对象本身调用的。

### 23. 在mapper中如何传递多个参数？

> 第一种：使用占位符的思想
>
> 在映射文件中使用#{0},#{1}代表传递进来的第几个参数,使用@param注解：来命名参数
>
> 第二种：使用Map集合作为参数来装载，根据key自动找到对应Map集合的value
>
> PS:由于一直使用SpringBoot、SpringCloud，对于SSM没有过多研究,主要是根据网络和自己的经历整理的。仅供参考



**滑动窗口**