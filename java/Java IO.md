# Java IO

## 一.File

### 1.File的三个构造方法

> 1.`new File(String filePath)`: 根据`路径`构造一个文件对象；
> 2.`new File(String parentPath,String childPath)` : 根据`上级路径+子路径`构造一个文件对象；
> 3.`new File(File parentFile,String childPath)` : 根据`上级文件+子路径`否早一个文件对象；

> `File.createNewFile()` : **真正在磁盘上创建文件的方法！**
> **【注】：当调用此方法时，如果对应的文件已经存在了，则创建失败，返回值为false!**

**示例：**

```java
import java.io.File;
import java.io.IOException;

public class ApplicationFileCreate {

    public static void main(String[] args) {
    	// 1.父级路径
  		String parentPath = "D:\\EDailyRoutine"; 
  		// 2.子路径，
        String fileName = "java-io-test\\002File_2.txt";
        File file = new File(parentPath, fileName);
        try {
        	// 执行创建动作
            boolean newFile = file.createNewFile(); 
            if (newFile){
                System.out.println("create02 文件创建成功");
            }else{
                System.out.println("create02 文件创建失败");
            }
        } catch (IOException e) {
            System.out.println("create02 文件创建异常");
            e.printStackTrace();
        }
    }

}
```

### 2.File中的常用方法

#### <font color='orange'>访问文件名</font>

> `getName()` : 获取文件名
> `getPath()` ： 获取文件路径
> `getAbsoluteFile()` ：获取绝对路径下的文件，返回值是文件
> `getAbsolutePath()` ：获取文件的绝对路径
> `getParent()` ： 获取文件的上级目录
> `renameTo(File newName)` ： 文件重命名

```java
import java.io.File;
import java.io.IOException;

public class ApplicationFileMehods {
    public static void main(String[] args) throws IOException {

        File file = new File("D:\\EDailyRoutine\\java-io-test\\001File.txt");
        System.out.println("文件名 ： " + file.getName());
        System.out.println("文件路径 ： " + file.getPath());
        System.out.println("文件绝对文件 ： " + file.getAbsoluteFile());
        System.out.println("文件绝对路径 ： " + file.getAbsolutePath());
        System.out.println("文件的父级文件 ： " + file.getParent());
        File file8 = new File("D:\\EDailyRoutine\\java-io-test\\001File_bak.txt");
        boolean b = file.renameTo(file8);
        System.out.println(b ? "文件改名成功" : "文件改名失败");
        System.out.println("==================================");

    }
}
```

```java
文件名 ： 001File.txt
文件路径 ： D:\EDailyRoutine\java-io-test\001File.txt
文件绝对文件 ： D:\EDailyRoutine\java-io-test\001File.txt
文件绝对路径 ： D:\EDailyRoutine\java-io-test\001File.txt
文件的父级文件 ： D:\EDailyRoutine\java-io-test
文件改名成功
==================================
```



#### <font color='orange'>检测文件状态</font>

> `exists()` ：检测文件是否存在
> `canWrite()` ： 检测文件是否可写
> `canRead()` ： 检测文件是否可 读
> `isFile()` ： 判断是不是一个文件
> `isDirectory()` ： 判断是不是一个目录



#### <font color='orange'>创建、删除文件</font>

> `createNewFile()` : 创建文件,必须在已经存在的一个目录下才可以创建成功
> `delete()` ： 删除文件or删除一个空的目录



#### <font color='orange'>目录操作</font>

> `mkdir` : 在已有目录下创建单级目录
> `mkdirs` : 在已有目录下创建多级目录
> `list` : 返回String[] ,所有的文件和目录的名称
> `listFiles` ： 返回File[],所有的文件和目录的绝对路径



## 二.流

### 1.流的分类

- 按照**操作数据单位**不同分为 ： `字节流`（二进制文件）、`字符流`（文本文件）
- 按照**数据流的流向**不同分为 ：`输入流`、`输出流`（以java程序为参考点）
  `【从文件到程序中 ： 输入流】`
  `【从程序到文件中 ： 输出流】`
- 按照**流的角色**的不同分为 ： `节点流`、`处理流/包装流`

> **下面是四个大的抽象基类：** （非常关键）

- **字节输入流 ： `InputStream`**
- **字节输出流 ： `OutputStream`**
- **字符输入流 ：`Reader`**
- **字符输出流 ：`Writer`**



### 2.节点流

#### 2.1字节输入流 FileInputStream

##### 1.继承关系图

> 首先了解一下 FileInputStream 的继承关系图，熟悉他的位置:
> **`FileInputStream` 是 `InputStream` 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_10,color_FFFFFF,t_70,g_se,x_16.png)



##### 2.API介绍

> **字节输入流**，作用就是以`字节`的为单位`读取文件内容到程序中`。

**1.字节输入流读取文件 `read()` 方法**

    /**
    * 方法作用 ： 按字节读入文件,每次读取一个字节的数据
    * 方法返回值 ： 返回读入的字节数据，如果是-1，代表读取完毕
    * 注 ： 1.因为是按照一个字节一个字节的方式进行读取，所以读取中文会出现乱码的情况
    *       2.流在使用完毕后要关闭资源，调用close()方法
    */

> 因为一个中文字符，在UTF-8编码下占3个字节。
> 所以，当按照一个字节一个字节的读取方式时，中文字符被拆开了，就乱码了。

**使用代码**

