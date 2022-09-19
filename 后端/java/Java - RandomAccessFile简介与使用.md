# Java - RandomAccessFile简介与使用

# 一、[api](https://so.csdn.net/so/search?q=api&spm=1001.2101.3001.7020)的研究

## 曾经的我们如何处理文本

以前我们要处理一个文件会怎么做？是不是如下图所示：

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNjE1MDQ5,size_16,color_FFFFFF,t_70.png)

上面是读取文件的方式，写文件也一样，我们需要使用输出流，如下图所示：

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNjE1MDQ5,size_16,color_FFFFFF,t_70-16632394620251.png)

ok 看到这里想必大家都会发现，我对一个文件的读写操作需要new两个类，分别是`读流`和`写流`，并且他们的方法并不多

## RandomAccessFile类帮我们处理文本

### 构造器：

```java
private RandomAccessFile(File file, String mode, boolean openAndDelete)
    throws FileNotFoundException
{
    String name = (file != null ? file.getPath() : null);
    int imode = -1;
    if (mode.equals("r"))
        imode = O_RDONLY;
    else if (mode.startsWith("rw")) {
        imode = O_RDWR;
        rw = true;
        if (mode.length() > 2) {
            if (mode.equals("rws"))
                imode |= O_SYNC;
            else if (mode.equals("rwd"))
                imode |= O_DSYNC;
            else
                imode = -1;
        }
    }
    if (openAndDelete)
        imode |= O_TEMPORARY;
    if (imode < 0)
        throw new IllegalArgumentException("Illegal mode \"" + mode
                                           + "\" must be one of "
                                           + "\"r\", \"rw\", \"rws\","
                                           + " or \"rwd\"");
    @SuppressWarnings("removal")
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkRead(name);
        if (rw) {
            security.checkWrite(name);
        }
    }
    if (name == null) {
        throw new NullPointerException();
    }
    if (file.isInvalid()) {
        throw new FileNotFoundException("Invalid file path");
    }
    fd = new FileDescriptor();
    fd.attach(this);
    path = name;
    open(name, imode);
    FileCleanable.register(fd);   // open sets the fd, register the cleanup
}
```

可以发现这里定义了四种模式：R RW RWD RWS

| r    | 以只读的方式打开文本，也就意味着不能用write来操作文件 |
| ---- | ----------------------------------------------------- |
| rw   | 读操作和写操作都是允许的                              |
| rws  | 每当进行写操作，同步的刷新到磁盘，刷新内容和元数据    |
| rwd  | 每当进行写操作，同步的刷新到磁盘，刷新内容            |

下面让我们看看RandomAccessFile为我们提供的方法：

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNjE1MDQ5,size_16,color_FFFFFF,t_70-166324004198329.png)

可以看出：

这个类集中了丰富的读写方法

下面我举几个比较典型的方法来说明一下这个类的强大之处

---

### 1、seek()：

指定文件的光标位置，通俗点说就是指定你的光标位置

然后下次读文件数据的时候从该位置读取。

------

### 2、getFilePointer()：

我们注意到这是一个long类型的返回值，

**字面意思就是返回当前的文件光标位置。**

这样方便我们后面读取插入。

---

### 3、length：

毫无疑问的方法，文件的长度，返回`long`类型。

注意它并不会受光标的影响。只会反应客观的文本长度。

---

### 4、read()

`read(byte[] b)、read(byte[] b,int off,int len)`

这些方法跟`readstream`中的方法一样，

例如最后一个：定义缓冲数组，从数组的off偏移量位置开始写，

读取转换为数组数据达到len个字节。

总之这是一个读文件内容的标准操作api。

---

### 5、readXXX() 

`readDouble()  readFloat() readBoolean() readInt()`

`readLong() readShort() readByte() readChar()`

这些方法都是去read每一个字符，个人感觉就是返回他们的ASCII码

当然如果专家们有异议可以指出，我测试的时候至少是这么感觉得。

大家也可以自己试一下。

比如`readLong`就是要求你的文本内容必须有八个字符，不然会报错。

伴随着也就是

`writeDouble() writeFloat() writeBoolean() writeInt()`

`writeLong() writeShort() writeByte() writeChar()`

----

### 6、readFully(byte[] b):

这个方法的作用就是将文本中的内容填满这个缓冲区b。

如果缓冲b不能被填满，那么读取流的过程将被阻塞，

如果发现是流的结尾，那么会抛出异常。

这个过程就比较像“凑齐一车人在发车，不然不走”。

----

### 7、getChannel：

它返回的就是`nio`通信中的`file`的唯一`channel`

---

### 8、skipBytes(int n)：

跳过n字节的位置，相对于当前的point。





可以看出`RandomAccessFile`实现了大部分文件输入输出流的方法，但是底层实现中他实现的是`DataInput`和`DataOutput`接口，并非是`FileInputStream`和`FileOutputStream`。RandomAccessFile使用很多native方法实现了对文件的操作，并且很多native方法跟inputstream都有重叠，比如`read()`方法。我想这么做可能是为了让这个DataInput接口的职责更明确吧。

