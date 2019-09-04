### 一、springboot 

Spring Boot是一个简化Spring开发的框架。用来监护spring应用开发，约定大于配置，去繁就简.

我们在使用Spring Boot时只需要配置相应的Spring Boot就可以用所有的Spring组件，简单的说，spring boot就是**整合了很多优秀的框架**，**不用我们自己手动的去写一堆xml配置然后进行配置**。从本质上来说，Spring Boot就是Spring,它做了那些没有它你也会去做的**Spring Bean**配置。

**3.单体应用与微服务**

单体应用是把所有的应用模块都写在一个应用中，导致项目越写越大，模块之间的耦合度也会越来越高。微服务是一种架构风格，用微服务可以将应用的模块**单独部署**，**对不同的模块进行不同的管理操作**，不同的模块生成小型服务，每个功能元素最后都可以成为一个可以独立替换、独立升级的功能单元，**各个小型服务之间通过http进行通信**。

**Spring Boot中的特殊注解**

**@SpringBootApplication是一个复合注解，包括@ComponentScan，@SpringBootConfiguration，@EnableAutoConfiguration**

**1.@SpringBootConfiguration**继承自@Configuration，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。

**2.@EnableAutoConfiguration**的作用启动自动的配置。

**3.@EnableAutoConfiguration**注解的意思就是Springboot根据你添加的jar包来配置你项目的默认配置，比如根据spring-boot-starter-web，来判断你的项目是否需要添加了webmvc和tomcat，就会自动的帮你配置web项目中所需要的默认配置。在下面博客会具体分析这个注解，快速入门的demo实际没有用到该注解。

**4.@ComponentScan**，扫描当前包及其子包下被@Component，@Controller，@Service，@Repository注解标记的类并纳入到spring容器中进行管理。是以前的<context:component-scan>（以前使用在xml中使用的标签，用来扫描包配置的平行支持）。所以本demo中的User为何会被spring容器管理

**@ResponseBody**

表示该方法的返回结果直接写入HTTP response body中，一般在**异步获取数据时使用**，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析为跳转路径，加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上@Responsebody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用

**@Controller**用于定义控制器类，在spring项目中**由控制器负责将用户发来的URL请求转发到对应的服务接口（service层**），一般这个注解在类中，通常方法需要**配合注解@RequestMapping**

**@Bean**

用@Bean标注方法等价于XML中配置的bean。

**@Value**

注入Spring boot application.properties配置的属性的值。

### 二、SpringMVC框架理解

[SpringMVC框架理解](<https://blog.csdn.net/litianxiang_kaola/article/details/79169148>)
  MVC设计模式的任务是将包含**业务数据的模块**与**显示模块**的视图解耦。

这是怎样发生的？在模型和视图之间**引入重定向层**可以解决问题。此重定向层是**控制器**，控制器将接收请求，执行**更新模型**的操作，然后**通知****视图**关于模型更改的消息。

### 三、mysql优化（建索引）

###### 索引  [索引检索为什么快？](<https://juejin.im/post/5c2c53396fb9a04a053fc7fe>)

关键字与数据的映射关系称为索引（==包含关键字和对应的记录在磁盘中的地址==）。关键字是从数据当中提取的用于标识、检索数据的特定内容。

<https://blog.csdn.net/qq_42262803/article/details/88728050>

<https://juejin.im/post/5c2c53396fb9a04a053fc7fe>

<https://cloud.tencent.com/developer/article/1004912>



```
package demo5;



public class test00 {
    public static void main(String[] args) {
        String str="1222333";
        String nStr;
        nStr=removeDuplucate(str);
        System.out.println(nStr);
    }

    public  static String removeDuplucate(String str){
        char[] chars=str.toCharArray();
        int len=chars.length;
        if(chars.length==0)
            return new String(chars);
        int flag=0;//标志位判断
        int count=0;//count 为出现的次数
        for(int i=0;i<len;i++){
            if(chars[flag]!=chars[i]){
                flag++;
                chars[flag]=chars[i];
                flag++;

                count++;
            }else{
                if(count<2){
                    chars[flag]=chars[i];
                    count++;
                }
            }
        }
        System.out.println(flag);
    return new String(chars,0,flag+1);
       
    }
}

```

