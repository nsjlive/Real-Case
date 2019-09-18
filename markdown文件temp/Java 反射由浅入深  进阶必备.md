#### Java 反射由浅入深 | 进阶必备

准备知识：

**面向对象**

> 我们都知道，java 是一门面向对象的语言。在面向对象的世界里，万事万物皆对象，除了静态成员（因为**静态成员属于某个类**，而不是对象）和**普通数据类型**。
>
> 在面向对象的语言中，我们擅长将现实世界中的一个实际存在的事物抽象并封装成一个类，并在类中添加相应的成员变量（属性）和方法，然后我们就可以创建该类的对象，该对象持有属于自己的成员变量和方法。
>
> 首先，Class是一个java类，跟Java API中定义的诸如Thread、Integer类、我们自己定义的类是一样，**也继承了Object**（Class是Object的直接子类）。总之，必须明确一点，它其实只是个类，只不过名字比较特殊。更进一步说，**Class是一个java中的泛型类型**。
>
> 既然万事万物皆对象，那么我们的类是不是对象呢？是的，我们写的每一个类都是对象，是 java.lang.Class 类的对象。也就是说，**每一个类既有自己的对象，同时也是 Class 类的对象**。那么，该如何表示 Class 类的对象呢，让我们接着往下看，以进一步了解 Class 类。

**java.lang.Class 类**

先来看看 `Class` 类的构造方法（有删减）：

```java
/*
     * Private constructor. 
     * Only the Java Virtual Machine
     * creates Class objects.
     */
    private Class(ClassLoader loader) {
        classLoader = loader;}
```

根据注释可知，**Class 类的构造方法是私有**的，只有 **Java 虚拟机可以创建该类的对象**，因此我们无法在代码中显式地声明一个 Class 对象。这就是我要强调的内容，其对于后面内容的理解十分重要。

**Class 类与其他类的关系**
**由类或类对象得到 Class 类的对象**

自定义一个类 MyClass，并声明该类的对象：

```java
class MyClass{}
 
MyClass mClass1 = new MyClass();
```

`在上面说过Class` 类的构造方法是私有的，只有 java 虚拟机可以调用该方法创建该类的对象。也就是说我们无法像定义普通类对象一样，通过 new 直接创建 `Class` 类的对象。

**1.通过类的静态成员表示。**

每个类都有一个隐含的**静态成员class**，表示如下：

```
Class c1 = MyClass.class;
```

**2.通过类对象的 getClass() 方法。**

由1不难理解，既然存在静态变量，那么通过对象的 getter 方法，就可以获取静态成员class：

```
Class c2 = mClass1.getClass();
```

**3.通过 Class 类的静态方法 forName() 方法获取 Class 的对象。**

区别于通过 new 创建对象（编译时静态加载），在开发时如果我们需要动态加载我们的功能模块，该方法可以帮助我们实现在程序运行时类的动态加载。

```java
    try {
        //注意，forName()需要传入类的全路径
        //如果当前类与参数类在同一包下即可省略包名
        mClass = Class.forName("custom.OtherClass");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
```

如果我们的程序中没有 OtherClass 这个类，使用 Class.forName() 动态加载时，在程序编译时刻是不会报错的，只有在运行时刻检测到没有该类才会出错。

类的**动态加载**和**静态加载**是 Java 中一个重要的知识点，但因为我们开发时通常都使用 IDE， 其帮助我们自动实现了程序的编译和运行，使得我们常常忽略了程序的编译时和运行时状态。说着说着，我发现这块我有必要再写一篇博客记录一下，今天就先不讨论了。

##### **由 Class 类的对象得到类的对象**

我们可以通过类或类对象得到 `Class` 类的对象，反过来，我们也可以由 `Class` 类的对象得到类的对象：

```java
MyClass mClass2 = (MyClass)c2.newInstance();
```

需要提醒你的是：`c2.newInstance()` 需要调用 `MyClass` 类的无参构造方法！如果 `MyClass` 类中存在显示的有参构造方法，会覆盖默认的无参构造方法，同时又没有显示的声明无参构造方法，那么执行这段代码时会直接导致程序Crash掉。下面为大家演示一下：

示例代码，请看注释：

