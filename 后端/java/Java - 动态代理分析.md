

Java - 动态代理分析

Java动态代理机制的出现，使得Java开发人员不用手工编写代理类，只要简单地制定一组接口及委托[类对象](https://so.csdn.net/so/search?q=类对象&spm=1001.2101.3001.7020)，便能动态地获得代理类。代理类会负责将所有的方法调用分配到委托对象上反射执行，配置执行过程中，开发人员还可以进行修改

## 代理[设计模式](https://so.csdn.net/so/search?q=设计模式&spm=1001.2101.3001.7020)

**代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息、过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。**
\1. 为了保持行为的一致性，代理类和委托类通常会实现相同的接口
\2. 引入代理能够控制对委托对象的直接访问，可以很好的隐藏和保护委托对象，也更加具有灵活性

## 相关的类和接口

要了解 Java 动态代理的机制，首先需要了解以下相关的类或接口：
\1. java.lang.reflect.Proxy：这是 Java 动态代理机制的主类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象
\2. java.lang.reflect.InvocationHandler：这是调用处理器接口，它自定义了一个invoke方法，用于几种处理在动态代理类对象上的方法调用。通常在该方法中实现对委托类的代理访问。
\3. java.lang.ClassLoader：Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个.class 文件中。

## 代理机制及其特点

首先让我们来了解一下如何使用 Java 动态代理。具体有如下四步骤：
\1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
\2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
\3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
\4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。

```
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
// 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用
InvocationHandler handler = new InvocationHandlerImpl(..); 

// 通过 Proxy 为包括 Interface 接口在内的一组接口动态创建代理类的类对象
Class clazz = Proxy.getProxyClass(classLoader, new Class[] { Interface.class, ... }); 

// 通过反射从生成的类对象获得构造函数对象
Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class }); 

// 通过构造函数对象创建动态代理类实例
Interface Proxy = (Interface)constructor.newInstance(new Object[] { handler });123456789101112
```

实际使用过程更加简单，因为 Proxy 的静态方法 newProxyInstance 已经为我们封装了步骤 2 到步骤 4 的过程，所以简化后的过程如下

```
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
InvocationHandler handler = new InvocationHandlerImpl(..); 

// 通过 Proxy 直接创建动态代理类实例
Interface proxy = (Interface)Proxy.newProxyInstance( classLoader, 
     new Class[] { Interface.class }, 
     handler );1234567
```

### 特点

动态生成的代理类本身的一些特点
\1. 包：如果所代理的接口都是 public 的，那么它将被定义在顶层包（即包路径为空），如果所代理的接口中有非 public 的接口（因为接口不能被定义为 protect或private，所以除 public之外就是默认的package访问级别，那么它将被定义在该接口所在包，这样设计的目的是为了最大程度的保证动态代理类不会因为包管理的问题而无法被成功定义并访问；
\2. 类修饰符：该代理类具有 final 和 public 修饰符，意味着它可以被所有的类访问，但是不能被再度继承；
\3. 类名：格式是“$ProxyN”，其中 N 是一个逐一递增的阿拉伯数字，代表 Proxy 类第 N 次生成的动态代理类，值得注意的一点是，并不是每次调用 Proxy 的静态方法创建动态代理类都会使得 N 值增加，原因是如果对同一组接口（包括接口排列的顺序相同）试图重复创建动态代理类，它会很聪明地返回先前已经创建好的代理类的类对象，而不会再尝试去创建一个全新的代理类，这样可以节省不必要的代码重复生成，提高了代理类的创建效率。
\4. 类继承关系：Proxy 类是它的父类，这个规则适用于所有由 Proxy 创建的动态代理类。而且该类还实现了其所代理的一组接口;
![这里写图片描述](E:\Development\Typora\images\SouthEast.png)
代理类实例的一些特点
\1. 每个实例都会关联一个InvocationHandler(调用处理器对象)，在代理类实例上调用其代理接口中声明的方法时，最终都会由InvocationHandler的invoke方法执行；
\2. java.lang.Object中有三个方法也同样会被分派到调用处理器的 invoke 方法执行，它们是 hashCode，equals 和 toString；

被代理接口的一组特点
\1. 要注意不能有重复的接口
\2. 接口对于类装载器必须可见，否则类装载器将无法链接它们
\3. 被代理的所有非 public 的接口必须在同一个包中，接口的数目不能超过65535

## 美中不足

Proxy只能对interface进行代理，无法实现对class的动态代理。观察动态生成的代理继承关系图可知原因，他们已经有一个固定的父类叫做Proxy，Java语法限定其不能再继承其他的父类

## 代码示例

最后以一个简单的动态代理例子结束

```
public class DynamicProxy {
    interface IHello{
        void sayHello();
    }

    static class Hello implements IHello{
        public void sayHello() {
            System.out.println("hello world");
        }
    }

    static class DynamicProxyTest implements InvocationHandler{
        Object originalObj;
        Object bind(Object originalObj){
            this.originalObj = originalObj;
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(),
                    originalObj.getClass().getInterfaces(),this);
        }
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("Welcome");
            return method.invoke(originalObj,args);
        }
    }

    public static void main(String[] args){
        //设置这个值，在程序运行完成后，可以生成代理类
        System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true");
        IHello hello = (IHello) new DynamicProxyTest().bind(new Hello());
        hello.sayHello();
    }
}12345678910111213141516171819202122232425262728293031
```

程序输出为：

```
Welcome
hello world
```