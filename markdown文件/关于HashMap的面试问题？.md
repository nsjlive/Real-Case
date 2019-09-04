一、什么是HashMap？

> HashMap是一个用于存储Key-Value键值对的集合（**散列桶**），每一个键值对也叫做Entry。这些个键值对（Entry）分散存储在一个数组当中，这个数组就是HashMap的主干，数组每一个元素的初始值都是Null。这些就是HashMap的定义了。

二、你为什么用到它？

> HashMap可以接受null键值和值，而Hashtable则不能；（原因就是**equlas()方法需要对象**，因为HashMap是后出的API经过处理才可以）
> HashMap是非synchronized，所以相对而说，HashMap很快；
> 以及HashMap储存的是键值对，以一种数据之间的对应关系。
>
> HashMap采用了**数组和链表**的数据结构，能在**查询**和**修改**方便继承了**数组的线性查找**和**链表的寻址修改**

三、你知道HashMap的工作原理吗？

> HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。
>
> 当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，**HashMap是在bucket中储存键对象和值对象**，作为Map.Entry。

四、你知道HashMap的**get()方法**的工作原理吗？（**重要**）

> 首先根据对象的Hash值进行数组方面的寻找，然后找到这个数组之后，**判断key是不是唯一**，如果key唯一，则直接返回，如果不唯一，则**使用equals进行值的判断，最后返回数据**。

五、当两个对象的hashcode相同会发生什么？

> 一些面试者会回答因为hashcode相同，所以两个对象是相等的，HashMap将会抛出异常，或者不会存储它们。
> 如果之前的问题回答的好，面试官的印象比较好，可能会提醒他们有equals()和hashCode()两个方法，并告诉他们两个对象就算hashcode相同，但是它们可能并不相等。
> 如果掌握的不太好，一些面试者可能就此放弃。那下面的问题也就不了了之了，等于放弃了一个很好的机会。
> 而这个问题的答案是：**因为hashcode相同，所以它们的bucket位置相同**，‘**碰撞’**会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。这个时候要理解根据hashcode来划分的数组，
> 如果数组的坐标相同，则进入链表这个数据结构中了，jdk1.7及以前为头插法，**jdk1.8之后是尾插法**，在**jdk1.8之后**，当链表长度到达8的时候，jdk1.8上升为**红黑树**

六、如果两个键的hashcode相同，你如何获取值对象？

> 许多情况下，面试者会在这个环节中出错，因为他们混淆了hashCode()和equals()方法。
>
> 因为在此之前hashCode()屡屡出现，而equals()方法仅仅在获取值对象的时候才出现。
>
> 一些优秀的开发者会指出**使用不可变的声明作final的对象**，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。
>
> **不可变性**使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

补充阅读：1. **为什么String, Interger这样的wrapper类适合作为键？**

> String, Interger这样的wrapper类作为HashMap的键是再适合不过了，而且String最为常用。
>
> 因为**String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了**。其他的wrapper类（**包装类**）也有这个特点。
>
> **不可变性是必要的**，因为为了要计算hashCode()，就要**防止键值改变**，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。
>
> **不可变性**还有其他的优点如**线程安全**。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。**如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这样就能提高HashMap的性能**。

2. **我们可以使用自定义的对象作为键吗？** 

> 这是前一个问题的延伸。当然你可能使用任何对象作为键，**只要它遵守了equals()和hashCode()方法的定义规则，并且当对象插入到Map中之后将不会再改变了**。如果这个自定义对象时不可变的，那么它已经满足了作为键的条件，因为当它创建之后就已经不能改变了。

七、如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？

【问到这个问题之后，要及时的意识到面试官要把你往线程安全的方向引入了，做好准备。】

> 默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，**并将原来的对象放入新的bucket数组中**。这个过程叫作**rehashing**，因为它调用hash方法找到新的bucket位置。

八、你了解重新调整HashMap大小存在什么问题吗？

你可能回答不上来，这时面试官会提醒你当多线程的情况下，可能产生条件竞争(race condition)。

> 当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)，原数组[j]位置上的桶移到了新数组[j+原数组长度]。如果条件竞争发生了，那么就死循环了。

（如果线程方面的知识储备还不错，那这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？，不直接使用concurrentHashMap）

补充：按照JDK文档中的规定，Hashtable不允许空键或值。 HashMap允许一个空键和任意数量的空值。这是为什么？

Hashtable是较老的类

从 Hashtable [JavaDoc ](http://www.it1352.com/"http://download.oracle.com/javase/6/docs/api/java)：

> 成功存储和检索Hashtable中的对象，对象用作键**必须实现hashCode方法和equals方法**。 **null 不是对象**，您不能调用 .equals（）或 .hashCode（）“，所以 Hashtable 无法计算散列值以用作键。
>
> HashMap更新，并且具有更高级的功能，这基本上是对Hashtable功能的改进。当创建HashMap时，它专门设计用来处理空值作为键，并将它们作为特殊情况处理。

hashMap是根据key的hashCode来寻找存放位置的，那当key为null时， 怎么存储呢？

在put方法里头，其实第一行就处理了key=null的情况。 

>  if (key == null)
>              // key为null调用putForNullKey(value)
>              return putForNullKey(value);



**如果读者对这个问题感兴趣，可以再来看看这些问题设计哪些知识点：**
● hashing的概念
● HashMap中解决碰撞的方法
● equals()和hashCode()的应用，以及它们在HashMap中的重要性
● 不可变对象的好处
● HashMap多线程的条件竞争
● 重新调整HashMap的大小
**总结**

> HashMap的工作原理
> HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。
> 当两个不同的键对象的hashcode相同时会发生什么？ 它们会储存在同一个bucket位置的链表中。键对象的equals()方法用来找到键值对。

转自：[HashMap面试常问问题](<https://blog.csdn.net/weixin_42636552/article/details/82016183>)

java中的null是值还是对象？

> 严格讲既不是基础类型也不是对象类型，是一个特殊的类型，并可以看作允许cast成任意类型的对象。