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





# 三、Collection - LinkedList源码解析

## 概述

*`LinkedList`*同时实现了*`List`*接口和*`Deque`*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列(*Queue*)，同时又可以看作一个栈(*Stack*)。这样看来，*LinkedList*简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用***LinkedList***，一方面是因为Java官方已经声明不建议使用*Stack*类，更遗憾的是，Java里根本没有一个叫做*Queue*的类(它是个接口名字)。关于栈或队列，现在的首选是***ArrayDeque***，它有着比*LinkedList*(当作栈或队列使用时)有着更好的性能。

![LinkedList_base](E:\Development\Typora\images\LinkedList_base-16622975964991.png)

*LinkedList*的实现方式决定了所有跟下标相关的操作都是线性时间，而在首段或者末尾删除元素只需要常数时间。为追求效率*LinkedList*没有实现同步(synchronized)，如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装。

## [¶](#linkedlists实现) LinkedList实现

### [¶](#底层数据结构) 底层数据结构

*LinkedList*底层**通过双向链表实现**，本节将着重讲解插入和删除元素时双向链表的维护过程，也即是之间解跟*List*接口相关的函数，而将*Queue*和*Stack*以及*Deque*相关的知识放在下一节讲。双向链表的每个节点用`内部类Node` 表示。*LinkedList*通过`first`和`last`引用分别指向链表的第一个和最后一个元素。注意这里没有所谓的哑元，当链表为空的时候`first`和`last`都指向`null`。

```java
transient int size = 0;

/**
 * 指向第一个节点的指针
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * 指向第一个节点的指针
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;
```

其中`Node`是私有的内部类:

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```



### 构造函数

```java
/**
 * Constructs an empty list.
 */
public LinkedList() {
}

/**
 * 按照集合的迭代器返回的顺序构造一个包含指定集合元素的列表。
 *
 * @param  c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```



### getFirst(), getLast()

获取第一个元素， 和获取最后一个元素:

```java
/**
 * Returns the first element in this list.
 *
 * @return the first element in this list
 * @throws NoSuchElementException if this list is empty
 */
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

/**
 * Returns the last element in this list.
 *
 * @return the last element in this list
 * @throws NoSuchElementException if this list is empty
 */
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```



###  removeFirst(), removeLast(), remove(e), remove(index)

![LinkedList_remove.png](E:\Development\Typora\images\LinkedList_remove-16622980990204.png)



删除元素 - 指的是删除第一次出现的这个元素, 如果没有这个元素，则返回false；判断的依据是equals方法， 如果equals，则直接`unlink`这个node；

**由于LinkedList可存放null元素，故也可以删除第一次出现null的元素；**

```java
/**
从此列表中删除第一次出现的指定元素（如果存在）。如果此列表不包含该元素，则它不变。更正式地说，删除具有最低索引i的元素，使得(o==null ? get(i)==null : o.equals(get(i))) （如果存在这样的元素）。如果此列表包含指定的元素（或等效地，如果此列表因调用而更改），则返回true 
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if this list contained the specified element
 */
