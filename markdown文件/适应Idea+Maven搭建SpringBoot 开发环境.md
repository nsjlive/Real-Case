适应Idea+Maven搭建SpringBoot 开发环境

集成Mybatis数据库

实现秒杀项目

Jdk1.8

------

#### 一、应用SpringBoot完成基础项目搭建

1. 使用Idea创建MAVEN项目；
2. 引入SpringBoot依赖包实现简单的WEB项目

> Pom 文件引入parent  和dependencies
>
> @EnableAutoConfiguation 开启 开启SpringBoot 的自动化配置；
>
> @RestConcoller  返回值对应的是json数据

```java
@RequestMapping("/")当用户访问根路径的时候，输出一个方法
public String home(){
	return "helloWorld";
	}
```

3. Mybatis接入SpringBoot项目；

> 在resource创建的application.properties，修改sort端口
>
> 引入Myslq依赖，采用阿里巴巴的德鲁伊**连接池**com.alibaba  / druid /1.1.3
>
> 下载SpringBoot对mybatis的支持 org.mybatis.spring.boot
>
> 创建mapping目录

3. Mybatis自动生成器使用方式

自动生成对应数据库的映射 

> mybatis-generator.xml  从mybatis官网下载；
>
> 创建mysql数据库；
>
> <数据库链接地址账号和密码>
>
> connectionURL="jdbc:mysql"   password   User     本地数据库的用户名和密码；
>
> <生成DataObject 类存放位置>
>
> <生成对应表及类名>

#### 二、用户模块开发

1. 使用SpringMvc 开发用户信息；

需要：controller 和sercive层（两个文件夹）

controller 层：

![1557652598582](C:\Users\n\Desktop\markdown文件\assets\1557652598582.png)

UserService接口

![1557652704874](C:\Users\n\Desktop\markdown文件\assets\1557652704874.png)

usermodel 

![1557652785923](C:\Users\n\Desktop\markdown文件\assets\1557652785923.png)

2. 定义通用的返回对象，返回正确信息；

![1557653070858](C:\Users\n\Desktop\markdown文件\assets\1557653070858.png)

3. 定义通用的返回对象，返回错误信息；

status:定义状态  success  fail  

使用通用错误码

​	![1557653246785](C:\Users\n\Desktop\markdown文件\assets\1557653246785.png)

4. 定义通用的返回对象，异常处理；

定义通用化解决方式

![1557653433188](C:\Users\n\Desktop\markdown文件\assets\1557653433188.png)

5. 用户模块管理--otp验证码获取

![1557653773199](C:\Users\n\Desktop\markdown文件\assets\1557653773199.png)

![1557653827698](C:\Users\n\Desktop\markdown文件\assets\1557653827698.png)

6. 用户模型管理--Metronic 模板简介；

7. 用户模型管理--getotp 页面实现、页面美化；

前端部分

![1557654016046](C:\Users\n\Desktop\markdown文件\assets\1557654016046.png)

1. 
2. 用户模型管理--用户注册功能实现；
3. 用户模型管理--用户登陆功能实现；

