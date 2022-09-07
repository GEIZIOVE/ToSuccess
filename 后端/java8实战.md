# 4.流









# 10.Optional

## 1.7 Optional类型

optional<T＞对象是一种包装器对象，① 要么包装了类型T的对象，② 要么没有包装任何对象。

对于第一种情况， 我们称这种值是存在的。 

### 1.7.1 获取Optional值

理想的方法：它在值不存在的情况下会产生一个可替代物， 而只有在值存在的情况下才会使用这个值。

第一条策略：

​	在没有任何匹配时， 我们会希望使用某种默认值， 可能是空字符串：

```java
String result = optionalString.orElse("") ;
```

**API java.util.Optional**

- ​	T  orElse(T other)  	

​			产生这个Optional的值， 或者在该Optional为空时， 产生 other。

![image-20220507200017344](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220507200017344.png)

### 1.7.2 消费Optional值

**ifPresent()**

方法会接受一个函数 。如果可选值存在， 那么它会被传递给该函数。 否则 ， 不会发生任何事情。

```java
optionalValue.ifPresent(v-> Process v);
```

eg:如果在该值存在的情况下想要将其添加到某个集中．

```
optionalValue.ifPresent(v-> results.add(v));
```

如果想要在可选值存在时执行一种动作，在可选值不存在时执行另一种动作，可以使用

 **ifPresentOrElse()**

```java
optionalValue.ifPresentOrElse(
	v ->System.out.println(“Found “ + v),
	() -> logger.warning（"No match"));
```

### 1.7.3 管道化Optional值 待

有关 Optional 类型正确用法的提示：

- Optional 类型的变量永远都不应该为 null。
- 不要使用 Optional 类型的域。 因为其代价是额外多出来一个对象。 在类的内部， 使用 null 表示缺失的域更易于操作。
- 不要在集合中放置Optional对象 ， 并且不要将它们用 作map的键 应该直接收集其中的值。

### 1.7.5 创建Optional值

![image-20220507200407518](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220507200407518.png)





## 10.2 Optional 类入门

​		变量存在时，Optional类只是对类简单**封装**。变量不存在时，缺失的值会被建模成一个“空”
的Optional对象，由方法Optional.empty()返回。

![image-20220512082428992](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220512082428992.png)

**Optional.empty()**：

​	返回Optional类的特定单一实例。

null引用和Optional.empty()的区别：

​	如果你尝试解引用一个null ， 一定会触发NullPointerException ， 不过使用
Optional.empty()就完全没事儿，

## 10.3 应用Optional 的几种模式

### 10.3.1 创建Optional 对象

1. **声明一个空的Optional**   

   **Optional.empty()**

​	Optional<Car> optCar = Optional.empty();

2. **依据一个非空值创建Optional**

```java
Optional.of（）
```

​	Optional<Car> optCar = Optional.of(car);

​	如果car是一个null，这段代码会**立即**抛出一个NullPointerException，而不是等到你
试图访问car的属性值时才返回一个错误。

3. **可接受null的Optional**

​	**Optional.ofNullable（）**

​	Optional<Car> optCar = Optional.ofNullable(car);

​	如果car是null，那么得到的Optional对象就是个空对象。

### 10.3.2 使用map 从Optional 对象中提取和转换值

从概念上和流的map很像。

你可以把Optional对象看成一种特殊的集合数据，**它至多包含一个元素**。如果Optional包含一个值，那函数就将该值作为参数传递给map，对该值进行转换。如果Optional为空，就什么也不做。

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

![image-20220512083216475](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220512083216475.png)

### 10.3.3 使用flatMap 链接Optional 对象

对于代码

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name =
        optPerson.map(Person::getCar)
        .map(Car::getInsurance)
        .map(Insurance::getName);
```

​	getCar返回的是一个Optional<Car>类型的对象，这意味着map方法的结果是一个Optional<Optional<Car>>类型的对象， 他对getInsurance（）方法的调用是非法的

![image-20220512084130939](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220512084130939.png)

![image-20220512083653785](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220512083653785.png)

​		这个例子中，传递给流的flatMap方法会将每个正方形转换为另一个流中的两个三角形。那么，map操作的结果就包含有三个新的流，每一个流包含两个三角形，但flatMap方法会将这种两层的流**合并**为一个包含六个三角形的单一流。类似地，传递给optional的flatMap方法的函数会将原始包含正方形的optional对象转换为包含三角形的optional对象。如果将该方法传递给map方法，结果会是一个Optional对象，而这个Optional对象中包含了三角形；**但flatMap方法会将这种两层的Optional对象转换为包含三角形的单一Optional对象。**	



对之前代码改为

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");
}
```

![image-20220512084611833](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220512084611833.png)

### 10.3.4 默认行为及解引用Optional 对象

** get()**

​	是这些方法中**最简单但又最不安全**的方法。如果变量存在，它直接返回封装的变量
值，否则就抛出一个NoSuchElementException异常。所以，除非你非常确定Optional
变量一定包含值，否则使用这个方法是个相当糟糕的主意。此外，这种方式即便相对于
嵌套式的null检查，也并未体现出多大的改进。
 orElse(T other)

​	是我们在代码清单10-5中使用的方法，正如之前提到的，它允许你在
Optional对象不包含值时提供一个默认值。
** orElseGet(Supplier<? extends T> other)**

​	是orElse方法的延迟调用版，Supplier方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作，你应该考虑采用这种方式（借此提升程序的性能），或者你需要非常确定某个方法仅在Optional为空时才进行调用，也可以考虑该方式（这种情况有严格的限制条件）。
** orElseThrow(Supplier<? extends X> exceptionSupplier)**

​	和get方法非常类似，它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希
望抛出的异常类型。
** ifPresent(Consumer<? super T>)**

​	让你能在变量值存在时执行一个作为参数传入的方法，否则就不进行任何操作。



## 10.4 相关API概括

![image-20220512084934859](E:\Development\Typora\images\image-20220512084934859.png)

![image-20220512084950987](E:\Development\Typora\images\image-20220512084950987.png)









# 12.新的日期和时间API