| ![img](E:\Development\Typora\images\20190314235401724.png) | ![img](E:\Development\Typora\images\20190314235421569.png) |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
|                                                            |                                                            |

 

从上述对比中可以看出，datainput更注视强化对数据的各种既定操作。并没有出现类似inputstream的方法，也许这就是我们常说的单一职责原则吧。

# 二、实际操作一下吧

![img](E:\Development\Typora\images\20190314235923230.png)

定义一个文件

编写一个方便我们使用的工厂类：

## 工厂类

```java
public class RAFTestFactory 
	private static final String url = "D:\\EclipseWorkspace\\text\\test.txt";
	private static final String [] model = {"r","rw","rws","rwd"};
	
	public static RandomAccessFile getRAFWithModelR() throws FileNotFoundException {
		RandomAccessFile raf = new RandomAccessFile(new File(url), model[0]);
		return raf;
	}
	
	public static RandomAccessFile getRAFWithModelRW() throws FileNotFoundException {
		RandomAccessFile raf = new RandomAccessFile(new File(url), model[1]);
		return raf;
	}
	
	public static RandomAccessFile getRAFWithModelRWS() throws FileNotFoundException {
		RandomAccessFile raf = new RandomAccessFile(new File(url), model[2]);
		return raf;
	}
	
	public static RandomAccessFile getRAFWithModelRWD() throws FileNotFoundException {
		RandomAccessFile raf = new RandomAccessFile(new File(url), model[3]);
		return raf;
	}
}
```

## **编写测试类1：**

```java
public class RAFTestMain {
	public static void main(String[] args) throws IOException {
		
		RandomAccessFile raf = RAFTestFactory.getRAFWithModelR();
		System.out.println("raf.length()->获取文本内容长度:"+raf.length());
		System.out.println("raf.getFilePointer()->获取文本头指针:"+raf.getFilePointer());
		raf.seek(4);
		System.out.println("raf.getFilePointer()->第二次获取文本头指针:"+raf.getFilePointer());
	}
}
```

看一下结果，体验一下seek的含义

```
raf.length()->获取文本内容长度:9
raf.getFilePointer()->获取文本头指针:0
raf.getFilePointer()->第二次获取文本头指针:4
```

 

## **编写测试类2：**

```java
public class RAFTestMain {
	public static void main(String[] args) throws IOException {
		
		RandomAccessFile raf = RAFTestFactory.getRAFWithModelR();
		raf.write(1);
	}
}
```

![img](E:\Development\Typora\images\20190315000453920.png)

为什么出现这个结果，就是因为我们使用了 `R model`，也就是只读模式，这是写入会报错。

**编写测试类3：**

```java
public class RAFTestMain {
	public static void main(String[] args) throws IOException {
		RandomAccessFile raf = RAFTestFactory.getRAFWithModelRW();
		String word = "ljh";
		raf.write(word.getBytes());
	}
}
```

![img](E:\Development\Typora\images\20190315000735539.png)

可以看到我们写的位置就是当前光标的位置，这个时候让我们结合seek和write试验一下吧。

**编写测试类4：**

```java
public class RAFTestMain {
	public static void main(String[] args) throws IOException {
		RandomAccessFile raf = RAFTestFactory.getRAFWithModelRW();
		raf.seek(4);
		String word = "ljh";
		raf.write(word.getBytes());
	}
}
```

![img](E:\Development\Typora\images\20190315000935631.png)

看，此时我们很方便的实现了插入操作。这是其他类无法实现的功能。也是这个类的强大之处。

 

**编写测试类5：**

最后我们看看

`readDouble() readFloat() readBoolean() readInt() readLong() readShort() readByte() readChar()`

`writeDouble() writeFloat() writeBoolean() writeInt() writeLong() writeShort() writeByte() writeChar()`

他们都怎么用的

```java
public class RAFTestMain {
	public static void main(String[] args) throws IOException {
		RandomAccessFile raf = RAFTestFactory.getRAFWithModelRW();
		raf.writeByte(3);
		raf.writeChar('a');
		raf.writeShort(5);
		raf.writeInt(6);
		raf.writeLong(792929347162343l);
		raf.writeFloat(8.5f);
		raf.writeDouble(9.44d);
		raf.writeBoolean(true);
	}
}
```

![img](E:\Development\Typora\images\20190315001738152.png)

真是仙的不行，反正我是看不出个所以然。

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxNjE1MDQ5,size_16,color_FFFFFF,t_70-16632394620264.png)

 

# 三、对该工具类的价值分析

1、大型文本日志类文件的快速定位获取数据：

得益于`seek`的巧妙设计，我认为我们可以从超大的文本中快速定位我们的游标，例如每次存日志的时候，我们可以建立一个索引缓存，索引是日志的起始日期，value是文本的poiniter 也就是光标，这样我们可以快速定位某一个时间段的文本内容

2、并发读写

emmm也是得益于seek的设计，我认为多线程可以轮流操作seek控制光标的位置，从未达到不同线程的并发写操作。

3、更方便的获取二进制文件

通过自带的读写转码（readDouble、writeLong等），我认为可以快速的完成字节码到字符的转换功能，对使用者来说比较友好。