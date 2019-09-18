您好，我叫倪少剑，来自湖北天门，现在就读于电子科技大学，自动化工程学院，控制工程专业，研究生三年级。研究方向是智能电网。

研究生期间参与导师的国家自然科学基金项目：电力系统脆弱机理及控制稳定研究，我参与的方面是电力系统态势预测研究，同时与教研室的博士完成一篇论文，关于电力系统无人机巡逻系统的改进（我做的主要工作是算法的具体实现	）。

假期有过一段时间去南方电网鼎信实习的经历，对软件开发有了一层新的认识（发现于学校、教研室的技术栈不一样，侧重点不一样）。

之所以投小米公司，是因为喜欢小米的文化，身边同学推崇。电子科大很多同学特别喜欢小米公司。

##### 使用SETNX实现分布式锁

多个进程执行以下Redis命令：

SETNX lock.foo <current Unix time + lock timeout + 1>

> 如果 SETNX 返回1，说明该进程获得锁，SETNX将键 lock.foo 的值设置为锁的超时时间（当前时间 + 锁的有效时间）。
> 如果 SETNX 返回0，说明其他进程已经获得了锁，进程不能进入临界区。进程可以在一个循环中不断地尝试 SETNX 操作，以获得锁。

##### 解决死锁

正常第一反应利用SETNX实现分布式锁可能是这样的

```java
if(SETNX key value){//如果设置成功表示拿到了锁
    return true;
}
return false;
```

然后释放锁的时候就直接 DEL掉；
简单思路是这样，但是这样会有很多问题

> 如果一个进程获得锁之后，断开了与redis的连接（进程挂断或者网络中断），那么锁一直的不断释放，其他的进程就一直获取不到锁，就出现了 “死锁”
>
> 在检测到锁超时后，进程不能直接简单地执行 DEL 删除键的操作以获得锁。



掌握常用的设计模式；

了解常见的分布式存储计算框架；

spring声明式事务管理默认对非检查型异常和运行**时异常进行事务回滚**，而对检查型异常则不进行回滚操作。

Java里面异常分为两大类：checkedexception(检查异常)和unchecked exception(未检查异常-也叫运行时异常**RuntimeException**)

例如：常见的运行时异常，**RuntimeException异常：**

> NullPointerException
>
> ClassCastException
>
> ArrayIndexOutOfBoundsException
>
> StringIndexOutOfBoundsException
>
> NumberFormatException
>
> ArithmeticException   零做除数

检查异常 Checked ，一般在编译器就被检出，

> ​           ClassNotFoundException
>
> ​           NosuchFieldException
>
> ​           NosuchMethodException

#### **2：spring事务回滚**

spring声明式事务管理默认对非检查型异常和**运行时异常进行事务回滚**，而对检查型异常则不进行回滚操作。

**1**)，Spring事务回滚机制是这样的：当所拦截的方法有指定异常抛出，事务才会自动进行回滚！

我们需要注意的地方有四点： 如果你在开发当中引入Spring进行事务管理，但是**事务没能正常的自动回滚**，可以对照下面四点，缺一不可！

> ①被拦截方法-—— 注解式：方法或者方法所在类被@Transactional注解；
>
> 拦截配置式：<tx:method />应该包含对该方法，名称格式的定义；且方法需要在expression定义的范围内；
>
> ②异常—— 该方法的执行过程必须出现异常，这样事务管理器才能被触发，并对此做出处理；
>
> ③指定异常——  默认配置下，事务只会对Error与RuntimeException及其子类这些UNChecked异常，做出回滚。  一般的Exception这些Checked异常不会发生回滚（如果一般Exception想回滚要做出配置）；
>
> ④异常抛出—— 即方法中出现的指定异常，只有在被事务管理器捕捉到以后，事务才会据此进行事务回滚；
>
> 1，不捕捉，会回滚：没有trycatch
>
> 2，如果异常被try{}捕捉到，那么事务管理器就无法再捕捉异常，所以就无法做出反应，事务不回滚；
>
> 3，如果异常被try{}捕捉了，我们还可以在Catch(){}中throw   new  RuntimeException()，手动抛出运行时异常供事务管理器捕捉；    
>
>    

2，在实际开发中，有时并没有异常发生，但是由于事务结果未满足具体业务需求，所以我们不得不手动回滚事务！
有如下两种方法：

> ①手动抛出异常（如果你没有配置一般异常事务回滚，请抛出运行时异常）  
> throw new RuntimeException();
>
> ②编程式实现手动回滚       
>
> TransactionAspectSupport.currentTransactionStatus().setRollbackOnly(); 

**注意：尽管可以采用编程式方法回滚事务，但“回滚”只是事务的生命周期之一，所以要么编程实现事务的全部必要周期，要么仍要配置事务切点，即，将事务管理的其他周期交由****Spring****的标识！**

##### **3：spring事务的默认配置**

Spring的事务管理默认是针对unchecked exception（未检查 RuntimeException）回滚，也就是默认对Error异常和RuntimeException异常以及其子类进行事务回滚，**且必须对抛出异常**，**若使用try-catch对其异常捕获则不会进行回滚！（Error异常和RuntimeException异常抛出时不需要方法调用throws或try-catch语句）；**

> checked异常,checked异常必须由try-catch语句包含或者由方法throws抛出，且事务默认对checked异常不进行回滚。
> 在service类前加上@Transactional，声明这个service所有方法需要事务管理。每一个业务方法开始时都会打开一个事务。

Spring默认情况下会对运行期例外(RunTimeException)进行事务回滚。这个例外是unchecked

如果遇到checked意外就不回滚。

如何改变默认规则：

1 让checked例外也回滚：在整个方法前加上 @Transactional(rollbackFor=Exception.class)

