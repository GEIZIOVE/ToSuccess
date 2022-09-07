# Java - 序列化和反序列化

## 官方文档理解

要使类的成员变量可以序列化和反序列化，必须实现Serializable接口。任何可序列化类的子类都是可序列化的。Serializable接口没有提供任何方法和字段，只是标记可以序列化。

为了允许不可序列化类的子类可序列化，子类要承担父类的public，protected和包内可访问(default)修饰的字段。该父类必须有一个子类可访问的无参构造器，去初始化它的属性。

在反序列化过程中，不可序列化类的字段通过public或protected修饰的无参构造器初始化。这个构造器必须是子类可访问的。而可序列化子类的字段从流中恢复。

> 这里没有提到使用friendly修饰的构造器，应该是不确定子类和父类属于同一包中。如果是一个包，应该也可以。因为friendly修饰的构造器也可以被同一个包下的子类访问。

如果对一个不可序列化对象进行序列化操作时，会抛出NotSerializableException标记该类不可序列化。

如果一个类在序列化和反序列化中需要制定一些特殊操作，必须实现一下方法签名的特殊方法：



```java
 private void writeObject(java.io.ObjectOutputStream out)
     throws IOException
 private void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException;
 private void readObjectNoData()
     throws ObjectStreamException;
```

`writeObject()`方法职责是将特定类的对象属性输出，这样，相应的`readObject()`可以恢复。可以调用`out.defaultWriteObject()`使用默认保存对象属性的机制。`out.defaultWriteObject()`自身不必考虑使用的变量是属于父类还是子类。通过使用`writeObject`或通过使用DataOutput支持的基本数据类型的方法将各个字段写入ObjectOutputStream来保存状态。

`readObject()`主要责任是从流中读取并恢复类的字段。它可以通过调用`in.defaultReadObject()`采用默认机制恢复非静态和non-transient字段。`in.defaultReadObject()`使用流中的信息来将流中保存的对象的字段分配给当前对象中相应命名的字段。当类添加了新字段，它依旧可以使用。`in.defaultReadObject()`自身不必考虑使用的变量是属于父类还是子类。通过使用`writeObject`或通过使用DataOutput支持的基本数据类型的方法将各个字段写入ObjectOutputStream来保存状态。

`readObjectNoData()`：Serializable对象反序列化时，由于序列化与反序列化提供的class版本不同，序列化的class的super class不同于序列化时的class的super class；或者收到有敌意的流；或接收不完整；都会对初始化对象字段值时造成影响。针对这些情况可以在在该方法中实现这些字段的初始化。如果没有定义`readObjectNoData()`，这些字段会初始化成JVM默认值。

序列化类在将对象写入流中时，可以指定一个替代对象写入。但是必须实现精确的方法签名：



```java
ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
```

如果`writeReplace()`存在，序列化时会被调用。它的权限修饰符可以是任何一个，子类遵循Java权限访问规则。

当一个类的对像从流中读取时，指派另一个类作为返回值。必须实现精确的方法签名：



```java
ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
```

该方法有着与`writeReplace()`类似的调用和访问规则。

Java允许为序列化的类提供一个serialVersionUID的常量标识该类的版本。只要serialVersionUID的值不变，Java就会把它们当作相同的序列化版本。例如，一个类升级后，它的serialVersionUID类变量值保持不变，序列化机制也会把它们当成同一个类版本。



```java
ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
```

如果反序列时，发送方的类(指序列化时使用的类文件)与接受方的(指反序列化时使用的类文件)类的各自serialVersionUID不同，那么会抛出InvalidClassException异常。

JVM就会根据类的各个方面计算出一个serialVersionUID的值。不同的编译器下会产生不同的serialVersionUID值。serialVersionUID值不同则会导致反序列化程序编译失败。解决办法是显示指定一个serialVersionUID。这样，即使在某个对象被序列化后，它所对应的类被修改了，该对象也依然可以被正确的反序列化。

## 实践

### 可序列化子类默认实现序列化