public boolean remove(Object o) {
    if (o == null) { //删除第一次出现null的元素
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

/**
 * Unlinks non-null node x.
 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {// 第一个元素
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {// 最后一个元素
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null; // 让GC生效
    size--;
    modCount++;
    return element;
}
```

`remove(int index)`使用的是下标计数， 只需要判断该index是否有元素即可，如果有则直接unlink这个node。

```java
/**
 * 移除此列表中指定位置的元素。将任何后续元素向左移动（从它们的索引中减去 1）。返回从列表中删除的元素
 *
 * @param index the index of the element to be removed
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index); //判断是否越界
    return unlink(node(index));
}
```



删除head元素:

```java
/**
 * Removes and returns the first element from this list.
 *
 * @return the first element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```


```java
/**
 * Unlinks non-null first node f.
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

删除last元素:

```java
/**
 * Removes and returns the last element from this list.
 *
 * @return the last element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

/**
 * Unlinks non-null last node l.
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```



### add()

*add()方法有两个版本，一个是`add(E e)`，该方法在\*`LinkedList`*的**末尾**插入元素，因为有`last`指向链表末尾，在末尾插入元素的花费是常数时间。只需要简单修改几个相关引用即可；另一个是`add(int index, E element)`，该方法是在指定下表处插入元素，需要先通过线性查找找到具体位置，然后修改相关引用完成插入操作。

```java
/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * Links e as last element.
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null); //(pre,当前,next)
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

![LinkedList_add](E:\Development\Typora\images\LinkedList_add.png)



`add(int index, E element)`, 当index==size时，等同于`add(E e)`(向链表最后添加); 如果不是，则分两步: 1.先根据index找到要插入的位置,即node(index)方法；2.修改引用，完成插入操作。

```java
/**
 * Inserts the specified element at the specified position in this list.
 * Shifts the element currently at that position (if any) and any
 * subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```



上面代码中的`node(int index)`函数有一点小小的trick，因为链表双向的，可以从开始往后找，也可以从结尾往前找，具体朝那个方向找取决于条件`index < (size >> 1)`，也即是index是靠近前端还是后端。从这里也可以看出，linkedList通过index检索元素的效率没有arrayList高。



```java
/**
 * Returns the (non-null) Node at the specified element index.
 */
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

###  addAll()

`addAll(index, c)` 实现方式并不是直接调用`add(index,e)`来实现，主要是因为效率的问题，另一个是fail-fast中`modCount`只会增加1次；

```java
/**
 * Appends all of the elements in the specified collection to the end of
 * this list, in the order that they are returned by the specified
 * collection's iterator.  The behavior of this operation is undefined if
 * the specified collection is modified while the operation is in
 * progress.  (Note that this will occur if the specified collection is
 * this list, and it's nonempty.)
 *
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * Inserts all of the elements in the specified collection into this
 * list, starting at the specified position.  Shifts the element
 * currently at that position (if any) and any subsequent elements to
 * the right (increases their indices).  The new elements will appear
 * in the list in the order that they are returned by the
 * specified collection's iterator.
 *
 * @param index index at which to insert the first element
 *              from the specified collection
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### clear()

为了让GC更快可以回收放置的元素，需要将node之间的引用关系赋空。

```java
/**
 * Removes all of the elements from this list.
 * The list will be empty after this call returns.
 */
public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```



###  Positional Access 方法

通过`index`获取元素

```java
/**
 * Returns the element at the specified position in this list.
 *
 * @param index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

将某个位置的元素重新赋值:

```java
/**
 * Replaces the element at the specified position in this list with the
 * specified element.
 *
 * @param index index of the element to replace
 * @param element element to be stored at the specified position
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```



将元素插入到指定index位置:

```java
/**
 * Inserts the specified element at the specified position in this list.
 * Shifts the element currently at that position (if any) and any
 * subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

删除指定位置的元素:

```java
/**
 * Removes the element at the specified position in this list.  Shifts any
 * subsequent elements to the left (subtracts one from their indices).
 * Returns the element that was removed from the list.
 *
 * @param index the index of the element to be removed
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

其它位置的方法:

```java
/**
 * Tells if the argument is the index of an existing element.
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}

/**
 * Tells if the argument is the index of a valid position for an
 * iterator or an add operation.
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

/**
 * Constructs an IndexOutOfBoundsException detail message.
 * Of the many possible refactorings of the error handling code,
 * this "outlining" performs best with both server and client VMs.
 */
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}

private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### 查找操作

查找操作的本质是查找元素的下标:

查找第一次出现的index, 如果找不到返回-1；

```java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index {@code i} such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 *
 * @param o element to search for
 * @return the index of the first occurrence of the specified element in
 *         this list, or -1 if this list does not contain the element
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

查找最后一次出现的index, 如果找不到返回-1；

```java
/**
 * Returns the index of the last occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the highest index {@code i} such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 *
 * @param o element to search for
 * @return the index of the last occurrence of the specified element in
 *         this list, or -1 if this list does not contain the element
 */
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### Queue 方法


```java
/**
 * Retrieves, but does not remove, the head (first element) of this list.
 *
 * @return the head of this list, or {@code null} if this list is empty
 * @since 1.5
 */
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

/**
 * Retrieves, but does not remove, the head (first element) of this list.
 *
 * @return the head of this list
 * @throws NoSuchElementException if this list is empty
 * @since 1.5
 */
public E element() {
    return getFirst();
}

/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list, or {@code null} if this list is empty
 * @since 1.5
 */
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list
 * @throws NoSuchElementException if this list is empty
 * @since 1.5
 */
public E remove() {
    return removeFirst();
}

/**
 * Adds the specified element as the tail (last element) of this list.
 *
 * @param e the element to add
 * @return {@code true} (as specified by {@link Queue#offer})
 * @since 1.5
 */
public boolean offer(E e) {
    return add(e);
}
```

###  Deque 方法

```java
/**
 * Inserts the specified element at the front of this list.
 *
 * @param e the element to insert
 * @return {@code true} (as specified by {@link Deque#offerFirst})
 * @since 1.6
 */
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

/**
 * Inserts the specified element at the end of this list.
 *
 * @param e the element to insert
 * @return {@code true} (as specified by {@link Deque#offerLast})
 * @since 1.6
 */
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

/**
 * Retrieves, but does not remove, the first element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the first element of this list, or {@code null}
 *         if this list is empty
 * @since 1.6
 */
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
 }

/**
 * Retrieves, but does not remove, the last element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the last element of this list, or {@code null}
 *         if this list is empty
 * @since 1.6
 */
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}

/**
 * Retrieves and removes the first element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the first element of this list, or {@code null} if
 *     this list is empty
 * @since 1.6
 */
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * Retrieves and removes the last element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the last element of this list, or {@code null} if
 *     this list is empty
 * @since 1.6
 */
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

/**
 * Pushes an element onto the stack represented by this list.  In other
 * words, inserts the element at the front of this list.
 *
 * <p>This method is equivalent to {@link #addFirst}.
 *
 * @param e the element to push
 * @since 1.6
 */
public void push(E e) {
    addFirst(e);
}

/**
 * Pops an element from the stack represented by this list.  In other
 * words, removes and returns the first element of this list.
 *
 * <p>This method is equivalent to {@link #removeFirst()}.
 *
 * @return the element at the front of this list (which is the top
 *         of the stack represented by this list)
 * @throws NoSuchElementException if this list is empty
 * @since 1.6
 */
public E pop() {
    return removeFirst();
}

/**
 * Removes the first occurrence of the specified element in this
 * list (when traversing the list from head to tail).  If the list
 * does not contain the element, it is unchanged.
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if the list contained the specified element
 * @since 1.6
 */
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

/**
 * Removes the last occurrence of the specified element in this
 * list (when traversing the list from head to tail).  If the list
 * does not contain the element, it is unchanged.
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if the list contained the specified element
 * @since 1.6
 */
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```



# 四、[¶](https://pdai.tech/md/java/collection/java-collection-Queue&Stack.html#collection---stack-amp-queue-源码解析)Collection - Stack & Queue 源码解析



## Stack & Queue概述

Java里有一个叫做*`Stack`*的类，却没有叫做*`Queue`的类(它是个接口名字)。当需要使用栈时，Java已不推荐使用*Stack*，而是推荐使用更高效的*`ArrayDeque`*；既然*Queue*只是一个接口，当需要使用队列时也就首选*ArrayDeque*了(次选是*LinkedList*)。

## [¶](#queue) Queue

*Queue*接口继承自Collection接口，除了最基本的Collection的方法之外，它还支持额外的`insertion, extraction`和*`inspection`*操作。这里有两组格式，共6个方法，一组是抛出异常的实现；另外一组是返回值的实现(没有则返回null)。

|         | Throws exception | Returns special value |
| ------- | ---------------- | --------------------- |
| Insert  | add(e)           | offer(e)              |
| Remove  | remove()         | poll()                |
| Examine | element()        | peek()                |

## [¶](#deque) Deque

​	注意Deque是接口傲！！！

`Deque`是"double ended queue", 表示双向的队列，英文读作"deck". Deque 继承自 Queue接口，**除了支持Queue**的方法之外，还支持`insert`, `remove`和`examine`操作，由于Deque是双向的，所以可以对队列的头和尾都进行操作，它同时也支持两组格式，一组是抛出异常的实现；另外一组是返回值的实现(没有则返回null)。共12个方法如下:

|         | First Element - Head |               | Last Element - Tail |               |
| ------- | -------------------- | ------------- | ------------------- | ------------- |
|         | Throws exception     | Special value | Throws exception    | Special value |
| Insert  | addFirst(e)          | offerFirst(e) | addLast(e)          | offerLast(e)  |
| Remove  | removeFirst()        | pollFirst()   | removeLast()        | pollLast()    |
| Examine | getFirst()           | peekFirst()   | getLast()           | peekLast()    |

当把`Deque`当做FIFO（先进先出）的`queue`来使用时，元素是从`deque`的尾部添加，从头部进行删除的； 所以`deque`的部分方法是和`queue`是等同的。具体如下:

| Queue Method | Equivalent Deque Method |
| ------------ | ----------------------- |
| add(e)       | addLast(e)              |
| offer(e)     | offerLast(e)            |
| remove()     | removeFirst()           |
| poll()       | pollFirst()             |
| element()    | getFirst()              |
| peek()       | peekFirst()             |

*Deque*的含义是“double ended queue”，即双端队列，它既可以当作栈使用，也可以当作队列使用。下表列出了*Deque*与*Queue*相对应的接口:

| Queue Method | Equivalent Deque Method | 说明                                   |
| ------------ | ----------------------- | -------------------------------------- |
| `add(e)`     | `addLast(e)`            | 向队尾插入元素，失败则抛出异常         |
| `offer(e)`   | `offerLast(e)`          | 向队尾插入元素，失败则返回`false`      |
| `remove()`   | `removeFirst()`         | 获取并删除队首元素，失败则抛出异常     |
| `poll()`     | `pollFirst()`           | 获取并删除队首元素，失败则返回`null`   |
| `element()`  | `getFirst()`            | 获取但不删除队首元素，失败则抛出异常   |
| `peek()`     | `peekFirst()`           | 获取但不删除队首元素，失败则返回`null` |

下表列出了*Deque*与*Stack*对应的接口:

| Stack Method | Equivalent Deque Method | 说明                                   |
| ------------ | ----------------------- | -------------------------------------- |
| `push(e)`    | `addFirst(e)`           | 向栈顶插入元素，失败则抛出异常         |
| 无           | `offerFirst(e)`         | 向栈顶插入元素，失败则返回`false`      |
| `pop()`      | `removeFirst()`         | 获取并删除栈顶元素，失败则抛出异常     |
| 无           | `pollFirst()`           | 获取并删除栈顶元素，失败则返回`null`   |
| `peek()`     | `getFirst()`            | 获取但不删除栈顶元素，失败则抛出异常   |
| 无           | `peekFirst()`           | 获取但不删除栈顶元素，失败则返回`null` |

上面两个表共定义了*Deque*的12个接口。添加，删除，取值都有两套接口，它们功能相同，区别是对失败情况的处理不同。**一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值(`false`或`null`)**。除非某种实现对容量有限制，大多数情况下，添加操作是不会失败的。**虽然\Deque\的接口有12个之多，但无非就是对容器的两端进行操作，或添加，或删除，或查看**。明白了这一点讲解起来就会非常简单。

***ArrayDeque*和*LinkedList*是*Deque*的两个通用实现**，由于官方更推荐使用*AarryDeque*用作栈和队列，加之上一篇已经讲解过*LinkedList*，本文将着重讲解*ArrayDeque*的具体实现。



从名字可以看出*ArrayDeque*底层通过`数组`实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即**`循环数组`(circular array)**，也就是说数组的任何一点都可能被看作起点或者终点。*ArrayDeque*是非线程安全的(not thread-safe)，当多个线程同时使用的时候，需要程序员手动同步；另外，该容器不允许放入`null`元素。

![ArrayDeque_base.png](E:\Development\Typora\images\ArrayDeque_base.png)

上图中我们看到，**`head`指向首端第一个有效元素，`tail`指向尾端第一个可以插入元素的空位**。因为是循环数组，所以`head`不一定总等于0，`tail`也不一定总是比`head`大。

## [¶](#方法剖析) 方法剖析

### [¶](#addfirst) addFirst()

`addFirst(E e)`的作用是在*Deque*的首端插入元素，也就是在`head`的前面插入元素，在空间足够且下标没有越界的情况下，只需要将`elements[--head] = e`即可。

![ArrayDeque_addFirst.png](E:\Development\Typora\images\ArrayDeque_addFirst.png)

实际需要考虑: 1.空间是否够用，以及2.下标是否越界的问题。上图中，如果`head`为`0`之后接着调用`addFirst()`，虽然空余空间还够用，但`head`为`-1`，下标越界了。下列代码很好的解决了这两个问题。

```java
//addFirst(E e)
public void addFirst(E e) {
    if (e == null)//不允许放入null
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;//2.下标是否越界
    if (head == tail)//1.空间是否够用
        doubleCapacity();//扩容
}
```

上面的`elements`是`ArrayQueue`的底层实现 --数组

```java
    transient Object[] elements; // non-private to simplify nested class access
```

上述代码我们看到，**空间问题是在插入之后解决的**，因为`tail`总是指向下一个可插入的空位，也就意味着`elements`数组至少有一个空位，所以插入元素的时候不用考虑空间问题。

下标越界的处理解决起来非常简单，`head = (head - 1) & (elements.length - 1)`就可以了，**这段代码相当于取余，同时解决了`head`为负值的情况**。因为`elements.length`必需是`2`的指数倍，`elements - 1`就是二进制低位全`1`，跟`head - 1`相与之后就起到了取模的作用，如果`head - 1`为负数(其实只可能是-1)，则相当于对其取相对于`elements.length`的补码。

下面再说说扩容函数`doubleCapacity()`，其逻辑是申请一个更大的数组(原数组的两倍)，然后将原数组复制过去。过程如下图所示:

![ArrayDeque_doubleCapacity.png](E:\Development\Typora\images\ArrayDeque_doubleCapacity.png)



图中我们看到，复制分两次进行，第一次复制`head`右边的元素，第二次复制`head`左边的元素。

```java
//doubleCapacity()
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // head右边元素的个数
    int newCapacity = n << 1;//原空间的2倍
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);//复制右半部分，对应上图中绿色部分
    System.arraycopy(elements, 0, a, r, p);//复制左半部分，对应上图中灰色部分
    elements = (E[])a;
    head = 0;
    tail = n;
}

```

###  addLast()

`addLast(E e)`的作用是在*Deque*的尾端插入元素，也就是在`tail`的位置插入元素，由于`tail`总是指向下一个可以插入的空位，因此只需要`elements[tail] = e;`即可。插入完成后再检查空间，如果空间已经用光，则调用`doubleCapacity()`进行扩容。

![ArrayDeque_addLast.png](E:\Development\Typora\images\ArrayDeque_addLast.png)



```java
public void addLast(E e) {
    if (e == null)//不允许放入null
        throw new NullPointerException();
    elements[tail] = e;//赋值
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)//下标越界处理
        doubleCapacity();//扩容
}

```

### pollFirst()

`pollFirst()`的作用是删除并返回*Deque*首端元素，也即是`head`位置处的元素。如果容器不空，只需要直接返回`elements[head]`即可，当然还需要处理下标的问题。由于`ArrayDeque`中不允许放入`null`，当`elements[head] == null`时，意味着容器为空。

```java
public E pollFirst() {
    int h = head;
    E result = elements[h];
    if (result == null)//null值意味着deque为空
        return null;
    elements[h] = null;//let GC work
    head = (h + 1) & (elements.length - 1);//下标越界处理
    return result;
}
```

### pollLast()

`pollLast()`的作用是删除并返回*Deque*尾端元素，也即是`tail`位置前面的那个元素。

```java
public E pollLast() {
    int t = (tail - 1) & (elements.length - 1);//tail的上一个位置是最后一个元素
    E result = elements[t];
    if (result == null)//null值意味着deque为空
        return null;
    elements[t] = null;//let GC work
    tail = t;
    return result;
}
  
        @pdai: 代码已经复制到剪贴板
    
```

### [¶](#peekfirst) peekFirst()

`peekFirst()`的作用是返回但不删除*Deque*首端元素，也即是`head`位置处的元素，直接返回`elements[head]`即可。

```java
public E peekFirst() {
    return elements[head]; // elements[head] is null if deque empty
}
  
        @pdai: 代码已经复制到剪贴板
    
```

### [¶](#peeklast) peekLast()

`peekLast()`的作用是返回但不删除*Deque*尾端元素，也即是`tail`位置前面的那个元素。

```java
public E peekLast() {
    return elements[(tail - 1) & (elements.length - 1)];
}
  
```





# 五、Map - HashSet & HashMap 源码解析



著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/java/collection/java-map-HashMap&HashSet.html

## Java7 HashMap

### [¶](#概述) 概述

之所以把*`HashSet`*和*`HashMap`*放在一起讲解，是因为二者在Java里有着相同的实现，前者仅仅是对后者做了一层包装，也就是说*HashSet*里面有一个*HashMap*(适配器模式)。因此本文将重点分析*HashMap*。

*HashMap*实现了*Map*接口，即允许放入`key`为`null`的元素，也允许插入`value`为`null`的元素；除该类未实现同步外，其余跟`Hashtable`大致相同；跟*TreeMap*不同，该容器**不保证**元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个*HashMap*的顺序可能会不同。 根据对冲突的处理方式不同，哈希表有两种实现方式，一种开放地址方式(Open addressing)，另一种是冲突链表方式(Separate chaining with linked lists)。**Java7 \*HashMap\*采用的是冲突链表方式**。

![HashMap_base](E:\Development\Typora\images\HashMap_base-16623646264329.png)

从上图容易看出，如果选择合适的哈希函数，`put()`和`get()`方法可以在常数时间内完成。但在对*HashMap*进行迭代时，需要遍历整个table以及后面跟的冲突链表。因此对于迭代比较频繁的场景，不宜将*HashMap*的初始大小设的过大。

有两个参数可以影响*HashMap*的性能: 	

​		<font color='red'>初始容量(inital capacity)和负载系数(load factor)。</font>

初始容量指定了初始`table`的大小，负载系数用来指定自动扩容的临界值。当`entry`的数量超过`capacity*load_factor`时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。

将对象放入到*HashMap*或*HashSet*中时，有两个方法需要特别关心: `hashCode()`和`equals()`。**`hashCode()`方法决定了对象会被放到哪个`bucket`里，当多个对象的哈希值冲突时，`equals()`方法决定了这些对象是否是“同一个对象”**。所以，如果要将自定义的对象放入到`HashMap`或`HashSet`中，需要**@Override** `hashCode()`和`equals()`方法。

### [¶](#get) get()

`get(Object key)`方法根据指定的`key`值返回对应的`value`，该方法调用了`getEntry(Object key)`得到相应的`entry`，然后返回`entry.getValue()`。因此`getEntry()`是算法的**核心**。 算法思想是首先通过`hash()`函数得到对应`bucket`的下标，然后依次遍历冲突链表，通过`key.equals(k)`方法来判断是否是要找的那个`entry`。

![HashMap_getEntry](E:\Development\Typora\images\HashMap_getEntry-166236513146212.png)

上图中`hash(k)&(table.length-1)`等价于`hash(k)%table.length`，原因是*HashMap*要求`table.length`必须是2的指数，因此`table.length-1`就是二进制低位全是1，跟`hash(k)`相与会将哈希值的高位全抹掉，剩下的就是余数了。

```java
//getEntry()方法
final Entry<K,V> getEntry(Object key) {
	......
	int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[hash&(table.length-1)];//得到冲突链表
         e != null; e = e.next) {//依次遍历冲突链表中的每个entry
        Object k;
        //依据equals()方法判断是否相等
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}

```

​	

未完待续…
