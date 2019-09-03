#### Arraylist和Linkedlist的区别

Arraylist：底层是基于动态数组，根据下表随机访问数组元素的效率高，向数组尾部添加元素的效率高；但是，删除数组中的数据以及向数组中间添加数据效率低，因为需要移动数组。例如最坏的情况是删除第一个数组元素，则需要将第2至第n个数组元素各向前移动一位。而之所以称为动态数组，是因为Arraylist在数组元素超过其容量大，Arraylist可以进行扩容（针对JDK1.8  数组扩容后的容量是扩容前的1.5倍），Arraylist源码中最大的数组容量是Integer.MAX_VALUE-8，对于空出的8位，目前解释是 ：①存储Headerwords；②避免一些机器内存溢出，减少出错几率，所以少分配③最大还是能支持到Integer.MAX_VALUE（当Integer.MAX_VALUE-8依旧无法满足需求时）。以下是Arraylist部分源码：
![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件\assets\1)

从图中可以看出,[ArrayList](https://link.juejin.im?target=http%3A%2F%2Fwww.liuhaihua.cn%2Farchives%2Ftag%2Farraylist)与[LinkedList](https://link.juejin.im?target=http%3A%2F%2Fwww.liuhaihua.cn%2Farchives%2Ftag%2Flinkedlist)都是List接口的实现类,因此都实现了List的所有未实现的方法,只是实现的方式有所不同,(从中可以看出**面向接口的好处**, 对于不同的需求就有不同的实现!),而List接口继承了Collection接口,Collection接口又继承了Iterable接口,因此可以看出List同时拥有了Collection与Iterable接口的特性.

## List接口介绍

查阅API，看List的介绍：有序的 collection（也称为序列）。此接口的用户可以对列表中每个元素的插入位置进行精确地控制。用户可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。与 set 不同，列表通常允许重复的元素。

**List接口：**

l  它是一个元素**存取有序**的集合。例如，存元素的顺序是11、22、33。那么集合中，元素的存储就是按照11、22、33的顺序完成的）。

l  它是一个**带有索引**的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。

l  集合中可以有重复的元素，通过元素的equals方法，来比较是否为重复的元素。

## ArrayList

ArrayList集合数据存储的结构是数组结构。元素**增删慢，查找快**，由于日常开发中使用最多的功能为查询数据、遍历数据，所以ArrayList是最常用的集合。

ArrayList是List接口的**可变数组**的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。
每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。

随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

注意，此实现**不是同步**的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。这通常是通过同步那些用来封装列表的对象来实现的。但如果没有这样的对象存在，则该列表需要运用{@link Collections#synchronizedList Collections.synchronizedList}来进行“包装”，该方法最好是在创建列表对象时完成，为了避免对列表进行突发的非同步操作。

```java
List list = Collections.synchronizedList(new ArrayList(...));
```

建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

#### 扩容

**数组**有个明显的特点就是它的容量是**固定不变**的，一旦数组被创建则容量则无法改变。所以在往数组中添加指定元素前，首先要考虑的就是其容量是否饱和。

若接下来的添加操作会时数组中的元素超过其容量，则必须对其进行**扩容**操作。受限于数组容量固定不变的特性，扩容的本质其实就是创建一个**容量更大**的新数组，再将旧数组的元素**复制**到新数组当中去。

这里以 ArrayList 的 添加操作为例，来看下 ArrayList 内部数组扩容的过程。

```java
public boolean add(E e) {
	// 关键 -> 添加之前，校验容量
	ensureCapacityInternal(size + 1); 
	
	// 修改 size，并在数组末尾添加指定元素
	elementData[size++] = e;
	return true;
}
```

可以发现 ArrayList 在进行添加操作前，会检验内部数组容量并**选择性地**进行数组扩容。在 ArrayList 中，通过私有方法 **ensureCapacityInternal** 来进行数组的扩容操作。下面来看具体的实现过程：

- 扩容操作的第一步会去判断当前 ArrayList 内部数组是否为空，为空则将最小容量 minCapacity 设置为 10。

```
// 内部数组的默认容量
private static final int DEFAULT_CAPACITY = 10;

// 空的内部数组
private static final Object[] EMPTY_ELEMENTDATA = {};

// 关键 -> minCapacity = seize+1，即表示执行完添加操作后，数组中的元素个数 
private void ensureCapacityInternal(int minCapacity) {
	// 判断内部数组是否为空
	if (elementData == EMPTY_ELEMENTDATA) {
		// 设置数组最小容量（>=10）
		minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
	}
	ensureExplicitCapacity(minCapacity);
}
```

- 接着判断添加操作会不会导致内部数组的容量饱和

```java
private void ensureExplicitCapacity(int minCapacity) {
	modCount++;
	
	// 判断结果为 true，则表示接下来的添加操作会导致元素数量超出数组容量
	if (minCapacity - elementData.length > 0){
		// 真正的扩容操作
		grow(minCapacity);
	}
}
```

- 数组容量不足，则进行扩容操作，关键的地方有两个：**扩容公式**、通过从旧数组复制元素到新数组完成扩容操作。

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
	
	int oldCapacity = elementData.length;
	
	// 关键-> 容量扩充公式
	int newCapacity = oldCapacity + (oldCapacity >> 1);
	
	// 针对新容量的一系列判断
	if (newCapacity - minCapacity < 0){
		newCapacity = minCapacity;
	}
	if (newCapacity - MAX_ARRAY_SIZE > 0){
		newCapacity = hugeCapacity(minCapacity);
	}
		
	// 关键 -> 复制旧数组元素到新数组中去
	elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
	if (minCapacity < 0){
		throw new OutOfMemoryError();
	}
			
	return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

关于 ArrayList 扩容操作，整个过程如下图：

![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件\assets\1aq)

## LinkedList

## ![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件\assets\1qw)

- LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行队列操作。
- LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
- LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
- LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
- LinkedList 是非同步的。

![img](C:\Users\n\OneDrive - std.uestc.edu.cn\秋招\markdown文件\assets\1asd)

如上图所示，LinkedList底层使用的**双向链表结构**，有一个头结点和一个尾结点，双向链表意味着我们可以从头开始正向遍历，或者是从尾开始逆向遍历，并且可以针对头部和尾部进行相应的操作。

```java
private static class Node<E> {
	E item;Node<E> next;
	Node<E> prev;
	Node(Node<E> prev, E element, Node<E> next) 
		{
			this.item = element;
			this.next = next;this.prev = prev;
		}
}
```

**总结：**

1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。

1:ArrayList和ListedList都是list的实现类(其实还有Vector和它的子类stack(同步效率底很少使用了)),

ArrayList和LinkedList都不是线程安全的,而vector是线程安全的这个算是两者的相同之处吧

2:底层方面来说,ArrayList是用动态数组来实现的,LinkList是用链表来实现的

3:对于随机访问get()和set()方法-(相当于改查),ArrayList的效率要比linkList的效率好一点(linkList要移动指针啦,这个老铁们应该理解吧).

4:对于add()和remove()方法

   4.1:.arrayList耗时主要在arraycopy这个函数上了函数覆盖方式实现,增加的时候,函数后移,index空缺插入,删除的时候,后面依次后移,删除末尾.

linkList主要耗时间在查找index位置上了,当index位置靠后,数量又大的时候,效率就低于arrrlist