这应该是毋庸置疑的。因为父类实现了Serializable接口，那么子类必然可序列化。

### 不可序列化父类的子类可否序列化

从文档中得知，只要父类有一个子类可访问的无参构造器能够初始化父类自身的字段，就可行。那么没有访问权限修饰符号修饰的无参构造器可行吗？



```java
public class FriendlyConstructorFather {
    private int number;
    FriendlyConstructorFather() {
        this.number = 12;
    }
    public int getNumber() {
        return this.number;
    }

    public void setNumber(int num) {
        this.number = num;
    }

}

public class FriendlyConstructorSon extends FriendlyConstructorFather
    implements Serializable{
    private int sonNum;
    public FriendlyConstructorSon() {

    }

    public int getSonNum() {
        return sonNum;
    }

    public void setSonNum(int number) {
        sonNum = number;
    }
}


public class TestFriendlyConstructor {
    public static void main(String[] agrs) {
        FriendlyConstructorSon son = new FriendlyConstructorSon();
        son.setSonNum(2);

        FileOutputStream fileOut = null;
        ObjectOutputStream objectOut = null;

        File file = new File("../file/TestFriendlyConstructor.txt");

        try {
            try {
                fileOut = new FileOutputStream(file);
                objectOut = new ObjectOutputStream(fileOut);

                objectOut.writeObject(son);
            }finally {
                objectOut.close();
            }
        }catch(IOException e) {
            e.printStackTrace();
        }

        FileInputStream fileIn = null;
        ObjectInputStream objectIn = null;

        try {
            try {
                fileIn = new FileInputStream(file);
                objectIn = new ObjectInputStream(fileIn);

                FriendlyConstructorSon resultSon =
                    (FriendlyConstructorSon) objectIn.readObject();

                System.out.println(
                    "the father's number is " + resultSon.getNumber());

                System.out.println(
                    "the son's num is " + resultSon.getSonNum());
            }finally{
                objectIn.close();
            }
        }catch(IOException e) {
            e.printStackTrace();
        }catch(ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

执行后效果：



```python
the father's number is 12
the son's num is 2
```

发现是可行的，但是要求子类必须和父类在同一个包中。

### readObjectNoDate方法使用情况

原始Person.java



```java
public class Person implements Serializable{
    private int age;
    public Person() {

    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }
}
```

并且通过序列化ObjectOutputStream输出到test.txt文件中进行保存。然后升级Person类：



```java
public class Animal implements Serializable {
    private String name;

    public Animal() {

    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    private void readObjectNoData() throws ObjectStreamException{
        this.name = "zhangsan";
    }
}

public class Person extends Animal implements Serializable {
    private int age;