```java
import java.io.*;
public class ApplicationFileInputStream {
    public static void main(String[] args) {
         //1.创建 File 对象
        String filePath = "D:\\EDailyRoutine\\java-io-test\\004FileInputStreamRead.txt";
        File file = new File(filePath);
        //2.创建 FileInputStream 对象
        FileInputStream inputStream = null;
        //3.创建一个变量，接收read()方法读取到的内容
        int readData = 0;
        try {
            inputStream = new FileInputStream(file); // 此处会抛出 FileNotFoundException
            readData = inputStream.read(); // 此处会抛出 IOException
            //4.循环读取文件的内容
            while (readData != -1){
                System.out.print((char)readData); // int --> char 转换一下
                readData = inputStream.read(); // 再次读取下一个字节的数据
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 5.使用完毕后，都要关闭流
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

```java
hello world
hello FileInputStream
hello method read
å¤§å®¶å¥½ æµ‹è¯•read()æ–¹æ³•
```

**2.`read(byte[] b)` 方法**

```
/**
 * 方法作用 ： 按字节读入文件，每次将读取的数据放入到byte数组b中
 * 方法返回值 ： 返回读取到的数据的长度，如果是-1，代表读取完毕
 * 注 ：1.因为是一组一组的读取数据，所以可能会出现中文乱码的问题
 *      2.流在使用完毕后要关闭资源，调用close()方法
 */
```

```java
package com.northcastle.inputStream_;

import java.io.*;

