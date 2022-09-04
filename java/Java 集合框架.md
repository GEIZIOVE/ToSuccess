# Java 集合框架

# 一、 Collection 类关系图

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/java/collection/java-collection-all.html

[Collection 类关系图](#collection-类关系图)

- [知识体系结构](#知识体系结构)
- [介绍](#介绍)
- Collection
  - Set
    - [TreeSet](#treeset)
    - [HashSet](#hashset)
    - [LinkedHashSet](#linkedhashset)
  - List
    - [ArrayList](#arraylist)
    - [Vector](#vector)
    - [LinkedList](#linkedlist)
  - Queue
    - [LinkedList](#linkedlist-1)
    - [PriorityQueue](#priorityqueue)
- Map
  - [TreeMap](#treemap)
  - [HashMap](#hashmap)
  - [HashTable](#hashtable)
  - [LinkedHashMap](#linkedhashmap)
- [参考内容](#参考内容)

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/java/collection/java-collection-all.html

## 知识体系结构

![img](E:\Development\Typora\images\java_collections_overview.png)

## [¶](#介绍) 介绍

​		**容器**，就是可以容纳其他Java对象的对象。*Java Collections Framework(JCF)*为Java开发者提供了通用的容器，其始于JDK 1.2，优点是:

- 降低编程难度
- 提高程序性能
- 提高API间的互操作性
- 降低学习难度
- 降低设计和实现相关API的难度
- 增加程序的重用性

Java容器里只能放对象，对于基本类型(int, long, float, double等)，需要将其包装成对象类型后(Integer, Long, Float, Double等)才能放到容器里。**很多时候拆包装和解包装能够自动完成**。这虽然会导致额外的性能和空间开销，但简化了设计和编程。

## [¶](#collection) Collection

> 容器主要包括 `Collection` 和 `Map` 两种，`Collection` 存储着对象的集合，而 `
>
> ` 存储着键值对(两个对象)的映射表。

### [¶](#set) ①Set

#### [	¶](#treeset) TreeSet

**优:**基于红黑树实现，支持`有序性`操作，例如根据一个范围查找元素的操作。

**缺:**查找效率不如 `HashSet`，HashSet 查找的时间复杂度为 <font color='orange'>O(1)</font>，TreeSet 则为<font color='orange'> O(logN)</font>。

#### 	[¶](#hashset) HashSet

**优：**基于哈希表实现，支持快速查找

**缺：**不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 `Iterator` 遍历 HashSet 得到的<font color='cornflowerblue'>结果是不确定的</font>。

#### [	¶](#linkedhashset) LinkedHashSet

集成上面两个的优点

**具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。**

### [¶](#list) ②List

#### [¶](#arraylist) ArrayList

基于**<font color='red'>动态数组</font>**实现，支持随机访问(也可以顺序访问)

#### [¶](#vector) Vector

和 ArrayList 类似，但它是<font color='red'>线程安全</font>的。

#### [¶](#linkedlist) LinkedList

**优：**基于双向链表实现，**只能顺序访问**

**缺：**但是可以快速地在链表中间插入和删除元素。

​		不仅如此，LinkedList 还可以用作栈、队列和双向队列。

### [¶](#queue) ③Queue

#### [¶](#linkedlist-2) 	LinkedList

可以用它来实现**双向队列**。

#### [¶	](#priorityqueue) PriorityQueue

基于**堆结构**实现，可以用它来实现优先队列。

## [¶](#map) Map

### [¶](#treemap) TreeMap

基于红黑树实现。

### [¶](#hashmap) HashMap

基于哈希表实现。

### [¶](#hashtable) HashTable

和 HashMap 类似，但它是<font color='red'>线程安全</font>的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。

### [¶](#linkedhashmap) LinkedHashMap

使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用(LRU)顺序。

## [¶](#参考内容) 参考内容

- CarpenterLee/JCFInternals https://github.com/CarpenterLee/JCFInternals



# 二、Collection - ArrayList 源码解析

- Collection - ArrayList 源码解析
  - [概述](#概述)
  - ArrayList的实现
    - [底层数据结构](#底层数据结构)
    - [构造函数](#构造函数)
    - [自动扩容](#自动扩容)
    - [add(), addAll()](#add-addall)
    - [set()](#set)
    - [get()](#get)
    - [remove()](#remove)
    - [trimToSize()](#trimtosize)
    - [indexOf(), lastIndexOf()](#indexof-lastindexof)
    - [Fail-Fast机制:](#fail-fast机制)
  - [参考](#参考)

## [¶](#概述) 概述

*`ArrayList`*实现了*`List`*接口，是**<font color='cornflowerblue'>顺序容器</font>**，即元素存放的数据与放进去的顺序相同，允许放入`null`元素，底层通过**<font color='red'>数组<实现**。除该类未实现同步外，其余跟*`Vector`*大致相同。每个*`ArrayList`*都有一个容量(capacity)，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。

​		当向容器中添加元素时，**如果容量不足，容器会自动增大底层数组的大小**。前面已经提过，Java泛型只是编译器提供的语法糖，所以这里的数组是一个`Object`数组，以便能够容纳任何类型的对象。

![ArrayList_base](E:\Development\Typora\images\ArrayList_base-16622187148223.png)



​		**`	size(), isEmpty(), get(), set()`**方法均能在**常数时间**内完成，`add()`方法的时间开销跟插入位置有关，`addAll()方`法的时间开销跟添加元素的个数成正比。其余方法大都是线性时间。

​		为追求效率，ArrayList没有实现同步(synchronized)，**如果需要多个线程并发访问，用户可以手动同步，也可使用`Vector`替代。**



## ArrayList的实现

### [¶](https://pdai.tech/md/java/collection/java-collection-ArrayList.html#底层数据结构)底层数据结构

```JAVA
//默认初始容量。
private static final int DEFAULT_CAPACITY = 10;

//用于空实例的共享空数组实例。
private static final Object[] EMPTY_ELEMENTDATA = {};

//用于默认大小的空实例的共享空数组实例。我们将其与 EMPTY_ELEMENTDATA 区分开来，以了解添加第一个元素时要膨胀多少。
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/** 
* 存储 ArrayList 元素的数组缓冲区。
* ArrayList 的容量是此数组缓冲区的长度。任何带有 elementData==DEFAULTCAPACITY_EMPTY_ELEMENTDATA 
* 的空 ArrayList 都将在添加第一个元素时展开为DEFAULT_CAPACITY。
**/
transient Object[] elementData; // non-private to simplify nested class access

//The size of the ArrayList (the number of elements it contains).
private int size;
```



### 构造函数

```java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//构造一个初始容量为 10 的空列表。
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 按照集合的迭代器返回的顺序构造一个包含指定集合元素的列表。
 * 参形：c – 其元素要放入此列表的集合
 * 抛出：NullPointerException – 如果指定的集合为空
 */
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

###  自动扩容

​		每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个公开的方法`ensureCapacity(int minCapacity)`来实现。在实际添加大量元素前，我也可以使用`ensureCapacity`来**手动**增加`ArrayList`实例的容量，以减少递增式再分配的数量。

​		数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的**1.5**倍。这种操作的代价是很高的，因此在实际使用时，**我们应该尽量避免数组容量的扩张**。当我们可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用`ensureCapacity`方法来手动增加ArrayList实例的容量。

```java
/**
 * 如有必要，增加此ArrayList实例的容量，以确保它至少可以容纳最小容量参数指定的元素数量。
 *
 * @param   minCapacity   the desired minimum capacity
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY; //10
    
		//这里表示最小扩容的大小为10...
    if (minCapacity > minExpand) { 
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++; //此列表在结构上的修改次数+1

    // overflow-conscious code 
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
 *	要分配的数组的最大大小。一些 VM 在数组中保留一些标题字。尝试分配更大的数组可能会导致 OutOfMemoryError：请求的数组大小超过 VM 限制
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 *	增加容量以确保它至少可以容纳最小容量参数指定的元素数量。  
 *
 * @param minCapacity the desired minimum capacity
 */
//扩容的实际方法
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); //自动扩展1.5倍
    if (newCapacity - minCapacity < 0) //若自动扩展的大小不够 则用传过来的大小 避免浪费
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :  //2^31-1.
        MAX_ARRAY_SIZE;
}
```

补充解释 `overflow-conscious code`

在计算机中，`a - b < 0` 和 `a < b` 并不完全等价。在没有发生溢出的情况下，两者是等价的，但是如果发生了溢出，情况将会变得不同。考虑下面的代码：

```java
int a = Integer.MAX_VALUE;
int b = Integer.MIN_VALUE;
if (a < b) {
    System.out.println("a < b");
}
if (a - b < 0) {
    System.out.println("a - b < 0");
}
```

运行上面的代码可以发现，输出的是 “a - b < 0”。很显然，`a < b` 明显是错误的，而 `a - b` 由于发生了溢出，因此结果是负的。

我们来看看上面的`grow()`方法

假设 `oldCapacity` 接近 `Integer.MAX_VALUE` 的时候，那么 `newCapacity = newCapacity = oldCapacity + (oldCapacity >> 1)` ，此时 `newCapacity` 必然发生溢出，从而变成负数。

在 `newCapacity` 发生溢出时 ，如果使用 `newCapacity < minCapacity` 和 来进行判断，判断结果很明显为 `true`，此时就会走 if 表达式中的内容，将 `newCapacity` 设置为 `minCapacity`。这种情况从逻辑上来说很明显不正确。

因此这里使用的是 `if (newCapacity - minCapacity < 0)` 和 `if (newCapacity - MAX_ARRAY_SIZE > 0)` 来判断。当 `newCapacity` 为负数时，`newCapacity - minCapacity` 必然也会发生溢出，也就意味着 `newCapacity - minCapacity` 的结果是大于0的，这种情况下就会跳过执行 `if (newCapacity - MAX_ARRAY_SIZE > 0)` 中的逻辑。此时第二个if条件 `newCapacity - MAX_ARRAY_SIZE` 的结果肯定也是大于0的，此时 `newCapacity = hugeCapacity(minCapacity)`，从而消除了 `newCapacity` 发生溢出的情况。



![ArrayList_grow](E:\Development\Typora\images\ArrayList_grow.png)





###  add(), addAll()

跟C++ 的*vector*不同，*ArrayList*没有`push_back()`方法，对应的方法是`add(E e)`，*ArrayList*也没有`insert()`方法，对应的方法是`add(int index, E e)`。这两个方法都是向容器中添加新元素，这可能会导致*capacity*不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过`grow()`方法完成的。



```java
/**
 * 将指定元素附加到此列表的末尾。
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

/**
 * 在此列表中的指定位置插入指定元素。将当前位于该位置的元素（如果有）和任何后续元素向右移动（将其索引加一）
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```



![ArrayList_add](E:\Development\Typora\images\ArrayList_add.png)



`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。

`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本，一个是在末尾添加的`addAll(Collection<? extends E> c)`方法，一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。

```java
/**
 * 按照指定集合的​​迭代器返回的顺序，将指定集合中的所有元素附加到此列表的末尾。如果在操作正在进行时修改了指定的集合，则此操作的行为是未定义的。 （这意味着如果指定的集合是这个列表，并且这个列表是非空的，那么这个调用的行为是未定义的。
 *
 * @param c collection containing elements to be added to this list
 * @return <tt>true</tt> if this list changed as a result of the call
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount 自动扩容
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

/**
 * 将指定集合中的所有元素插入此列表，从指定位置开始。将当前位于该位置的元素（如果有）和任何后续元素向右移动（增加它们的索引）。新元素将按照指定集合的​​迭代器返回的顺序出现在列表中。
 *
 * @param index index at which to insert the first element from the
 *              specified collection
 * @param c collection containing elements to be added to this list
 * @return <tt>true</tt> if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount 自动扩容

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/java/collection/java-collection-ArrayList.html

### set()

既然底层是一个数组*ArrayList*的`set()`方法也就变得非常简单，直接对数组的指定位置赋值即可。

```java
public E set(int index, E element) {
    rangeCheck(index);//下标越界检查
    E oldValue = elementData(index);
    elementData[index] = element;//赋值到指定位置，复制的仅仅是引用
    return oldValue;
}
   
```

### [¶](#get) get()

`get()`方法同样很简单，唯一要注意的是由于底层数组是Object[]，得到元素后需要进行类型转换。

### remove()

`remove()`方法也有两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
    return oldValue;
}

```

关于Java GC这里需要特别说明一下，**有了垃圾收集器并不意味着一定不会有内存泄漏**。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋`null`值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。

###  trimToSize()

ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能。它可以通过`trimToSize`方法来实现。代码如下:



```java
/**
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an <tt>ArrayList</tt> instance.
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```



更多方法请查看源码