```java
public class TestDemo {
    public static void main(String args[]){
        //得到 Class 对象 mClass
        Class mClass = MyClass.class;
        try {
            // 由 mClass 实例化 MyClass 的对象，
            // 需要调用 MyClass 的无参构造方法
            MyClass mMyClass = 
                    (MyClass) mClass.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
 
class MyClass{
    //有参构造方法，会覆盖默认的无参构造方法
    public MyClass(String s){}
}
```

程序异常：

```java
java.lang.InstantiationException: MyClass
    at java.lang.Class.newInstance(Class.java:427)
    at TestDemo.main(TestDemo.java:12)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:497)
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
Caused by: java.lang.NoSuchMethodException: MyClass.<init>()
    at java.lang.Class.getConstructor0(Class.java:3082)
    at java.lang.Class.newInstance(Class.java:412)
    ... 6 more
```

解决办法就是在 MyClass 中的显示的添加一个 无参构造方法，就不再提供示例了。

针对这一点，相信有许多人在使用第三方框架或者开源库时，遇到过因为在类中添加了带参数的构造方法而导致程序出错的情况！针对这个，我的理解是：**有些框架是基于反射实现的**，它根据我们传入的类对象，使用其 newInstance() 方法获取 Class 对象，进而进行后续的反射操作（不在本文的讨论范围）。可是因为我们**无意覆盖了默认的无参构造方法**，导致程序无法正常获取 Class 对象，所以就出错了。说到这儿，您应该能理解其中缘由了吧！

这次分享的内容比较少，归结起来就是如何获取 Class 类的对象，以及如何由 Class 对象得到类对象。至于如何使用 Class 对象进行反射操作，如何实现程序运行时动态加载类，在后面的分享中会继续向大家介绍的。


#### 一、Java 反射机制

> Java 反射机制在程序运行时，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。这种 动态的获取信息 以及 动态调用对象的方法 的功能称为 java 的反射机制。
> 反射机制很重要的一点就是**“运行时”**，其使得我们可以在程序运行时加载、探索以及使用编译期间**完全未知的 .class 文件**。换句话说，Java 程序可以加载一个运行时才得知名称的 .class 文件，然后获悉其完整构造，并生成其对象实体、或对其 fields（变量）设值、或调用其 methods（方法）。

不知道上面的理论你能否明白，反正刚接触反射时我一脸懵比，后来写了几个例子之后：哦~~原来是这个意思！

若暂时不明白理论没关系，先往下看例子，之后再回来看相信你就能明白了。

#### 二、使用反射获取类的信息

为使得测试结果更加明显，我首先定义了一个 `FatherClass` 类（默认继承自 `Object` 类），然后定义一个继承自 `FatherClass` 类的 `SonClass` 类，如下所示。可以看到测试类中变量以及方法的访问权限不是很规范，是为了更明显得查看测试结果而故意设置的，实际项目中不提倡这么写。

**FatherClass.java**

```java
public class FatherClass {
    public String mFatherName;
    public int mFatherAge;

    public void printFatherMsg(){}
}
```

**SonClass.java**

```java
public class SonClass extends FatherClass{

    private String mSonName;
    protected int mSonAge;
    public String mSonBirthday;

    public void printSonMsg(){
        System.out.println("Son Msg - name : "
                + mSonName + "; age : " + mSonAge);
    }

    private void setSonName(String name){
        mSonName = name;
    }

    private void setSonAge(int age){
        mSonAge = age;
    }

    private int getSonAge(){
        return mSonAge;
    }

    private String getSonName(){
        return mSonName;
    }
}
```

​	1. 获取类的所有变量信息

```
/**
 * 通过反射获取类的所有变量
 */
private static void printFields(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());

    //2.1 获取所有 public 访问权限的变量
    // 包括本类声明的和从父类继承的
    Field[] fields = mClass.getFields();

    //2.2 获取所有本类声明的变量（不问访问权限）
    //Field[] fields = mClass.getDeclaredFields();

    //3. 遍历变量并输出变量信息
    for (Field field :
            fields) {
        //获取访问权限并输出
        int modifiers = field.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //输出变量的类型及变量名
        System.out.println(field.getType().getName()
                 + " " + field.getName());
    }
}
```

以上代码注释很详细，就不再解释了。需要注意的是注释中 2.1 的 `getFields()` 与 2.2的 `getDeclaredFields()` 之间的区别，下面分别看一下两种情况下的输出。看之前强调一下：
`SonClass` extends `FatherClass` extends  `Object` ：