    public Person() {}

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

编译新的Person类后，使用新的Person.class文件从test.txt反序列化加载Person对象。并且使用`getName()`获取字段`name`值，执行输出如下：



```csharp
the age is 25
the name is zhangsan
```

可以看到从信息不完整的序列化流中得到了完整的Person类，这要归功于`readObjectNoData()`。它初始化了`name`字段。如果在这种Person发生升级的情况下，没有定义`readObjectNoData()`那么`name`字段会初始化它们的默认值。`readObjectNoData()`一般用于序列化对象和反序列化对象父类不同的情况，还有就是为了防止信息不完整，可以使用它来进一步保证初始化。如果了类中有自定义的`readObject()`，出现上述情况时，会用`readObjectNoData()`替代它。

这里可以注意下，虽然升级了Person，且没有显示指定SerializableUID。序列化机制依旧认为升级前后的Person是同一个版本。这是因为：

- 只是修改了类的方法，不会影响反序列化。
- 只是修改了类的static Field或transient Field，不会影响反序列化。
- 修改了类的非static和非transient Field，会影响序列化。

如果此时升级Person时，继承的Animal中有与原始Person相同的字段。那么`readObjectNoDate()`的初始化无效果，会使用它们的默认值初始化。而且不显示指定SerializableUID会抛出InvalidClassException异常。唯一的解决办法就是显示指定SerializableUID，即可执行。

### 自定义序列化

#### case one

在一些特殊情况下，类中某些实例变量是敏感信息不希望被序列化，或者这些实例变量的类型不可序列化为避免发生NotSerializableException异常。可以通过关键词transient修饰这些实例变量，指定类在序列化时无需理会它们。这样一来，反序列化后得到的对象中这些字段会被初始化为默认值。

序列化注意事项：

- 对象的类名、Field（包括基本类型、数组及对其他对象的引用）都会被序列化，对象的static Field，transient Field及方法不会被序列化；
- 实现Serializable接口的类，如不想某个Field被序列化，可以使用transient关键字进行修饰；
- 保证序列化对象的引用类型Filed的类也是可序列化的，如不可序列化，可以使用transient关键字进行修饰，否则会序列化失败；
- 反序列化时必须要有序列化对象的类的class文件，而且方法不会被序列化；
- 当通过文件网络读取序列化对象的时候，必需按写入的顺序来读取。

使用transient关键字修饰实例变量避免序列化非常便捷，但该变量将被完全隔离在序列化机制之外，这样导致在反序列化恢复的对象无法取得该实例变量值。Java还提供了一种自定义序列化机制，可以让程序控制如何序列化各实例变量，甚至完全不序列化某些实例变量(与使用transient关键字的效果相同)。



```java
public class CustomSerializable implements Serializable {
    private String account;
    private transient String password;
    private int passwordCount;

    public CustomSerializable(String name, String password)
        throws Exception{
        passwordCount = password.length();
        for(int i = 0; i < passwordCount; i++) {
            char c = password.charAt(i);
            if(c < '0' && '9' < c) {
                throw new Exception("the password is not correct!");
            }
        }

        this.account = name;
        this.password = password;

    }

    private String changePassword() {
        byte[] bArray = new byte[passwordCount];
        for(int i = 0; i < passwordCount; i++) {
            bArray[i] = '*';
        }
        return new String(bArray);
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        System.out.println("custom writeObject method execute!");
        out.defaultWriteObject();
        out.writeObject(changePassword());
    }

    private void readObject(ObjectInputStream in)
        throws IOException, ClassNotFoundException {
        System.out.println("custom readObject method execute!");
        in.defaultReadObject();
        this.password = (String) in.readObject();
    }

    @Override
    public String toString() {
        return "Account: " + account + "\nPassword: " + password +
            "\nPasswrod Count: " + passwordCount;
    }
}
```

在代码中，使用了`defaultWrite/ReadObject()`去执行默认的序列化机制。

在序列化中(自定义的writeObject())，由于password是一个敏感信息，所以使用transient修饰，将其排除在默认序列化机制外。然后针对敏感信息自定义一套序列化操作，保护信息安全。

在反序列化中(自定义的readObject())，首先使用默认反序列化机制去初始化可序列化字段，然后针对自定义序列化中的password字段，使用相应的`readObject()`读取，并且赋值给反序列化对象中的同名字段。

执行效果，无自定义序列化和反序列化，transient修饰的password字段读写：

![img](E:\Development\Typora\images\webp.webp)

transient关键字.png

自定义序列化和反序列化后，transient修饰的password字段读写：

![img](E:\Development\Typora\images\webp-16622201738051.webp)

Serializable自定义序列化.png

在代码中还看到`writeObject()`,`readObject()`,`readObjectNoDate()`都是private修饰的，但是序列化过程中一样可以被外部的ObjectOut/InputStream调用。

ObjectOutputStream在执行自己的writeObject方法前会先通过反射在要被序列化的对象的类中查找有无自定义的writeObject方法，如有的话，则会优先调用自定义的writeObject方法。因为查找反射方法时使用的是getPrivateMethod，所以自定以的writeObject方法的作用域要被设置为private。通过自定义writeObject和readObject方法可以完全控制对象的序列化与反序列化。(详情可见ObjectOutputStream中的writeSerialData方法，以及ObjectInputStream中的readSerialData方法。)

> 引用自[Java对象序列化与反序列化](https://link.jianshu.com/?t=http://www.cnblogs.com/amunote/p/4171799.html)

#### case two

Java序列化机制提供一种更彻底的序列化方式，`writeReplace()`和`readResolve()`。前者是在序列化过程中替换成其他对象，后者是在反序列化中替换掉`readObject()`返回的实例。

注意两者一般不同时使用，因为同时存在时，只会去执行前者，而且不会调用自定义的`writeObject()`和`readobject()`。而后者执行之前回去执行`writeObject()`和`readobject()`，然后替换掉`readObject()`返回的实例，并且抛起它。

在CustomSerializable.java中添加下列代码：



```csharp
    private Object writeReplace() throws ObjectStreamException {
        System.out.println("custom writeReplace method execute!");
        return "No Permission to access the account'password!";
    }
```

执行序列化读取：

![img](E:\Development\Typora\images\webp-16622201738062.webp)

writeReplace method.png

在CustomSerializable.java中添加下列代码：



```csharp
    private Object readResolve() throws ObjectStreamException {
        System.out.println("custom readResolve method execute!");
        return "The account's password is a private!";
    }
```

执行序列化读取：

![img](E:\Development\Typora\images\webp-16622201738063.webp)

readResolve Method.png

注意，`writeReplace()`和`readResolve()`可以被任何权限修饰符修饰，且子类访问这两个方法遵循Java的权限访问规则。这样做的目的子类可以使用父类中已有的序列化操作，也可以覆写它们。

**一般`readResolve()`用于单例模式**

一般类实现单例模式的用途是保证该类的实例只有一个。但是对象序列化后，通过反序列化得到的对象是重构的，也就是在堆内存中重新创建的。所以无法保障唯一实例。



```java
public class SingletonSer implements Serializable {
    private static SingletonSer instance;
    private String name;
    private int age;
    private SingletonSer(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public static SingletonSer getInstance() {
        if(instance == null) {
            instance = new SingletonSer("FoolishDev", 25);
        }
        return instance;
    }

    private Object readResolve() throws ObjectStreamException {
        return getInstance();
    }
}
```

执行序列化读取，在代码中对序列化前后实例引用进行比较(==)，得到



```bash
they are same? true
```

#### 总结

Serializable反序列化时，不会去调用类的构造器，除非类的父类不可序列化，会去调用子类可访问的无参构造器来初始化父类字段。如果父类和子类都可序列化，且各自自定义了序列化和反序列化方法，在整个序列化过程中父类和子类自定义的操作互不影响，各自执行。

## Externalizable

### 官方文档理解

Externalizable实例类可以实现序列化，而且承担了自身内容存储和恢复的责任。Externalizable子类需要实现两个方法，`writeExternal()`和`readExternal()`，通过这两个方法可以完全控制它的对象和它父类的流的内容和格式。这两个方法必须明确的和父类协调，处理它的状态，同时替代了自定义的`writeObject()`和`readObject()`。

类的序列化可以使用Serializable和Externalizable接口。对象的持久化可以使用这两个接口。任何一个对象要存储，首先回去判断有无实现Externalizable接口，如果有那么执行`writeExternal()`，如果没有但是实现了Serializable接口，那么使用ObjectOutputStream。

> 一旦类实现了Externalizable就会替代了Serializable机制。

当一个Externalizable对象重建时，使用public no-arg 构造器进行创建实例，然后调用`readExternal()`。Serializable对象创建是从流中读取信息。Externalizable类一样可以使用`writeReplace()`和`readResolve()`替换对象。

### 实践

#### Externalizable父类构造器必须是公共的？



```java
    public class Supertype implements Externalizable{
    private String className;
    private Date date;

    Supertype() {
        System.out.println("super no arg constructor executed!");
        className = "Super";
        date = new Date();
    }

    public void setClassName(String name) {
        this.className = name;
    }

    @Override
    public String toString() {
        return "Class name is " + className + ". The date is " + date.toString();
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(className);
        out.writeObject(date);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        className = (String) in.readObject();
        date = (Date) in.readObject();
    }
}


    public class Subtype extends Supertype{
    private String nickName;
    public Subtype() {
        System.out.println("sub no arg constructor executed!");
        nickName = "Sub";
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        super.writeExternal(out);
        out.writeObject(nickName);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        super.readExternal(in);
        nickName = (String) in.readObject();
    }

    public String getNickName() {
        return nickName;
    }

    public void setNickName(String name) {
        this.nickName = name;
    }

    @Override
    public String toString() {
        return super.toString() + ". The nick name is " + nickName;
    }
}
```

执行序列化操作后，输出：



```kotlin
super no arg constructor executed!
sub no arg constructor executed!
super no arg constructor executed!
sub no arg constructor executed!
Class name is Supertype. The date is Sun Feb 26 16:10:45 CST 2017. The nick name is Subtype
```

发现父类的构造器可以被子类访问即可，并没有强制规定为public权限。如果不可被子类访问，创建子类对象时会抛出异常java.lang.IllegalAccessError。如果子类的构造器不是public修饰，反序列时会抛出异常java.io.InvalidClassException。

而且还需要注意，即使在反序列化时调用了Externalizable类的无参构造器去初始化了字段，最后依旧会使用序列化流中的信息赋值给同名字段。

最后代码中子类并没有覆写父类中的`writeExternal()`和`readExternal()`，而是继承了。这样保证了父类字段信息不丢失。如果子类覆写了这两个方法，那么父类字段会使用构造器去初始化(实际上就是调用了子类的公共无参构造器，然后执行内部引用的父类构造器去初始化)。

> 这样联想到Serializable文档中提到`writeObject()`和`readObject()`无需考虑字段属于父类还是子类。因为它们权限是private，所以是各自执行字段的初始化，互不影响。

### 不可序列化父类的子类实现可序列化

如何去初始化父类中的私有字段，以及子类可访问的字段？



```java
    public class NoExternalSuper {
    private String className;
    private Date date;

    protected NoExternalSuper() {
        System.out.println("no-externalizable super no arg constructor executed!");
        className = "Super";
        date = new Date();
    }

    public void setClassName(String name) {
        this.className = name;
    }

    @Override
    public String toString() {
        return "Class name is " + className + ",date is " + date.toString();
    }
}

    public class NoExternalSub
    extends NoExternalSuper implements Externalizable{
    private String nickName;
    public NoExternalSub() {
        System.out.println("no-externalizable sub no arg constructor executed!");
        nickName = "Sub";
    }

    @Override
    public void writeExternal(ObjectOutput out)
        throws IOException {
        out.writeObject(nickName);
    }

    @Override
    public void readExternal(ObjectInput in)
        throws IOException, ClassNotFoundException {
        nickName = (String) in.readObject();
    }

    @Override
    public String toString() {
        return super.toString() + ".The nick name is " + nickName;
    }
}
```

执行序列化操作后输出结果：



```kotlin
no-externalizable super no arg constructor executed!
no-externalizable sub no arg constructor executed!
no-externalizable super no arg constructor executed!
no-externalizable sub no arg constructor executed!
Class name is Super,date is Sun Feb 26 16:36:34 CST 2017.The nick name is Sub
```

可以看出，父类使用自身构造器初始化自身字段(就是子类公共无参构造器引用的父类构造器)。而子类可访问的字段当然可以在子类中做出相应序列化和反序列化操作。

#### 总结

一个实现Externalizable接口的序列化类，必须有一个public-no-arg constructor，且会完全替代Serializable机制，让开发者完全控制和自定义序列化和反序列化。所以需要注意协调处理父类的属性状态。它虽然使用起来复杂，但是性能比Serializable好。它实现持久化的方法是通过ObjectOutput和ObjectInput。