2 让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)

3 不需要事务管理的(只查询的)方法：@Transactional(propagation=Propagation.NOT_SUPPORTED)


注意： 如果异常被try｛｝catch｛｝了，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw Exception｝。

### 开发中实现spring的事务有5种方式

Spring+Hibernate的实质：
就是把Hibernate用到的数据源Datasource，Hibernate的SessionFactory实例，事务管理器HibernateTransactionManager，都交给Spring管理。

那么再没整合之前Hibernate是如何实现事务管理的呢？
通过**ServletFilter实现数据库事务的管理**，这样就避免了在数据库操作中每次都要进行数据库事务处理。

##### 一.事务的4个特性：

原子性：一个事务中所有对数据库的操作是一个不可分割的操作序列，要么全做，要么全部做。
一致性：数据不会因为事务的执行而遭到破坏。
隔离性：一个事务的执行，不受其他事务（进程）的干扰。即并发执行的个事务之间互不干扰。
持久性：一个事务一旦提交，它对数据库的改变将是永久的。

#####   二.事务的实现方式：

实现方式共有两种：编码方式；声明式事务管理方式。

基于AOP技术实现的声明式**事务管理**，实质就是**：在方法执行前后进行拦截**，**然后在目标方法开始之前创建并加入事务，执行完目标方法后根据执行情况提交或回滚事务。**

声明式事务管理又有两种方式：**基于XML配置文件的方式**；另一个是在业务方法上进行**@Transactional注解**，将事务规则应用到业务逻辑中。  

##### 三.创建事务的时机：

是否需要创建事务，是由事务传播行为控制的。读数据不需要或只为其指定只读事务，而数据的插入，修改，删除就需要事务管理了。
配置文件中关于事务配置总是由三个组成部分，分别是DataSource、TransactionManager、和代理机制这三部分，无论哪种配置方式，一般变化的只是代理机制这部分。

> ​    DataSource、TransactionManager这两部分只是会根据数据访问方式有所变化，比如使用Hibernate进行数据访问时，DataSource实际为SessionFactory，TransactionManager的实现为HibernateTransactionManager。

 根据代理机制的不同，总结了五种Spring事务的配置方式，配置文件如下：

第一种方式：每个Bean都有一个代理



##### 4.spring AOP的实现方式 AopProxy接口

　　jdk动态代理

　　cglib动态增强

#### Tomcat 线程池优化，

#### 其实就是与tomcat的连接使用线程池。

当客户端由请求时，线程处理请求，处理完毕就休眠

> 1）创建线程开销大，
>
> 2）线程池：预先建立好线程，等待任务派发。

1. 用Executors可以new很多种不同参数的线程池
2. submit方法是把任务放进线程池的队列
3. 线程池的任务都做完后，即使main函数运行完了，线程池并不会自己关闭，线程也并不会自己销毁，需要我们手动销毁
4. 手动关闭线程池，直接在最后面加调用shutdown()，这个函数会在线程池处理完所有队列里的任务后自动关闭线程池

1：配置executor属性

打开/conf/server.xml文件，在Connector之前配置一个线程池（这个executor可以自己手动去掉注释）：

```java
<Executor name="myThreadPool"
           namePrefix="catalina-exec-"
           maxThreads="250"
           maxIdleTime="60000"
           prestartminSpareThreads="true" 
           minSpareThreads="50"/> 
```

重要参数说明：

name：共享线程池的名字。这是Connector为了共享线程池要引用的名字，该名字必须唯一。默认值：None；

namePrefix:在JVM上，每个运行线程都可以有一个name 字符串。这一属性为线程池中每个线程的name字符串设置了一个前缀，Tomcat将把线程号追加到这一前缀的后面。默认值：tomcat-exec-；

maxThreads：该线程池可以容纳的最大线程数。默认值：200，一般建议设置500~ 800 ，要根据自己的硬件设施条件和实际业务需求而定。 

maxIdleTime：在Tomcat关闭一个空闲线程之前，允许空闲线程持续的时间(以毫秒为单位)。只有当前活跃的线程数大于minSpareThread的值，才会关闭空闲线程。默认值：60000(一分钟)。

minSpareThreads：Tomcat应该始终打开的最小不活跃线程数。默认值：25。Tomcat启动初始化的线程数，在tomcat初始化的时候就初始化

2：配置Connector

```java
<Connector executor="myThreadPool"  
           port="8080" protocol="HTTP/1.1"  
           connectionTimeout="20000"  
            redirectPort="8443"   
           minProcessors="5"  
           maxProcessors="75"  
           acceptCount="1000"/> 
```

重要参数说明：
executor：表示使用该参数值对应的线程池；

minProcessors：服务器启动时创建的处理请求的线程数；

maxProcessors：最大可以创建的处理请求的线程数；

acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。


#### 4.6 查询总结

> select 一般在的后面的内容都是要查询的字段
>
> from 要查询到表
>
> where
>
> group by
>
> having 分组后带有条件只能使用having
>
> order by 它必须放到最后面

#### 4.1 简单查询

1. 查询所有商品

select * from product；

2. 查询商品名和商品价格

select pname,price from product;

3. 查询所有商品信息使用表别名

select * from product as p;

4.查询商品名，使用列别名

select pname as p fromproduct

5.去掉重复值(按照价格)

select distinct(price) from product;

#### 3.3 面试题

说说delete与truncate的区别？

> delete删除的时候是一条一条的删除记录，它配合事务，可以将删除的数据找回。
>
> truncate删除，它是将整个表摧毁，然后再创建一张一模一样的表。它删除的数据无法找回。

先准备数据

insert into tbl_user values(null,’老王’,’666’);