public class ApplicationFileInputStream {
    public static void main(String[] args) {
                //1.创建 File 对象
        String filePath = "D:\\EDailyRoutine\\java-io-test\\004FileInputStreamRead.txt";
        File file = new File(filePath);
        //2.创建 FileInputStream 对象
        FileInputStream inputStream = null;
        //3.创建一个变量，接收read(byte[] b)方法读取到的内容
        byte[] buff = new byte[8]; // 每次读取8个字节的数据
        int len = 0; // 接收每次实际读取到的数据长度
        try {
            inputStream = new FileInputStream(file); // 此处会抛出 FileNotFoundException
            len = inputStream.read(buff); // 首次读取数据，此处会抛出 IOException
            //4.循环读取数据
            while (len != -1){
                System.out.print(new String(buff,0,len)); // 字节数组转换成字符串输出
                len = inputStream.read(buff); // 再次进行读取
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 5.使用完毕后，都要关闭流
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

> 运行结果：`存在中文乱码问题`
> 因为，此时读取是按照一组一组的字节数组进行读取的，
> 所以汉字所占的字节非常有可能被拆开，导致读取乱码。

```
hello world
hello FileInputStream
hello method read
���家好 测试read()方法
```



#### 2.2字节输出流 FileOutputStream

##### 1.继承关系图

> 首先了解一下 FileInputStream 的继承关系图，熟悉他的位置:
> **FileOutputStream 是 OutputStream 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_13,color_FFFFFF,t_70,g_se,x_16.png)

##### 2.API介绍

> `字节输出流`，就是将内容`从程序写入到文件中`。
> 它所涉及到的主要就是`写出`的方法。

1.`FileOutputStream` :   

​	1.	如果文件存在，可以直接写入内容 

​	2.	如果文件不存在，则会**先创建文件**再写入内容  

【注】FileOutputStream的构造方法中，如果带有参数 true,则证明是追加的方式写入文件。流在使用完毕后，一定要进行资源的关闭，就是要调用`close()`方法。

2.方法介绍 ：

1. `write(int b)` : 向文件中写入一个字节

2. `write(byte[] b)` : 向文件中写入一个字节数组

3. `write(byte[] b,int offset,int len)` : 向文件中写入字节数组的一部分，从offset开始，到len长度的位置   

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class ApplicationFileOutputStream {
    public static void main(String[] args) {

        //1.创建File对象
        String filePath = "D:\\EDailyRoutine\\java-io-test\\004FileOutputStreamWrite.txt";
        File file = new File(filePath);
        //2.声明FileOutputStream输出流对象
        FileOutputStream fileOutputStream = null;
        try {
            // 参数true : 表示追加内容到文件中去
            // 如果不写，默认是false,表示每次的写入操作都会覆盖原来的内容
            fileOutputStream = new FileOutputStream(file, true);

            //3.1 写入一个字节
            fileOutputStream.write('a');
            fileOutputStream.write('\n');
            //3.2 写入一个字节数组
            String str = "Hello 你好";
            fileOutputStream.write(str.getBytes());
            fileOutputStream.write('\n');
            //3.3 写入一个字节数据的部分数据
            String str2 = "Hello 你好";
            fileOutputStream.write(str.getBytes(),1,8); // “ello 你” 就是这个字符串
            fileOutputStream.write('\n');
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fileOutputStream != null){
                    fileOutputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```



#### 2.3文件复制（使用字节流实现）

##### 1.需求描述

> 当前有一张图片，将此图片`复制一份`。

##### 2.思路分析

> 1.图片是一种[二进制](https://so.csdn.net/so/search?q=二进制&spm=1001.2101.3001.7020)的文件，所以使用字节流来进行读写操作；
> 2.将原文件读入到程序中，然后再将读取的内容写入到一个新的文件中即可；
> 3.字节输出流，要使用追加的方式，这样才能写出一个完整的文件；
> 4.输入流和输出流使用完成后都需要进行关闭。

##### 3.代码实现

```java
import java.io.*;
public class ApplicationFileCopy {

    public static void main(String[] args) {
      //1.字节流复制文件
        fileCopy01();
    }

    /**
     * 1.使用字节输入流和字节输出流完成文件的复制
     * 实现思路 ： 一边读，一边写
     */
    public static void fileCopy01(){
        //1.两个文件 ：源文件和目标文件
        File fileSource = new File("D:\\EDailyRoutine\\java-io-test\\aaa.JPG");
        File fileCopy = new File("D:\\EDailyRoutine\\java-io-test\\aaa_copy02.jpg");
        //2.声明输入流和输出流对象
        FileInputStream inputStream = null;
        FileOutputStream outputStream = null;
        //3.读取文件的缓存数组和长度记录
        byte[] buff = new byte[1024];
        int len = 0;

        try {
            //4.创建输入流和输出流对象
            inputStream = new FileInputStream(fileSource);
            outputStream = new FileOutputStream(fileCopy,true); // 追加写入文件

            //5.开始进行读写操作
            System.out.println("==> 复制文件 begin : "+System.currentTimeMillis()+"<==");
            while ((len = inputStream.read(buff)) != -1){ // 读
                outputStream.write(buff,0,len); // 写
            }
            System.out.println("==> 复制文件 end : "+System.currentTimeMillis()+"<==");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally{
            // 关闭两个流
            try {
                if (inputStream != null){
                    inputStream.close();
                }
                if (outputStream != null){
                    outputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

![在这里插入图片描述](E:\Development\Typora\images\b00eead352fd423c97c7ae6790a9499e.png)



#### 2.4字符输入流 FileReader

##### 1.继承关系图

> 首先了解一下 FileReader 的继承关系图，熟悉他的位置:
> **`FileReader` 是 `InputStreamReader` 的一个子类。**
> **`InputStreamReader` 是 `Reader` 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_12,color_FFFFFF,t_70,g_se,x_16.png)

##### 2.[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020) 介绍

> **字符输入流**，作用就是以`字符`的为单位`读取文件内容到程序中`。
> 特别适合处理`文本文件`。

**1.`read()` 方法**

作  用 ： 按字符读取文件内容，每次读取一个字符
参  数 ： 无参数
返回值 ： 1.返回读取到的字符；

​					  2.如果读取完毕，返回-1

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

/**
 * author : northcastle
 * createTime:2021/12/13
 * 使用 FileReader 读取文件内容
 */
public class ApplicationFileReader {
    public static void main(String[] args) {
 		//1.创建文件
        String filePath = "D:\\EDailyRoutine\\java-io-test\\reader01.txt";
        File file = new File(filePath);
        //2.声明字符输入流 FileReader
        FileReader fileReader = null;
        //3.创建读取的结果的接收值
        int readChar = 0;
        try {
            // 4.创建字符流，执行读取操作
            fileReader = new FileReader(file);
            readChar = fileReader.read();//读取首个字符
            while (readChar != -1){ // 循环读取
                System.out.print((char) readChar); // 输出这个字符即可
                readChar = fileReader.read(); // 继续读下一个字符
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (fileReader != null){
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
   
}
```



**2.`read(char[] c)` 方法**

作  用 ： 按字符读取文件内容，每次可以读取 char[] 长度个字符

参  数 ： 字符数组，每次读取的内容放到这个数组中暂存

返回值 ： 1.返回读取到的字符数组的实际长度；2.如果是-1，则读取完毕

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

/**
 * author : northcastle
 * createTime:2021/12/13
 * 使用 FileReader 读取文件内容
 */
public class ApplicationFileReader {
    public static void main(String[] args) {
        //1.创建文件
        String filePath = "D:\\EDailyRoutine\\java-io-test\\reader01.txt";
        File file = new File(filePath);
        //2.创建字符输入流 FileReader
        FileReader fileReader = null;
        //3.创建读取暂存数组
        char[] buff = new char[8];
        int len = 0;
        try {
            //4.执行读取操作
            fileReader = new FileReader(file);
            len = fileReader.read(buff); //首次读取
            while (len != -1){ // 根据读取的状态，循环读取
                System.out.print(new String(buff,0,len));
                len = fileReader.read(buff);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (fileReader != null){
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### 2.5字符输出流 FileWriter

##### 1.继承关系图

> 首先了解一下 FileWriter 的继承关系图，熟悉他的位置:
> **`FileWriter` 是 `OutputStreamWriter` 的一个子类。**
> **`OutputStreamWriter` 是 `Writer` 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_19,color_FFFFFF,t_70,g_se,x_16.png)

##### 2.[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)介绍

> `字符输出流`，就是将内容`按字符从程序写入到文件中`。

```
 /**
   * 1.FileWriter :
   *      如果文件存在，可以直接写入内容
   *      如果文件不存在，则会先创建文件再写入内容
   *      【注】FileWriter 的构造方法中，如果带有参数 true,则证明是追加的方式写入文件。
   *           流在使用完毕后，一定要进行资源的关闭，就是要调用close方法。
   * 【如果FileWriter不进行close()或者flush()的话，内容不会写入到文件中去！！！】
   */
    /**
    * 2.写入到文件中的方法：
    *     write(int) : 写入单个字符
    *     write(char) : 写入单个字符
    *     write(char[]) : 写入字符数组
    *     write(char[],off,len) : 写入字符数组的一部分，off:从哪儿开始，len：长度
    *     write(String) : 写入一个字符串
    *     write(String,off,len) : 写入字符串的一部分，off:从哪儿开始，len: 长度
    */
```



```java
package com.northcastle.writer_;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

/**
 * author : northcastle
 * createTime:2021/12/13
 * 使用FileWriter向文件中写入内容
 */
public class ApplicationFileWriter {

    public static void main(String[] args) {
        //1.创建文件
        String filePath = "D:\\EDailyRoutine\\java-io-test\\write01.txt";
        File file = new File(filePath);
        //2.创建字符输出流 FileWriter
        FileWriter fileWriter = null;
        try {
            fileWriter = new FileWriter(file,true); // true 表示向文件中追加
            fileWriter.write(97);// a
            fileWriter.write('\n'); // 写一个换行

            fileWriter.write('大');
            fileWriter.write('\n'); // 写一个换行

            char[] chars = {'大','家','好'};
            fileWriter.write(chars);
            fileWriter.write('\n'); // 写一个换行

            fileWriter.write(chars,1,2); // 家，好
            fileWriter.write('\n'); // 写一个换行

            String str = "我喜欢学习java";
            fileWriter.write(str);
            fileWriter.write('\n'); // 写一个换行

            fileWriter.write(str,2,6);
            fileWriter.write('\n'); // 写一个换行
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (fileWriter != null){
                try {
                    fileWriter.close(); // 真正执行保存的地方
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### 2.6小结

> [字符流](https://so.csdn.net/so/search?q=字符流&spm=1001.2101.3001.7020)和字节流，在使用的思路上有很大的相似之处，特在此作一下小结。
> **本小结主要涉及到 四个类：**
> `字节输入流 ： FileInputStream`
> `字节输出流 ：FileOutputStream`
> `字符输入流 ：FileReader`
> `字符输出流 ：FileWriter`

##### 输入流的比较

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16.png)

##### 输出流的比较

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16604986537633.png)

### 3.处理流

实际上是对节点流的包装，需要先实例化节点流，然后再实例化处理流

#### 1.处理流 BufferedReader

##### 1.继承关系图

> 首先了解一下 `BufferedReader` 的继承关系图，熟悉他的位置:
> **`BufferedReader` 是 `Reader` 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_16,color_FFFFFF,t_70,g_se,x_16.png)

##### 2.[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020) 介绍

> **BufferedReader**，读取`文本文件`的内容。
> 它有一个Reader 类型的属性，可以对Reader类型的对象进行扩展。
> 【此处使用到的就是 `装饰器模式`】

**读取文本文件，本质是对节点流的一层包装，使功能使用更方便**

```java
/**
* BufferedReader 字符处理流 ：
*     作用 ： 读取文本文件，本质是对节点流的一层包装，使功能使用更方便
*                  【构造方法的参数中传入一个 Reader 类型的对象即可】
*     注意 ： 在关闭流时，只需要关闭最外层的处理流即可（源码中就是把内层的节点流给关掉了）。
* 主要方法 ： readLine():
*     作用 ： 读一行的文件
*     返回值 ： String lineTex,读取到的一行的文本内容
*               null : 当读取结束时返回null,
*                      [如果最后一行是空行，则会返回null]
*               [注意] ：返回值中不包含换行符
*/
```

 `readLine()` 方法

```java
package com.northcastle.reader_;

import java.io.*;

public class ApplicationBufferReader {
    public static void main(String[] args) {
        //1.创建一个文件对象
        String filePath = "D:\\EDailyRoutine\\java-io-test\\bufferedReader01.txt";
        File file = new File(filePath);
        //2.声明、创建一个节点流对象
        FileReader fileReader = null;
        //3.声明、创建一个处理流对象
        BufferedReader bufferedReader = null;
        //4.声明一个变量接收读取到的内容
        String contextLine = "";
        //6.关闭处理流
        try {
            fileReader = new FileReader(file); // 实例化节点流 ： FileReader
            bufferedReader = new BufferedReader(fileReader); // 实例化处理流 ： BufferedReader
            contextLine = bufferedReader.readLine(); // 首次读取文本文件的内容
            //5.循环读取文本中的内容，并在控制台打印
            while (contextLine != null){
                //lineNum++;
                //System.out.println(Arrays.toString(contextLine.getBytes()) ); // 打印读取到的内容
                System.out.println(contextLine); // 打印读取到的内容
                contextLine = bufferedReader.readLine(); // 继续读取下一行的内容
            }
            System.out.println("文件读取结束 success!");

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (bufferedReader != null){
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }


    }
}
```

> 运行结果：`可以正常读取到文件中的内容`，
> 【注意】：源文件中 最后有三个空行，但在程序中只有两个空行
> 原因是：**程序读取到最后一行的时候，返回了null（空行），所以没有进行打印**

#### 2.处理流 BufferedWriter

**写入文本文件，本质是对节点流的一层包装，使功能使用更方便**

##### 1.继承关系图

> 首先了解一下 `BufferedWriter` 的继承关系图，熟悉他的位置:
> **`BufferedWriter`是 `Writer` 的一个子类。**

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16604991873326.png)

##### 2.[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020) 介绍

> **BufferedWriter**，一个`字符`输出流。
> 它有一个Writer 类型的属性，可以对Writer 类型的对象进行扩展。
> 【此处使用到的就是 `装饰器模式`】

```java
   /**
    * BufferedWriter 字符处理流 ：
    *     作用 ： 写入文本文件，本质是对节点流的一层包装，使功能使用更方便
    *     注意 ： 在关闭流时，只需要关闭最外层的处理流即可（源码中就是把内层的节点流给关掉了）。
    * 主要方法 ：
    *     write(String content): 写一行的文件（不带换行符）
    *     newLine() : 写入一个换行符，随系统一致的
    *
    */
```

> 文件路径 ： `D:\EDailyRoutine\java-io-test\bufferedWriter01.txt`

**`write()、newLine()` 方法**

```java
package com.northcastle.writer_;

import java.io.*;

public class ApplicationBufferedWriter {

    public static void main(String[] args) {
        //1.准备一个文件对象，数据的写入位置
        String filePath = "D:\\EDailyRoutine\\java-io-test\\bufferedWriter01.txt";
        File file = new File(filePath);
        //2.声明+创建一个节点流对象
        FileWriter fileWriter = null;
        //3.声明+创建一个处理流对象
        BufferedWriter bufferedWriter = null;
        try {
            fileWriter = new FileWriter(file,false);  // 实例化节点流
            bufferedWriter = new BufferedWriter(fileWriter);// 实例化处理流
            //4.向文件中写入内容
            bufferedWriter.write("开始向文本文件中写入内容");
            bufferedWriter.newLine(); // 换行
            bufferedWriter.write("内容是追加的");
            bufferedWriter.newLine();
            bufferedWriter.newLine();
            bufferedWriter.newLine();
            bufferedWriter.write("----------------------");
            bufferedWriter.newLine();
            bufferedWriter.newLine();
            bufferedWriter.newLine();
            System.out.println("写入完成！");
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //5.关闭处理流
            if (bufferedWriter != null){
                try {
                    bufferedWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

#### 3.文本文件复制（使用字符处理流实现）

##### 1.需求描述

> 当前有一个文本文件`txt`，将此文件`复制一份`。

##### 2.思路分析

> 1.txt是一种文本文件，所以使用[字符流](https://so.csdn.net/so/search?q=字符流&spm=1001.2101.3001.7020)进行读写操作；
> 2.将原文件读入到程序中，然后再将读取的内容写入到一个新的文件中即可；
> 3.字符输出流，要使用追加的方式，这样才能写出一个完整的文件；
> 4.使用处理流的时候，只需要关闭最外层的流即可，内部的流可以自动关闭。

##### 3.代码实现

```java
package com.northcastle.fileOperation;

import java.io.*;

public class ApplicationFileCopyBufferReaderWriter {
    public static void main(String[] args) {
        //1.准备两个文件，一个读取，一个写入
        String filePathReader = "D:\\EDailyRoutine\\java-io-test\\bufferedReader01.txt";
        String filePathWriter = "D:\\EDailyRoutine\\java-io-test\\bufferedReader01_copy.txt";
        //2.声明读取和写出的处理流对象
        BufferedReader bufferedReader = null;
        BufferedWriter bufferedWriter = null;
        //3.声明读取的对象
        String contentLint = null;
        try {
            bufferedReader = new BufferedReader(new FileReader(filePathReader));
            bufferedWriter = new BufferedWriter(new FileWriter(filePathWriter,true));
            //4.循环读取与写入
            while ((contentLint = bufferedReader.readLine()) != null){
                bufferedWriter.write(contentLint);
                bufferedWriter.newLine();
            }
            System.out.println("copy 完成");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //5.关闭流
            try {
                if (bufferedReader != null){
                    bufferedReader.close();
                }
                if (bufferedWriter != null){
                    bufferedWriter.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
}
```



### 4.转换流

#### 1.字节-字符转换流 - InputStreamReader+OutputStreamWriter

##### 1.由[乱码](https://so.csdn.net/so/search?q=乱码&spm=1001.2101.3001.7020)引出的问题

> 1.`字符流`在处理文本文件时比较方便，且对文本文件的读写效率要高；
> 2.在读取文件的时候，如果文件的格式和读入流的格式不一致，则会导致读入的内容`乱码`。

> 下面，我们通过一个例子来观察一下这个问题：
> （以下操作在windows上进行）

1.1 另存为文件的格式为 GBK

> 文件 ： `D:\EDailyRoutine\java-io-test\\字符读取乱码演示.txt`
> `另存为`一下，选择编码格式为 `GB2312`.

> **文件内容中包含中文,内容如下：**
> hello FileInputReader
> 你好，这里演示FileInputReader读取文件乱码的操作。

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16605000705168.png)

1.2 使用BufferedReader读取中文乱码

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
/*
* 文件编码格式为 GB2312;输入流编码为 UTF8
* 二者的编码不一致，导致文件乱码。
*/
public class ApplicationBufferedReaderLuanMa {
    public static void main(String[] args) throws IOException {
        String filePath = "D:\\EDailyRoutine\\java-io-test\\字符读取乱码演示.txt";
        BufferedReader bufferedReader = new BufferedReader(new FileReader(filePath));
        String context = null;
        while ((context = bufferedReader.readLine()) != null){
            System.out.println(context);
        }
    }
}
```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050012018610.png)

##### 2.什么是字节-字符转换流

> `转换流` 是将 `字节流对象` 装换成 `字符流对象`，主要是在转换的过程中可以`指定编码格式`。
> `转换流`的应用对象主要有两个 `InputStreamReader`和`OutputStreamWriter`。
> 通过二者的命名可以认识到， 这两个转换流对象都是 `字符流对象`。

> **InputStreamReader : 是Reader的子类，可以将 InputStream 转换成 Reader。**
> **OutputStreamWriter : 是Writer的子类，可以将 OutputStream 转换成 Writer。**
> 【**二者在转换过程中，可以指定编码字符集，构造方法中直接传入参数即可**】

> 可以看一下两个对象的继承关系图：

》1.InputStreamReader
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050017279412.png)
》2.OutputStreamWriter:
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050017279513.png)

##### 3.InputStreamReader的使用

> 使用方式很简单，直接将 [InputStream](https://so.csdn.net/so/search?q=InputStream&spm=1001.2101.3001.7020) 的对象放入 InputStreamReader 的构造器中即可。

3.1 要读取的文件及编码格式

> 文件路径 ： `D:\\EDailyRoutine\\java-io-test\\字符读取乱码演示.txt`

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050020316616.png)

```java
import java.io.*;

/**
 * author : northcastle
 * createTime:2022/1/9
 * 字节流转换成字符流，并指定文件编码格式，进行文件的读取
 */
public class ApplicationInputStreamReader {
    public static void main(String[] args)  {
        String filePath = "D:\\EDailyRoutine\\java-io-test\\字符读取乱码演示.txt";
        //1.声明普通的 字节输入流  对象
        FileInputStream fileInputStream = null; 
        //2.声明读取文件的编码格式
        String charset = "GB2312";
        //3.声明 转换流 对象
        InputStreamReader inputStreamReader = null;
        //4.声明 处理流 对象
        BufferedReader bufferedReader = null;
        //5.声明一个String对象来接收读取到的内容
        String context = "";

        try {
            //6.实例化 字节流 对象
            fileInputStream = new FileInputStream(filePath);
            //7.实例化 转换流 对象，此处可以指定编码格式
            inputStreamReader = new InputStreamReader(fileInputStream,charset);
            //8.实例化 处理流对象
            bufferedReader = new BufferedReader(inputStreamReader);
            //9.执行读取操作，并把读取到的内容打印到控制台中
            while ((context = bufferedReader.readLine()) != null){
                System.out.println(context);
            }
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //10.关闭最外层的处理流
            if (bufferedReader != null){
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050026884318.png)



##### 4.OutputStreamWriter的使用

```java
import java.io.*;

/**
 * author : northcastle
 * createTime:2022/1/9
 * 本例演示：使用转换流，指定编码方式写入数据到文件中
 */
public class ApplicationOutputStramWriter {
    public static void main(String[] args) {
        //0.声明要写入的文件的路径
        String filePath = "D:\\EDailyRoutine\\java-io-test\\字符写入-转换流演示.txt";
        //1.声明 字节输出流对象
        FileOutputStream fileOutputStream = null;
        //2.声明 转换流对象
        OutputStreamWriter outputStreamWriter = null;
        //3.声明 处理流对象
        BufferedWriter bufferedWriter = null;

        try {
            //4.实例化 字节输出流对象
            fileOutputStream = new FileOutputStream(filePath);
            //5.实例化 转换流对象，同时指定编码格式为 utf8
            outputStreamWriter = new OutputStreamWriter(fileOutputStream,"utf8");
            //6.实例化 缓冲流对象
            bufferedWriter = new BufferedWriter(outputStreamWriter);
            //7.往文件中写入内容
            bufferedWriter.write("Hello OutputStreamWriter");
            bufferedWriter.newLine();
            bufferedWriter.write("这里是测试字节字符转换流的操作 OutputStreamWriter");

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //8.关闭最外层的 缓冲流即可
            if (bufferedWriter != null){
                try {
                    bufferedWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



### 5.对象流

#### `ObjectInputStream`+`ObjectOutputStream`

#### 1.由一个需求引入对象流

##### 1.1 需求描述

```
1.将【基本数据类型 int a = 10】, 保存到文件中，并能够从文件中恢复；
2.将 【对象类型 new Dog("旺财"，3)】,保存到文件中，并能够从文件中恢复。
```

##### 1.2 需求分析

```
上面的两个需求可做如下分析：
1.需要将数据保存到文件中，（这个可以通过文件的写操作实现）
2.保存数据的同时，需要将 数据的类型也一起保存起来？（这个使用普通的流无法实现）
3.读取文件的时候，也要把 数据的类型读取出来？（这个使用普通的流无法实现）
```

##### 1.3 需求解决

```
上述的需求，就是需要把 【基本数据类型】和【对象类型】进行【序列化】和【反序列化】。
```

#### 2.补充-序列化的概念

```
【序列化】：就是 保存数据时，同时保存 数据的类型+数据值
【反序列化】：就是 读取数据时，同时读取 数据的类型+数据值
12
[注意]：
    在将对象进行序列化的时候，该对象的类必须是可序列化的。
    实现方式推荐 通过实现 Serializable 接口。
```

#### 3.初识对象流

##### 3.1 ObjectOutputStream

> **ObjectOutputStream** : 提供`序列化`功能，可以实现**保存 数据类型+数据值 到文件中**去。
> 类的继承关系图如下：

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050068840420.png)

##### 3.2 ObjectInputStream

> **ObjectInputStream** : 提供`反序列化`功能，可以实现 **从文件中 读取 数据类型+数据值**。
> 类的继承关系图：

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166050068840521.png)

#### 4.对象流的使用规则

> 在使用 对象流的时候，需要注意以下几个方面：

```
1.自定义的类需要 实现 Serializable 接口来确保可以进行序列化；
2.序列化的类中，建议添加 SerialVersionUID来提高版本的兼容性；
3.序列化对象时，默认将对象中的所有属性都进行序列化，但static 和 transient 修饰的成员除外；
4.序列化对象时，要求对象的属性的类型，也需要实现序列化接口；
5.序列化具有继承性，即一个类实现了序列化接口，则其子类也默认实现了序列化接口。
6.使用对象流在进行 序列化 和 反序列化 的时候，顺序要一致，即先写出去的，就要先读进来。
123456
```

#### 5.ObjectOutputStream使用案例

> 将 对象 序列化 保存

##### 5.1 一个自定义类，实现了Serializable接口

```java
@Data
public class Dog implements Serializable {
    private String name;
    private Integer age;
    private Integer color;

}
```

##### 5.2 对象流的使用

主要看`writeObject（）`方法

```java
public class ApplicationObjectOutputStream {
    public static void main(String[] args) {
        //1.声明一个输出的文件位置
        String filePath = "D:\\EDailyRoutine\\java-io-test\\ObjectOutputStream.data";
        //2.声明一个文件输出流
        FileOutputStream fos = null;
        //3.声明一个对象输出流
        ObjectOutputStream oos = null;

        try {
            //4.实例化 文件输出流
            fos = new FileOutputStream(filePath);
            //5.实例化 对象输出流
            oos = new ObjectOutputStream(fos);
            //6.执行写操作 ： 进行序列化保存
            oos.writeInt(100); // 写 Integer
            oos.writeFloat(200.123f); // 写 Float
            oos.writeDouble(300.2d); // 写 Double
            oos.writeBoolean(false); // 写 Boolean
            oos.writeChar(95); // 写 Char
            oos.writeUTF("你好，我是对象输出流！"); // 写字符串 String
            oos.writeObject(new Dog("旺财",3,123)); // 写自定义的对象
            oos.writeUTF("============================"); // 写字符串
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
            // 7.关闭最外层的 处理流 ObjectOutputStream
            if (oos != null){
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```

执行结果：

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16.png)

#### 6.ObjectInputStream使用案例

> 将数据 反序列化 读入到内存中.
> 此案例结合 上面的第5小结中序列化的数据。
> **【注】：反序列化的顺序，必须和 序列化的顺序相同！**

```java
public class ApplicationObjectInputStream {
    public static void main(String[] args)  {
        //1.指定文件路径
        String filePath = "D:\\EDailyRoutine\\java-io-test\\ObjectOutputStream.data";
        //2.声明 文件输入流
        FileInputStream fis = null;
        //3.声明 对象输入流
        ObjectInputStream ois = null;

        try {
            //4.实例化 文件输入流
            fis = new FileInputStream(filePath);
            //5.实例化 对象输入流
            ois = new ObjectInputStream(fis);
            //6.执行反序列化操作：读取内容到程序中
            System.out.println(ois.readInt()); // 读 Integer
            System.out.println(ois.readFloat()); // 读 Float
            System.out.println(ois.readDouble()); // 读 Double
            System.out.println(ois.readBoolean()); // 读 Boolean
            System.out.println(ois.readChar()); // 读 Char
            System.out.println(ois.readUTF()); // 读 字符串 String
            System.out.println(ois.readObject()); // 读 对象
            System.out.println(ois.readUTF()); // 对 字符串 String

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }finally {
            //7.关闭对象输入流
            if (ois != null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



### 6.打印流（可不看）

#### 1.认识两个打印流

> 两个打印流 `PrintStream` 和`PrintWriter`。
> 顾名思义，打印流，就是专门用来输出内容的；
> 所以，打印流`只有输出流`，而没有输入流。

##### 1.1 PrintStream

> 字节打印流:可以包装一个 [OutputStream](https://so.csdn.net/so/search?q=OutputStream&spm=1001.2101.3001.7020)、File对象 或者 文件路径。

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16605060823165.png)

##### 1.2 [PrintWriter](https://so.csdn.net/so/search?q=PrintWriter&spm=1001.2101.3001.7020)

> 字符打印流：可以包装一个 OutputStream、Writer、File或者文件路径。
> 【[构造方法](https://so.csdn.net/so/search?q=构造方法&spm=1001.2101.3001.7020)中带有 布尔类型 的参数，true 表示 println、printf、format 方法 会自动刷新缓存】
> 但个人不建议这样使用，使用字节流时，还是在最后手动close(),执行真正的写入操作吧。

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16605060823166.png)

#### 2.PrintStream使用案例

##### 2.1 标准的输出流 System.out

> 标准的输出流 System.out 就是一个字节打印流。
> 可以使用 `write()` 方法;
> 也可以使用 `print()` 方法。

```Java
public class ApplicationPrintStream {
    public static void main(String[] args) throws IOException {
        PrintStream out = System.out; // System.out 就是一个自己打印流
        out.print(100);
        out.println("hello,我是字节打印流 PrintStream "); // println() 方法就是调用了write方法
        out.write("大家好".getBytes());
        out.close();
    }
}
```

##### 2.2 包装OutputStream对象

```java
public class ApplicationPrintStream {
    public static void main(String[] args) throws IOException {
    
         //1.声明一个文件路径
         String filePath = "D:\\EDailyRoutine\\java-io-test\\PrintStream02.data";
         //2.实例化一个 FileOutputStream 对象
         FileOutputStream fos = new FileOutputStream(filePath);
         //3.包装 OutputStream 对象，实例化一个 PrintStream 对象
         PrintStream printStream = new PrintStream(fos);
         //4.开始输出内容
         printStream.write("我是printStram对象，哈哈哈".getBytes());
         printStream.println();
         printStream.println("使用println方法打印内容");
         //5.关闭流
         printStream.close();
    }
}
```

> 运行结果：内容输出正常

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_17,color_FFFFFF,t_70,g_se,x_16.png)

#### 3.PrintStream使用案例

> 注意 ： 使用字节打印流时，建议在最后都要执行一边 `close()` 方法，将写出的内容真正写入到文件中。

```java
public class ApplicationPrintWriter {
    public static void main(String[] args) throws FileNotFoundException {

        String filePath = "D:\\EDailyRoutine\\java-io-test\\PrintWriter01.data";
        PrintWriter printWriter = new PrintWriter(filePath);
        printWriter.println("我来使用一下【PrinterWriter对象】这个打印流到文件");
        printWriter.write("我是write方法来写的");
        printWriter.close(); // 必须是在关闭流的时候，才能真正将内容写入到文件中
    }
}
```

> 运行结果：内容正常输出

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_17,color_FFFFFF,t_70,g_se,x_16-166050636943613.png)

##### 3.2 包装OutputStream对象打印输出

```java
public class ApplicationPrintWriter {
    public static void main(String[] args) throws FileNotFoundException {
     
        /**
         * 二 ： 包装一个 OutputStream 对象进行输出
         */
        String filePath02 = "D:\\EDailyRoutine\\java-io-test\\PrintWriter02.data";
        FileOutputStream fos = new FileOutputStream(filePath02);
        PrintWriter printWriter1 = new PrintWriter(fos);
        printWriter1.println("包装一个 OutputStream 对象进行输出");
        printWriter1.write("hello 哈哈哈");
        printWriter1.close();// 必须是在关闭流的时候，才能真正将内容写入到文件中
    }
}
```

> 运行结果：内容正常输出

![在这里插入图片描述](E:\Development\Typora\images\3a6f25cccb734c538b69e89d2ca4b194.png)

##### 3.3 包装Writer对象打印输出

```java
public class ApplicationPrintWriter {
    public static void main(String[] args) throws IOException {
        /**
         * 三 ： 包装一个 Writer 对象进行输出
         */
        String filePath03 = "D:\\EDailyRoutine\\java-io-test\\PrintWriter03.data";
        FileWriter fw = new FileWriter(filePath03);
        PrintWriter printWriter2 = new PrintWriter(fw);
        printWriter2.println("包装一个 Writer 对象进行输出");
        printWriter2.write("欢迎来到 PrintWriter 打印流");
        printWriter2.close();// 必须是在关闭流的时候，才能真正将内容写入到文件中

    }
}
```

> 运行结果：内容正常输出

![在这里插入图片描述](E:\Development\Typora\images\66f891a268f1465a8a66b453e5a11be8.png)



## 三.Properties类

### 1.什么是[properties](https://so.csdn.net/so/search?q=properties&spm=1001.2101.3001.7020)文件

> `*.properties` 文件是一种常见的 `key-value 格式`的配置文件。
> 格式 ： 键=值
>
> 1. 键和值之间用`=`连接，不需要空格
> 2. 值不需要用引号引起来，默认是String 类型
> 3. **如果存在多个相同的key，则后面的值会覆盖前面的值**

### 2.什么是Properties类

> 1. `Properties.java` 是 `java.util`包下的一个工具类；
> 2. `Properties.java` 是`HashTable`的一个[子类](https://so.csdn.net/so/search?q=子类&spm=1001.2101.3001.7020)，所以它也是一个集合类。
> 3. `Properties.java` 的主要使用的[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)方法有：
>    `load()` : 加载properties文件到内存中
>    `list()` : 指定输出位置，将properties文件中的内容输出出来
>    `getProperty(key)`: 根据key获取对应的值
>    `setProperty()`: 设置Properties类的内容
>    `store()`: 把Properties对象的内容持久化到文件中去

4.**继承关系图**如下 ：

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16.png)

### 3.Properties类读取文件内容

#### 3.1 properties文件

> 文件内容如下 ：

```properties
name = 这是名称name
age=18
school_name=青岛大学
school_addr=山东省青岛市市南区
name=这是名称2
```

#### 3.2 案例代码

```java
public class PropertiesReadForCSDN {
    public static void main(String[] args) {
        //1.要读取的文件位置：绝对路径
        String filePath = "D:\\EDailyRoutine\\java-io-test\\properties_file.properties";
        //2.声明 字符输入流 对象
        FileReader fr = null;
        //3.声明 Properties 对象
        Properties propertiesObj = null;

        try {
            //4.实例化 字符输入流 对象
            fr = new FileReader(filePath);
            //5.实例化 Properties 对象
            propertiesObj = new Properties();
            //6.加载 properties 文件
            propertiesObj.load(fr);
            //7.把文件中的内容输出出来 ; 具体的输出位置由参数指定
            propertiesObj.list(System.out);
            System.out.println("==============================");
            //8.获取指定的值，默认所有的值都是String类型
            String name = propertiesObj.getProperty("name");
            System.out.println("name : -->"+name+"<--"); // 如果有重复的key的时候，后面的值会覆盖前面的值
            String age = propertiesObj.getProperty("age");
            System.out.println("age : "+age);
            String school_name = propertiesObj.getProperty("school_name");
            System.out.println("school_name : "+school_name);
            String school_addr = propertiesObj.getProperty("school_addr");
            System.out.println("school_addr : "+school_addr);


        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //9.关闭流
            if (fr != null){
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### 3.3 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_17,color_FFFFFF,t_70,g_se,x_16.png)

### 4.Properties类写内容到文件中

#### 4.1 代码案例

```java
public class PropertiesWriterForCSDN {
    public static void main(String[] args) {
        //1.输出文件路径
        String filePath = "D:\\EDailyRoutine\\java-io-test\\properties_file02.properties";
        //2.声明 字符输出流对象
        FileWriter fr = null;
        //3.声明 Properties 对象
        Properties propertiesObj = null;

        try {
            //4.实例化 字符输出流对象
            fr = new FileWriter(filePath);
            //5.实例化 Properties 对象
            propertiesObj = new Properties();
            //6.赋值
            propertiesObj.setProperty("bookName","深入理解Java虚拟机");
            propertiesObj.setProperty("bookPrice","123.88");
            propertiesObj.setProperty("bookName","深入理解Java虚拟机(第三版)");
            propertiesObj.setProperty("bookAuthor","zhouzhi明");
            //7.持久化到文件中 ： 第二个参数是备注信息，如果是中文，则在properties中是被转码的
            propertiesObj.store(fr,"这是一个备注信息");
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //8.关闭字符输出流
            if (fr != null){
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### 4.2 执行结果

> 文件内容写入正常

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16605142284835.png)



## 四.相对路径