- 调用 `getFields()` 方法，输出 `SonClass` 类以及其所继承的父类( 包括 `FatherClass` 和 `Object` ) 的 `public` 方法。注：`Object` 类中没有成员变量，所以没有输出。

```
  类的名称：obj.SonClass
  public java.lang.String mSonBirthday
  public java.lang.String mFatherName
  public int mFatherAge
```

- 调用 `getDeclaredFields()` ， 输出 `SonClass` 类的所有成员变量，不问访问权限。

```
  类的名称：obj.SonClass
  private java.lang.String mSonName
  protected int mSonAge
  public java.lang.String mSonBirthday
```

## 2. 获取类的所有方法信息

```java
/**
 * 通过反射获取类的所有方法
 */
private static void printMethods(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());

    //2.1 获取所有 public 访问权限的方法
    //包括自己声明和从父类继承的
    Method[] mMethods = mClass.getMethods();

    //2.2 获取所有本类的的方法（不问访问权限）
    //Method[] mMethods = mClass.getDeclaredMethods();

    //3.遍历所有方法
    for (Method method :
            mMethods) {
        //获取并输出方法的访问权限（Modifiers：修饰符）
        int modifiers = method.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //获取并输出方法的返回值类型
        Class returnType = method.getReturnType();
        System.out.print(returnType.getName() + " "
                + method.getName() + "( ");
        //获取并输出方法的所有参数
        Parameter[] parameters = method.getParameters();
        for (Parameter parameter:
             parameters) {
            System.out.print(parameter.getType().getName()
                    + " " + parameter.getName() + ",");
        }
        //获取并输出方法抛出的异常
        Class[] exceptionTypes = method.getExceptionTypes();
        if (exceptionTypes.length == 0){
            System.out.println(" )");
        }
        else {
            for (Class c : exceptionTypes) {
                System.out.println(" ) throws "
                        + c.getName());
            }
        }
    }
}
```

同获取变量信息一样，需要注意注释中 2.1 与 2.2 的区别，下面看一下打印输出：

- 调用 `getMethods()` 方法
	  获取 `SonClass` 类所有 `public` 访问权限的方法，包括从父类继承的。打印信息中，`printSonMsg()` 方法来自 `SonClass` 类， `printFatherMsg()` 来自 `FatherClass` 类，其余方法来自 Object 类。

```java
  类的名称：obj.SonClass
  public void printSonMsg(  )
  public void printFatherMsg(  )
  public final void wait(  ) throws java.lang.InterruptedException
  public final void wait( long arg0,int arg1, ) throws java.lang.InterruptedException
  public final native void wait( long arg0, ) throws java.lang.InterruptedException
  public boolean equals( java.lang.Object arg0, )
  public java.lang.String toString(  )
  public native int hashCode(  )
  public final native java.lang.Class getClass(  )
  public final native void notify(  )
  public final native void notifyAll(  )
```

- 调用 `getDeclaredMethods()` 方法

打印信息中，输出的都是 `SonClass` 类的方法，不问访问权限

```
  类的名称：obj.SonClass
  private int getSonAge(  )
  private void setSonAge( int arg0, )
  public void printSonMsg(  )
  private void setSonName( java.lang.String arg0, )
  private java.lang.String getSonName(  )
```

#### 三、访问或操作类的私有变量和方法

在上面，我们成功获取了类的变量和方法信息，验证了在运行时 **动态的获取信息** 的观点。那么，仅仅是获取信息吗？我们接着往后看。

都知道，对象是无法访问或操作类的私有变量和方法的，但是，通过反射，我们就可以做到。没错，反射可以做到！下面，让我们一起探讨如何利用反射访问 **类对象的私有方法** 以及修改 **私有变量或常量**。

老规矩，先上测试类。

注：

1. 请注意看测试类中变量和方法的修饰符（访问权限）；
2. 测试类仅供测试，不提倡实际开发时这么写 : )

**TestClass.java**

```java
public class TestClass {

    private String MSG = "Original";

    private void privateMethod(String head , int tail){
        System.out.print(head + tail);
    }

    public String getMsg(){
        return MSG;
    }
}
```

后续补充，参考：[Java 反射由浅入深 | 进阶必备](<https://juejin.im/post/598ea9116fb9a03c335a99a4>)

