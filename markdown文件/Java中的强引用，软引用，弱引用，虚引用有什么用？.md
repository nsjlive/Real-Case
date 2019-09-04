#### Java中的强引用，软引用，弱引用，虚引用有什么用？

**一.概念解释**

**强引用**

是使用最普遍的引用：Object o=new Object();  特点：不会被GC

- 将对象的引用显示地置为null：o=null;     // 帮助垃圾收集器回收此对象

- 举例ArrayList的实现源代码：

	![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件temp\assets\dd6f826c4e0c045f3701978f311636e1_hd.jpg)

**软引用**

用来描述一些还有用但是并非必须的对象，在Java中用java.lang.ref.SoftReference类来表示。对于软引用关联着的对象，只有在内存不足的时候JVM才会回收该对象。因此，这一点可以很好地用来解决**OOM**的问题，并且这个特性很适合用来实现缓存：**比如网页缓存、图片缓存等**。

1. 浏览器网页缓存实例：

![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件temp\assets\assets\34a44802709c83869b50c5e16b8256db_hd.jpg)

2. 软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

3. **弱引用**与软引用的区别在于：只具有弱引用的对象**拥有更短暂的生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，**一旦发现了只具有弱引用的对象**，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

	如果这个对象是**偶尔**的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用 Weak Reference 来记住此对象。 

**4. 虚引用**也称为幻影引用：一个对象是都有虚引用的存在都**不会对生存时间都构成影响**，也无法通过虚引用来获取对一个对象的真实引用。唯一的用处：能在对象被GC时收到系统通知，JAVA中用PhantomReference来实现虚引用。

#### 2.对比不同:

![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件temp\assets\assets\65b7abe9bf2fcd249c789024d95bb67a_hd.jpg)

