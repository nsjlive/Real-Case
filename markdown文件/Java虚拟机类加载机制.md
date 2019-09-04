# Java虚拟机类加载机制

*成员变量的作用域是整个类，所以成员变量可以在类里面任何地方声明*

静态代码块是以static修饰的代码块，反之没有用static修饰的代码块为非静态代码块:

```java
//静态代码块：
static{ 
若干代码 
}
```

```java
{//非静态代码块：
若干代码
}
```

它们之间的区别主要如下:参考：[静态代码块与非静态代码块的区别](<https://blog.csdn.net/zhou_438/article/details/81270523>)

1. 静态代码块在**虚拟机加载类的时候就执行（只执行一次）**，而非静态代码块**每一次new的时候都会执行一次**。
2. 由于静态代码块在虚拟机加载类的时候就执行，因此在非静态代码块在静态代码块后执行

```java
public class SSClass
{
    static
    {
        System.out.println("SSClass");
    }
}    
public class SuperClass extends SSClass
{
    static
    {
        System.out.println("SuperClass init!");
    }
 
    public static int value = 123;
 
    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
public class SubClass extends SuperClass
{
    static
    {
        System.out.println("SubClass init");
    }
 
    static int a;
 
    public SubClass()
    {
        System.out.println("init SubClass");
    }
}
public class NotInitialization
{
    public static void main(String[] args)
    {
        System.out.println(SubClass.value);
    }
}
```

运行结果：

```java
`SSClass``SuperClass init!``123`
```

答案答对了嚒？
也许有人会疑问：**为什么没有输出SubClass init**。ok~解释一下：**对于静态字段，只有直接定义这个字段的类才会被初始化**，因此**通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而==不会触发子类的初始化==**。
上面就牵涉到了虚拟机类加载机制。如果有兴趣，可以继续看下去。

```java
package jvm.classload;

public class StaticTest
{
    public static void main(String[] args)
    {
        staticFunction();
    }

    static StaticTest st = new StaticTest();

    static
    {
        System.out.println("1");
    }

    {
        System.out.println("2");
    }

    StaticTest()
    {
        System.out.println("3");
        System.out.println("a="+a+",b="+b);
    }

    public static void staticFunction(){
        System.out.println("4");
    }

    int a=110;
    static int b =112;
}

```

> Java中赋值顺序：**父类优先，静态优先**  **构造最后**

1. 父类静态变量和静态代码块（**先声明的先执行**）；
2. 子类静态变量和静态代码块（先声明的先执行）；
3. *==父类的变量和代码块==*（**先声明的先执行**）；
4. *==父类的构造函数==*；
5. 子类的变量和代码块（**先声明的先执行**）；
6. 子类的构造函数。

ok,按照这个理论输出是什么呢？答案输出:1 4，这样正确嚒？肯定不正确啦，这里不是说上面的规则不正确，而是说不能简单的套用这个规则。
  正确的答案是：

```
2
3
a=110,b=0
1
4
```

这里主要的点之一：**实例初始化不一定要在类初始化结束之后才开始初始化。**

类的生命周期是：加载->验证->**准备**->解析->**初始化**->使用->卸载，只有在准备阶段和初始化阶段才会涉及类变量的初始化和赋值，因此只针对这两个阶段进行分析；

1. 类的**准备阶段**需要做是为==类变量==分配内存并设置默认值，因此类变量==*st为null*、b为0==；(**静态变量是类变量**)

2. 类的**初始化阶段**需要做是==*执行类构造器*==（类构造器是编译器收集所有静态语句块和类变量的赋值语句按语句在源码中的顺序合并生成类构造器，对象的构造方法是<init>()，类的构造方法是<clinit>()，可以在堆栈信息中看到），因此：
	* **先执行第一条静态变量的赋值语句**即st = new StaticTest ()，**此时**会进行**对象的初始化**，对象的初始化是**先初始化成员变量** **再执行构造方法**，因此设置a为110->打印2->执行构造方法(打印3,此时a已经赋值为110，但是**b只是设置了默认值0，并未完成赋值动作**)，等对象的初始化完成后继续执行之前的类构造器的语句，接下来就不详细说了，按照语句在源码中的顺序执行即可。

这里面还牵涉到一个冷知识，就是在嵌套初始化时有一个特别的逻辑。特别是内嵌的这个变量恰好是个静态成员，而且是本类的实例。

这会导致一个有趣的现象：“实例初始化竟然出现在静态初始化之前”。
  其实并没有提前，你要知道java记录初始化与否的时机。
  看一个简化的代码，把关键问题解释清楚：

```java
public class Test {
    public static void main(String[] args) {
        func();
    }
    static Test st = new Test();//st为静态成员
    static void func(){}
}
```

根据上面的代码，有以下步骤：

1. 首先在执行此段代码时，首先由main方法的调用触发静态初始化。

2. 在初始化Test 类的静态部分时，遇到**st**这个成员,但凑巧这个变量引用的是**本类的实例**。

	那么问题来了，此时静态初始化过程还没完成就要初始化实例部分了。是这样么？

3. 从人的角度是的。但从java的角度，一旦开始初始化静态部分，无论是否完成，后续都不会再重新触发静态初始化流程了。

	因此在**实例化st变量时**，实际上是把**实例初始化嵌入到了静态初始化流程中**，并且在楼主的问题中，嵌入到了静态初始化的起始位置。这就导致了**实例初始化完全至于静态初始化之前**。这也是导致a有值b没值的原因

4. 最后再考虑到文本顺序，结果就显而易见了。

本文参考：[Java虚拟机类加载机制——案例分析](<https://blog.csdn.net/u013256816/article/details/50837863>)

