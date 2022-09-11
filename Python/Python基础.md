# 	Python基础

## 基础语法

### 变量

#### 基本介绍

 变量，是用于在内存中存放程序数据的容器，怎么理解呢？

 计算机的最核心功能就是“计算”， 计算需要数据源，数据源要存在内存里，比如我要把小明的姓名、身高、年龄信息存下来，后面程序会调用，怎么存呢，直接设置一个“变量名=值”， 就可以

```python
x = 1
y = 2
print(x + y)
```



#### 命名规则

1. `变量名只能是 字母、数字或下划线的任意组合`

2. 变量名的第一个字符不能是数字

3. 以下关键字不能声明为变量名

   `[‘and’, ‘as’, ‘assert’, ‘break’, ‘class’, ‘continue’, ‘def’, ‘del’, ‘elif’, ‘else’, ‘except’, ‘exec’, ‘finally’, ‘for’, ‘from’, ‘global’, ‘if’, ‘import’, ‘in’, ‘is’, ‘lambda’, ‘not’, ‘or’, ‘pass’, ‘print’, ‘raise’, ‘return’, ‘try’, ‘while’, ‘with’, ‘yield’]`



#### **常用定义方式**

 驼峰体

```python
AgeOfOldboy = 56 
NumberOfStudents = 80
```

 下划线

```python
age_of_oldboy = 56 
number_of_students = 80
```

> java写多了，第一种用多了！听说官方推荐使用第二种，emm…，我寻思着怎么跟SQL一样呀！没办法，我们只能入乡随俗了！！！ 😒

#### **定义变量不好的方式举例**

- 变量名为中文、拼音
- 变量名过长
- 变量名词不达意

#### 常量

 常量即指不变的量，如pai 3.141592653…, 或在程序运行过程中不会改变的量

 举例，假如老男孩老师的年龄会变，那这就是个变量，但在一些情况下，他的年龄不会变了，那就是常量。`在Python中没有一个专门的语法代表常量，程序员约定俗成用变量名全部大写代表常量`

```
AGE_OF_OLDBOY = 56
```



#### 例子

```python
# 变量的定义
x = 1
y = 2
studentName = '张三'
student_name = '李四'
student1 = '赵武'
print(x + y)
print("驼峰名字："+studentName)
print("下划线名字："+student_name)
print("包含数字名字："+student1)

# 变量修改
x = 3
print(x+y)

# 常量
STUDENT_NAME = "峰哥"
print(STUDENT_NAME)
```



### 注释

 随着学习的深入，用不了多久，你就可以写复杂的上千甚至上万行的代码啦，有些代码你花了很久写出来，过了些天再回去看，发现竟然看不懂了，哈哈，这太正常了。 另外，你以后在工作中会发现，一个项目多是由几个甚至几十个开发人员一起做，你要调用别人写的代码，别人也要用你的，如果代码不加注释，你自己都看不懂，更别说别人了，这样写会挨打的。所以为了避免这种尴尬的事情发生，一定要增加你代码的可读性。

 代码注释分单行和多行注释， 单行注释用`#`，多行注释可以用**三对**双引号`“”” “””`

```python
"""
 这里是多行注释
 这里是多行注释
 这里是多行注释
"""

# 这里是单行注释
```

**代码注释原则:**

1. 不用给全部代码加注释，只需要在自己觉得重要或不好理解的部分加注释即可
2. 注释可以用中文或英文，但绝对不要拼音噢
3. 注释不光要给自己看，还要给别人看，所以请认真写



### 数据类型

 我们人类可以很容易的分清数字与字符的区别，但是计算机并不能呀，计算机虽然很强大，但从某种角度上看又很傻，除非你明确的告诉它，1是数字，“汉”是文字，否则它是分不清1和‘汉’的区别的，因此，在每个编程语言里都会有一个叫数据类型的东东，其实就是对常用的各种数据类型进行了明确的划分，你想让计算机进行数值运算，你就传数字给它，你想让他处理文字，就传字符串类型给他。Python中常用的数据类型包括多种，今天我们暂只讲4种， 数字、字符串、布尔类型、列表。

#### 数字(int,float)

- **int（整型）**

 在64位系统上，整数的位数为64位，取值范围为-2**63～2**63-1，即-9223372036854775808～9223372036854775807

- **long（长整型）**

 跟C语言不同，Python的长整数没有指定位宽，即：Python没有限制长整数数值的大小，但实际上由于机器内存有限，我们使用的长整数数值不可能无限大。

 注意，自从Python2.2起，如果整数发生溢出，Python会自动将整数数据转换为长整数，所以如今在长整数数据后面不加字母L也不会导致严重后果了。

> ***注意：在Python3里不再有long类型了，全都是int***

- **float (浮点型)**

  即小数

 例子

```python
# int
age_int = 1

# float
score_float = 89.999

print(type(age_int))
print(type(score_float))
```

 输出

```python
<class 'int'>
<class 'float'>
```



#### 字符串(str)

 `在Python中,加了引号的字符都被认为是字符串！`

 字符串是一个有序的字符的集合，用于存储和表示基本的文本信息，’ ‘或’’ ‘’或’’’ ‘’’中间包含的内容称之为字符串

**特性**

1. 按照从左到右的顺序定义字符集合，下标从0开始顺序访问，有序

2. ![在这里插入图片描述](E:\Development\Typora\images\20200727163713338.png)

   在这里插入图片描述

3. 可以进行切片操作

4. 不可变，字符串是不可变的，不能像列表一样修改其中某个元素，所有对字符串的修改操作其实都是相当于生成了一份新数据。

```python
# 字符串
name_str = "你好呀"
favorite_str = '我喜欢python'
desc_str = '''1
    python是一门好语言
    还行吧
2'''


print(name_str)
print(desc_str)
print("切片：", name_str[0:2])  # 你好
print("长度：", len(name_str))  # 3
print("'好'的索引：", name_str.index("好"))  # 1

user_note = "go!gO!Go!"
print("o的个数(可以指定位置)：", user_note.count('o', 1, 5))  # 1
print("查找列表中指定的值(find,可以指定位置)：", user_note.find('o', 2, -1))  # 7
print("查找列表中指定的值(find,可以指定位置)：", user_note.find('o', 2, 5))   # -1 (找不到)
print("首字母大写,其他小写(capitalize)：", user_note.capitalize())  # Go!go!go!
print("字符串全变为小写(casefold)：", user_note.casefold())  # go!go!go!
print("满足字符串的长度，不足的用指定符号来凑齐(center)：", user_note.center(20, '-'))   # -----go!gO!Go!------
print("满足字符串的长度，不足的用指定符号来凑齐(ljust)", user_note.ljust(20, "-"))  # go!gO!Go!-----------
print("满足字符串的长度，不足的用指定符号来凑齐(rjust)", user_note.rjust(20, "-"))  # -----------go!gO!Go!
print("是否以Go!结尾(endswith)：", user_note.endswith("Go!"))  # True
print("是否以go开始(startswith)：", user_note.startswith("go"))  # True

# 两种方式
string_format = "这个地方是{0},距离现在有{1}km,您去不去呢？{select}"
print(string_format.format("上海", "1000", select="Go!"))  # 这个地方是上海,距离现在有1000km,您去不去呢？Go!

# 数组拼接
string_arr = ["你好", "呀", "!", "去吧"]
print("".join(string_arr))  # 你好呀!去吧
```

 输出

```python
你好呀
1
    python是一门好语言
    还行吧
2
切片： 你好
长度： 3
'好'的索引： 1
o的个数(可以指定位置)： 1
查找列表中指定的值(find,可以指定位置)： 7
查找列表中指定的值(find,可以指定位置)： -1
首字母大写,其他小写(capitalize)： Go!go!go!
字符串全变为小写(casefold)： go!go!go!
满足字符串的长度，不足的用指定符号来凑齐(center)： -----go!gO!Go!------
满足字符串的长度，不足的用指定符号来凑齐(ljust) go!gO!Go!-----------
满足字符串的长度，不足的用指定符号来凑齐(rjust) -----------go!gO!Go!
是否以Go!结尾(endswith)： True
是否以go开始(startswith)： True
这个地方是上海,距离现在有1000km,您去不去呢？Go!
你好呀!去吧
```

> 注意多行字符的格式，有空行和空格(输出中的1和2)！
>
> 更多的操作(常用，自己看API进行，我这里就不一一列举了)：
>
> [![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70.png)](https://img-blog.csdnimg.cn/20200727163904602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 casefold和lower函数。这两个函数的主要功能都是将字符串中的元素变成小写，但是最重要的区别就是lower函数只支持ascill表中的字符，而casefold则支持很多不同种类的语言。比如说β，lower只能显示出原形而casefold则能显示他的小写–ss

```python
def capitalize(self):  
    首字母大写
def casefold(self):  
    把字符串全变小写
    >> > c = 'Alex Li'
    >> > c.casefold()
    'alex li'
def center(self, width, fillchar=None):  
    >> > c.center(50, "-")
    '---------------------Alex Li----------------------'
def count(self, sub, start=None, end=None):  
    """
    S.count(sub[, start[, end]]) -> int
    >>> s = "welcome to apeland"
    >>> s.count('e')
    3
    >>> s.count('e',3)
    2
    >>> s.count('e',3,-1)
    2
def encode(self, encoding='utf-8', errors='strict'):  
    """
    编码，日后讲
def endswith(self, suffix, start=None, end=None):
    >> > s = "welcome to apeland"
    >> > s.endswith("land") 判断以什么结尾
    True
def find(self, sub, start=None, end=None):  
    """
    S.find(sub[, start[, end]]) -> int
    Return the lowest index in S where substring sub is found,
    such that sub is contained within S[start:end].  Optional
    arguments start and end are interpreted as in slice notation.
    Return -1 on failure.
    """
    return 0
def format(self, *args, **kwargs):  # known special case of str.format
    >> > s = "Welcome {0} to Apeland,you are No.{1} user."
    >> > s.format("Eva", 9999)
    'Welcome Eva to Apeland,you are No.9999 user.'
    >> > s1 = "Welcome {name} to Apeland,you are No.{user_num} user."
    >> > s1.format(name="Alex", user_num=999)
    'Welcome Alex to Apeland,you are No.999 user.'
def format_map(self, mapping):  
    """
    S.format_map(mapping) -> str
    Return a formatted version of S, using substitutions from mapping.
    The substitutions are identified by braces ('{' and '}').
    """
    讲完dict再讲这个
def index(self, sub, start=None, end=None):  
    """
    S.index(sub[, start[, end]]) -> int
    Return the lowest index in S where substring sub is found, 
    such that sub is contained within S[start:end].  Optional
    arguments start and end are interpreted as in slice notation.
    Raises ValueError when the substring is not found.
    """
def isdigit(self):  
    """
    S.isdigit() -> bool
    Return True if all characters in S are digits
    and there is at least one character in S, False otherwise.
    """
    return False
def islower(self):  
    """
    S.islower() -> bool
    Return True if all cased characters in S are lowercase and there is
    at least one cased character in S, False otherwise.
    """
def isspace(self):  
    """
    S.isspace() -> bool
    Return True if all characters in S are whitespace
    and there is at least one character in S, False otherwise.
    """
def isupper(self):  
    """
    S.isupper() -> bool
    Return True if all cased characters in S are uppercase and there is
    at least one cased character in S, False otherwise.
    """
def join(self, iterable):  
    """
    S.join(iterable) -> str
    Return a string which is the concatenation of the strings in the
    iterable.  The separator between elements is S.
    """
    >>> n = ['alex','jack','rain']
    >>> '|'.join(n)
    'alex|jack|rain'
def ljust(self, width, fillchar=None):  
    """
    S.ljust(width[, fillchar]) -> str
    Return S left-justified in a Unicode string of length width. Padding is
    done using the specified fill character (default is a space).
    """
    return ""
def lower(self):  
    """
    S.lower() -> str
    Return a copy of the string S converted to lowercase.
    """
    return ""
def lstrip(self, chars=None):  
    """
    S.lstrip([chars]) -> str
    Return a copy of the string S with leading whitespace removed.
    If chars is given and not None, remove characters in chars instead.
    """
    return ""
def replace(self, old, new, count=None):  
    """
    S.replace(old, new[, count]) -> str
    Return a copy of S with all occurrences of substring
    old replaced by new.  If the optional argument count is
    given, only the first count occurrences are replaced.
    """
    return ""
def rjust(self, width, fillchar=None):  
    """
    S.rjust(width[, fillchar]) -> str
    Return S right-justified in a string of length width. Padding is
    done using the specified fill character (default is a space).
    """
    return ""
def rsplit(self, sep=None, maxsplit=-1):  
    """
    S.rsplit(sep=None, maxsplit=-1) -> list of strings
    Return a list of the words in S, using sep as the
    delimiter string, starting at the end of the string and
    working to the front.  If maxsplit is given, at most maxsplit
    splits are done. If sep is not specified, any whitespace string
    is a separator.
    """
    return []
def rstrip(self, chars=None):  
    """
    S.rstrip([chars]) -> str
    Return a copy of the string S with trailing whitespace removed.
    If chars is given and not None, remove characters in chars instead.
    """
    return ""
def split(self, sep=None, maxsplit=-1):  
    """
    S.split(sep=None, maxsplit=-1) -> list of strings
    Return a list of the words in S, using sep as the
    delimiter string.  If maxsplit is given, at most maxsplit
    splits are done. If sep is not specified or is None, any
    whitespace string is a separator and empty strings are
    removed from the result.
    """
    return []
def startswith(self, prefix, start=None, end=None):  
    """
    S.startswith(prefix[, start[, end]]) -> bool
    Return True if S starts with the specified prefix, False otherwise.
    With optional start, test S beginning at that position.
    With optional end, stop comparing S at that position.
    prefix can also be a tuple of strings to try.
    """
    return False
def strip(self, chars=None):  
    """
    S.strip([chars]) -> str
    Return a copy of the string S with leading and trailing
    whitespace removed.
    If chars is given and not None, remove characters in chars instead.
    """
    return ""
def swapcase(self):  
    """
    S.swapcase() -> str
    Return a copy of S with uppercase characters converted to lowercase
    and vice versa.
    """
    return ""
def upper(self):  
    """
    S.upper() -> str
    Return a copy of S converted to uppercase.
    """
    return ""
def zfill(self, width):  
    """
    S.zfill(width) -> str
    Pad a numeric string S with zeros on the left, to fill a field
    of the specified width. The string S is never truncated.
    """
    return ""
```



 **字符串拼接**

 数字可以进行加减乘除等运算，字符串呢？让我大声告诉你，也能？what ?是的，但只能进行”相加”和”相乘”运算。

```python
# 字符串
name_str = "你好呀"
favorite_str = '我喜欢python'
desc_str = '''
    python是一门好语言
    还行吧
'''

print("name_str+favorite_str+desc_str+age_int："+name_str+favorite_str+desc_str)
print("name_str * 10："+name_str*10)
```



 输出

```python
name_str+favorite_str+desc_str+age_int：你好呀我喜欢python
    python是一门好语言
    还行吧

name_str * 10：你好呀你好呀你好呀你好呀你好呀你好呀你好呀你好呀你好呀你好呀
```



> 注意，字符串的拼接只能是双方都是字符串，不能跟数字或其它类型拼接

```python
print("name_str+favorite_str+desc_str+age_int："+name_str+favorite_str+desc_str+age_int)
```

 **输出**

[![img](E:\Development\Typora\images\20200726202647452.png)](https://img-blog.csdnimg.cn/20200726202647452.png)



 但我们可以这么写

逗号

```python
print("name_str+favorite_str+desc_str+age_int："+name_str+favorite_str+desc_str, age_int)
```

 **输出**

```python
name_str+favorite_str+desc_str+age_int：你好呀我喜欢python1
    python是一门好语言
    还行吧
2 1
```

> 最后我的1是int类型



#### 布尔(bool)

 布尔类型很简单，就两个值 ，一个True(真)，一个False(假), 主要用记逻辑判断

 但其实你们并不明白对么？ let me explain, 我现在有2个值 ， a=3, b=5 , 我说a>b你说成立么? 我们当然知道不成立，但问题是计算机怎么去描述这成不成立呢？或者说a< b是成立，计算机怎么描述这是成立呢？

 没错，答案就是，用布尔类型

```python
flag = True
not_flag = False
print(type(flag))
print(not_flag)
print(1 < 2)
```

 输出

```python
<class 'bool'>
False
True
```



#### 字节(bytes)

 bytes类型是指一堆字节的集合，在python中以b开头的字符串都是bytes类型

```python
b'\xe5\xb0\x8f\xe7\x8c\xbf\xe5\x9c\x88' #b开头的都代表是bytes类型，是以16进制来显示的，2个16进制代表一个字节。 utf-8是3个字节代表一个中文，所以以上正好是9个字节
```

 计算机只能存储2进制， 我们的字符、图片、视频、音乐等想存到硬盘上，也必须以正确的方式编码成2进制后再存。

- 对于文字，我们可以以gbk编码，也可以以utf-8、ASCII编码。
- 对于图片，必须编码成PNG,JPEG等格式
- 对于音乐，必须编码成MP3,WAV等

 在python中， 数据转成2进制后不是直接以0101010的形式表示的，而是用一种叫bytes(字节)的类型来表示，人类不可读。字符串转成bytes后长成这个样子

```python
>>> s = "小猿圈"
>>> s.encode("utf-8")  # 以utf-8编码 
b'\xe5\xb0\x8f\xe7\x8c\xbf\xe5\x9c\x88' #b开头的都代表是bytes类型，是以16进制来显示的，2个16进制代表一个字节。 utf-8是3个字节代表一个中文，所以以上正好是9个字节
```

 在python中，字符串必须编码成bytes后才能存到硬盘上。 唉，你说，我之前学的文件操作时也没有把字符串编码后再存呀， 哈，那是python默认帮你干了这个事，在python3中文件存储的默认编码是utf-8.

 当然你可以自行改变文件的默认编码，但意味着你存的数据

```python
f = open(file="encode_test",encoding="gbk",mode="w")
```

 这样，你写入的数据就是按gbk编码的了。



#### 数组(list)

```python
user_names = ["Tom", "Alex", "Jack", "Rain", "Mack"]
print("第二个名称："+user_names[1])
# 指定位置添加
user_names.insert(3, "Drake")
# 最后添加
user_names.append("Taylor")
# 修改(-1为修改最后一个)
user_names[0] = "Tomcat"
user_names[-1] = "Mack-"
# 删除(删除指定的元素,只删除第一个)
user_names.remove("Mack")

# 判断元素是否在数组里
print("Alex是否在数组中：", "Alex" in user_names)
print("Alex1是否在数组中：", "Alex1" in user_names)
# 获取元素索引
print("Alex的索引(index)：", user_names.index("Alex"))

# 输出
print("操作后的输出", user_names)

# 翻转
user_names.reverse()
print("数组翻转(reverse)", user_names)

# 指定元素的总量
print("指定元素Alex的总量(count)：", user_names.count("Alex"))

# 数组的长度
print("数组的长度(len)：", len(user_names))

# 排序
user_names.sort()
print("排序后的数组：", user_names)

# 复制
user_names_copy = user_names.copy()
print("数组的复制(copy)：", user_names_copy)

# 删除最后一个元素
user_names.pop()
print("删除最后一个元素(pop)：", user_names)

# 列表的合并
user_names.extend(user_names_copy)
print("数组的合并(extend)：", user_names)

# 列表的嵌套
user_names.insert(2, ["James", "Kobe"])
print("增加元素后：", user_names)
print("第3个元素中的第1一个：", user_names[2][0])

# 删除(删除指定索引的元素)
del user_names[0]
print("删除第一个元素后：", user_names)

# 切片
print("取出数组中索引为[1,4)的元素：", user_names[1:4])
print("取出数组中索引为[1,-2)的元素：", user_names[1:-2])
print("取出第1个之后的元素：", user_names[1:])
print("取出数前五个元素：", user_names[:5])
print("取出数组中索引为[-5,-1)的元素：", user_names[-5: -1])
print("整个数组：", user_names[:])
# 切片，改变步长
print("取出数组中索引为[1,4)，步长为2的元素：", user_names[1:4:2])
print("倒着写[-1,-5),步长为-1：", user_names[-1:-5:-1])
print("整个数组，步长为2：", user_names[::2])
print("整个数组翻转：", user_names[::-1])

# 循环
for i in user_names:
    print(i, end="  ")
print()
i = 0
while i < len(user_names):
    print(user_names[i], end="  ")
    i += 1
print()


# 清空(删除所有的元素)
user_names.clear()
print("清空(clear)：", user_names)
```

 输出



```python
第二个名称：Alex
Alex是否在数组中： True
Alex1是否在数组中： False
Alex的索引(index)： 1
操作后的输出 ['Tomcat', 'Alex', 'Jack', 'Drake', 'Rain', 'Mack-']
数组翻转(reverse) ['Mack-', 'Rain', 'Drake', 'Jack', 'Alex', 'Tomcat']
指定元素Alex的总量(count)： 1
数组的长度(len)： 6
排序后的数组： ['Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
数组的复制(copy)： ['Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
删除最后一个元素(pop)： ['Alex', 'Drake', 'Jack', 'Mack-', 'Rain']
数组的合并(extend)： ['Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
增加元素后： ['Alex', 'Drake', ['James', 'Kobe'], 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
第3个元素中的第1一个： James
删除第一个元素后： ['Drake', ['James', 'Kobe'], 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
取出数组中索引为[1,4)的元素： [['James', 'Kobe'], 'Jack', 'Mack-']
取出数组中索引为[1,-2)的元素： [['James', 'Kobe'], 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-']
取出第1个之后的元素： [['James', 'Kobe'], 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
取出数前五个元素： ['Drake', ['James', 'Kobe'], 'Jack', 'Mack-', 'Rain']
取出数组中索引为[-5,-1)的元素： ['Drake', 'Jack', 'Mack-', 'Rain']
整个数组： ['Drake', ['James', 'Kobe'], 'Jack', 'Mack-', 'Rain', 'Alex', 'Drake', 'Jack', 'Mack-', 'Rain', 'Tomcat']
取出数组中索引为[1,4)，步长为2的元素： [['James', 'Kobe'], 'Mack-']
倒着写[-1,-5),步长为-1： ['Tomcat', 'Rain', 'Mack-', 'Jack']
整个数组，步长为2： ['Drake', 'Jack', 'Rain', 'Drake', 'Mack-', 'Tomcat']
整个数组翻转： ['Tomcat', 'Rain', 'Mack-', 'Jack', 'Drake', 'Alex', 'Rain', 'Mack-', 'Jack', ['James', 'Kobe'], 'Drake']
Drake  ['James', 'Kobe']  Jack  Mack-  Rain  Alex  Drake  Jack  Mack-  Rain  Tomcat  
Drake  ['James', 'Kobe']  Jack  Mack-  Rain  Alex  Drake  Jack  Mack-  Rain  Tomcat  
清空(clear)： []
```

> 因为列表的是通过下标来标记元素位置的。 下标从0开始，每添加一个元素，就自动+1
>
> 其他方法可以自行查看



##### 增加

 **追加**，数据会追加到尾部

```
# 最后添加
user_names.append("Taylor")
```

 **插入**，可插入任何位置

```
# 指定位置添加
user_names.insert(3, "Drake")
```

 **合并**,可以把另一外列表的值合并进来

```
# 列表的合并
user_names.extend(user_names_copy)
```

 **复制**，将一个数组中的元素复制给另一个数组

```
# 复制
user_names_copy = user_names.copy()
```

 **列表嵌套**

```
# 列表的嵌套
user_names.insert(2, ["James", "Kobe"])
```



##### 删除

 **del 直接删**

```
# 删除(删除指定索引的元素)
del user_names[0]
```

 **pop 删**

```
# 删除最后一个元素
user_names.pop()
```

 **clear 清空**

```
# 清空(删除所有的元素)
user_names.clear()
```

 **remove 删除**

```
# 删除(删除指定的元素,只删除第一个)
user_names.remove("Mack")
```



##### 修改

```
# 修改(-1为修改最后一个)
user_names[0] = "Tomcat"
user_names[-1] = "Mack-"
```



##### 查询

 **arr[index]**查询指定索引位置的元素

```
print("第二个名称："+user_names[1])
```

 **in**,判断指定元素在数组中是否存在

```
# 判断元素是否在数组里
print("Alex是否在数组中：", "Alex" in user_names)
print("Alex1是否在数组中：", "Alex1" in user_names)
```

 **index**，判断指定元素在数组中的索引

```
# 获取元素索引
print("Alex的索引(index)：", user_names.index("Alex"))
```

 **count**，查询指定元素在数组中的数量

```
# 指定元素的总量
print("指定元素Alex的总量(count)：", user_names.count("Alex"))
```

 **len**，查询数组的长度

```
# 数组的长度
print("数组的长度(len)：", len(user_names))
```



##### 翻转

 **reverse**，指定数组所有元素进行一个翻转

```
# 翻转
user_names.reverse()
```

 **切片**，原数组并没有反转

```
print("整个数组翻转：", user_names[::-1])
```



##### 切片

 切片就像切面包，可以同时取出元素的多个值

```
names[start:end:step] #step默认为1
print("取出数组中索引为[1,4)的元素：", user_names[1:4])
print("取出数组中索引为[1,-2)的元素：", user_names[1:-2])
print("取出第1个之后的元素：", user_names[1:])
print("取出数前五个元素：", user_names[:5])
print("取出数组中索引为[-5,-1)的元素：", user_names[-5: -1])
print("整个数组：", user_names[:])
```

 **步长， 允许跳着取值**

```
print("取出数组中索引为[1,4)，步长为2的元素：", user_names[1:4:2])
print("倒着写[-1,-5),步长为-1：", user_names[-1:-5:-1])
print("整个数组，步长为2：", user_names[::2])
print("整个数组翻转：", user_names[::-1])
```



##### 排序

```
# 排序
user_names.sort()
```

 汉字也可以进行排序，排序的优化级规则是按这张表来的

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16628279372267.png)](https://img-blog.csdnimg.cn/20200727160420563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



##### 循环

```
# 循环
for i in user_names:
    print(i, end="  ")
print()
i = 0
while i < len(user_names):
    print(user_names[i], end="  ")
    i += 1
print()
```



##### 解构

```
user_info = ["张三", "中国", 20]
name, area, age = user_info
print("%s--%s--%s" % (name, area, age))
```

 输出

```
张三--中国--20
```

#### 元组(tuple)

 有些时候我们的列表数据不想被人修改时怎么办？ 就可以用元组存放，元组又被称为`只读列表`，不能修改。

 定义：与列表类似，只不过［］改成（）

 特性：

1.可存放多个值

2.不可变

3.按照从左到右的顺序定义元组元素，下标从0开始顺序访问，有序

 **创建**

```
user_names = ("James", "Tom", "Drake", ["Rain", "Jack"])
或者
user_names2 = tuple((1, 2, 3, 4))
```

 **常用操作**

```
print("所有元素：", user_names)
print("查看第二个元素：", user_names[1])
print("Tom的索引：", user_names.index("Tom"))
print("元组的长度：", len(user_names))
print("Tom的个数：", user_names.count("Tom"))
print("判断是否含有Tom：", "Tom" in user_names)
print("查询第[1， 3)个元素(切片)：", user_names[1:3])
# 还有循环，不列出来了
```

 输出

```
所有元素： ('James', 'Tom', 'Drake', ['Rain', 'Jack'])
查看第二个元素： Tom
Tom的索引： 1
元组的长度： 4
Tom的个数： 1
判断是否含有Tom： True
查询第[1， 3)个元素(切片)： ('Tom', 'Drake')
```

 **注意：元组本身不可变，如果元组中还包含其他可变元素，这些可变元素可以改变**

```
user_names[-1].append("金角大王")
print("元组：", user_names)
```

 **输出**

```
元组： ('James', 'Tom', 'Drake', ['Rain', 'Jack', '金角大王'])
```

 为啥呢？ 因为元组只是存每个元素的内存地址，上面[‘金角大王’, ‘Jack’]这个列表本身的内存地址存在元组里确实不可变，但是这个列表包含的元素的内存地址是存在另外一块空间里的，是可变的。

[![img](E:\Development\Typora\images\20200727162602164.png)](https://img-blog.csdnimg.cn/20200727162602164.png)



#### 字典(Dictionary)

**特性：**

1. key-value结构
2. key必须为不可变数据类型、必须唯一
3. 可存放任意多个value、可修改、可以不唯一
4. 无序
5. 查询速度快，且不受dict的大小影响，至于为何快？我们学完hash再解释

##### 定义

```
# 方式一
print("-----方式一-----")
info_dict_1 = {
    "name": "Alex",
    "position": "CEO",
    "salary": 60000
}
print("类型：", type(info_dict_1))
print("名字：", info_dict_1.get("name"), " 完整信息：", info_dict_1)

# 方式二
print("-----方式二-----")
info_dict_2 = dict(name="Alex", position="CEO", salary=60000)
print("类型：", type(info_dict_2))
print("名字：", info_dict_2.get("name"), " 完整信息：", info_dict_2)

# 方式三
print("-----方式三-----")
info_dict_3 = dict({"name": "Alex", "position": "CEO", "salary": 60000})
print("类型：", type(info_dict_3))
print("名字：", info_dict_3.get("name"), " 完整信息：", info_dict_3)

# 方式四
print("-----方式四-----")
info_dict_4 = {}.fromkeys([1, 2, 3, 4, 5, 6, 7, 8, 9, 10],100)
print("完整信息：", info_dict_4)
```

 输出

```
-----方式一-----
类型： <class 'dict'>
名字： Alex  完整信息： {'name': 'Alex', 'position': 'CEO', 'salary': 60000}
-----方式二-----
类型： <class 'dict'>
名字： Alex  完整信息： {'name': 'Alex', 'position': 'CEO', 'salary': 60000}
-----方式三-----
类型： <class 'dict'>
名字： Alex  完整信息： {'name': 'Alex', 'position': 'CEO', 'salary': 60000}
-----方式四-----
完整信息： {1: 100, 2: 100, 3: 100, 4: 100, 5: 100, 6: 100, 7: 100, 8: 100, 9: 100, 10: 100}
```



##### 查询

```
print("查询长度：", len(info_dict_3))
print("查询名称1：", info_dict_3["name"])
print("查询名称2：", info_dict_3.get("name"))
print("查询名称2(不存在，返回为None)：", info_dict_3.get("area"))
print("查询名称2(不存在，返回指定默认值)：", info_dict_3.get("area", "找不到呀！！！"))
print("字典中是否存在area信息：", "area" in info_dict_3)
print("获取所有的key-value：", info_dict_3.items())
print("获取所有的key：", info_dict_3.keys())
print("获取所有的value：", info_dict_3.values())
```

 输出

```
查询长度： 3
查询名称1： Alex
查询名称2： Alex
查询名称2(不存在，返回为None)： None
查询名称2(不存在，返回指定默认值)： 找不到呀！！！
字典中是否存在area信息： False
获取所有的key-value： dict_items([('name', 'Alex'), ('position', 'CEO'), ('salary', 60000)])
获取所有的key： dict_keys(['name', 'position', 'salary'])
获取所有的value： dict_values(['Alex', 'CEO', 60000])
```



##### 新增

```
# 新增
info_dict_3.setdefault("age", 20)
info_dict_3["area"] = "上海"
print("新增两个键值对后，完整信息：", info_dict_3)
```

 输出

```
新增两个键值对后，完整信息： {'name': 'Alex', 'position': 'CEO', 'salary': 60000, 'age': 20, 'area': '上海'}
```



##### 修改

```
# 修改
info_dict_2["name"] = "Drake"
info_dict_2.update({"salary": 65000, "position": "manager"})
info_dict_2.update(zip(keys, values))  # keys和values都是一个数组
print("修改名称和薪资后，完整信息：", info_dict_2)
```

 输出

```
修改名称和薪资后，完整信息： {'name': 'Drake', 'position': 'manager', 'salary': 65000}
```



##### 删除

```
# 删除最后一个(随机删除,因为字典是无序的)
print("删除最后一个键值对后，完整信息：", info_dict_3, "\t删除的键值对：", info_dict_3.popitem())
# 指定删除1
print("删除'name'的键值对后，完整信息：", info_dict_3, "\t删除的值：", info_dict_3.pop("name"))
# 指定删除2
del info_dict_3["age"]
print("删除'age'的键值对后，完整信息：", info_dict_3)
# 全部删除
info_dict_3.clear()
print("全部删除后，完整信息：", info_dict_3)
```

 输出

```
删除最后一个键值对后，完整信息： {'name': 'Alex', 'position': 'CEO', 'salary': 60000, 'age': 20} 	删除的键值对： ('area', '上海')
删除'name'的键值对后，完整信息： {'position': 'CEO', 'salary': 60000, 'age': 20} 	删除的值： Alex
删除'age'的键值对后，完整信息： {'position': 'CEO', 'salary': 60000}
全部删除后，完整信息： {}
```



##### 循环

```
# 循环
# 推荐，效率速度最快
for key in info_dict_2:
    print(key, "：", info_dict_2[key], end="\t")

print()
for key in info_dict_2.keys():
    print(key, "：", info_dict_2[key], end="\t")

print()
for value in info_dict_2.values():
    print(value, end="\t")

print()
for key, value in info_dict_2.items():
    print(key, "：", value, end="\t")
```

 输出

```
name ： Drake	position ： manager	salary ： 65000	
name ： Drake	position ： manager	salary ： 65000	
Drake	manager	65000	
name ： Drake	position ： manager	salary ： 65000	
```

> 推荐第一种方式，效率速度最快
>
> 第四种有种解构的意味，一开始我还以为跟java一样，害 😂



#### 集合(set)

集合跟我们学的列表有点像，也是可以存一堆数据，不过它有几个独特的特点，令其在整个Python语言中占有一席之地，

1. 里面的元素不可变，代表你不能存一个list、dict 在集合里，字符串、数字、元组等不可变类型可以存
2. 天生去重，在集合里没办法存重复的元素
3. 无序，不像列表一样通过索引来标记在列表中的位置 ，元素是无序的，集合中的元素没有先后之分，如集合{3,4,5}和{3,5,4}算作同一个集合

基于上面的特性，我们可以用集合来干2件事，**去重和关系运算**

##### 去重

```
lans = ["Java", "Vue", "Java", "Vue", "Python"]
print("langs去重：", list(set(lans)))
```

 输出

```
langs去重： ['Java', 'Python', 'Vue']
```



##### 查询

```
print("类型：", type(lan))
print("信息：", lan)
print("长度：", len(lan))
print("是否存在Java：", "Java" in lan)
```

 输出

```
类型： <class 'set'>
信息： {'Java', 'Python', 'Ruby', 'Vue'}
长度： 4
是否存在Java： True
```



##### 新增

```
lan.add("Golang")
print("新增一个数值后：", lan)
lan.add(("Netty", "SpringBoot", "ShardingSphere"))
print("新增一个元组后：", lan)
```

 输出

```
新增一个数值后： {'Java', 'Golang', 'Ruby', 'Python', 'Vue'}
新增一个元组后： {'Java', 'Golang', 'Ruby', 'Python', 'Vue', ('Netty', 'SpringBoot', 'ShardingSphere')}
```



##### 删除

```
lan.discard("Vue")  # 如果这个值不存在，do nothing.
lan.remove("Java")  # 如果这个值不存在，报错
lan.pop()  # 随机删除
print("删除三个元素后：", lan)
lan.clear()
print("全部删除：", lan)
```

 输出

```
删除三个元素后： {'Ruby', 'Python', ('Netty', 'SpringBoot', 'ShardingSphere')}
全部删除： set()
```



##### 修改(想p吃)

##### 关系运算

```
print("lan的值：", lan)
print("lans：", set(lans))

print("lan和lans的交集1：", lan.intersection(set(lans)))
print("lan和lans的交集2：", lan & set(lans))

print("lan和lans的并集1：", lan.union(set(lans)))
print("lan和lans的并集2：", lan | set(lans))

print("lan和lans的差集1(lan-langs)：", lan.difference(set(lans)))
print("lan和lans的差集1(lans-lan)：", set(lans).difference(lan))
print("lan和lans的差集2(lan-langs)：", lan - set(lans))
print("lan和lans的差集2(lans-lan)：", set(lans) - lan)

print("对称差集1(去掉两个集合中都存在的)：", lan.symmetric_difference(set(lans)))
print("对称差集2(去掉两个集合中都存在的)：", lan ^ set(lans))



print("lan和lans是否不相交：", lan.isdisjoint(lans))
print("lan是否包含lans：", lan.issuperset(lans))
print("lans是否包含lan：", lan.issubset(lans))

lan.difference_update(set(lans))
print("lan和lans的差集并更新(lan-langs)：", lan)
lan.intersection_update(set(lans))
lan.symmetric_difference_update(set(lans))
```

 输出

```
lan的值： {('Netty', 'SpringBoot', 'ShardingSphere'), 'Python', 'Ruby'}
lans： {'Python', 'Java', 'Vue'}

lan和lans的交集1： {'Python'}
lan和lans的交集2： {'Python'}

lan和lans的并集1： {'Java', 'Vue', 'Python', ('Netty', 'SpringBoot', 'ShardingSphere'), 'Ruby'}
lan和lans的并集2： {'Java', 'Vue', 'Python', ('Netty', 'SpringBoot', 'ShardingSphere'), 'Ruby'}

lan和lans的差集1(lan-langs)： {('Netty', 'SpringBoot', 'ShardingSphere'), 'Ruby'}
lan和lans的差集1(lans-lan)： {'Vue', 'Java'}
lan和lans的差集2(lan-langs)： {('Netty', 'SpringBoot', 'ShardingSphere'), 'Ruby'}
lan和lans的差集2(lans-lan)： {'Vue', 'Java'}

对称差集1(去掉两个集合中都存在的)： {'Java', 'Vue', 'Ruby', ('Netty', 'SpringBoot', 'ShardingSphere')}
对称差集2(去掉两个集合中都存在的)： {'Java', 'Vue', 'Ruby', ('Netty', 'SpringBoot', 'ShardingSphere')}


lan和lans是否相交： False
lan是否包含lans： False
lans是否包含lan： False

lan和lans的差集并更新(lan-langs)： {'Golang', ('Netty', 'SpringBoot', 'ShardingSphere'), 'Ruby'}
```

### 读取用户指令

```
name = input("请输入您的名字：")
print("名字："+name)
age = input("请输入您的年龄：")
print("年龄：", age)

msg = '''
-----用户 信息------
Name: %s
Age : %d
''' % (name, age)

print(msg)
```

 输出

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16628279372268.png)](https://img-blog.csdnimg.cn/20200726210737600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 **怎么回事？？？**

[![img](E:\Development\Typora\images\2020072621080248.jpg)](https://img-blog.csdnimg.cn/2020072621080248.jpg)

> **注意，input()方法接收的只是字符串，即使你输入的是数字，它也会按字符串处理**

解决方法

```
age = int(input("请输入您的年龄："))
```

> 用一个int包裹起来，应该是进行了一个类型转换，看下面这个例子
>
> ```
> print(type(int("20")))
> ```
>
> **输出结果是：<class ‘int’>**



### 运算符

 计算机可以进行的运算有很多种，可不只加减乘除这么简单，运算按种类可分为算数运算、比较运算、逻辑运算、赋值运算、成员运算、身份运算、位运算，今天我们暂只学习算数运算、比较运算、逻辑运算、赋值运算

#### 算数运算

 以下假设变量：**a=10，b=20**

| 运算符 | 描述                                            | 实例                                               |
| ------ | ----------------------------------------------- | -------------------------------------------------- |
| +      | 加 - 两个对象相加                               | a + b 输出结果 30                                  |
| -      | 减 - 得到负数或是一个数减去另一个数             | a - b 输出结果 -10                                 |
| *      | 乘 - 两个数相乘或是返回一个被重复若干次的字符串 | a * b 输出结果 200                                 |
| /      | 除 - x除以y                                     | b / a 输出结果 2                                   |
| %      | 取模 - 返回除法的余数                           | b % a 输出结果 0                                   |
| **     | 幂 - 返回x的y次幂                               | a**b 为10的20次方， 输出结果 100000000000000000000 |
| //     | 取整除 - 返回商的整数部分（**向下取整**）       | 9//2 输出结果为4，9.0//2.0输出结果为4.0            |



#### 比较运算

 以下假设变量：**a=10，b=20**

| 运算符 | 描述                                                         | 实例                                     |
| ------ | ------------------------------------------------------------ | ---------------------------------------- |
| ==     | 等于 - 比较对象是否相等                                      | (a == b) 返回 False。                    |
| !=     | 不等于 - 比较两个对象是否不相等                              | (a != b) 返回 true.                      |
| <>     | 不等于 - 比较两个对象是否不相等。python3 已废弃。            | (a <> b) 返回 true。这个运算符类似 != 。 |
| >      | 大于 - 返回x是否大于y                                        | (a > b) 返回 False。                     |
| <      | 小于 - 返回x是否小于y。所有比较运算符返回1表示真，返回0表示假。这分别与特殊的变量True和False等价。 | (a < b) 返回 true。                      |
| >=     | 大于等于 - 返回x是否大于等于y。                              | (a >= b) 返回 False。                    |
| <=     | 小于等于 - 返回x是否小于等于y。                              | (a <= b) 返回 true。                     |



#### 赋值运算

 以下假设变量：**a=10，b=20**

| 运算符 | 描述             | 实例                                  |
| ------ | ---------------- | ------------------------------------- |
| =      | 简单的赋值运算符 | c = a + b 将 a + b 的运算结果赋值为 c |
| +=     | 加法赋值运算符   | c += a 等效于 c = c + a               |
| -=     | 减法赋值运算符   | c -= a 等效于 c = c - a               |
| *=     | 乘法赋值运算符   | c *= a 等效于 c = c * a               |
| /=     | 除法赋值运算符   | c /= a 等效于 c = c / a               |
| %=     | 取模赋值运算符   | c %= a 等效于 c = c % a               |
| **=    | 幂赋值运算符     | c **= a 等效于 c = c ** a             |
| //=    | 取整除赋值运算符 | c //= a 等效于 c = c // a             |



#### 位运算符

 以下假设变量：**a 为 60(0011 1100)，b 为 13(0000 1101)**

| 运算符 | 描述                                                         | 实例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| &      | 按位与运算符：参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0 | (a & b) 输出结果 12 ，二进制解释： 0000 1100                 |
| \|     | 按位或运算符：只要对应的二个二进位有一个为1时，结果位就为1。 | (a \| b) 输出结果 61 ，二进制解释： 0011 1101                |
| ^      | 按位异或运算符：当两对应的二进位相异时，结果为1              | (a ^ b) 输出结果 49 ，二进制解释： 0011 0001                 |
| ~      | 按位取反运算符：对数据的每个二进制位取反,即把1变为0,把0变为1 。~x 类似于 -x-1 | (~a ) 输出结果 -61 ，二进制解释： 1100 0011，在一个有符号二进制数的补码形式。 |
| <<     | 左移动运算符：运算数的各二进位全部左移若干位，由 << 右边的数字指定了移动的位数，高位丢弃，低位补0。 | a << 2 输出结果 240 ，二进制解释： 1111 0000                 |
| >>     | 右移动运算符：把">>"左边的运算数的各二进位全部右移若干位，>> 右边的数字指定了移动的位数 | a >> 2 输出结果 15 ，二进制解释： 0000 1111                  |



#### 逻辑运算

| 运算符 | 逻辑表达式 | 描述                                                         | 实例                    |
| ------ | ---------- | ------------------------------------------------------------ | ----------------------- |
| and    | x and y    | 布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。 | (a and b) 返回 20。     |
| or     | x or y     | 布尔"或" - 如果 x 是非 0，它返回 x 的值，否则它返回 y 的计算值。 | (a or b) 返回 10。      |
| not    | not x      | 布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。 | not(a and b) 返回 False |



#### 身份运算和None

 python 中有很多种数据类型， 查看一个数据的类型的方法是type().

 判断一个数据类型是不是str, or int等，可以用身份运算符is

| 运算符 | 描述                                        | 实例                                                         |
| :----- | :------------------------------------------ | :----------------------------------------------------------- |
| is     | is 是判断两个标识符是不是引用自一个对象     | **x is y**, 类似 **id(x) == id(y)** , 如果引用的是同一个对象则返回 True，否则返回 False |
| is not | is not 是判断两个标识符是不是引用自不同对象 | **x is not y** ， 类似 **id(a) != id(b)**。如果引用的不是同一个对象则返回结果 True，否则返回 False。 |

```
print("-----is--is not-----")
age = 2
print(type(age))  # <class 'int'>
print(type(age) is int)  # True
print(type(age) is not int)  # False
print(not type(age) is int)  # False
print(type(age) == int)  # True
print(type(age) != int)  # False

print("-----None-----")
name = None
print(type(name))   # <class 'NoneType'>
print(name is None)  # True
print(name is not None)  # False
```

 输出

```
-----is--is not-----
<class 'int'>
True
False
False
True
False
-----None-----
<class 'NoneType'>
True
False
```



#### 三元运算符

```
print("age大于3" if age > 3 else "age小于3")
```

 输出

```
age小于3
```

### 流程控制if-else

 假如把写程序比做走路，那我们到现在为止，一直走的都是直路，还没遇到过分叉口，想象现实中，你遇到了分叉口，然后你决定往哪拐必然是有所动机的。你要判断那条岔路是你真正要走的路，如果我们想让程序也能处理这样的判断怎么办？ 很简单，只需要在程序里预设一些条件判断语句，满足哪个条件，就走哪条岔路。这个过程就叫流程控制。

 基本上在各个语言中，都是用语法if…else…来实现，可分为单分支、双分支、多分支

```
age = 49
if age > 50:
    print("年龄大于50")
elif age == 50:
    print("年龄等于50")
else:
    print("年龄小于50")
```

 你会发现，上面的if代码里，每个条件的下一行都缩进了4个空格，这是为什么呢？这就是Python的一大特色，强制缩进，目的是为了让程序知道，每段代码依赖哪个条件，如果不通过缩进来区分，程序怎么会知道，当你的条件成立后，去执行哪些代码呢？

在其它的语言里，大多通过`{}`来确定代码块，在有`{}`来区分代码块的情况下，缩进的作用就只剩下让代码变的整洁了。

 Python是门超级简洁的语言，发明者定是觉得用`{}`太丑了，所以索性直接不用它，那怎么能区分代码块呢？答案就是**强制缩进**。

> **Python的缩进有以下几个原则:**
>
> - 顶级代码必须顶行写，即如果一行代码本身不依赖于任何条件，那它必须不能进行任何缩进
> - 同一级别的代码，缩进必须一致
> - 官方建议缩进用4个空格，当然你也可以用2个，如果你想被人笑话的话。



### 流程控制之while循环

- break用于完全结束一个循环，跳出循环体执行循环后面的语句

- continue和break有点类似，区别在于continue只是终止本次循环，接着还执行后面的循环，break则完全终止循环

- 与其它语言else 一般只与if 搭配不同，在Python 中还有个while …else 语句

  while 后面的else 作用是指，**当while 循环正常执行完，中间没有被break 中止的话，就会执行else后面的语句**

```
import random

count = 1
age = None
while count <= 10 and (age is None or age < 100):
    print("------第", count, "次------")
    age = int(input("请输入您的年龄："))
    ran = random.uniform(0, 2)
    if age*ran > 100:
        print("您的年龄：", age, ",随机数为", ran, "age×ran=", age*ran, ">100，退出程序")
        break
    if age == 0:
        print("您的年龄为0，请重新输入")
        continue
    elif age > 50:
        print("您的年龄：", age, ",年龄大于50,随机数为", ran, "age×ran=", age*ran)
    elif age == 50:
        print("您的年龄：", age, "年龄等于50,随机数为", ran, "age×ran=", age*ran)
    else:
        print("您的年龄：", age, "年龄小于50,随机数为", ran, "age×ran=", age*ran)
    count += 1

else:
    if count > 10:
        print("执行了10次，退出")
    else:
        print("出现了不符合规则的情况，退出")

print("------out of while loop ------")
```



 输出

```
------第 1 次------
请输入您的年龄：0
您的年龄为0，请重新输入
------第 1 次------
请输入您的年龄：11
您的年龄： 11 年龄小于50,随机数为 1.9762225675476903 age×ran= 21.738448243024592
------第 2 次------
请输入您的年龄：50
您的年龄： 50 年龄等于50,随机数为 1.113568284883348 age×ran= 55.6784142441674
------第 3 次------
请输入您的年龄：60
您的年龄： 60 ,年龄大于50,随机数为 1.3357098610902078 age×ran= 80.14259166541247
------第 4 次------
请输入您的年龄：100
您的年龄： 100 ,随机数为 1.944775672644064 age×ran= 194.4775672644064 >100，退出程序
------out of while loop ------
```



```
...省略之前的9次
------第 10 次------
请输入您的年龄：10
您的年龄： 10 年龄小于50,随机数为 0.20069877928471436 age×ran= 2.0069877928471436
执行了10次，退出
------out of while loop ------
```



 从输出中可以看出，当break跳出循环时，while的else语句时不会执行的！！！

 而当循环不满足while条件时，才会执行我们while相应的else语句！！！

> random的使用，请点击 👉 [random](https://blog.codekiller.top/2021/02/03/Python基础/#random)



## 相关提高

### 变量的创建和修改(内存)

#### 例子

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author     : codekiller
# @Time       : 2020/7/27 0:54
# @Email      : jcl1345414527@163.com
# @File       : variable2.py
# @Description: 变量的创建过程和修改

name = "oldboy"
print("name(oldboy)的内存地址：", id(name)) #name(oldboy)的内存地址： 3018749901168

name = "alex"
print("name(alex)的内存地址：", id(name)) #name(alex)的内存地址： 3018749903152

name = "oldboy"
print("name(oldboy)的内存地址：", id(name)) #name(oldboy)的内存地址： 3018749901168

name_copy = name
print("name_copy(name)的内存地址：", id(name_copy)) #name_copy(name)的内存地址： 3018749901168

name = "alex"
# 修改name的值为alex后，name的值： alex  name_copy的值： oldboy
print("修改name的值为alex后，name的值：", name, " name_copy的值：", name_copy) 

age = 1
print("age(1)的内存地址：", id(age)) #age(1)的内存地址： 140704774887072

age = 2
print("age(2)的内存地址：", id(age)) #age(2)的内存地址： 140704774887104

age = 1
print("age(1)的内存地址：", id(age)) #age(1)的内存地址： 140704774887072

#省略了浮点数，布尔类型和数组
```



 输出

```
name(oldboy)的内存地址： 3018749901168
name(alex)的内存地址： 3018749903152
name(oldboy)的内存地址： 3018749901168
name_copy(name)的内存地址： 3018749901168
修改name的值为alex后，name的值： alex  name_copy的值： oldboy
age(1)的内存地址： 140704774887072
age(2)的内存地址： 140704774887104
age(1)的内存地址： 140704774887072
```

> 我这里试了一下字符串，整数，浮点数，布尔类型(True/False)，发现只要值是是一样的，那么指向的内存地址就是一样的
>
> 并且数组一旦创建，不管什么操作，指向的内存地址也是不变的！



#### 分析

##### 变量创建

 首先，当我们定义了一个变量name = ‘oldboy’的时候，在内存中其实是做了这样一件事：

 程序开辟了一块内存空间，将‘oldboy’存储进去，再让变量名name指向‘oldboy’所在的内存地址。如下图所示：

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621317.png)](https://img-blog.csdnimg.cn/20200727010620203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 我们可以通过id()方法查看这个变量在内存中的地址



##### 变量修改

 从上面的例子中，我们发现修改一次后，内存地址会发生改变，但当我们再次改回来的时候，内存地址其实还是第一次的地址，为什么？？？

```
name = "oldboy"
print("name(oldboy)的内存地址：", id(name)) #name(oldboy)的内存地址： 3018749901168

name = "alex"
print("name(alex)的内存地址：", id(name)) #name(alex)的内存地址： 3018749903152

name = "oldboy"
print("name(oldboy)的内存地址：", id(name)) #name(oldboy)的内存地址： 3018749901168
```

 实际的原理什么样的呢？ 程序先申请了一块内存空间来存储‘oldboy’，让name变量名指向这块内存空间

 执行到name=‘alex’之后又申请了另一块内存空间来存储‘alex’，并让原本指向‘oldboy’内存的链接断开，让name再指向‘alex’。所以原来的变量 “oldboy” 其实还在内存中，我们声明的变量仅仅是改变了一个内存地址引用！

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621318.png)](https://img-blog.csdnimg.cn/20200727011038838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



 上面的例子中，还有一个注意点，当一个变量(name)赋值给另一个变量(name_copy)时，其实是`将name的内存地址赋给了name_copy`，并不是直接指向name的，所以当name发生改变时，name_copy的值是不会发生改变的！

```
name = "alex"
# 修改name的值为alex后，name的值： alex  name_copy的值： oldboy
print("修改name的值为alex后，name的值：", name, " name_copy的值：", name_copy) 
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621319.png)](https://img-blog.csdnimg.cn/20200727011612216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 字符编码之文字是如何显示的

#### 介绍

 计算机只认识二进制，生活中的数字要想让计算机理解就必须转换成二进制。十进制到二进制的转换只能解决计算机理解数字的问题，那么文字要怎么让计算机理解呢？

 于是我们就选择了一种曲线救国的方式，既然数字可以转换成十进制，我们只要想办法把文字转换成数字，这样文字不就可以表示成二进制了么？

[![img](E:\Development\Typora\images\20200727231019573.png)](https://img-blog.csdnimg.cn/20200727231019573.png)

 可是文字应该怎么转换成数字呢？就是强制转换啊，简单粗暴呀。 我们自己强行约定了一个表，把文字和数字对应上，这张表就相当于翻译，我们可以拿着一个数字来对比对应表找到相应的文字，反之亦然。

#### ASCII码

 这张表就是计算机显示各种文字、符号的基石呀

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621420.png)](https://img-blog.csdnimg.cn/20200727230950682.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于[拉丁字母](http://baike.baidu.com/item/拉丁字母)的一套电脑编码系统，主要用于显示现代[英语](http://baike.baidu.com/item/英语/109997)和其他[西欧](http://baike.baidu.com/item/西欧)语言。它是现今最通用的单字节编码系统，并等同于[国际](http://baike.baidu.com/item/国际)标准ISO/IEC 646。

 由于计算机是美国人发明的，因此，最早只有127个字母被编码到计算机里，也就是大小写英文字母、数字和一些符号，这个编码表被称为`ASCII`编码，比如大写字母 `A`的编码是`65`，小写字母 `z`的编码是`122`。后128个称为[扩展ASCII](http://baike.baidu.com/item/扩展ASCII)码。

 那现在我们就知道了上面的字母符号和数字对应的表是早就存在的。那么根据现在有的一些十进制，我们就可以转换成二进制的编码串。

比如

> 一个空格对应的数字是0 翻译成二进制就是0（注意字符’0’和整数0是不同的）
>
> 一个对勾√对应的数字是251 翻译成二进制就是11111011

论断句的重要性与必要性：

上次在网上看到个新闻，讲是个小偷在上海被捕时高喊道：“我一定要当上海贼王！”

正是由于这些字符串长的长，短的短，写在一起让我们难以分清每一个字符的起止位置，所以聪明的人类就想出了一个解决办法，既然一共就这255个字符，那最长的也不过是11111111八位，不如我们就把所有的二进制都转换成8位的，不足的用0来替换。

这样一来，刚刚的两个空格一个对勾就写作000000000000000011111011，读取的时候只要每次读8个字符就能知道每个字符的二进制值啦。

在这里，每一位0或者1所占的空间单位为bit(比特)，这是计算机中最小的表示单位

**每8个bit组成一个字节，这是计算机中最小的存储单位(毕竟你是没有办法存储半个字符的)orz～**

> 1. `bit 位，计算机中最小的表示单位`
> 2. `8bit = 1bytes 字节，最小的存储单位，1bytes缩写为1B`
> 3. `1KB=1024B`
> 4. `1MB=1024KB`
> 5. `1GB=1024MB`
> 6. `1TB=1024GB`
> 7. `1PB=1024TB`
> 8. `1EB=1024PB`
> 9. `1ZB=1024EB`
> 10. `1YB=1024ZB`
> 11. `1BB=1024YB`



#### GB2312 & GBK

 英文问题是解决了， 我们中文如何显示呢？ 美国佬设计ASSCII码的时候应该是没考虑中国人有一天也能用上电脑， 所以根本没考虑中文的问题，上世界80年代，电脑进入中国，把砖家们难倒了，妈的你个一ASSCII只能存256个字符，我常用汉字就几千个，怎么玩？？？勒紧裤腰带还苏联贷款的时候我们都挺过来啦，这点小事难不到我们， 既然美帝的ASCII不支持中文，那我们自己搞张编码表不就行了， 于是我们设计出了GB2312编码表，长成下面的样子。一共存了6763个汉字。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621421.png)](https://img-blog.csdnimg.cn/2020072723163298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 这个表格比较大，像上面的一块块的文字区域有72个，这导致通过一个字节是没办法表示一个汉字的(因为一个字节最多允许256个字符变种，你现在6千多个，只能2个字节啦，2**16=65535个变种)。

 有了gb2312，我们就能愉快的写中文啦。

 但我们写字竟然会出现中英混杂的情况，比如“我是小猿圈，我的英文名叫Apeland.”， 这种你怎么办？这就要求你必须在gb2312里同时支持英文，但是还不能是2个字节表示一个英文字母。人家ASCII用一个字符，你用2个，那一个2mb大小的英文文档只要一改编码，就立刻变成4mb, 太坑爹，中国人你有钱也不能这么造呀。 所以中国砖家们又通过神奇手段兼容了ASSCII, 即遇到中文用2个字节，遇到英文直接用ASCII的编码。怎么做到的呢？

 如何区别连在一起的2个字节是代表2个英文字母，还是一个中文汉字呢？ 中国人如此聪明，决定，**如果2个字节连在一起，且每个字节的第1位(也就是相当于128的那个2进制位)如果是1，就代表这是个中文，这个首位是128的字节被称为高字节。 也就是2个高字节连在一起，必然就是一个中文。** 你怎么如此笃定？因为0-127已经表示了英文的绝大部分字符，128-255是ASCII的扩展表，表示的都是极特殊的字符，一般没什么用。所以中国人就直接拿来用了。

 自1980年发布gb2312之后，中文一直用着没啥问题，随着个人电脑进入千家万户，有人发现，自己的名字竟然打印不出来，因为起的太生僻了。

 于是1995年， 砖家们又升级了gb2312, 加入更多字符，连什么藏语、维吾尔语、日语、韩语、蒙古语什么的统统都包含进去了，国家统一亚洲的野心从这些基础工作中就可见一斑哈。 这个编码叫GBK，一直到现在，我们的windows电脑中文版本的编码就是GBK.



#### 编码混战时代

 中国人在搞自己编码的同时，世界上其它非英语国家也得用电脑呀，于是都搞出了自己的编码，你可以想得到的是，全世界有上百种语言，日本把日文编到**Shift_JIS**里，韩国把韩文编到**Euc-kr**里，

 各国有各国的标准，就会不可避免地出现冲突，结果就是，在多语言混合的文本中，显示出来会有乱码。之前你从玩个日本游戏，往自己电脑上一装，就显示乱码了。**需要安装语言包，才能正常显示文字**。

 这么乱极大了阻碍了不同国家的信息传递，于是联合国出面，发誓要解决这个混乱局面。

 因此，**Unicode**应运而生。**Unicode**把所有语言都统一到一套编码里，这样就不会再有乱码问题了。Unicode 2-4字节 已经收录136690个字符，并还在一直不断扩张中…

 **Unicode**标准也在不断发展，但最常用的是用两个字节表示一个字符（如果要用到非常偏僻的字符，就需要**4**个字节）。现代[操作系统](http://lib.csdn.net/base/operatingsystem)和大多数编程语言都直接支持**Unicode**。

 Unicode有2个特点：

1. 支持全球所有语言
2. 可以跟各种语言的编码自由转换，也就是说，即使你gbk编码的文字 ，想转成unicode很容易。

 为何unicode可以跟其它语言互相转换呢？ 因为有跟所有语言都有对应关系哈，这样做的好处是可以让那些已经用gbk或其它编码写好的软件容易的转成unicode编码 ，利于unicode的推广。 下图就是unicode跟中文编码的对应关系

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621422.png)](https://img-blog.csdnimg.cn/20200727232619858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### UTF-8

 新的问题又出现了：如果统一成**Unicode**编码，乱码问题从此消失了。但是，如果你写的文本基本上全部是英文的话，用**Unicode**编码比**ASCII**编码需要多一倍的存储空间，由于计算机的内存比较大，并且字符串在内容中表示时也不会特别大，所以内容可以使用unicode来处理，但是存储和网络传输时一般数据都会非常多，那么增加1倍将是无法容忍的！！！

 为了解决存储和网络传输的问题，出现了Unicode Transformation Format，学术名UTF，即：对unicode字符进行转换，以便于在存储和网络传输时可以节省空间!

- UTF-8： 使用1、2、3、4个字节表示所有字符；优先使用1个字符、无法满足则使增加一个字节，最多4个字节。英文占1个字节、欧洲语系占2个、东亚占3个，其它及特殊字符占4个
- UTF-16： 使用2、4个字节表示所有字符；优先使用2个字节，否则使用4个字节表示。
- UTF-32： 使用4个字节表示所有字符；

 总结：**UTF 是为unicode编码 设计 的一种 在存储 和传输时节省空间的编码方案。**

 如果你要传输的文本包含大量英文字符，用**UTF-8**编码就能节省空间：



### 什么是哈希？

#### 介绍

 hash,一般翻译做散列、杂凑，或音译为哈希，是把任意长度的[输入](https://baike.baidu.com/item/输入/5481954)（又叫做预映射pre-image）通过散列算法变换成固定长度的[输出](https://baike.baidu.com/item/输出/11056752)，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间。

 它其实就是一个算法，最简单的算法就是加减乘除，比方，我设计个数字算法，输入+7=输出，比如我输入1，输出为8；输入2，输出为9。

 哈希算法不过是一个更为复杂的运算，它的输入可以是字符串，可以是数据，可以是任何文件，经过哈希运算后，变成一个固定长度的输出，该输出就是哈希值。**但是哈希算法有一个很大的特点，就是你不能从结果推算出输入,所以又称为不可逆的算法**

```
print(hash("冲呀"))  # -1373361932094575644
print(hash("莫崽子"))  # 2455353328478895534
```

 如上所示，输入“冲呀”三个字，经过哈希运算后，会得到一个随机数列，而且不管你的输入文件多大，最后得到的结果都是这么一个固定长度的数列，即使你输入的是一部电影，输出也是这么大。而且通过数列不能推导出输入。



#### 哈希特性

 **不可逆**：在具备编码功能的同时，哈希算法也作为一种加密算法存在。即，你无法通过分析哈希值计算出源文件的样子，换句话说：你不可能通过观察香肠的纹理推测出猪原来的样子。

 **计算极快**：20G高清电影和一个5K文本文件复杂度相同，计算量都极小，可以在0.1秒内得出结果。也就是说，不管猪有多肥，骨头多硬，做成香肠都只要眨眨眼的时间，



#### 哈希的用途

哈希算法的不可逆特性使其在以下领域使用广泛

1. 密码，我们日常使用的各种电子密码本质上都是基于hash的，你不用担心支付宝的工作人员会把你的密码泄漏给第三方，因为你的登录密码是先经过 hash+各种复杂算法得出密文后 再存进支付宝的数据库里的
2. 文件完整性校验，通过对文件进行hash，得出一段hash值 ，这样文件内容以后被修改了，hash值就会变。 MD5 Hash算法的”数字指纹”特性，使它成为应用最广泛的一种文件完整性[校验和](https://baike.baidu.com/item/校验和)(Checksum)算法，不少Unix系统有提供计算md5 checksum的命令。
3. 数字签名，[数字签名技术](https://baike.baidu.com/item/数字签名技术)是将摘要信息用发送者的私钥加密，与[原文](https://baike.baidu.com/item/原文)一起传送给接收者。接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用[HASH函数](https://baike.baidu.com/item/HASH函数)对收到的[原文](https://baike.baidu.com/item/原文)产生一个摘要信息，与解密的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过，因此数字签名能够验证信息的完整性。

此外，hash算法在区块链领域也使用广泛。

#### 基于hash的数据类型有哪些？

Python 中基于hash的2个数据类型是`dict` and `set` , 之前说dict查询速度快，为何快？ 说set天生去重，怎么做到的？其实都是利用了hash的特性，我们下面来剖析

#### dict 为何查询速度超快，且不受dict大小影响 ？

 解析：假设我要存14亿人的基本信息

```
data = {
    "张三":[23742364782642342323234,28,"山东济南"],
    "李四":[12124234232311214458271,25,"北京昌平"],
    "王五":[23030293483727384383929,33,"山东济南"],
    "赵六":[42302033030302482634674,28,"河北保定"],
    # "alex":["xxxx"],
    # "黑姑娘":["xxxx"]
    # ...
}
```

 dict 的每个key 都要先经过hash生成一段固定长度的hash值，假设生成的hash值如下

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621423.png)](https://img-blog.csdnimg.cn/20200728000912425.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 dict会把这些数字`按大小排序`好放在一个列表里kd = [-10, 53, 67, 81, 99, 123]

 当我们想查找”赵六”的信息时， 会把“赵六”先hash, 得到99这个值，然后拿这个值去到kd列表里找，想象这个列表有14亿个值 ，如何快速找到99？ 二分法就行，具体看剖析视频。

 只要找到了99的位置，就可以定位到赵六对应的value的值了。 通过2分法查找，每次数据量都会少一半，这样查找最多31次(2**31=2147483648)就能从20亿信息里找到这个人的信息。

 当然 dict 真实的查找算法比这个还要复杂些， 我只是通过这个例子让大家理解下为何基于hash的数据类型查找速度会快很多。



#### set为何是天生去重的？

 因为每存一个值到set里时， 都要先经过hash，然后通过得出的这个hash值算出应该存在set里的哪个位置，存的时候会先检查那个位置上有没有值 ，有的话就对比是否相等，如果相等，则不再存储此值。 如果不相等(即为空)，则把新值 存在这。



### 深浅拷贝

#### 浅拷贝

```
data = {
    "name": "alex",
    "age": 18,
    "scores": {
        "语文": 130,
        "数学": 60,
        "英语": 98,
    }
}

# 浅拷贝
data_copy = data.copy()

# 再看一下各自的内存地址,可以发现指向的内存地址不一样
print("data的内存地址：", id(data), "\tdata_copy的内存地址：", id(data_copy))
print("data的age的内存地址：", id(data["age"]), "\tdata_copy的age的内存地址：", id(data_copy["age"]))
print("data的scores的内存地址：", id(data["scores"]), "\tdata_copy的scores的内存地址：", id(data_copy["scores"]))
print()

# 改变一下data_copy的age值
data_copy["age"] = 20
data_copy["scores"]["语文"] = 140
# 查看各自的数据,可以发现age的值只有data_copy发生改变，字典都发生改变了，因为修改字典内部并不能改变内存的引用地址
print("修改后data数据：%s\ndata_copy数据：%s" % (data, data_copy))
print("修改后data的age的内存地址：", id(data["age"]), "\t修改后data_copy的age的内存地址：", id(data_copy["age"]))
print("修改后data的scores的内存地址：", id(data["scores"]), "\tdata_copy的scores的内存地址：", id(data_copy["scores"]))
```

 输出

```
data的内存地址： 2238712599232 	data_copy的内存地址： 2238712642176
data的age的内存地址： 140705936513216 	data_copy的age的内存地址： 140705936513216
data的scores的内存地址： 2238712599168 	data_copy的scores的内存地址： 2238712599168

修改后data数据：{'name': 'alex', 'age': 18, 'scores': {'语文': 140, '数学': 60, '英语': 98}}
data_copy数据：{'name': 'alex', 'age': 20, 'scores': {'语文': 140, '数学': 60, '英语': 98}}
修改后data的age的内存地址： 140705936513216 	修改后data_copy的age的内存地址： 140705936513280
修改后data的scores的内存地址： 2238712599168 	data_copy的scores的内存地址： 2238712599168
```

 看输出 ， 很神奇，两个Dict里age的值是独立的，但score字典里的分数值貌似是共享的

 **因为浅copy会仅复制dict的第一层数据，更深层的scores下面的值依然是共享一份**。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282932621424.png)](https://img-blog.csdnimg.cn/20200728110358108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 注意图中2个dict中的name都是alex,内存地址也一样，在没改前，两个name都确实指向同一个内存地址，但只要改任何一个的值，内存地址都会变更， 如age这个key一样。



#### 深拷贝

 若你想彻底使上面的2个dict完全独立，无论有多少层数据。那就要用python工具包里的一个工具了，

```
import copy

data = {
    "name": "alex",
    "age": 18,
    "scores": {
        "语文": 130,
        "数学": 60,
        "英语": 98,
    }
}

data_copy = copy.deepcopy(data)

# 再看一下各自的内存地址,可以发现指向的内存地址不一样
print("data的内存地址：", id(data), "\ndata_copy的内存地址：", id(data_copy))
print("data的age的内存地址：", id(data["age"]), "\tdata_copy的age的内存地址：", id(data_copy["age"]))
print("data的scores-的内存地址：", id(data["scores"]), "\tdata_copy的scores的内存地址：", id(data_copy["scores"]))
print()

# 改变一下data_copy的age值
data_copy["age"] = 20
data_copy["scores"]["语文"] = 150
# 查看各自的数据,可以发现age和字典的值只有data_copy发生改变，因为是深拷贝，改变字典我的值，相应字典的内存引用也发生改变
print("修改后data数据：%s\n修改后data_copy数据：%s" % (data, data_copy))
print("修改后data的age的内存地址：", id(data["age"]), "\tdata_copy的age的内存地址：", id(data_copy["age"]))
print("修改后data的scores的内存地址：", id(data["scores"]), "\tdata_copy的scores的内存地址：", id(data_copy["scores"]))
```

 输出

```
data的内存地址： 2388009767616 
data_copy的内存地址： 2388041388608
data的age的内存地址： 140705936513216 	data_copy的age的内存地址： 140705936513216
data的scores的内存地址： 2388009767552 	data_copy的scores的内存地址： 2388041388352

修改后data数据：{'name': 'alex', 'age': 18, 'scores': {'语文': 140, '数学': 60, '英语': 98}}
修改后data_copy数据：{'name': 'alex', 'age': 20, 'scores': {'语文': 150, '数学': 60, '英语': 98}}
修改后data的age的内存地址： 140705936513216 	data_copy的age的内存地址： 140705936513280
修改后data的scores的内存地址： 2388009767552 	data_copy的scores的内存地址： 2388041388352
```

 最后，这东西有什么用呢？ 坦白讲，以后开发中多数情况下你用不到，但是你有要知道有这个知识点，说不定哪天有个需求就要求你必须确保你的2个复制出来的dict,list必须是独立的了。



### 编码的转换

 在py3里，**内存里的字符串是以unicode编码的**，unicode的其中一个特性就是跟所有语言编码都有映射关系。所以你的utf-8格式的文件，在windows电脑上若是不能看，就可以把utf-8先解码成unicode,再由unicode编码成gbk就可以了。

[![img](E:\Development\Typora\images\20200728113824472.png)](https://img-blog.csdnimg.cn/20200728113824472.png)

 注意，不管在Windows or Mac or Linux上，你的pycharm IDE都可以支持各种文件编码，所以即使是utf-8的文件，在windows下的pycharm里也可以正常显示

```
name = "张龙子"
name_GBK = name.encode("GBK")
print("编码格式：", chardet.detect(name_GBK)["encoding"], "数据：", name_GBK, "数据：", name_GBK.decode("GBK"))
name_UTF_8 = name_GBK.decode("GBK").encode("UTF-8")
print(name_UTF_8)
```

 输出

```
编码格式： KOI8-R 数据： b'\xd5\xc5\xc1\xfa\xd7\xd3' 数据： 张龙子
b'\xe5\xbc\xa0\xe9\xbe\x99\xe5\xad\x90'
```



## IO

### 介绍

 用word操作一个文件的流程如下：

1. 找到文件，双击打开
2. 读或修改
3. 保存&关闭

 用python操作文件也差不多：

```
f=open(filename)  # 打开文件
f.write("我是野生程序员") # 写操作
f.read()  #读操作
f.close() #保存并关闭
```



### 操作模式

- r 只读模式
- w 创建模式，若文件已存在，则覆盖旧文件
- a 追加模式，新数据会写到文件末尾



### 创建模式

```
# 新增文件
file = open(file="F:\\Animation\\0.txt", mode="w")
file.write("""开始
    你好呀！
    快来吧!!
结束""")
file.flush()
file.close()
```



### 追加模式

```
# 追加文件
file = open(file="F:\\Animation\\1.txt", mode="a")
file.write("增加一行\n")
file.close()
```



### 只读模式

```
import chardet

# 只读文件
file = open(file="F:\\Animation\\0.txt", mode="r")

print("是否可读：", file.readable())
print("是否可写：", file.writable())

file.seek(2, 0)  # 把操作文件的光标移到指定位置,从0开始
print("此时位置：", file.tell(), file.readline(1))  # 数字为限制读取该行几个字符，默认为-1，即为全部

file.seek(2)
print("此时位置：", file.tell(), file.readlines(2))  # 数字为限制读取到 当前位置行的剩余数据~第hint个字符的那一行所有数据，默认为-1，即全部

info = file.read(6)
print(info)  # 数字为读取几个字符，默认为-1，即全部字符
print("编码格式：", chardet.detect(info.encode()).get("encoding"))

file.close()
```



 输出

```
是否可读： True
是否可写： False
此时位置： 2 始
此时位置： 2 ['始\n', '    你好呀！\n']
    快来
编码格式： utf-8
```

> 编码格式要安装 `pip install chardet`
>
> 我发现我的编译器和文件设置的都是utf-8编码，但是文件存储的还是GBK格式，因为win上面默认的文件编码格式就是GBK，linux和mac上面默认是UTF-8
>
> 可以通过设置encoding来改变格式
>
> ```
> file = open(file="F:\\Animation\\4.txt", mode="r",  encoding="utf-8")
> ```
>
> 至于为什么编码格式输出为utf-8，因为编译器自动帮我们将GBK转为了utf-8



### 循环

 文件数据

```
马纤羽     深圳    173    50    13744234523
乔亦菲     广州    172    52    15823423525
罗梦竹     北京    175    49    18623423421
刘诺涵     北京    170    48    18623423765
岳妮妮     深圳    177    54    18835324553
贺婉萱     深圳    174    52    18933434452
叶梓萱     上海    171    49    18042432324
```

 循环读取

```
# 循环
file = open(file="F:\\Animation\\2.txt", mode="r")
for line in file:  # line <class 'str'>  叶梓萱     上海    171    49    18042432324
    line = line.split()  # line <class 'list'>  ['贺婉萱', '深圳', '174', '52', '18933434452']
    name, address, height, weight, phone = line
    height = float(height)
    weight = float(weight)
    if height > 170 and weight <= 50:
        print(line, name, "--", address, "--", height, "--", weight, "--", phone)
file.close()
```

 输出

```
['马纤羽', '深圳', '173', '50', '13744234523'] 马纤羽 -- 深圳 -- 173.0 -- 50.0 -- 13744234523
['罗梦竹', '北京', '175', '49', '18623423421'] 罗梦竹 -- 北京 -- 175.0 -- 49.0 -- 18623423421
['叶梓萱', '上海', '171', '49', '18042432324'] 叶梓萱 -- 上海 -- 171.0 -- 49.0 -- 18042432324
```



### 截断

```
file = open(file="F:\\Animation\\0.txt", mode="a")
print("截断所有：", file.truncate())  # 截断所有，即不变
print("截断指定：", file.truncate(4))  # 截断到第4个字节,值剩下两个字符(中文)：开始
file.close()
```

 输出

```
截断所有： 38
截断指定： 4
```



### 修改(注意覆盖问题)

```
file = open("F:\\Animation\\3.txt", "r+")
file.seek(7)
file.write("二柱")  
file.close()
```

 结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720245.png)](https://img-blog.csdnimg.cn/20200728084125767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```
file = open("F:\\Animation\\3.txt", "r+")
file.seek(7)
file.write("二柱子")
file.close()
```

 结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720246.png)](https://img-blog.csdnimg.cn/20200728084223363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 没错，进行了一个覆盖，那么，**为什么原有数据会被覆盖呢？**

 这是硬盘的存储原理导致的，当你把文件存到硬盘上，就在硬盘上划了一块空间，存数据，等你下次打开这个文件 ，seek到一个位置，每改一个字，就是把原来的覆盖掉，如果要插入，是不可能的，因为后面的数据在硬盘上不会整体向后移。所以就出现 当前这个情况 ，你想插入，却变成了会把旧内容覆盖掉。

> **但是人家word, vim 都可以修改文件 呀，你这不能修改算个什么玩意？**

 我并没说就不能修改了，你想修改当然可以，就是不要在硬盘上修改，把内容全部读到内存里，数据在内存里可以随便增删改查，修改之后，把内容再全部写回硬盘，把原来的数据全部覆盖掉。vim word等各种文本编辑器都是这么干的。

> **说的好像有道理，但你又没看过word软件的源码，你凭什么这么笃定？**

 哈哈，我不需要看源码，硬盘 的存储原理决定了word必须这么干 ，不信的话，还有个简单的办法来确认我说的，就是用word or vim读一个编辑一个大文件 ，至少几百MB的，你 会发现，加载过程会花个数十秒，这段时间干嘛了？ cpu 去玩了？去上厕所啦？ 当然不是，是在努力把数据 从硬盘上读到内存里。

> **但是文件如果特别大，比如5个GB,读到内存，就一下子吃掉了5GB内存，好费资源呀，有没有更好的办法呢？**

 如果不想占内存，只能用另外一种办法啦，就是边读边改， 什么意思？ 不是不能改么？是不能改原文件 ，但你可以打开旧文件 的同时，生成一个新文件呀，**边从旧的里面一行行的读，边往新的一行行写**，遇到需要修改就改了再写到新文件 ，这样，在内存里一直只存一行内容。就不占内存了。 但这样也有一个缺点，就是虽然不占内存 ，但是占硬盘，每次修改，都要生成一份新文件，虽然改完后，可以把旧的覆盖掉，但在改的过程中，还是有2份数据 的。

 来个例子吧！！！GO！！

```
import os

file_name = "F:\\Animation\\3.txt"
file_new_name = "F:\\Animation\\4.txt"
file = open(file_name, "r")
file_new = open(file_new_name, "w")
str_old = "深圳"
str_new = "杭州"

# 循环修改，产生一个新文件
for line in file:
    if str_old in line:
        line = line.replace(str_old, str_new)
    file_new.write(line)
file_new.close()
file.close()

# 再修改和文件名称
os.replace(file_new_name, file_name)
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720347.png)](https://img-blog.csdnimg.cn/2020072809091050.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 查看编码方式

> 先安装 `pip install chardet`

```
import chardet

file = open(file="F:\\Animation\\0.txt", mode="r")
print("编码格式：", chardet.detect(info.encode()).get("encoding"))
file.close()
```

 输出

```
编码格式： utf-8
```



### 二进制读

```
# 二进制方式不能指定encoding，因为获取的就是二进制字节码
file = open(file="F:\\Animation\\4.txt", mode="rb")
print("完整数据：", file.read().decode("utf-8"))

file.seek(3)  # 默认是0，从头开始移动3个字节
print(file.readline(3).decode("utf-8"))

file.seek(-3, 1)  # 从当前位置移动-3个字节，只有1和2才能是offset<0
print(file.readline(3).decode("utf-8"))

file.seek(-3, 2)  # 从文件末尾移动-3个字节
print(file.readline(3).decode("utf-8"))

file.seek(-3, 2)  # 从文件末尾移动-3个字节
print(file.readline(3).decode("utf-8").encode("GBK"))  # 转成GBK编码

file.close()
```

 输出

```
完整数据： 你好呀去吧
好
好
吧
b'\xb0\xc9'
```



### 相关方法

#### 表格

| 序号 | 方法及描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | close()关闭文件。关闭后文件不能再进行读写操作。              |
| 2    | [file.flush()](https://www.runoob.com/python/file-flush.html) 刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入。 |
| 3    | [file.fileno()](https://www.runoob.com/python/file-fileno.html) 返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。 |
| 4    | [file.isatty()](https://www.runoob.com/python/file-isatty.html) 如果文件连接到一个终端设备返回 True，否则返回 False。 |
| 5    | [file.next()](https://www.runoob.com/python/file-next.html) 返回文件下一行。 |
| 6    | [file.read([size\])](https://www.runoob.com/python/python-file-read.html) 从文件读取指定的字节数，如果未给定或为负则读取所有。size指定读取几个字节 |
| 7    | [file.readline([size\])](https://www.runoob.com/python/file-readline.html) 读取整行，包括 “\n” 字符。size可以指定读取当前行的几个字符 |
| 8    | [file.readlines([sizeint\])](https://www.runoob.com/python/file-readlines.html) 读取所有行并返回列表，若给定sizeint>0，则是设置一次读多少字节，这是为了减轻读取压力。注意的是：`sizeint为限制读取到 当前位置行的剩余数据~第hint个字符的那一行所有数据` |
| 9    | [file.seek(offset[, whence\])](https://www.runoob.com/python/file-seek.html) 设置文件当前位置 ，offset: 偏移量，whence：三种方式(0：从头开始；1：从当前位置开始；2：从末尾开始)。1和2：open函数**只能以二进制模式**打开文件，才能正常运行，否则就会报出上面的错误。**如果没有以二进制b的方式打开，则offset无法使用负值（即向左侧移动）** |
| 10   | [file.tell()](https://www.runoob.com/python/file-tell.html) 返回文件当前位置。 |
| 11   | [file.truncate([size\])](https://www.runoob.com/python/file-truncate.html) 截取文件，截取的字节通过size指定，默认为当前文件位置。 |
| 12   | [file.write(str)](https://www.runoob.com/python/python-file-write.html) 将字符串写入文件，返回的是写入的字符长度。 |
| 13   | [file.writelines(sequence)](https://www.runoob.com/python/file-writelines.html) 向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。 |



#### open方法

```
def open(file, mode='r', buffering=None, encoding=None, errors=None, newline=None, closefd=True)
```

**可选模式**

| 模式 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| t    | 文本模式 (默认)。                                            |
| x    | 写模式，新建一个文件，如果该文件已存在则会报错。             |
| b    | 二进制模式。                                                 |
| +    | 打开一个文件进行更新(可读可写)。                             |
| U    | 通用换行模式（不推荐）。                                     |
| r    | 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。 |
| rb   | 以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。一般用于非文本文件如图片等。 |
| r+   | 打开一个文件用于读写。文件指针将会放在文件的开头。           |
| rb+  | 以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。一般用于非文本文件如图片等。 |
| w    | 打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| wb   | 以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。 |
| w+   | 打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| wb+  | 以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。 |
| a    | 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| ab   | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| a+   | 打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。 |
| ab+  | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。 |





## 函数

### what？

 函数一词来源于数学，但编程中的「函数」概念，与数学中的函数是有很大不同的，具体区别，我们后面会讲，编程中的函数在英文中也有很多不同的叫法。在BASIC中叫做subroutine(子过程或子程序)，在Pascal中叫做procedure(过程)和function，在C中只有function，在Java里面叫做method。

 **定义: 函数是指将一组语句的集合通过一个名字(函数名)封装起来，要想执行这个函数，只需调用其函数名即可**

 **特性:**

1. 减少重复代码
2. 使程序变的可扩展
3. 使程序变得易维护



### 定义

```
# 定义
def hello(note):
    print("Hello Function！{name}".format(name=note))

# 调用
hello("Go!")
```

> 参数可以让你的函数更灵活，不只能做死的动作，还可以根据调用时传参的不同来决定函数内部的执行流程



### 函数参数

#### **形参变量**

 只有在被调用时才分配内存单元，在调用结束时，即刻释放所分配的内存单元。因此，形参只在函数内部有效。函数调用结束返回主调用函数后则不能再使用该形参变量

#### **实参**

 可以是常量、变量、表达式、函数等，无论实参是何种类型的量，在进行函数调用时，它们都必须有确定的值，以便把这些值传送给形参。因此应预先给实参赋值

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720348.png)](https://img-blog.csdnimg.cn/20200728120051750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

#### 默认参数

```
# 默认参数
def calc(a, b, c=2):
    return a ** b // c


print(calc(3, 3))
print(calc(3, 3, 4))
```

 输出

```
13
6
```

> 也就是说，我们可以不指定c
>
> - 若不传入，则就是2
> - 若传入，则就是传入的值



#### 关键参数

 正常情况下，给函数传参数要按顺序，不想按顺序就可以用关键参数，只需指定参数名即可(指定了参数名的参数就叫关键参数)，***但记住一个要求就是，关键参数必须放在位置参数(以位置顺序确定对应关系的参数)之后***

```
def stu_register(name, age, course='PY' ,country='CN'):
    print("----注册学生信息------")
    print("姓名:", name)
    print("age:", age)
    print("国籍:", country)
    print("课程:", course)
```

 调用可以这样

```
stu_register("王山炮",course='PY', age=22,country='JP' )
```

 但绝不可以这样

```
stu_register("王山炮",course='PY',22,country='JP' )
```

 当然这样也不行

```
stu_register("王山炮",22,age=25,country='JP' )
```

 这样相当于给age赋值2次，会报错！

> 注意，参数优先级顺序是**位置参数>关键参数**



#### **非固定参数**

 若你的函数在定义时不确定用户想传入多少个参数，就可以使用非固定参数

```
def stu_register(name, age, course='PY', country='CN', *args, **kwargs):
    print("----注册学生信息------")
    print("姓名:", name)
    print("age:", age)
    print("国籍:", country)
    print("课程:", course)
    print(args, kwargs)
    
stu_register("Drake", 18, phone=14456789521, sex="M")
stu_register("Drake", 18, 'Java', 'EN', 14456789521, "M")
```

 args是一个元组，kwargs是一个字典

 输出

```
----注册学生信息------
姓名: Drake
age: 18
国籍: CN
课程: PY
() {'phone': 14456789521, 'sex': 'M'}
----注册学生信息------
姓名: Drake
age: 18
国籍: EN
课程: Java
(14456789521, 'M') {}
```



### 函数返回值与作用域

 函数外部的代码要想获取函数的执行结果，就可以在函数里用return语句把结果返回

- 函数在执行过程中只要遇到return语句，就会停止执行并返回结果，so 也可以理解为 return 语句代表着函数的结束
- 如果未在函数中指定return,那这个函数的返回值为None

```
def get_str():
    return "你好"


print(get_str())


def get_tuple():
    return "好呀", "去吧", "嗯呐"


print(get_tuple())
```

 输出

```
你好
('好呀', '去吧', '嗯呐')
```

> 当有多个值时，返回的是一个元组



### 全局与局部变量

```
name = "定义一个全局变量"


def change_name():
    name = "局部变量"
    print("方法内输出name：", name)


change_name()
print("方法外输出name：", name)
```

 输出

```
方法内输出name： 局部变量
方法外输出name： 定义一个全局变量
```

为什么在函数内部改了name的值后， 在外面print的时候却没有改呢？ 因为这两个name根本不是一回事

- 在函数中定义的变量称为局部变量，在程序的一开始定义的变量称为全局变量。
- 全局变量作用域(即有效范围)是整个程序，局部变量作用域是定义该变量的函数。
- 变量的查找顺序是**局部变量>全局变量**
- 当全局变量与局部变量同名时，在定义局部变量的函数内，局部变量起作用；在其它地方全局变量起作用。
- 在函数里是不能直接修改全局变量的



> **就是想在函数里修改全局变量怎么办？**

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720349.jpeg)](https://img-blog.csdnimg.cn/20200728200935243.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```
name = "定义一个全局变量"

def change_name_global():
    global  name
    name = "局部变量"
    print("方法内输出name：", name)

change_name_global()
print("方法外输出name：", name)
```

 输出

```
方法内输出name： 局部变量
方法外输出name： 局部变量
```

> `global name`**的作用就是要在函数里声明全局变量name ，意味着最上面的**`name = “定义一个全局变量”`**即使不写，程序最后面的print也可以打印name**



### 传递列表、字典、集合产生的现象

```
d = {"name":"Alex","age":26,"hobbie":"大保健"}
l = ["Rebeeca","Katrina","Rachel"]
name = "你去吧"
def change_data(info,girls,name):
    info["hobbie"] = "学习"
    girls.append("XiaoYun")

change_data(d,l,name)
print(d,l,name)
```

 输出

```
{'name': 'Alex', 'age': 26, 'hobbie': '学习'} ['Rebeeca', 'Katrina', 'Rachel', 'XiaoYun'] 你去吧
```

> 其实也没啥好说的，参数的传递其实就是**内存地址**引用的传递，元组，字典等修改内部的数据，但是其本身的内存地址引用不会发生改变！而字符，数字等发生改变了，其实数据的内存地址也就发生改变了。



### 嵌套函数

```
def fun1():
    name = "fun1"
    def fun2():
        name = "fun2"
        print(name)
    fun2()
    print(name)

fun1()
```

 输出

```
fun2
fun1
```

 通过上面的例子，我们理解了，每个函数里的变量是互相独立的，变量的查找顺序也是从当前层依次往上层找。



### 匿名函数

```
fun = lambda x, y: x**y

print(fun(2, 3))
```

 输出

```
8
```

 匿名函数用着还是很爽的好吧，这里举个例子：

```
res = map(lambda x:x**2, [1,5,7,2,9])
for value in res:
    print(value)
```

 输出

```
1
25
49
4
81
```



### 高阶函数

 变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

```
def add(x, y):
    return x+y


def calc(x, y, z, add):
    return add(x, y)-z


print(calc(5, 7, 2, add))
```

 输出

```
10
```

只需满足以下任意一个条件，即是高阶函数

- 接受一个或多个函数作为输入
- return 返回另外一个函数



### 内置函数

 Python的len为什么你可以直接用？肯定是解释器启动时就定义好了

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720350.png)](https://img-blog.csdnimg.cn/2020072820590560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> *内置参数详解* https://docs.python.org/3/library/functions.html?highlight=built#ascii

```
abs # 求绝对值

all #Return True if bool(x) is True for all values x in the iterable.If the iterable is empty, return True.

any #Return True if bool(x) is True for any x in the iterable.If the iterable is empty, return False.

ascii #Return an ASCII-only representation of an object,ascii(“中国”) 返回”‘\u4e2d\u56fd’”

bin #返回整数的2进制格式

bool # 判断一个数据结构是True or False, bool({}) 返回就是False, 因为是空dict

bytearray # 把byte变成 bytearray, 可修改的数组

bytes # bytes(“中国”,”gbk”)

callable # 判断一个对象是否可调用

chr # 返回一个数字对应的ascii字符 ， 比如chr(90)返回ascii里的’Z’

classmethod #面向对象时用，现在忽略

compile #py解释器自己用的东西，忽略

complex #求复数，一般人用不到

copyright #没用

credits #没用

delattr #面向对象时用，现在忽略

dict #生成一个空dict

dir #返回对象的可调用属性

divmod #返回除法的商和余数 ，比如divmod(4,2)，结果(2, 0)

enumerate #返回列表的索引和元素，比如 d = [“alex”,”jack”]，enumerate(d)后，得到(0, ‘alex’) (1, ‘jack’)

eval #可以把字符串形式的list,dict,set,tuple,再转换成其原有的数据类型。

exec #把字符串格式的代码，进行解义并执行，比如exec(“print(‘hellworld’)”)，会解义里面的字符串并执行

exit #退出程序

filter #对list、dict、set、tuple等可迭代对象进行过滤， filter(lambda x:x>10,[0,1,23,3,4,4,5,6,67,7])过滤出所有大于10的值

float #转成浮点

format #没用

frozenset #把一个集合变成不可修改的

getattr #面向对象时用，现在忽略

globals #打印全局作用域里的值

hasattr #面向对象时用，现在忽略

hash #hash函数

help

hex #返回一个10进制的16进制表示形式,hex(10) 返回’0xa’

id #查看对象内存地址

input

int

isinstance #判断一个数据结构的类型，比如判断a是不是fronzenset, isinstance(a,frozenset) 返回 True or False

issubclass #面向对象时用，现在忽略

iter #把一个数据结构变成迭代器，讲了迭代器就明白了

len

list

locals

map # map(lambda x:x**2,[1,2,3,43,45,5,6,]) 输出 [1, 4, 9, 1849, 2025, 25, 36]

max # 求最大值

memoryview # 一般人不用，忽略

min # 求最小值

next # 生成器会用到，现在忽略

object #面向对象时用，现在忽略

oct # 返回10进制数的8进制表示

open

ord # 返回ascii的字符对应的10进制数 ord(‘a’) 返回97，

print

property #面向对象时用，现在忽略

quit

range

repr #没什么用

reversed # 可以把一个列表反转

round #可以把小数4舍5入成整数 ，round(10.15,1) 得10.2

set

setattr #面向对象时用，现在忽略

slice # 没用

sorted

staticmethod #面向对象时用，现在忽略

str

sum #求和,a=[1, 4, 9, 1849, 2025, 25, 36],sum(a) 得3949

super #面向对象时用，现在忽略

tuple

type

vars #返回一个对象的属性，面向对象时就明白了

zip #可以把2个或多个列表拼成一个， a=[1, 4, 9, 1849, 2025, 25, 36]，b = [“a”,”b”,”c”,”d”]，
list(zip(a,b)) #得结果 
[(1, 'a'), (4, 'b'), (9, 'c'), (1849, 'd')]
```



### 名称空间

 又名name space, 顾名思义就是存放名字的地方，存什么名字呢？举例说明，若变量x=1，1存放于内存中，那名字x存放在哪里呢？**名称空间正是存放名字x与1绑定关系的地方**

 python里面有很多名字空间，每个地方都有自己的名字空间，互不干扰，不同空间中的两个相同名字的变量之间没有任何联系。

 名称空间有4种(`LEGB`):

- `locals`:函数内部的名字空间，一般包括函数的局部变量以及形式参数
- `enclosing function`:在嵌套函数中外部函数的名字空间, 若fun2嵌套在fun1里，对`fun2`来说， `fun1`的名字空间就是enclosing.
- `globals`:当前的模块空间，模块就是一些`py`文件。也就是说，globals()类似全局变量。
- `builtins`: 内置模块空间，也就是内置变量或者内置函数的名字空间，print(dir(**builtins**))可查看包含的值。

**不同变量的作用域不同就是由这个变量所在的名称空间决定的。**

作用域即范围

- 全局范围：全局存活，全局有效
- 局部范围：临时存活，局部有效

> 查看作用域方法 globals(),locals()



### 作用域查找顺序

 当程序引用某个变量的名字时，就会从当前名字空间开始搜索。搜索顺序规则便是: `LEGB`。**即locals -> enclosing function -> globals -> builtins。** 一层一层的查找，找到了之后，便停止搜索，如果最后没有找到,则抛出在`NameError`的异常。

```
level = 'L0'
n = 22
def func():
    level = 'L1'
    n = 33
    print(locals())
    def outer():
        n = 44
        level = 'L2'
        print("outer:",locals(),n)
        def inner():
            level = 'L3'
            print("inner:",locals(),n) #此外打印的n是多少？
        inner()
    outer()
func()
```

 输出

```
{'level': 'L1', 'n': 33}
outer: {'level': 'L2', 'n': 44} 44
inner: {'level': 'L3', 'n': 44} 44
```



### 闭包

 关于闭包，即函数定义和函数表达式位于另一个函数的函数体内(嵌套函数)。而且，这些内部函数可以访问它们所在的外部函数中声明的所有局部变量、参数。当其中一个这样的内部函数在包含它们的外部函数之外被调用时，就会形成闭包。也就是说，内部函数会在外部函数返回后被执行。而当这个内部函数执行时，它仍然必需访问其外部函数的局部变量、参数以及其他内部函数。这些局部变量、参数和函数声明（最初时）的值是外部函数返回时的值，但也会受到内部函数的影响。

```
def outer():
    name = 'alex'
    def inner():
        print("在inner里打印外层函数的变量",name)
    return inner # 注意这里只是返回inner的内存地址，并未执行


f = outer() # .inner at 0x1027621e0>
f()  # 相当于执行的是inner()
```

 输出

```
在inner里打印外层函数的变量 alex
```

 注意此时outer已经执行完毕，正常情况下outer里的内存都已经释放了，但此时由于闭包的存在，我们却还可以调用inner, 并且inner内部还调用了上一层outer里的name变量。这种粘粘糊糊的现象就是闭包。

> 闭包的意义：**返回的函数对象，不仅仅是一个函数对象，在该函数外还包裹了一层作用域，这使得，该函数无论在何处调用，优先使用自己外层包裹的作用域**



### 装饰器

```
account = {
    "is_authenticated":False,# 用户登录了就把这个改成True
    "username":"admin", # 假装这是DB里存的用户信息
    "password":"123456" # 假装这是DB里存的用户信息
}
def login(func):
    def inner(*args, **kwargs):
        if account["is_authenticated"] is False:
            username = input("user:")
            password = input("pasword:")
            if username == account["username"] and password == account["password"]:
                print("welcome login....")
                account["is_authenticated"] = True
            else:
                print("wrong username or password!")
        if account["is_authenticated"] is True:  # 主要改了这
            func(*args, **kwargs)  # 认证成功了就执行传入进来的函数
    return inner

def home():
    print("---首页----")

def america():
    print("----欧美专区----")

@login
def japan(note):
    print("----日韩专区----")
    print(note)

def henan(*args, **kwargs):
    print("----河南专区----")
    print(kwargs["note"])

home()

america = login(america) # 需要验证就调用 login，把需要验证的功能 当做一个参数传给login
henan = login(henan)
america()
henan(note="什么玩意呀(滑稽)~~~")

japan("日韩的好呀~~~")
```

 输出

```
---首页----
user:admin
pasword:123456
welcome login....
----欧美专区----
----河南专区----
什么玩意呀(滑稽)~~~
----日韩专区----
日韩的好呀~~~
```

 想实现一开始你写的america = login(america)不触发你真正的america函数的执行，只需要在这个login里面再定义一层函数，第一次调用america = login(america)只调用到外层login，这个login虽然会执行，但不会触发认证了，因为认证的所有代码被封装在login里层的新定义 的函数里了，login只返回 里层函数的函数名，这样下次再执行america()时， 就会调用里层函数啦。。。

 `@login`注解就相当于一个语法糖吧，我们并不需要显示的去调用装饰方法。



### 列表生成式

 现在有个需求，现有列表a=`[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`,要求你把列表里的每个值加1，你怎么实现？你可能会想到2种方式

 方式一：

```
for index,i in enumerate(a):
    a[index] +=1
print(a)
```

 方式二(一般想到的是这种方法)：

```
a = map(lambda x:x+1, a)
```

 方式三(牛逼呀)：

```
print([i+1 for i in a])
```



### 生成器generator

 通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且，创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。比如我要循环100万次，按py的语法，for i in range(1000000)会先生成100万个值的列表。但是循环到第50次时，我就不想继续了，就退出了。但是90多万的列表元素就白为你提前生成了。

```
for i in range(1000000):
    if i == 50: 
        break
    print(i)
```

 所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？

 像上面这个循环，每次循环只是+1而已，我们完全可以写一个算法，让他执行一次就自动+1，这样就不必创建完整的list，从而节省大量的空间。**在Python中，这种一边循环一边计算后面元素的机制，称为生成器：generator。**

#### 简单使用

 要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的`[]`改成`()，`就创建了一个generator：

```
temp = (x**x for x in range(10))
print(type(temp))
print(next(temp))
print(next(temp))
print(temp.__next__())

for i in temp:
    print(i, end="\t")
```

 输出

```
<class 'generator'>
1
1
4
27	256	3125	46656	823543	16777216	387420489	
```

 如果要一个一个打印出来，可以通过`next()`函数获得generator的下一个返回值：

 我们讲过，generator保存的是算法，每次调用`next(g)`就计算出`g`的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出`StopIteration`的错误。

 当然，上面这种不断调用`next(g)`实在是太变态了，正确的方法是使用`for`循环，因为generator也是可迭代(遍历)对象：

```
for i in temp:
    print(i, end="\t")
```



#### 函数方式

 generator非常强大。如果推算的算法比较复杂，用类似列表生成式的for循环无法实现的时候，还可以用函数来实现。

比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：

 1, 1, 2, 3, 5, 8, 13, 21, 34, …

 实现100以内的斐波那契数代码：

```
def fib_g(max):
    a, b = 0, 1
    value = 0
    while value < max:
        value = a+b
        a = b
        b = value
        yield value # 程序走到这，就会暂停下来，返回count到函数外面，直到被next方法调用时唤醒

g = fib_g(20)
print(next(g))
print(next(g))
print(g.__next__())
```

 输出

```
1
2
3
```

 这就是定义generator的另一种方法。如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator：

> 这里，最难理解的就是generator和函数的执行流程不一样。函数是顺序执行，遇到return语句或者最后一行函数语句就返回。**而变成generator的函数，在每次调用next()的时候执行，遇到yield语句暂停并返回数据到函数外，再次被next()调用时从上次返回的yield语句处继续执行**。



### 并发编程(yield的另一种使用)

 虽然我们还没学并发编程，但我们肯定听过cpu 多少核多少核之类的，cpu的多核就是为了可以实现并行运算，让你同时边听歌、边聊qq、边刷知乎。单核的cpu同一时间只能干一个事，所以你用单核电脑同时做好几件事的话，就会变的很慢，因为cpu要在不同程序任务间来回切换。

 通过yield, 我们可以实现单核下并发做多件事的效果。

```
def yield_test():
    print("---")
    while True:
        n = yield
        print("receive from outside:", n)

g = yield_test()  # 注意此时并不会执行函数
print("1 ", type(g))
g.__next__()   # 执行一下next可以使上面的函数走到yield那句。 这样后面的send语法才能生效
print("2 ", type(g))

for i in range(10):
    g.send(i)  # send的作用=next, 同时还把数据传给了上面函数里的yield
```

 输出

```
1  <class 'generator'>
---
2  <class 'generator'>
receive from outside: 0
receive from outside: 1
receive from outside: 2
receive from outside: 3
receive from outside: 4
receive from outside: 5
receive from outside: 6
receive from outside: 7
receive from outside: 8
receive from outside: 9
```

> **注意：调用send(x)给生成器传值时，必须确保生成器已经执行过一次next()调用, 这样会让程序走到yield位置等待外部第2次调用。**



### 迭代器

 我们已经知道，可以直接作用于`for`循环的数据类型有以下几种：

1. 一类是集合数据类型，如`list`、`tuple`、`dict`、`set`、`str`等；
2. 一类是`generator`，包括生成器和带`yield`的generator function。

 这些可以直接作用于`for`循环的对象统称为**可迭代对象：`Iterable，可迭代的意思就是可遍历、可循环`。**

 **可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator**

> 可以使用`isinstance()`判断一个对象是否是`Iterable`对象：
>
> 可以使用isinstance()判断一个对象是否是`Iterator`对象：

```
from collections import Iterable
from collections import Iterator

print(isinstance([], Iterable))  # True   是可迭代对象
print(isinstance([], Iterator))  # False  不是迭代器对象

print(isinstance((x for x in range(10)), Iterable))  # True
print(isinstance((x for x in range(10)), Iterator))  # True
```

 生成器都是`Iterator`对象，但`list`、`dict`、`str`虽然是`Iterable`，却不是`Iterator`。

 把`list`、`dict`、`str`等`Iterable`变成`Iterator`可以使用`iter()`函数：

```
print(isinstance(iter([]), Iterable))  # True
print(isinstance(iter([]), Iterator))  # True

# 示例
iterator = iter([1,2,3,4,5])
print(iterator.__next__())
for value in iterator:
    print(value, end="\t")
    
----输出----
1
2	3	4	5	
```

 你可能会问，为什么`list`、`dict`、`str`等数据类型不是`Iterator`？

 这是因为Python的`Iterator`对象表示的是一个数据流，Iterator对象可以被`next()`函数调用并不断返回下一个数据，直到没有数据时抛出`StopIteration`错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过`next()`函数实现按需计算下一个数据，所以`Iterator`的计算是惰性的，只有在需要返回下一个数据时它才会计算。

 `Iterator`甚至可以表示一个无限大的数据流，例如全体自然数。而使用list是永远不可能存储全体自然数的。



> **小结**
>
> 凡是可作用于`for`循环的对象都是`Iterable`类型；
>
> 凡是可作用于`next()`函数的对象都是`Iterator`类型，它们表示一个惰性计算的序列；
>
> 集合数据类型如`list`、`dict`、`str`等是`Iterable`但不是`Iterator`，不过可以通过`iter()`函数获得一个`Iterator`对象。



## 模块

### 为什么是模块?

 在计算机程序的开发过程中，随着程序代码越写越多，在一个文件里代码就会越来越长，越来越不容易维护。

 为了编写可维护的代码，我们把很多函数分组，分别放到不同的文件里，这样，每个文件包含的代码就相对较少，很多编程语言都采用这种组织代码的方式。在Python中，一个.py文件就可以称之为一个模块（Module）。



### 使用模块有什么好处？

1. 最大的好处是大大提高了代码的可维护性。其次，编写代码不必从零开始。当一个模块编写完毕，就可以被其他地方引用。我们在编写程序的时候，也经常引用其他模块，包括Python内置的模块和来自第三方的模块。
2. 使用模块还可以避免函数名和变量名冲突。每个模块有独立的命名空间，因此相同名字的函数和变量完全可以分别存在不同的模块中，所以，我们自己在编写模块时，不必考虑名字会与其他模块冲突



### 模块分类

模块分为三种：

- 内置标准模块（又称标准库）执行help(‘modules’)查看所有python自带模块列表
- 第三方开源模块，可通过pip install 模块名 联网安装
- 自定义模块



### 模块导入&调用

```
import module_a  #导入
from module import xx
from module.xx.xx import xx as rename #导入后重命令
from module.xx.xx import *  #导入一个模块下的所有方法，不建议使用
__import__("test_name")  # 动态导入
__import__("basicGrammar.module.my_module")  # 动态导入(跨包)
importlib.import_module("test_name")  # 动态导入
importlib.import_module("basicGrammar.module.my_module")  # 动态导入(跨包)

module_a.xxx  #调用
```

> *注意：模块一旦被调用，即相当于执行了另外一个py文件里的代码*



### 自定义模块

 这个最简单， 创建一个.py文件，就可以称之为模块，就可以在另外一个程序里导入

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720351.png)](https://img-blog.csdnimg.cn/20200729135007338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 模块查找路径（跨目录导入）

 发现，自己写的模块只能在当前路径下的程序里才能导入，换一个目录再导入自己的模块就报错说找不到了， 这是为什么？

 这与导入模块的查找路径有关

```
import sys
print(sys.path)
```

 输出（注意不同的电脑可能输出的不太一样）

```
['D:\\workspace_python\\basis_learn01\\basicGrammar\\module', 
 'D:\\workspace_python\\basis_learn01', 
 'E:\\PyCharm\\PyCharm 2020.1.3\\plugins\\python\\helpers\\pycharm_display',  'E:\\PyCharm\\Python38\\python38.zip', 
 'E:\\PyCharm\\Python38\\DLLs', 
 'E:\\PyCharm\\Python38\\lib', 
 'E:\\PyCharm\\Python38', 
 'D:\\workspace_python\\basis_learn01\\venv', 
 'D:\\workspace_python\\basis_learn01\\venv\\lib\\site-packages', 
 'E:\\PyCharm\\PyCharm 2020.1.3\\plugins\\python\\helpers\\pycharm_matplotlib_backend']
```

 你导入一个模块时，Python解释器会按照上面列表顺序去依次到每个目录下去匹配你要导入的模块名，只要在一个目录下匹配到了该模块名，就立刻导入，不再继续往后找。

> *注意列表第一个元素为空，即代表当前目录，所以你自己定义的模块在当前目录会被优先导入。*

 我们自己创建的模块若想在任何地方都能调用，那就得确保你的模块文件至少在模块路径的查找列表中。

 我们一般把自己写的模块放在一个带有“site-packages”字样的目录里，我们从网上下载安装的各种第三方的模块一般都放在这个目录。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720352.png)](https://img-blog.csdnimg.cn/20200729135756583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720353.png)](https://img-blog.csdnimg.cn/20200729140351103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 第3方开源模块的安装使用

 https://pypi.python.org/pypi 是python的开源模块库，截止2019年4.30日 ，已经收录了**175870**个来自全世界python开发者贡献的模块,几乎涵盖了你想用python做的任何事情。 事实上每个python开发者，只要注册一个账号就可以往这个平台上传你自己的模块，这样全世界的开发者都可以容易的下载并使用你的模块。

 直接通过pip安装

```
pip install paramiko #paramiko 是模块名
```

 pip命令会自动下载模块包并完成安装。

 软件一般会被自动安装你python安装目录的这个子目录里

```
'D:\\workspace_python\\basis_learn01\\venv\\lib\\site-packages',
```

#### 使用国内镜像(暂时)

 pip命令默认会连接在国外的python官方服务器下载，速度比较慢，你还可以使用国内的豆瓣源，数据会定期同步国外官网，速度快好多

```
pip install -i http://pypi.douban.com/simple/ alex_sayhi --trusted-host pypi.douban.com   #alex_sayhi是模块名
```

> -i 后面跟的是豆瓣源地址
>
> —trusted-host 得加上，是通过网站https安全验证用的

#### 使用国内镜像(永久)

- 在文件夹的地址栏输入 %appdata% （即进入这个文件夹）。
- 在当前文件夹下新建一个pip文件夹。
- 进入pip文件夹，新建一个pip.ini文件
- 在pip.ini文件中写下如下内容:

```
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
 
[install]
trusted-host=mirrors.aliyun.com
```



### 包&跨目录导入模块

 当你的模块文件越来越多，就需要对模块文件进行划分，比如把负责跟数据库交互的都放一个文件夹，把与页面交互相关的放一个文件夹，

```
my_proj/
├── apeland_web  #代码目录
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
└── my_proj    #配置文件目录
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

 像上面这样，**一个文件夹管理多个模块文件，这个文件夹就被称为包**

 一个包就是一个文件夹，但该文件夹下必须存在 **init**.py 文件, 该文件的内容可以为空， **int**.py用于标识当前文件夹是一个包。

 这个**init**.py的文件主要是用来对包进行一些初始化的，当当前这个package被别的程序调用时，**init**.py文件会先执行，一般为空， 一些你希望只要package被调用就立刻执行的代码可以放在**init**.py里，一会后面会演示。



#### 跨模块导入

目录结构如下

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720354.png)](https://img-blog.csdnimg.cn/20200730013831351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 根据上面的结构，如何实现在`包.py`里调用`my_package`模块？

 直接导入的话，会报错，说找到不模块

[![img](E:\Development\Typora\images\20200730014321268.png)](https://img-blog.csdnimg.cn/20200730014321268.png)



#### 方式一

 我们自己创建的模块若想在任何地方都能调用，那就得确保你的模块文件至少在模块路径的查找列表中。

 我们一般把自己写的模块放在一个带有“site-packages”字样的目录里，我们从网上下载安装的各种第三方的模块一般都放在这个目录。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720355.png)](https://img-blog.csdnimg.cn/20200730014215951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```
from my_package.test import test_p

test_p()
```



#### 方式二

 **添加环境变量，把父亲级的路径添加到sys.path中，就可以了，这样导入 就相当于从父亲级开始找模块了。**

```
import sys
import os

# 固定获取目录(不可取)
# base_dir = "D:\\workspace_python\\basis_learn01\\basicGrammar"

print(os.path.dirname(os.path.dirname(__file__)))

# 动态的获取目录
base_dir = os.path.dirname(os.path.dirname(__file__))
sys.path.append(base_dir)

from my_package.test import test_p

test_p()
```



#### 方式三(官方推荐)

 虽然通过添加环境变量的方式可以实现跨模块导入，但是官方不推荐这么干，因为这样就需要在每个目录下的每个程序里都写一遍添加环境变量的代码。

 官方推荐的玩法是，在项目里创建个入口程序，整个程序调用的开始应该是从入口程序发起，这个入口程序一般放在项目的顶级目录

 这样做的好处是，项目中的二级目录 module.包.py中再调用他表亲my_package.test.py时就不用再添加环境变量了。

 原因是由于manage.py在顶层，manage.py启动时项目的环境变量路径就会自动变成….xxx/basicGrammar/这一级别

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720356.png)](https://img-blog.csdnimg.cn/20200730015643713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720357.png)](https://img-blog.csdnimg.cn/20200730020230269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



> 其实三种方式本质上都是通过`模块查找路径`来跨目录导入的

#### 方式四(动态加载)

```
import importlib

__import__("basicGrammar.module.my_module")
importlib.import_module("basicGrammar.module.my_module")
```





### random

```
#1.随机的浮点数，范围是在0.0~1.0之间：random.random()
import random
random.random()
0.644354136192532

#2.函数随机生成一个[a,b]范围内的浮点数：random.uniform(a, b)
random.uniform(0, 100)
24.333751706253736

#3.随机生成一个范围[a, b]内的整数：random.randint(a, b)
random.randint(1,10)
6

#4.随机选取一个元素返回：random.choice()
可以用于字符串、列表、元组等
random.choice([1,2,3])  #列表
3
random.choice((1,2,3))   #元组
2
random.choice("hello world")  #字符串
'h'
#随机生成字符
random.choice('abcdefghijklmnopqrstuvwxyz!@#$%^&*()')
'l'

#5.随机打乱元素：random.shuffle()
l = [1,2,3,4]
random.shuffle(l)
print(l)
[2, 4, 3, 1]

#6.从序列a中截取指定长度n的片段:random.sample(a, n)
a = [1,2,3,4,5]
b = "hello world"
n = 2
random.sample(a, n)
[5, 3]
random.sample(b, n)
['o', 'r']

#7.随机选取a到b间的奇数1/偶数2：random.randrange(a, b, 2)
random.randrange(0, 11, 1)   #奇数
5
random.randrange(0, 11, 2)   #偶数
10
```



简单描述

1. 随机的浮点数，范围是在0.0~1.0之间：random.random()；
2. 函数随机生成一个[a,b]范围内的浮点数：random.uniform(a, b)；
3. 随机生成一个范围[a, b]内的整数：random.randint(a, b)；
4. 随机选取一个元素返回或随机生成字符：random.choice()；
5. 随机打乱元素：random.shuffle()；
6. 从序列a中截取指定长度n的片段:random.sample(a, n)；
7. 随机选取a到b间的奇数1/偶数2：random.randrange(a, b, 2)。



### chardet

 安装

```
pip install chardet
```

 使用

```
import chardet

file = open(file="F:\\Animation\\0.txt", mode="r")
info = file.read(6)
print("编码格式：", chardet.detect(info.encode()).get("encoding"))
file.close()
```

 输出

```
编码格式： utf-8
```



### os

```
import os

# 修改文件的名称
os.replace(file_new_name, file_name)   # win
# os.rename(file_new_name, file_name)  # mac
```

 更多方法

```
得到当前工作目录，即当前Python脚本工作的目录路径: os.getcwd()
返回指定目录下的所有文件和目录名:os.listdir()
函数用来删除一个文件:os.remove()
删除多个目录：os.removedirs（r“c：\python”）
检验给出的路径是否是一个文件：os.path.isfile()
检验给出的路径是否是一个目录：os.path.isdir()
判断是否是绝对路径：os.path.isabs()
检验给出的路径是否真地存:os.path.exists()
返回一个路径的目录名和文件名:os.path.split()     e.g os.path.split('/home/swaroop/byte/code/poem.txt') 结果：('/home/swaroop/byte/code', 'poem.txt') 
分离扩展名：os.path.splitext()       e.g  os.path.splitext('/usr/local/test.py')    结果：('/usr/local/test', '.py')
获取路径名：os.path.dirname()
获得绝对路径: os.path.abspath()  
获取文件名：os.path.basename()
运行shell命令: os.system()
读取操作系统环境变量HOME的值:os.getenv("HOME") 
返回操作系统所有的环境变量： os.environ 
设置系统环境变量，仅程序运行时有效：os.environ.setdefault('HOME','/home/alex')
给出当前平台使用的行终止符:os.linesep    Windows使用'\r\n'，Linux and MAC使用'\n'
指示你正在使用的平台：os.name       对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'
重命名：os.rename（old， new） 如果命名文件已经存在，则无法创建该文件
替换: os.replace(src, dst) 就算命名文件已经存在，也可以创建该文件
创建多级目录：os.makedirs（r“c：\python\test”）
创建单个目录：os.mkdir（“test”）
获取文件属性：os.stat（file）
修改文件权限与时间戳：os.chmod（file）
获取文件大小：os.path.getsize（filename）
结合目录名与文件名：os.path.join(dir,filename)
改变工作目录到dirname: os.chdir(dirname)
获取当前终端的大小: os.get_terminal_size()
杀死进程: os.kill(10884,signal.SIGKILL)
遍历目录中的文件(含递归)：os.walk() 
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720458.png)](https://img-blog.csdnimg.cn/20200729150349566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### sys

 读取外部指令

```
import sys

command = sys.argv
old_area = command[1]
new_area = command[2]
print("%s---%s" % (old_area, new_area))
```

 手动运行

[![img](E:\Development\Typora\images\20200728091657340.png)](https://img-blog.csdnimg.cn/20200728091657340.png)

 常用方法

```
sys.argv           命令行参数List，第一个元素是程序本身路径
sys.exit(n)        退出程序，正常退出时exit(0)
sys.version        获取Python解释程序的版本信息
sys.maxint         最大的Int值
sys.path           返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform       返回操作系统平台名称
sys.stdout.write('please:')  #标准输出 , 引出进度条的例子， 注，在py3上不行，可以用print代替
val = sys.stdin.readline()[:-1] #标准输入
sys.getrecursionlimit() #获取最大递归层数
sys.setrecursionlimit(1200) #设置最大递归层数
sys.getdefaultencoding()  #获取解释器默认编码
sys.getfilesystemencoding  #获取内存数据存到文件里的默认编码
```



### copy

 深拷贝

```
import copy

data_copy = copy.deepcopy(data)
```



### time & datetime模块

 在平常的代码中，我们常常需要与时间打交道。在Python中，与时间处理有关的模块就包括：time，datetime,calendar(很少用，不讲)，下面分别来介绍。

 我们写程序时对时间的处理可以归为以下3种：

- **时间的显示**，在屏幕显示、记录日志等
- **时间的转换**，比如把字符串格式的日期转成Python中的日期类型
- **时间的运算**，计算两个日期间的差值等

#### time

**在Python中，通常有这几种方式来表示时间：**

1. 时间戳（timestamp）, 表示的是从1970年1月1日00:00:00开始按秒计算的偏移量。例子：1554864776.161901
2. 格式化的时间字符串，比如“2020-10-03 17:54”
3. 元组（struct_time）共九个元素。由于Python的time模块实现主要调用C库，所以各个平台可能有所不同，mac上：time.struct_time(tm_year=2020, tm_mon=4, tm_mday=10, tm_hour=2, tm_min=53, tm_sec=15, tm_wday=2, tm_yday=100, tm_isdst=0)

```
索引（Index）    属性（Attribute）    值（Values）
0     tm_year（年）                 比如2011
1     tm_mon（月）                  1 - 12
2     tm_mday（日）                 1 - 31
3     tm_hour（时）                 0 - 23
4     tm_min（分）                  0 - 59
5     tm_sec（秒）                  0 - 61
6     tm_wday（weekday）            0 - 6（0表示周日）
7     tm_yday（一年中的第几天）       1 - 366
8     tm_isdst（是否是夏令时）        默认为-1
```



#### UTC时间

 UTC（Coordinated Universal Time，世界协调时）亦即格林威治天文时间，世界标准时间。在中国为UTC+8，又称东8区。DST（Daylight Saving Time）即夏令时。



#### time模块的方法

- time.localtime([secs])：将一个时间戳转换为当前时区的struct_time。若secs参数未提供，则以当前时间为准。
  - time.struct_time(tm_year=2020, tm_mon=7, tm_mday=29, tm_hour=15, tm_min=15, tm_sec=50, tm_wday=2, tm_yday=211, tm_isdst=0)
- time.gmtime([secs])：和localtime()方法类似，gmtime()方法是将一个时间戳转换为UTC时区（0时区）的struct_time。
- time.time()：返回当前时间的时间戳。1596006950.818224
- time.mktime(t)：将一个struct_time转化为时间戳。
- time.sleep(secs)：线程推迟指定的时间运行,单位为秒。
- time.asctime([t])：把一个表示时间的元组或者struct_time表示为这种形式：’Sun Oct 1 12:04:38 2019’。如果没有参数，将会将time.localtime()作为参数传入。
- time.ctime([secs])：把一个时间戳（按秒计算的浮点数）转化为time.asctime()的形式。如果参数未给或者为None的时候，将会默认time.time()为参数。它的作用相当于time.asctime(time.localtime(secs))。
- time.strftime(format[, t])：把一个代表时间的元组或者struct_time（如由time.localtime()和time.gmtime()返回）转化为格式化的时间字符串。如果t未指定，将传入time.localtime()。
  - 举例：time.strftime("%Y %m %d %H:%M:%S %p %j %z %Z", time.localtime()）
  - 输出：2020 07 29 15:36:17 PM 211 +0800 中国标准时间
- time.strptime(string[, format])：把一个格式化时间字符串转化为struct_time。实际上它和strftime()是逆操作。
  - 举例：time.strptime(‘2017-10-3 17:54’,”%Y-%m-%d %H:%M”)
  - 输出：time.struct_time(tm_year=2017, tm_mon=10, tm_mday=3, tm_hour=17, tm_min=54, tm_sec=0, tm_wday=1, tm_yday=276, tm_isdst=-1)
- 字符串转时间格式对应表

| Meaning | Notes                                                        |                           |
| ------- | ------------------------------------------------------------ | ------------------------- |
| `%a`    | Locale’s abbreviated weekday name.                           |                           |
| `%A`    | Locale’s full weekday name.                                  |                           |
| `%b`    | Locale’s abbreviated month name.                             |                           |
| `%B`    | Locale’s full month name.                                    |                           |
| `%c`    | Locale’s appropriate date and time representation.           |                           |
| `%d`    | Day of the month as a decimal number [01,31].                |                           |
| `%H`    | Hour (24-hour clock) as a decimal number [00,23].            |                           |
| `%I`    | Hour (12-hour clock) as a decimal number [01,12].            |                           |
| `%j`    | Day of the year as a decimal number [001,366].               |                           |
| `%m`    | Month as a decimal number [01,12].                           |                           |
| `%M`    | Minute as a decimal number [00,59].                          |                           |
| `%p`    | Locale’s equivalent of either AM or PM.                      | (1)                       |
| `%S`    | Second as a decimal number [00,61].                          | (2)                       |
| `%U`    | Week number of the year (Sunday as the first day of the week) as a decimal number [00,53]. All days in a new year preceding the first Sunday are considered to be in week 0. | (3)                       |
| `%w`    | Weekday as a decimal number [0(Sunday),6].                   |                           |
| `%W`    | Week number of the year (Monday as the first day of the week) as a decimal number [00,53]. All days in a new year preceding the first Monday are considered to be in week 0. | (3)                       |
| `%x`    | Locale’s appropriate date representation.                    |                           |
| `%X`    | Locale’s appropriate time representation.                    |                           |
| `%y`    | Year without century as a decimal number [00,99].            |                           |
| `%Y`    | Year with century as a decimal number.                       |                           |
| `%z`    | Time zone offset indicating a positive or negative time difference from UTC/GMT of the form +HHMM or -HHMM, where H represents decimal hour digits and M represents decimal minute digits [-23:59, +23:59]. |                           |
| `%Z`    | Time zone name (no characters if no time zone exists).       |                           |
|         | `%%`                                                         | A literal `‘%’`character. |

 最后为了容易记住转换关系，看下图

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282935720458.png)](https://img-blog.csdnimg.cn/20200729150349566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### datetime

 相比于time模块，datetime模块的接口则更直观、更容易调用

 **datetime模块定义了下面这几个类：**

- datetime.date：表示日期的类。常用的属性有year, month, day；
- datetime.time：表示时间的类。常用的属性有hour, minute, second, microsecond；
- datetime.datetime：表示日期时间。
- datetime.timedelta：表示时间间隔，即两个时间点之间的长度。
- datetime.tzinfo：与时区有关的相关信息。（这里不详细充分讨论该类，感兴趣的童鞋可以参考python手册）

 **我们需要记住的方法仅以下几个：**

1. d=datetime.datetime.now() 返回当前的datetime日期类型

```
d.timestamp(),d.today(), d.year,d.timetuple()等方法可以调用
```

1. 时间戳和datetime日期类型的转换

```
print(datetime.datetime.now().timestamp())   # 1596018092.231518
print(datetime.datetime.fromtimestamp(time.time()))  # 2020-07-29 18:21:32.231519
```

1. 时间运算和时间替换

   ```
   print(datetime.datetime.now()-datetime.timedelta(days=3, minutes=2))
   print(datetime.datetime.now().replace(year=2009,month=10,day=7,minute=48,second=50))
   
   ---
   2020-07-26 18:21:22.140702
   2009-10-07 18:48:50.140702
   ```

2. 时区和时间字符串转换

   ```
   print(datetime.datetime.now().astimezone(pytz.timezone("Asia/Shanghai")).strftime("%Y %m %d %H:%M:%S"))  # 2020 07 29 18:36:00
   print(datetime.datetime.now().astimezone(pytz.timezone("Africa/Asmara")).strftime("%Y %m %d %H:%M:%S"))  # 2020 07 29 13:36:00
   ```



### pytz(时区)

 安装

```
pip install pytz
import pytz

# 查看所有时区
print(pytz.all_timezones)
print(type(pytz.timezone("Asia/Shanghai")))
print(datetime.datetime.now().astimezone(pytz.timezone("Asia/Shanghai")).strftime("%Y %m %d %H:%M:%S"))
print(datetime.datetime.now().astimezone(pytz.timezone("Africa/Asmara")).strftime("%Y %m %d %H:%M:%S"))
```

 输出

```
['Africa/Abidjan', 'Africa/Accra', 'Africa/Addis_Ababa', 'Africa/Algiers', 'Africa/Asmara', .....]
<class 'pytz.tzfile.Asia/Shanghai'>
2020 07 29 18:36:55
2020 07 29 13:36:55
```



### pickcle & json(序列化)

#### 什么叫序列化？

 序列化是指把内存里的数据类型转变成字符串，以使其能存储到硬盘或通过网络传输到远程，因为硬盘或网络传输时只能接受bytes

#### 为什么要序列化？

 你打游戏过程中，打累了，停下来，关掉游戏、想过2天再玩，2天之后，游戏又从你上次停止的地方继续运行，你上次游戏的进度肯定保存在硬盘上了，是以何种形式呢？游戏过程中产生的很多临时数据是不规律的，可能在你关掉游戏时正好有10个列表，3个嵌套字典的数据集合在内存里，需要存下来？你如何存？把列表变成文件里的多行多列形式？那嵌套字典呢？根本没法存。所以，若是有种办法可以直接把内存数据存到硬盘上，下次程序再启动，再从硬盘上读回来，还是原来的格式的话，那是极好的。

 用于序列化的两个模块

- json，用于字符串 和 python数据类型间进行转换
- pickle，用于python特有的类型 和 python的数据类型间进行转换



#### pickle

 pickle模块提供了四个功能：dumps、dump、loads、load

 **dumps和loads**

```
print(pickle.dumps(info))
print(pickle.dumps(name))

print(pickle.loads(b"\x80\x04\x95\x1b\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04name\x94\x8c\x04Alex\x94\x8c\x03age\x94K\x14u."))
print(pickle.loads(b"\x80\x04\x95\x08\x00\x00\x00\x00\x00\x00\x00\x8c\x04Alex\x94."))
```

 输出

```
b'\x80\x04\x95\x1b\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04name\x94\x8c\x04Alex\x94\x8c\x03age\x94K\x14u.'
b'\x80\x04\x95\x08\x00\x00\x00\x00\x00\x00\x00\x8c\x04Alex\x94.'
{'name': 'Alex', 'age': 20}
Alex
```



 **dump和load**

```
file = open("F:\\Animation\\10.txt", "wb")
pickle.dump(info, file)
pickle.dump(name, file)
file.close()

"F:\\Animation\\10.txt", "wb"
file = open("F:\\Animation\\10.txt", "rb")
print(pickle.load(file))  # {'name': 'Alex', 'age': 20}
print(pickle.load(file))  # Alex
file.close()
```

 输出

```
{'name': 'Alex', 'age': 20}
Alex
```

[![img](E:\Development\Typora\images\20200729185807556.png)](https://img-blog.csdnimg.cn/20200729185807556.png)

#### json

 Json模块也提供了四个功能：dumps、dump、loads、load，用法跟pickle一致

 **dumps和loads**

```
print(json.dumps(name))  # 注意json dumps生成的是字符串，不是bytes
print(json.dumps(info))
print(json.loads(json.dumps(info)))
```

 输出

```
"Alex"
{"name": "Alex", "age": 20}
{'name': 'Alex', 'age': 20}
```

 **dump和load**

> 需要注意的一点是json在文件中序列化的数据最好是字典的格式。因为读取数据的时候只能以`字典的数据结构`取出一次。

```
# 因为json的load只能load一次，将数据转为字典。所以序列化的数据只能是字典
file = open("F:\\Animation\\10.txt", "w")
info.setdefault("姓名", name)
json.dump(info, file)
file.close()

file = open("F:\\Animation\\10.txt", "r")
print(json.load(file))
file.close()
```

 输出

```
{'name': 'Alex', 'age': 20, '姓名': 'Alex'}
```

[![img](E:\Development\Typora\images\2020072920245652.png)](https://img-blog.csdnimg.cn/2020072920245652.png)



#### json与pickle的比较

**JSON:**

- 优点：跨语言(不同语言间的数据传递可用json交接)、体积小
- 缺点：只能支持int\str\list\tuple\dict

**Pickle:**

- 优点：专为python设计，支持python所有的数据类型
- 缺点：只能在python中使用，存储数据占空间大

> 其实就是Json和序列化的一个比较



### 加密算法

#### hash

 Hash，一般翻译做“散列”，也有直接音译为”哈希”的，就是把任意长度的输入（又叫做预映射，pre-image），通过散列算法，变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，而不可能从散列值来唯一的确定输入值。

 简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

 HASH主要用于信息安全领域中加密算法，他把一些不同长度的信息转化成杂乱的128位的编码里,叫做HASH值.也可以说，hash就是找到一种数据内容和数据存放地址之间的映射关系



#### MD5

**什么是MD5算法**

 MD5讯息摘要演算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码杂凑函数，可以产生出一个128位的散列值（hash value），用于确保信息传输完整一致。MD5的前身有MD2、MD3和MD4。

**MD5功能**

 输入任意长度的信息，经过处理，输出为128位的信息（数字指纹）；

 不同的输入得到的不同的结果（唯一性）；

**MD5算法的特点**

1. 压缩性：任意长度的数据，算出的MD5值的长度都是固定的
2. 容易计算：从原数据计算出MD5值很容易
3. 抗修改性：对原数据进行任何改动，修改一个字节生成的MD5值区别也会很大
4. 强抗碰撞：已知原数据和MD5，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。

**MD5算法是否可逆？**

 MD5不可逆的原因是其是一种散列函数，使用的是hash算法，在计算过程中原文的部分信息是丢失了的。

**MD5用途**

1. 防止被篡改：
   - 比如发送一个电子文档，发送前，我先得到MD5的输出结果a。然后在对方收到电子文档后，对方也得到一个MD5的输出结果b。如果a与b一样就代表中途未被篡改。
   - 比如我提供文件下载，为了防止不法分子在安装程序中添加木马，我可以在网站上公布由安装文件得到的MD5输出结果。
   - SVN在检测文件是否在CheckOut后被修改过，也是用到了MD5.
2. 防止直接看到明文：
   - 现在很多网站在数据库存储用户的密码的时候都是存储用户密码的MD5值。这样就算不法分子得到数据库的用户密码的MD5值，也无法知道用户的密码。（比如在UNIX系统中用户的密码就是以MD5（或其它类似的算法）经加密后存储在文件系统中。当用户登录的时候，系统把用户输入的密码计算成MD5值，然后再去和保存在文件系统中的MD5值进行比较，进而确定输入的密码是否正确。通过这样的步骤，系统在并不知道用户密码的明码的情况下就可以确定用户登录系统的合法性。这不但可以避免用户的密码被具有系统管理员权限的用户知道，而且还在一定程度上增加了密码被破解的难度。）
3. 防止抵赖（数字签名）：
   - 这需要一个第三方认证机构。例如A写了一个文件，认证机构对此文件用MD5算法产生摘要信息并做好记录。若以后A说这文件不是他写的，权威机构只需对此文件重新产生摘要信息，然后跟记录在册的摘要信息进行比对，相同的话，就证明是A写的了。这就是所谓的“数字签名”。

**使用**

```
import hashlib

md5 = hashlib.md5()
md5.update("你好呀".encode("utf-8"))
print(md5.digest())
print(md5.hexdigest())

md5.update("你好呀".encode("utf-8"))  # 相加
print(md5.digest())
print(md5.hexdigest())

md5_2 = hashlib.md5("你好呀".encode("utf-8"))
print(md5_2.hexdigest())
```

 输出

```
b'Oe\xfd\xb3>\x0f+\xd0\xdek\xd1\xb4\x1f\xde\xa9h'
4f65fdb33e0f2bd0de6bd1b41fdea968
b'\x0e\x10\xe6\xe2.\x82_\x9e\xbfe\xa7\x12\xde\x80\xd0\xec'
0e10e6e22e825f9ebf65a712de80d0ec
4f65fdb33e0f2bd0de6bd1b41fdea968
```

> 一般还需要加盐



#### SHA-1

 安全哈希算法（Secure Hash Algorithm）主要适用于数字签名标准（Digital Signature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA）。对于长度小于2^64位的消息，SHA1会产生一个160位的消息摘要。当接收到消息的时候，这个消息摘要可以用来验证数据的完整性。

 SHA是美国国家安全局设计的，由美国国家标准和技术研究院发布的一系列密码散列函数。

 由于MD5和SHA-1于2005年被山东大学的教授王小云破解了，科学家们又推出了SHA224, SHA256, SHA384, SHA512，当然位数越长，破解难度越大，但同时生成加密的消息摘要所耗时间也更长。**目前最流行的是加密算法是SHA-256 .**

**使用**

```
print("-----SH1-----")
print(hashlib.sha1("你好呀".encode("utf-8")).hexdigest())
print(hashlib.sha1("你好呀".encode("utf-8")).hexdigest())

print("-----SH256-----")
print(hashlib.sha256("你好呀".encode("utf-8")).hexdigest())
print(hashlib.sha256("你好呀".encode("utf-8")).hexdigest())
```

 输出

```
-----SH1-----
06161148546813f145022d12c7df76b8546edcea
06161148546813f145022d12c7df76b8546edcea
-----SH256-----
d2033138c3b3be1321ad29d0aff15a4b1b47934a2b91afcea6f59b96a9fed115
d2033138c3b3be1321ad29d0aff15a4b1b47934a2b91afcea6f59b96a9fed115
```



#### MD5与SHA-1的比较

 由于MD5与SHA-1均是从MD4发展而来，它们的结构和强度等特性有很多相似之处，SHA-1与MD5的最大区别在于其摘要比MD5摘要长32 比特。对于强行攻击，产生任何一个报文使之摘要等于给定报文摘要的难度：MD5是2128数量级的操作，SHA-1是2160数量级的操作。产生具有相同摘要的两个报文的难度：MD5是264是数量级的操作，SHA-1 是280数量级的操作。因而,SHA-1对强行攻击的强度更大。但由于SHA-1的循环步骤比MD5多80:64且要处理的缓存大160比特:128比特，SHA-1的运行速度比MD5慢。



### shutil & zipfile & tarfile

高级的 文件、文件夹、压缩包 处理模块

**shutil.copyfileobj(fsrc, fdst[, length])**

将文件内容拷贝到另一个文件中

```
import shutilshutil.copyfileobj(open('old.xml','r'), open('new.xml', 'w'))
```

**shutil.copyfile(src, dst)**

拷贝文件

```
shutil.copyfile('f1.log', 'f2.log') #目标文件无需存在
```

**shutil.copymode(src, dst)**

仅拷贝权限。内容、组、用户均不变

```
shutil.copymode('f1.log', 'f2.log') #目标文件必须存在
```

**shutil.copystat(src, dst)**

仅拷贝状态的信息，包括：mode bits, atime, mtime, flags

```
shutil.copystat('f1.log', 'f2.log') #目标文件必须存在
```

**shutil.copy(src, dst)**

拷贝文件和权限

```
import shutilshutil.copy('f1.log', 'f2.log')
```

**shutil.copy2(src, dst)**

拷贝文件和状态信息

```
import shutilshutil.copy2('f1.log', 'f2.log')
```

**shutil.ignore_patterns(\*patterns)**

**shutil.copytree(src, dst, symlinks=False, ignore=None)**

递归的去拷贝文件夹

```
import shutilshutil.copytree('folder1', 'folder2', ignore=shutil.ignore_patterns('*.pyc', 'tmp*')) #目标目录不能存在，注意对folder2目录父级目录要有可写权限，ignore的意思是排除
```

**shutil.rmtree(path[, ignore_errors[, onerror]])**

递归的去删除文件

```
import shutilshutil.rmtree('folder1')
```

**shutil.move(src, dst)**

递归的去移动文件，它类似mv命令，其实就是重命名。

```
import shutilshutil.move('folder1', 'folder3')
```

**shutil.make_archive(base_name, format,…)**

创建压缩包并返回文件路径，例如：zip、tar

可选参数如下：

- base_name： 压缩包的文件名，也可以是压缩包的路径。只是文件名时，则保存至当前目录，否则保存至指定路径，

如 data_bak =>保存至当前路径

如：/tmp/data_bak =>保存至/tmp/

- format： 压缩包种类，“zip”, “tar”, “bztar”，“gztar”
- root_dir： 要压缩的文件夹路径（默认当前目录）
- owner： 用户，默认当前用户
- group： 组，默认当前组
- logger： 用于记录日志，通常是logging.Logger对象

```
#将 /data 下的文件打包放置当前程序目录
import shutil
ret = shutil.make_archive("data_bak", 'gztar', root_dir='/data')

#将 /data下的文件打包放置 /tmp/目录
import shutil
ret = shutil.make_archive("/tmp/data_bak", 'gztar', root_dir='/data')
```

shutil 对压缩包的处理是调用 ZipFile 和 TarFile 两个模块来进行的，详细：

zipfile压缩&解压缩

```
import zipfile
# 压缩
z = zipfile.ZipFile('laxi.zip', 'w')
z.write('a.log')
z.write('data.data')
z.close()

# 解压
z = zipfile.ZipFile('laxi.zip', 'r')
z.extractall(path='.')
z.close()
```

tarfile压缩&解压缩

```
import tarfile
# 压缩
>>> t=tarfile.open('/tmp/egon.tar','w')
>>> t.add('/test1/a.py',arcname='a.bak')
>>> t.add('/test1/b.py',arcname='b.bak')
>>> t.close()

# 解压
>>> t=tarfile.open('/tmp/egon.tar','r')
>>> t.extractall('/egon')
>>> t.close()
```



 **自定义压缩**

```
import zipfile
import os


zip_file = zipfile.ZipFile("F:\\Zip.zip","w")
file_list = []
for root_dir,dirs,files in os.walk("F:\\Animation"):
    print(root_dir,dirs,files)
    for file_name in files:
       file_list.append(os.path.join(root_dir, file_name))

for file in file_list:
    zip_file.write(file)


zip_file.close()
```





### re(正则表达式)

#### 引子

 请从以下文件里取出所有的手机号

```
姓名        地区    身高    体重    电话
况咏蜜     北京    171    48    13651054608
王心颜     上海    169    46    13813234424
马纤羽     深圳    173    50    13744234523
乔亦菲     广州    172    52    15823423525
罗梦竹     北京    175    49    18623423421
刘诺涵     北京    170    48    18623423765
岳妮妮     深圳    177    54    18835324553
贺婉萱     深圳    174    52    18933434452
叶梓萱    上海     171    49    18042432324
杜姗姗   北京      167    49    13324523342
```

 你能想到的办法是什么？

 必然是下面这种吧？

```
f = open("兼职白领学生空姐模特护士联系方式.txt",'r',encoding="gbk")
phones = []

for line in f:
    name,city,height,weight,phone = line.split()
    if phone.startswith('1') and len(phone) == 11:
        phones.append(phone)
        
print(phones)
```

 有没有更简单的方式？

 手机号是有规则的，都是数字且是11位，再严格点，就都是1开头，如果能把这样的规则写成代码，直接拿规则代码匹配文件内容不就行了？

```
file = open("F:\\Animation\\2.txt", "r")
file_info = file.read()
find_info = re.findall("[0-9]{11}", file_info)
print("类型：{0}，数据：{1}".format(type(find_info), find_info))
```

 输出

```
类型：<class 'list'>，数据：['13744234523', '15823423525', '18623423421', '18623423765', '18835324553', '18933434452', '18042432324']
```



#### re

 正则表达式就是字符串的匹配规则，在多数编程语言里都有相应的支持，python里对应的模块是re



#### 常用的表达式规则

```
'.'     默认匹配除\n之外的任意一个字符，若指定flag DOTALL,则匹配任意字符，包括换行
'^'     匹配字符开头，若指定flags MULTILINE,这种也可以匹配上(r"^a","\nabc\neee",flags=re.MULTILINE)
'$'     匹配字符结尾， 若指定flags MULTILINE ,re.search('foo.$','foo1\nfoo2\n',re.MULTILINE).group() 会匹配到foo1
'*'     匹配*号前的字符0次或多次， re.search('a*','aaaabac')  结果'aaaa'
'+'     匹配前一个字符1次或多次，re.findall("ab+","ab+cd+abb+bba") 结果['ab', 'abb']
'?'     匹配前一个字符1次或0次 ,re.search('b?','alex').group() 匹配b 0次
'{m}'   匹配前一个字符m次 ,re.search('b{3}','alexbbbs').group()  匹配到'bbb'
'{n,m}' 匹配前一个字符n到m次，re.findall("ab{1,3}","abb abc abbcbbb") 结果'abb', 'ab', 'abb']
'|'     匹配|左或|右的字符，re.search("abc|ABC","ABCBabcCD").group() 结果'ABC'
'(...)' 分组匹配， re.search("(abc){2}a(123|45)", "abcabca456c").group() 结果为'abcabca45'
'\A'    只从字符开头匹配，re.search("\Aabc","alexabc") 是匹配不到的，相当于re.match('abc',"alexabc") 或^
'\Z'    匹配字符结尾，同$ 
'\d'    匹配数字0-9
'\D'    匹配非数字
'\w'    匹配[A-Za-z0-9]
'\W'    匹配非[A-Za-z0-9]
's'     匹配空白字符、\t、\n、\r , re.search("\s+","ab\tc1\n3").group() 结果 '\t'
'(?P...)' 分组匹配 re.search("(?P[0-9]{4})(?P[0-9]{2})(?P[0-9]{4}
```



#### re的匹配语法有以下几种

- re.compile 指定一个匹配规则
- re.match 从头开始匹配
- re.search 匹配包含
- re.findall 把所有匹配到的字符放到以列表中的元素返回
- re.split 以匹配到的字符当做列表分隔符
- re.sub 匹配字符并替换
- re.fullmatch 全部匹配

#### Flags标志符

- re.I(re.IGNORECASE): 忽略大小写（括号内是完整写法，下同）
- re.M(MULTILINE): 多行模式，改变’^’和’$’的行为
- re.S(DOTALL): 改变’.’的行为,make the ‘.’ special character match any character at all, including a newline; without this flag, ‘.’ will match anything except a newline.
- re.X(re.VERBOSE) 可以给你的表达式写注释，使其更可读，下面这2个意思一样

```
a = re.compile(r"""\d + # the integral part
                \. # the decimal point
                \d * # some fractional digits""", 
                re.X)
b = re.compile(r"\d+\.\d*")
```



#### group

 可以拿到括号中匹配的值

```
print(re.match("(AB|CD|EF|GH|I|12|54)(AB|CD|EF|GH|I|12|54)", "ABCDEFGHI123456").groups())
print(re.match("(AB|CD|EF|GH|I|12|54)(AB|CD|EF|GH|I|12|54)", "ABCDEFGHI123456").group())
```

 输出

```
('AB', 'CD')
ABCD
```



#### re.compile(pattern, flags=0)

 Compile a regular expression pattern into a regular expression object, which can be used for matching using its match(), search() and other methods, described below.

 例子

```
compile_email = re.compile('\w+@\w+\.(com|cn|edu)')
print(compile_email.match("alex@oldboyedu.cn"))
```

 输出

```
<re.Match object; span=(0, 17), match='alex@oldboyedu.cn'>
```

 **等同于**

```
print(re.match('\w+@\w+\.(com|cn|edu)', "alex@oldboyedu.cn"))
```



#### re.match(pattern, string, flags=0)

从起始位置开始根据模型去字符串中匹配指定内容，匹配单个

- pattern 正则表达式
- string 要匹配的字符串
- flags 标志位，用于控制正则表达式的匹配方式

```
print(re.match("^jcl[1-3]{2}(AB|1|2){2}.*$", "jcl22AB1dsa")) #如果能匹配到就返回一个可调用的对象，否则返回None
```

 输出

```
<re.Match object; span=(0, 11), match='jcl22AB1dsa'>
```



#### re.search(pattern, string, flags=0)

 根据模型去字符串中匹配指定内容，匹配单个

```
print(re.search("jcl[1-3]{2}(AB|1|2){2}", "dsadajcl22AB1dsadjasoibjcl22AB1dsa"))
```

 输出

```
<re.Match object; span=(5, 13), match='jcl22AB1'>
```



#### re.findall(pattern, string, flags=0)

 match and search均用于匹配单值，即：只能匹配字符串中的一个，如果想要匹配到字符串中所有符合条件的元素，则需要使用 findall。

```
file = open("F:\\Animation\\2.txt", "r")
file_info = file.read()
find_info = re.findall("[0-9]{11}", file_info)
print("类型：{0}，数据：{1}".format(type(find_info), find_info))
```

 输出

```
类型：<class 'list'>，数据：['13744234523', '15823423525', '18623423421', '18623423765', '18835324553', '18933434452', '18042432324']
```



#### **re.sub(pattern, repl, string, count=0, flags=0)**

 用于替换匹配的字符串,比str.replace功能更加强大

```
print(re.sub("(ldh|zxy)+", "*", "dsaldhdjsiaolzxydasldhdusazxy"))
print(re.sub("(ldh|zxy)+", "*", "dsaldhdjsiaolzxydasldhdusazxy", count=2))
```

 输出

```
dsa*djsiaol*das*dusa*
dsa*djsiaol*dasldhdusazxy
```



#### re.split(pattern, string, maxsplit=0, flags=0)

 用匹配到的值做为分割点，把值分割成列表

```
print(re.split("(ldh|zxy)+", "dsaldhdjsiaolzxydasldhdusazxy"))
```

 输出

```
['dsa', 'ldh', 'djsiaol', 'zxy', 'das', 'ldh', 'dusa', 'zxy', '']
```



#### **re.fullmatch(pattern, string, flags=0)**

 整个字符串匹配成功就返回re object, 否则返回None

```
print(re.fullmatch('\w+@\w+\.(com|cn|edu)', "alex@oldboyedu.cn"))
```

 输出

```
<re.Match object; span=(0, 17), match='alex@oldboyedu.cn'>
```



## 软件开发目录设计规范

### 为什么要设计好目录结构?

 “设计项目目录结构”，就和”代码编码风格”一样，属于个人风格问题。对于这种风格上的规范，一直都存在两种态度:

1. 一类同学认为，这种个人风格问题”无关紧要”。理由是能让程序work就好，风格问题根本不是问题。
2. 另一类同学认为，规范化能更好的控制程序结构，让程序具有更高的可读性。

 我是比较偏向于后者的，因为我是前一类同学思想行为下的直接受害者。我曾经维护过一个非常不好读的项目，其实现的逻辑并不复杂，但是却耗费了我非常长的时间去理解它想表达的意思。从此我个人对于提高项目可读性、可维护性的要求就很高了。”项目目录结构”其实也是属于”可读性和可维护性”的范畴，我们设计一个层次清晰的目录结构，就是为了达到以下两点:

1. **可读性高**: 不熟悉这个项目的代码的人，一眼就能看懂目录结构，知道程序启动脚本是哪个，测试目录在哪儿，配置文件在哪儿等等。从而非常快速的了解这个项目。
2. **可维护性高**: 定义好组织规则后，维护者就能很明确地知道，新增的哪个文件和代码应该放在什么目录之下。这个好处是，随着时间的推移，代码/配置的规模增加，项目结构不会混乱，仍然能够组织良好。

 所以，我认为，保持一个层次清晰的目录结构是有必要的。更何况组织一个良好的工程目录，其实是一件很简单的事儿。

### 目录组织方式

 关于如何组织一个较好的Python工程目录结构，已经有一些得到了共识的目录结构。在Stackoverflow的[这个问题](http://stackoverflow.com/questions/193161/what-is-the-best-project-structure-for-a-python-application)上，能看到大家对Python目录结构的讨论。

 这里面说的已经很好了，我也不打算重新造轮子列举各种不同的方式，这里面我说一下我的理解和体会。

 假设你的项目名为foo, 我比较建议的最方便快捷目录结构这样就足够了:

```
Foo/
|-- bin/
|   |-- foo
|
|-- foo/
|   |-- tests/
|   |   |-- __init__.py
|   |   |-- test_main.py
|   |
|   |-- __init__.py
|   |-- main.py
|
|-- docs/
|   |-- conf.py
|   |-- abc.rst
|
|-- setup.py
|-- requirements.txt
|-- README
```

 简要解释一下:

- bin/: 存放项目的一些可执行文件，当然你可以起名script/之类的也行。
- foo/: 存放项目的所有源代码。(1) 源代码中的所有模块、包都应该放在此目录。不要置于顶层目录。(2) 其子目录tests/存放单元测试代码； (3) [程序的入口最好命名为main.py](http://xn--main-uh5fw1ov5evjv8ay97a4rn8uyi99bflk.py/)。
- docs/: 存放一些文档。
- [setup.py](http://setup.py/): 安装、部署、打包的脚本。
- requirements.txt: 存放软件依赖的外部Python包列表。
- README: 项目说明文件。

 除此之外，有一些方案给出了更加多的内容。比如`LICENSE.txt`,`ChangeLog.txt`文件等，我没有列在这里，因为这些东西主要是项目开源的时候需要用到。如果你想写一个开源软件，目录该如何组织，可以参考[这篇文章](http://www.jeffknupp.com/blog/2013/08/16/open-sourcing-a-python-project-the-right-way/)。

 下面，再简单讲一下我对这些目录的理解和个人要求吧。

### 关于README的内容

**这个我觉得是每个项目都应该有的一个文件**，目的是能简要描述该项目的信息，让读者快速了解这个项目。

它需要说明以下几个事项:

1. 软件定位，软件的基本功能。
2. 运行代码的方法: 安装环境、启动命令等。
3. 简要的使用说明。
4. 代码目录结构说明，更详细点可以说明软件的基本原理。
5. 常见问题说明。

### [关于requirements.txt和setup.py](http://xn--requirements-5b6sf83a.xn--txtsetup-1c2n.py/)

#### [setup.py](http://setup.py/)

 一般来说，用`setup.py`来管理代码的打包、安装、部署问题。业界标准的写法是用Python流行的打包工具[setuptools](https://pythonhosted.org/setuptools/setuptools.html#developer-s-guide)来管理这些事情。这种方式普遍应用于开源项目中。不过这里的核心思想不是用标准化的工具来解决这些问题，而是说，**一个项目一定要有一个安装部署工具**，能快速便捷的在一台新机器上将环境装好、代码部署好和将程序运行起来。

 我刚开始接触Python写项目的时候，安装环境、部署代码、运行程序这个过程全是手动完成，遇到过以下问题:

1. 安装环境时经常忘了最近又添加了一个新的Python包，结果一到线上运行，程序就出错了。
2. Python包的版本依赖问题，有时候我们程序中使用的是一个版本的Python包，但是官方的已经是最新的包了，通过手动安装就可能装错了。
3. 如果依赖的包很多的话，一个一个安装这些依赖是很费时的事情。
4. 新同学开始写项目的时候，将程序跑起来非常麻烦，因为可能经常忘了要怎么安装各种依赖。

 `setup.py`可以将这些事情自动化起来，提高效率、减少出错的概率。”复杂的东西自动化，能自动化的东西一定要自动化。”是一个非常好的习惯。

 setuptools的[文档](https://pythonhosted.org/setuptools/setuptools.html#developer-s-guide)比较庞大，刚接触的话，可能不太好找到切入点。学习技术的方式就是看他人是怎么用的，可以参考一下Python的一个Web框架，flask是如何写的: [setup.py](https://github.com/mitsuhiko/flask/blob/master/setup.py)

 当然，简单点自己写个安装脚本（`deploy.sh`）替代`setup.py`也未尝不可。

#### requirements.txt

 这个文件存在的目的是:

1. 方便开发者维护软件的包依赖。将开发过程中新增的包添加进这个列表中，避免在 `setup.py` 安装依赖时漏掉软件包。
2. 方便读者明确项目使用了哪些Python包。

 这个文件的格式是每一行包含一个包依赖的说明，通常是`flask>=0.10`这种格式，要求是这个格式能被`pip`识别，这样就可以简单的通过 `pip install -r requirements.txt`来把所有Python包依赖都装好了。具体格式说明： [点这里](https://pip.readthedocs.org/en/1.1/requirements.html)。

 `pip freeze > requirements`可以自动将我们安装的包名称写入指定文件中



### 关于配置文件的使用方法

 注意，在上面的目录结构中，没有将`conf.py`放在源码目录下，而是放在`docs/`目录下。

 很多项目对配置文件的使用做法是:

1. 配置文件写在一个或多个python文件中，[比如此处的conf.py](http://xn--conf-fr6g9yd58gtzaw71e.py/)。
2. 项目中哪个模块用到这个配置文件就直接通过`import conf`这种形式来在代码中使用配置。

 这种做法我不太赞同:

1. 这让单元测试变得困难（因为模块内部依赖了外部配置）
2. 另一方面配置文件作为用户控制程序的接口，应当可以由用户自由指定该文件的路径。
3. 程序组件可复用性太差，因为这种贯穿所有模块的代码硬编码方式，使得大部分模块都依赖`conf.py`这个文件。

 所以，我认为配置的使用，更好的方式是，

1. 模块的配置都是可以灵活配置的，不受外部配置文件的影响。
2. 程序的配置也是可以灵活控制的。

 能够佐证这个思想的是，用过nginx和mysql的同学都知道，nginx、mysql这些程序都可以自由的指定用户配置。

 所以，不应当在代码中直接`import conf`来使用配置文件。上面目录结构中的`conf.py`，是给出的一个配置样例，不是在写死在程序中直接引用的配置文件。可以通过给`main.py`启动参数指定配置路径的方式来让程序读取配置内容。当然，这里的`conf.py`你可以换个类似的名字，比如`settings.py`。或者你也可以使用其他格式的内容来编写配置文件，比如`settings.yaml`之类的。



## 内存管理和垃圾回收

### 一句话

> **引用计数器为主 标记清除和分代回收为辅 + 缓存机制**



### 引用计数器

#### 环状双向链表 rechain

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110391.png)](https://img-blog.csdnimg.cn/20200730122421160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 在python程序中创建的任何对象都会放在rechain链表中，

```
name = "莫崽子"
age = 18
hobby = ["篮球", "编程"]
```

- 内部会创建一些数据[上一个对象，下一个对象，类型，引用个数]

  name = “莫崽子”

- 内部会创建一些数据[上一个对象，下一个对象，类型，引用个数，val=18]

  age = 18

- 内部会创建一些数据[上一个对象，下一个对象，类型，引用个数，items=元素，元素个数]

  hobby = [“篮球”, “编程”]

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110392.png)](https://img-blog.csdnimg.cn/20200730122504880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 在c语言源码中可以看出每个对象都有相同的值：**PyObject(4个值)**

 由多个元素组成的对象：**PyObject结构体 + ob_size**

 例如：每个类型的对象在创建时都有PyObject中的那4部分数据；list/set/tuple等由多个元素组成对象创建时都有PyVarObject中的那5部分数据



#### 类型封装结构体

- float类型

  ```
  typedef struct {
      PyObject_HEAD
      double ob_fval;
  } PyFloatObject;
  ```

- int类型

  ```
  struct _longobject {
      PyObject_VAR_HEAD
      digit ob_digit[1];
  };
  /* Long (arbitrary precision) integer object interface */
  typedef struct _longobject PyLongObject; /* Revealed in longintrepr.h */
  ```

- str类型

  ```
  typedef struct {
      PyObject_HEAD
      Py_ssize_t length;          /* Number of code points in the string */
      Py_hash_t hash;             /* Hash value; -1 if not set */
      struct {
          unsigned int interned:2;
          /* Character size:
               - PyUnicode_WCHAR_KIND (0):
                 * character type = wchar_t (16 or 32 bits, depending on the
                   platform)
               - PyUnicode_1BYTE_KIND (1):
                 * character type = Py_UCS1 (8 bits, unsigned)
                 * all characters are in the range U+0000-U+00FF (latin1)
                 * if ascii is set, all characters are in the range U+0000-U+007F
                   (ASCII), otherwise at least one character is in the range
                   U+0080-U+00FF
               - PyUnicode_2BYTE_KIND (2):
                 * character type = Py_UCS2 (16 bits, unsigned)
                 * all characters are in the range U+0000-U+FFFF (BMP)
                 * at least one character is in the range U+0100-U+FFFF
               - PyUnicode_4BYTE_KIND (4):
                 * character type = Py_UCS4 (32 bits, unsigned)
                 * all characters are in the range U+0000-U+10FFFF
                 * at least one character is in the range U+10000-U+10FFFF
               */
          unsigned int kind:3;
          unsigned int compact:1;
          unsigned int ascii:1;
          unsigned int ready:1;
          unsigned int :24;
      } state;
      wchar_t *wstr;              /* wchar_t representation (null-terminated) */
  } PyASCIIObject;
  typedef struct {
      PyASCIIObject _base;
      Py_ssize_t utf8_length;     /* Number of bytes in utf8, excluding the
                                         * terminating \0. */
      char *utf8;                 /* UTF-8 representation (null-terminated) */
      Py_ssize_t wstr_length;     /* Number of code points in wstr, possible
                                         * surrogates count as two code points. */
  } PyCompactUnicodeObject;
  typedef struct {
      PyCompactUnicodeObject _base;
      union {
          void *any;
          Py_UCS1 *latin1;
          Py_UCS2 *ucs2;
          Py_UCS4 *ucs4;
      } data;                     /* Canonical, smallest-form Unicode buffer */
  } PyUnicodeObject;
  ```

- list类型

  ```
  typedef struct {
      PyObject_VAR_HEAD
      PyObject **ob_item;
      Py_ssize_t allocated;
  } PyListObject;
  ```

- tuple类型

  ```
  typedef struct {
      PyObject_VAR_HEAD8
      PyObject *ob_item[1];
  } PyTupleObject;
  ```

- dict类型

  ```
  typedef struct {
      PyObject_HEAD
      Py_ssize_t ma_used;
      PyDictKeysObject *ma_keys;
      PyObject **ma_values;
  } PyDictObject;
  ```

例子

```
data = 3.14

内部会创建：
	_ob_next = refchain中的上一个对象
    _ob_prev = refchain中的下一个对象
    ob_refcnt = 1
    ob_type = float
    ob_fval = 3.14
```

通过常见结构体可以基本了解到本质上每个对象内部会存储的数据。

扩展：在结构体部分你应该发现了`str类型`比较繁琐，那是因为python字符串在处理时需要考虑到编码的问题，在内部规定（见源码结构体）：

- 字符串只包含ascii，则每个字符用1个字节表示，即：latin1
- 字符串包含中文等，则每个字符用2个字节表示，即：ucs2
- 字符串包含emoji等，则每个字符用4个字节表示，即：ucs4



#### 引用计数器

```
v1 = 3.14
v2 = 999
v3 = (1,2,3)
```

 当python程序运行时，会根据数据类型的不同找到其对应的结构体，根据结构体中的字段来进行创建相关的数据，然后将对象添加到rechain双向链表中。

 在C源码中有两个关键的结构体：`pyObject`，`pyVarObject`

 每个对象中有ob_refcnt就是引用计数器，值默认为1，当有其他变量引用对象时，引用计数器就会发生变化。

- 引用

  ```
  a = 99999
  b = a
  ```

- 删除引用

  ```
  a = 99999
  b = a
  del b  # 删除变量；b对应的引用计数器为-1
  del a  # 删除变量：a对象的应用计数器为-1
  ```

  当一个对象的引用计数器为0时，意味着没有人再使用这个对象了，这个对象就是垃圾，垃圾回收

  回收：1.对象从refchain链表中移除 2.将对象销毁，内存归还

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110393.png)](https://img-blog.csdnimg.cn/20200730142335229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### 循环引用问题

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110394.png)](https://img-blog.csdnimg.cn/20200730143035246.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 标记清除

 目的：为了解决引用计数器循环引用的不足

 实现：在python的底层再维护一个链表，链表中专门放那些可能存在循环的对象(list/tuple/dict/set)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110495.png)](https://img-blog.csdnimg.cn/20200730144900377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 在Python内部某种情况下触发，会去扫描`可能存在循环应用的链表`中的每个元素，检查是否有循环引用，如果有则让双方的引用计数器-1；如果是0则垃圾回收



### 分代回收

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110496.png)](https://img-blog.csdnimg.cn/20200730145415316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 将可能存在循环引用的对象维护成3个链表

- 0代：0代中对象个数达到700个扫描一次(是垃圾就回收，不是垃圾)
- 1代：0代扫描10次，则1代扫描一次
- 2代：1代扫描10次，则2代扫描一次



### 情景模拟

第一步：当创建对象`age=19`时，会将对象添加到refchain链表中。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110497.png)](https://img-blog.csdnimg.cn/20200730152433748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

第二步：当创建对象`num_list = [11,22]`时，会将列表对象添加到 refchain 和 generations 0代中。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166282938110498.png)](https://img-blog.csdnimg.cn/20200730152459476.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

第三步：新创建对象使generations的0代链表上的对象数量大于阈值700时，要对链表上的对象进行扫描检查。

当0代大于阈值后，底层不是直接扫描0代，而是先判断2、1是否也超过了阈值。

- 如果2、1代未达到阈值，则扫描0代，并让1代的 count + 1 。
- 如果2代已达到阈值，则将2、1、0三个链表拼接起来进行全扫描，并将2、1、0代的count重置为0.
- 如果1代已达到阈值，则讲1、0两个链表拼接起来进行扫描，并将所有1、0代的count重置为0.

对拼接起来的链表在进行扫描时，主要就是剔除循环引用和销毁垃圾，详细过程为：

- 扫描链表，把每个对象的引用计数器拷贝一份并保存到 `gc_refs`中，保护原引用计数器。

- 再次扫描链表中的每个对象，并检查是否存在循环引用，如果存在则让各自的`gc_refs`减 1 。

- 再次扫描链表，将 `gc_refs` 为 0 的对象移动到`unreachable`链表中；不为0的对象直接升级到下一代链表中。

- 处理unreachable

  链表中的对象的 析构函数 和 弱引用，不能被销毁的对象升级到下一代链表，能销毁的保留在此链表。

  - 析构函数，指的就是那些定义了`__del__`方法的对象，需要执行之后再进行销毁处理。

- 弱引用，

- 最后将 `unreachable` 中的每个对象销毁并在refchain链表中移除（不考虑缓存机制）。

至此，垃圾回收的过程结束。



### 缓存机制

 从上文大家可以了解到当对象的引用计数器为0时，就会被销毁并释放内存。而实际上他不是这么的简单粗暴，因为反复的创建和销毁会使程序的执行效率变低。Python中引入了“缓存机制”机制。
 例如：引用计数器为0时，不会真正销毁对象，而是将他放到一个名为 `free_list` 的链表中，之后会再创建对象时不会在重新开辟内存，而是在free_list中将之前的对象来并重置内部的值来使用。

- float类型，维护的free_list链表最多可缓存100个float对象。

  ```
  v1 = 3.14    # 开辟内存来存储float对象，并将对象添加到refchain链表。
  print( id(v1) ) # 内存地址：4436033488
  del v1    # 引用计数器-1，如果为0则在rechain链表中移除，不销毁对象，而是将对象添加到float的free_list.
  v2 = 9.999    # 优先去free_list中获取对象，并重置为9.999，如果free_list为空才重新开辟内存。
  print( id(v2) ) # 内存地址：4436033488
  
  # 注意：引用计数器为0时，会先判断free_list中缓存个数是否满了，未满则将对象缓存，已满则直接将对象销毁。
  ```

- int类型，不是基于free_list，而是维护一个small_ints链表保存常见数据（小数据池），小数据池范围：`-5 <= value < 257`。即：重复使用这个范围的整数时，不会重新开辟内存。

  ```
  v1 = 38    # 去小数据池small_ints中获取38整数对象，将对象添加到refchain并让引用计数器+1。
  print( id(v1))  #内存地址：4514343712
  v2 = 38 # 去小数据池small_ints中获取38整数对象，将refchain中的对象的引用计数器+1。
  print( id(v2) ) #内存地址：4514343712
  
  # 注意：在解释器启动时候-5~256就已经被加入到small_ints链表中且引用计数器初始化为1，代码中使用的值时直接去small_ints中拿来用并将引用计数器+1即可。另外，small_ints中的数据引用计数器永远不会为0（初始化时就设置为1了），所以也不会被销毁。
  ```

- str类型，维护`unicode_latin1[256]`链表，内部将所有的`ascii字符`缓存起来，以后使用时就不再反复创建。

  ```
  v1 = "A"
  print( id(v1) ) # 输出：4517720496
  del v1
  v2 = "A"
  print( id(v1) ) # 输出：4517720496
  
  # 除此之外，Python内部还对字符串做了驻留机制，针对那么只含有字母、数字、下划线的字符串（见源码Objects/codeobject.c），如果内存中已存在则不会重新在创建而是使用原来的地址里（不会像free_list那样一直在内存存活，只有内存中有才能被重复利用）。
  v1 = "wupeiqi"
  v2 = "wupeiqi"
  print(id(v1) == id(v2)) # 输出：True
  ```

- list类型，维护的free_list数组最多可缓存80个list对象。

  ```
  v1 = [11,22,33]
  print( id(v1) ) # 输出：4517628816
  del v1
  v2 = ["武","沛齐"]
  print( id(v2) ) # 输出：4517628816
  ```

- tuple类型，维护一个free_list数组且数组容量20，数组中元素可以是链表且每个链表最多可以容纳2000个元组对象。元组的free_list数组在存储数据时，是按照元组可以容纳的个数为索引找到free_list数组中对应的链表，并添加到链表中。

  ```
  v1 = (1,2)
  print( id(v1) )
  del v1  # 因元组的数量为2，所以会把这个对象缓存到free_list[2]的链表中。
  v2 = ("武沛齐","Alex")  # 不会重新开辟内存，而是去free_list[2]对应的链表中拿到一个对象来使用。
  print( id(v2) )
  ```

- dict类型，维护的free_list数组最多可缓存80个dict对象。

  ```
  v1 = {"k1":123}
  print( id(v1) )  # 输出：4515998128
  del v1
  v2 = {"name":"武沛齐","age":18,"gender":"男"}
  print( id(v1) ) # 输出：4515998128
  ```



### c语言源码分析(int为例)

#### 创建

```
age = 19
```

 当在python中创建一个整型数据时，底层会触发他的如下源码:

```
PyObject *
    PyLong_FromLong(long ival)
{
    PyLongObject *v;
    ...
        // 优先去小数据池中检查，如果在范围内则直接获取不再重新开辟内存。（ -5 <= value < 257）
        CHECK_SMALL_INT(ival);
    ...
        // 非小数字池中的值，重新开辟内存并初始化
        v = _PyLong_New(ndigits);
    if (v != NULL) {
        digit *p = v->ob_digit;
        Py_SIZE(v) = ndigits*sign;
        t = abs_ival;
        ...
    }
    return (PyObject *)v;
}
#define NSMALLNEGINTS           5
#define NSMALLPOSINTS           257
#define CHECK_SMALL_INT(ival) \
        do if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) { \
            return get_small_int((sdigit)ival); \
        } while(0)
static PyObject *
    get_small_int(sdigit ival)
{
    PyObject *v;
    v = (PyObject *)&small_ints[ival + NSMALLNEGINTS];
    // 引用计数器 + 1
    Py_INCREF(v);
    ...
        return v;
}
PyLongObject *
    _PyLong_New(Py_ssize_t size)
{
    // 创建PyLongObject的指针变量
    PyLongObject *result;
    ...
        // 根据长度进行开辟内存
        result = PyObject_MALLOC(offsetof(PyLongObject, ob_digit) +
                                 size*sizeof(digit));
    ...
        // 对内存中的数据进行初始化并添加到refchain链表中。
        return (PyLongObject*)PyObject_INIT_VAR(result, &PyLong_Type, size);
}
// Include/objimpl.h
#define PyObject_NewVar(type, typeobj, n) \
                    ( (type *) _PyObject_NewVar((typeobj), (n)) )
static inline PyVarObject*
    _PyObject_INIT_VAR(PyVarObject *op, PyTypeObject *typeobj, Py_ssize_t size)
{
    assert(op != NULL);
    Py_SIZE(op) = size;
    // 对象初始化
    PyObject_INIT((PyObject *)op, typeobj);
    return op;
}
#define PyObject_INIT(op, typeobj) \
        _PyObject_INIT(_PyObject_CAST(op), (typeobj))
static inline PyObject*
    _PyObject_INIT(PyObject *op, PyTypeObject *typeobj)
{
    assert(op != NULL);
    Py_TYPE(op) = typeobj;
    if (PyType_GetFlags(typeobj) & Py_TPFLAGS_HEAPTYPE) {
        Py_INCREF(typeobj);
    }
    // 对象初始化，并把对象加入到refchain链表。
    _Py_NewReference(op);
    return op;
}
// Objects/object.c
// 维护了所有对象的一个环状双向链表
static PyObject refchain = {&refchain, &refchain};
void
    _Py_AddToAllObjects(PyObject *op, int force)
{
    if (force || op->_ob_prev == NULL) {
        op->_ob_next = refchain._ob_next;
        op->_ob_prev = &refchain;
        refchain._ob_next->_ob_prev = op;
        refchain._ob_next = op;
    }
}
void
    _Py_NewReference(PyObject *op)
{
    _Py_INC_REFTOTAL;
    // 引用计数器初始化为1。
    op->ob_refcnt = 1;
    // 对象添加到双向链表refchain中。
    _Py_AddToAllObjects(op, 1);
    _Py_INC_TPALLOCS(op);
}
```



#### 引用

```
value = 69
data = value
```

 类似于出现这种引用关系时，内部其实就是将对象的引用计数器+1，源码同float类型引用。



#### 销毁

```
value = 699
del value
```

 在项目中如果出现这种删除的语句，则内部会将引用计数器-1，如果引用计数器减为0，则直接进行垃圾回收。（int类型是基于小数据池而不是free_list做的缓存，所以不会在销毁时缓存数据）。
 C源码执行流程如下：

```
// Include/object.h
static inline void _Py_DECREF(const char *filename, int lineno,
                              PyObject *op)
{
    (void)filename; /* may be unused, shut up -Wunused-parameter */
    (void)lineno; /* may be unused, shut up -Wunused-parameter */
    _Py_DEC_REFTOTAL;
    // 引用计数器-1，如果引用计数器为0，则执行 _Py_Dealloc去垃圾回收。
    if (--op->ob_refcnt != 0) {
        #ifdef Py_REF_DEBUG
        if (op->ob_refcnt < 0) {
            _Py_NegativeRefcount(filename, lineno, op);
        }
        #endif
    }
    else {
        _Py_Dealloc(op);
    }
}
#define Py_DECREF(op) _Py_DECREF(__FILE__, __LINE__, _PyObject_CAST(op))
// Objects/object.c
void
    _Py_Dealloc(PyObject *op)
{
    // 找到int类型的 tp_dealloc 函数（int类中没有定义tp_dealloc函数，需要去父级PyBaseObject_Type中找tp_dealloc函数）
    // 此处体现所有的类型都继承object
    destructor dealloc = Py_TYPE(op)->tp_dealloc;
    // 在refchain双向链表中摘除此对象。
    _Py_ForgetReference(op);
    // 执行int类型的 tp_dealloc 函数，去进行垃圾回收。
    (*dealloc)(op);
}
void
    _Py_ForgetReference(PyObject *op)
{
    ...
        // 在refchain链表中移除此对象
        op->_ob_next->_ob_prev = op->_ob_prev;
    op->_ob_prev->_ob_next = op->_ob_next;
    op->_ob_next = op->_ob_prev = NULL;
    _Py_INC_TPFREES(op);
}
// Objects/longobjet.c
PyTypeObject PyLong_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
        "int",                                      /* tp_name */
        offsetof(PyLongObject, ob_digit),           /* tp_basicsize */
        sizeof(digit),                              /* tp_itemsize */
        0,                                          /* tp_dealloc */
        ...
        PyObject_Del,                               /* tp_free */
};
Objects/typeobject.c
    PyTypeObject PyBaseObject_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
        "object",                                   /* tp_name */
        sizeof(PyObject),                           /* tp_basicsize */
        0,                                          /* tp_itemsize */
        object_dealloc,                             /* tp_dealloc */
        ...
        PyObject_Del,                               /* tp_free */
};
static void
    object_dealloc(PyObject *self)
{
    // 调用int类型的 tp_free，即：PyObject_Del去销毁对象。
    Py_TYPE(self)->tp_free(self);
}
```



## 面向对象

### **编程范式**

 编程是 程序 员 用特定的语法+数据结构+算法组成的代码来告诉计算机如何执行任务的过程 ， 一个程序是程序员为了得到一个任务结果而编写的一组指令的集合，正所谓条条大路通罗马，实现一个任务的方式有很多种不同的方式， 对这些不同的编程方式的特点进行归纳总结出来的编程方式类别，即为编程范式。 不同的编程范式本质上代表对各种类型的任务采取的不同的解决问题的思路， 大多数语言只支持一种编程范式，当然也有些语言可以同时支持多种编程范式。 两种最重要的编程范式分别是面向过程编程和面向对象编程。

### **面向过程编程(Procedural Programming)**

 Procedural programming uses a list of instructions to tell the computer what to do step-by-step.

 面向过程编程依赖 - 你猜到了- procedures，一个procedure包含一组要被进行计算的步骤， 面向过程又被称为top-down languages， 就是程序从上到下一步步执行，一步步从上到下，从头到尾的解决问题 。基本设计思路就是程序一开始是要着手解决一个大的问题，然后把一个大问题分解成很多个小问题或子过程，这些子过程再执行的过程再继续分解直到小问题足够简单到可以在一个小步骤范围内解决。

 举个典型的面向过程的例子， 有个需求是对网站日志进行分析，生成邮件报告，整个流程分以下几步：

1. 到各台服务器上收集日志，因为有多台网站服务器，共同对外提供服务
2. 对日志进行各种维度分析，比如pv,uv, 来源地区、访问的设备等
3. 生成报告，发送邮件

代码如下

```
# 1 整合日志
def collect_logs():
    print("log on server A ,get access.log")
    print("log on server B ,get access.log")
    print("log on server C ,get access.log")
    print("combine logs in to one file")
# 2 日志分析
def log_analyze(log_file):
    print("pv、uv分析....")
    print("用户来源分析....")
    print("访问的设备来源分析....")
    print("页面停留时间分析....")
    print("入口页面分析....")
# 3 生成报告并发送
def send_report(report_data):
    print("connect email server...")
    print("send email....")
def main():
    collect_logs()
    log_analyze('my_db')
    send_report()
if __name__ == '__main__':
    main()
```

 这样做的问题也是显而易见的，就是如果你要对程序进行修改，对你修改的那部分有依赖的各个部分你都也要跟着修改， 举个例子，如果程序开头你设置了一个变量值 为1 ， 但如果其它子过程依赖这个值 为1的变量才能正常运行，那如果你改了这个变量，那这个子过程你也要修改，假如又有一个其它子程序依赖这个子过程 ， 那就会发生一连串的影响，随着程序越来越大， 这种编程方式的维护难度会越来越高。

 所以我们一般认为， 如果你只是写一些简单的脚本，去做一些一次性任务，用面向过程的方式是极好的，但如果你要处理的任务是复杂的，且需要不断迭代和维护 的， 那还是用面向对象最方便了。



### 面向对象编程(**Object-Oriented Programming**)

 OOP编程是利用“类”和“对象”来创建各种模型来实现对真实世界的描述，使用面向对象编程的原因一方面是因为它可以使程序的维护和扩展变得更简单，并且可以大大提高程序开发效率 ，另外，基于面向对象的程序可以使它人更加容易理解你的代码逻辑，从而使团队开发变得更从容。

 面向对象的几个核心特性如下：

- **Class 类**

 一个类即是对一类拥有相同属性的对象的抽象、蓝图、原型。在类中定义了这些对象的都具备的属性（variables(data)）、共同的方法

- **Object 对象**

 一个对象即是一个类的实例化后实例，一个类必须经过实例化后方可在程序中调用，一个类可以实例化多个对象，每个对象亦可以有不同的属性，就像人类是指所有人，每个人是指具体的对象，人与人之前有共性，亦有不同

- **Encapsulation 封装**

 在类中对数据的赋值、内部调用对外部用户是透明的，这使类变成了一个胶囊或容器，里面包含着类的数据和方法

- **Inheritance 继承**

 一个类可以派生出子类，在这个父类里定义的属性、方法自动被子类继承

- **Polymorphism 多态**

 多态是面向对象的重要特性,简单点说:“一个接口，多种实现”，指一个基类中派生出了不同的子类，且每个子类在继承了同样的方法名的同时又对父类的方法做了不同的实现，这就是同一种事物表现出的多种形态。

 编程其实就是一个将具体世界进行抽象化的过程，多态就是抽象化的一种体现，把一系列具体事物的共同点抽象出来, 再通过这个抽象的事物, 与不同的具体事物进行对话。

 对不同类的对象发出相同的消息将会有不同的行为。比如，你的老板让所有员工在九点钟开始工作, 他只要在九点钟的时候说：“开始工作”即可，而不需要对销售人员说：“开始销售工作”，对技术人员说：“开始技术工作”, 因为“员工”是一个抽象的事物, 只要是员工就可以开始工作，他知道这一点就行了。至于每个员工，当然会各司其职，做各自的工作。

 多态允许将子类的对象当作父类的对象使用，某父类型的引用指向其子类型的对象,调用的方法是该子类型的方法。这里引用和调用方法的代码编译前就已经决定了,而引用所指向的对象可以在运行期间动态绑定

### 面向对象vs面向过程总结

 面向过程的程序设计的核心是过程（流水线式思维），过程即解决问题的步骤，面向过程的设计就好比精心设计好一条流水线，考虑周全什么时候处理什么东西。

 **优点是：极大的降低了写程序的复杂度，只需要顺着要执行的步骤，堆叠代码即可。**

 **缺点是：一套流水线或者流程就是用来解决一个问题，代码牵一发而动全身。**

 面向对象的程序设计的核心是对象（上帝式思维），要理解对象为何物，必须把自己当成上帝，上帝眼里世间存在的万物皆为对象，不存在的也可以创造出来。面向对象的程序设计好比如来设计西游记，如来要解决的问题是把经书传给东土大唐，如来想了想解决这个问题需要四个人：唐僧，沙和尚，猪八戒，孙悟空，每个人都有各自的特征和技能（这就是对象的概念，特征和技能分别对应对象的属性和方法），然而这并不好玩，于是如来又安排了一群妖魔鬼怪，为了防止师徒四人在取经路上被搞死，又安排了一群神仙保驾护航，这些都是对象。然后取经开始，师徒四人与妖魔鬼怪神仙互相缠斗着直到最后取得真经。如来根本不会管师徒四人按照什么流程去取。

 面向对象的程序设计的

 **优点是：解决了程序的扩展性。对某一个对象单独修改，会立刻反映到整个体系中，如对游戏中一个人物参数的特征和技能修改都很容易。**

 **缺点：可控性差，无法向面向过程的程序设计流水线式的可以很精准的预测问题的处理流程与结果，面向对象的程序一旦开始就由对象之间的交互解决问题**，**即便是上帝也无法预测最终结果。于是我们经常看到一个游戏人某一参数的修改极有可能导致阴霸的技能出现，一刀砍死3个人，这个游戏就失去平衡。**

 应用场景：需求经常变化的软件，一般需求的变化都集中在用户层，互联网应用，企业内部软件，游戏等都是面向对象的程序设计大显身手的好地方。



### 类的定义

```
class Dog:  # 类名首字母要大写，驼峰体
    d_type = "京巴"  # 公共属性，又称类变量
    
    def say_hi(self):  # 类的方法，必须带一个self参数，代表实例本身，具体先不解释
        print("hello , I am a dog,my type is ",self.d_type) # 想调用类里的属性，都要加上self., 原因先不表

        d = Dog()  # 生成一个狗的实例
d2 = Dog()  # 生成一个狗的实例

d.say_hi()  # 调用狗这个类的方法
d2.say_hi()

print(d.d_type)  # 调用Dog类的公共属性
```

 以上代码就是定义好了Dog这个类，相当于先生成了一个模板，接下来生成了2个实例d, d2，相当于2条有血有肉的狗被创造出来了。

 d_type是类变量，是Dog类下所有实例共有的属性，它存在Dog类本身的内存里。你可以查看d1.d_type,d2.d_type的内存地址，指向的是同一处

 除了共有属性，有没有私有(`这里的私有指的就是实例属性`)的呢？ 比如每条狗的名字、年龄、主人都不一样。可以的，如下操作就行：

```
class Dog:
    d_type = "京巴"
    d_name = "小黑"

    # 构造方法
    def __init__(self, age, d_type="京巴", name="小黑"):
        print("---构造方法---")
        self.d_type = d_type
        self.d_name = name
        self.age = age

    def say_hi(self):
        print("hello,i am a dog,my type is", self.d_type, ",my name is", Dog.d_name, "my age is", self.age)


dog = Dog(5)
dog2 = Dog(2, "藏獒", "小白")

dog.say_hi()
dog2.say_hi()

print("---修改类属性---")
Dog.d_type = "哈巴"   # 改的是类属性
Dog.d_name = "小白黑"
dog.say_hi()
dog2.say_hi()

# 测试类属性和实例属性
print(Dog.d_type)
print(dog.age)
print(Dog.age)
```

 输出

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829414649119.png)](https://img-blog.csdnimg.cn/2020073020513152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 **age是实例的属性，type是类属性**



> **self就是代表实例本身。你实例化时python会自动把这个实例本身通过self参数传进去。**
>
> 相当于c++，java，js中的this



### 私有属性

 在属性的前方加个`__`就可以将属性变为私有的

```
class Dog:
    __name = "你好呀"

    def __init__(self, age):
        self.__age = age

    def say_hi(self):
        print(Dog.__name, self.__age)


dog = Dog(6)
dog.say_hi()

# 调用私有属性
print(dog.__age)
print(dog.__name)
```

 输出

[![img](E:\Development\Typora\images\20200730235909125.png)](https://img-blog.csdnimg.cn/20200730235909125.png)

> 不论是类属性，还是实例属性，只要是私有的，都只能在类内部进行调用，外部无法访问



 **私有属性真的不能访问吗，其实是可以的**

```
print(dog._Dog__age)
print(dog._Dog__name)
dog._Dog__age = 10
print(dog._Dog__age)
```

 输出

```
6
你好呀
10
```

[![img](E:\Development\Typora\images\20200731000722411.jpg)](https://img-blog.csdnimg.cn/20200731000722411.jpg)

 

### 类与类之间的关系

大千世界, 万物之间皆有规则和规律. 我们的类和对象是对大千世界中的所有事物进行归类. 那事物之间存在着相对应的关系. 类与类之间也同样如此. 在面向对象的世界中. 类与类中存在以下关系:

1. 依赖关系，狗和主人的关系
2. 关联关系，你和你的女盆友的关系就是关联关系
3. 组合关系，比聚合还要紧密.比如人的大脑, 心脏, 各个器官. 这些器官组合成一个人. 这时. 人如果挂了. 其他的东西也跟着挂了
4. 聚合关系，电脑的各部件组成完整的电脑，电脑里有CPU, 硬盘, 内存等。 每个组件有自己的生命周期， 电脑挂了. CPU还是好的. 还是完整的个体
5. 继承关系， 类的三大特性之一，子承父业



### 关联关系

```
class Person:
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex

    def do_private_stuff(self):
        pass

p1 = Person("drake", 25, "M")
p2 = Person("taylor", 24, "F")

p1.parter = p2
p2.parter = p1


print(p1.parter.name, p2.parter.name)
```

 输出

```
taylor drake
```

 可以创建个单独的类，存储2个人的关系状态，2个人在查自己的感情状态时，都到这个单独的实例里来查

```
class RelationShip:

    def make_couple(self, obj1, obj2):
        self.couple = [obj1, obj2]
        print("[%s] 和 [%s] 确定了男女关系..." % (obj1, obj2))

    def get_my_partner(self, obj):
        for person in self.couple:
            if person != obj:
                return person

    def break_up(self):
        print(self.couple[0].name, "和", self.couple[1].name, "分手了")
        self.couple.clear()


class Person:
    def __init__(self, name, age, sex, relation=None):
        self.name = name
        self.age = age
        self.sex = sex
        self.relation = relation  # 存储两人的关系对象

    def do_private_stuff(self):
        pass




relation = RelationShip()
p1 = Person("drake", 25, "M", relation)
p2 = Person("taylor", 24, "F", relation)

# 谈对象
relation.make_couple(p1, p2)

print(p1.name, "的对象：", p1.relation.get_my_partner(p1).name)
print(p2.name, "的对象：", p2.relation.get_my_partner(p2).name)

# 分手
p1.relation.break_up()
```

 输出

```
[<__main__.Person object at 0x000002AF7B375C10>] 和 [<__main__.Person object at 0x000002AF7B3E7880>] 确定了男女关系...
drake 的对象： taylor
taylor 的对象： drake
drake 和 taylor 分手了
```



### 继承(Inheritance)

#### 介绍

 比较官方的说法就是：

 继承（英语：inheritance）是面向对象软件技术当中的一个概念。如果一个类别A“继承自”另一个类别B，

 就把这个A称为“B的子类别”，而把B称为“A的父类别”也可以称“B是A的超类”。

 继承可以使得子类别具有父类别的各种属性和方法，而不需要再次编写相同的代码。

 在令子类别继承父类别的同时，可以重新定义某些属性，并重写某些方法，

 即覆盖父类别的原有属性和方法，使其获得与父类别不同的功能。另外，为子类别追加新的属性和方法也是常见的做法。 一般静态的面向对象编程语言，继承属于静态的，意即在子类别的行为在编译期就已经决定，无法在执行期扩充。

 字面意思就是：子承父业，合法继承家产，就是如果你是独生子，而且你也很孝顺，不出意外，

 你会继承你父母所有家产，他们的所有财产都会由你使用（败家子儿除外）。



#### **继承与抽象（先抽象再继承）**

 抽象即抽取类似或者说比较像的部分。

 抽象分成两个层次：

 1.将雷昂纳多和王思聪这俩对象比较像的部分抽取成类；

 2.将人，猪，狗这三个类比较像的部分抽取成父类。

 抽象最主要的作用是划分类别（可以隔离关注点，降低复杂度）

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829414649120.png)](https://img-blog.csdnimg.cn/20200730215301916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 **继承：是基于抽象的结果，通过编程语言去实现它，肯定是先经历抽象这个过程，才能通过继承的方式去表达出抽象的结构。**

 抽象只是分析和设计的过程中，一个动作或者说一种技巧，通过抽象可以得到类

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829414649121.png)](https://img-blog.csdnimg.cn/20200730215342910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



```
class Animal:
    type = "哺乳动物"

    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex

    def eat(self):
        print("%s is eating..." % self.name)


class Person(Animal):
    # 重写属性
    type = "人"

    def __init__(self, name, age, sex, hobbie):
        # 调用父类的构造器
        # Animal.__init__(self, name, age, sex)
        super(Person, self).__init__(name, age, sex)
        self.hobbie = hobbie

    def talk(self):
        print(Person.type, " %s is talking" % self.name)

    # 重写方法
    def eat(self):
        # 调用父类的方法
        super().eat()
        print(Animal.type, "person %s is  eating gracefully" % self.name)

class Dog(Animal):

    def chase_rabbit(self):
        print(Dog.type, "-狗追兔子")

p = Person("老八", 25, "M", ["打篮球", "踢足球"])
p.eat()
p.talk()

print("-----")
d = Dog("小黑", 5, "Female")
d.eat()
d.chase_rabbit()
```

 输出

```
老八 is eating...
哺乳动物 person 老八 is  eating gracefully
人  老八 is talking
-----
小黑 is eating...
哺乳动物 -狗追兔子
```



#### 多继承

```
class Fairy:
    """神仙类"""

    def __init__(self, name):
        self.name = name

    def fly(self):
        print("神仙都会飞...")

    def fight(self):
        print("神仙打架...")

class Monkey:
    """猴子类"""

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat_peach(self):
        print("猴子都喜欢吃桃子...")

    def fight(self):
        print("猴子打架...")

class MonkeyKing(Fairy,Monkey):
    """
     孙悟空
    """

    def play_goden_stick(self):
        super().fight()  # 调用Fairy中的方法
        print(self.name, "玩棒子...")

m = MonkeyKing("孙猴子")
m.fly()
m.eat_peach()
m.play_goden_stick()
# 调用父类的方法
m.fight()
```

 输出

```
神仙都会飞...
猴子都喜欢吃桃子...
神仙打架...
孙猴子 玩棒子...
神仙打架...
```

> 可以看到，子类调用的都是父类Fairy中的方法，说明了什么？？？



#### 具体调用关系

##### 说明

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829414649122.png)](https://img-blog.csdnimg.cn/20200730231026942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 途中显示的调用的关系是按深度优先来调用的。以下为具体分析(不是最终结论)

##### 第一个(G)

```
class A:
    def desc(self):
        print("A")

class B:
    def desc(self):
        print("B")

class C:
    def desc(self):
        print("C")

class D:
    def desc(self):
        print("D")

class E(A,B):
    def desc(self):
        print("E,继承自A和B")

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
    def desc(self):
        print("G,继承自E和F")

g = G()
g.desc()
```

 输出

```
G,继承自E和F
```

##### 第二个(E)

```
class A:
    def desc(self):
        print("A")

class B:
    def desc(self):
        print("B")

class C:
    def desc(self):
        print("C")

class D:
    def desc(self):
        print("D")


class E(A,B):
    def desc(self):
        print("E,继承自A和B")

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
    
    
        pass

g = G()
g.desc()
```

emsp;输出

```
E,继承自A和B
```



##### 第三个(A)

```
class A:
    def desc(self):
        print("A")

class B:
    def desc(self):
        print("B")

class C:
    def desc(self):
        print("C")

class D:
    def desc(self):
        print("D")


class E(A,B):
    
        pass

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
    
        pass

g = G()
g.desc()
```

 输出

```
A
```



##### 第四个(B)

```
class A:
    
        pass

class B:
    def desc(self):
        print("B")

class C:
    def desc(self):
        print("C")

class D:
    def desc(self):
        print("D")


class E(A,B):
    
        pass

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
    
        pass

g = G()
g.desc()
```

 输出

```
B
```



##### 第五个(F)

```
class A:
    
        pass

class B:
    
        pass

class C:
    def desc(self):
        print("C")

class D:
    def desc(self):
        print("D")


class E(A,B):
    
        pass

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
    
        pass

g = G()
g.desc()
```

 输出

```
F,继承自C和D
```



##### 第六个©

##### 第七个(D)



##### 结论

> **在python 2中，经典类采用的是深度优先查找法， 新式类采用的是广度优先**
>
> **在python 3中，无论是经典类，还是新式类，都是按广度优先查找**
>
> Python 2.x中默认都是经典类，只有显式继承了object才是新式类
>
> Python 3.x中默认都是新式类，不必显式的继承object
>
> 之所以在python3中全改成了广度优先，是因为用深度优先在某些特殊情况下，[会出现bug.](https://www.python.org/download/releases/2.3/mro/)

啊啊啊？？？ 上面演示的明明不是按深度优先继承的么？ 你现在说py3用的都是广度，这不是睁眼说瞎话么？其实严格来说也并不是完全的广度优先，而是使用了C3算法



#### 多继承原理之C3算法

```
class Base:
    def desc(self):
        print("Base")


class A(Base):
    
        pass


class B(Base):
    def desc(self):
        print("B")

class C(Base):
    def desc(self):
        print("C")


class D:
    def desc(self):
        print("D")


class E(A,B):
        pass

class F(C,D):
    def desc(self):
        print("F,继承自C和D")


class G(E, F):
        pass

g = G()
g.desc()
```

 输出

```
B
```

**去掉B继承的Base，再查看结果**

```
class B:
    def desc(self):
        print("B")
        
-----
F,继承自C和D
```

此时调用的是F，B跳过了？？？

**去掉F和C继承desc()方法，再查看结果**

```
class C(Base):
     # def desc(self):
     #     print("C")
     
        pass

class F(C,D):
    # def desc(self):
    #     print("F,继承自C和D")
    
        pass

    
-----
Base
```

此时调用了Base

> 如果有一个公共继承类，只会先在继承了Base中的类进行深度优先查找，如果继承了Base的类都没有该方法，就会调用Base中的方法，如果Base中也没有，就会从非继承类中查找

 **上述的调用顺序（初始的代码）可以用G.mro()方法查看**

```
[<class '__main__.G'>, <class '__main__.E'>, <class '__main__.A'>, <class '__main__.B'>, <class '__main__.F'>, <class '__main__.C'>, <class '__main__.Base'>, <class '__main__.D'>, <class 'object'>]
```

 G->E->A->B->F->C->Base->D->obejct（object是因为py3中默认是新式类）



### 封装(Encapsulation)

#### 介绍

 封装可以被认为是一个保护屏障，防止该类的代码和数据被外部类定义的代码随机访问。

 要访问该类的代码和数据，必须通过严格的接口控制。

 封装最主要的功能在于我们能修改自己的实现代码，而不用修改那些调用我们代码的程序片段。

 适当的封装可以让程式码更容易理解与维护，也加强了代码数据的安全性。

#### 封装的优点

- 良好的封装能够减少耦合。
- 类内部的结构可以自由修改。
- 可以对成员变量进行更精确的控制。
- 隐藏信息，实现细节。

#### 封装原则

1. 将不需要对外提供的内容都隐藏起来；
2. 把属性都隐藏，提供公共方法对其访问。



### 多态(Polymorphism )

 有时一个对象会有多种表现形式，比如网站页面有个button按钮， 这个button的设计可以不一样(单选框、多选框、圆角的点击按钮、直角的点击按钮等)，尽管长的不一样，但它们都有一个共同调用方式，就是onClick()方法。我们直要在页面上一点击就会触发这个方法。点完后有的按钮会变成选中状态、有的会提交表单、有的甚至会弹窗。这种多个对象共用同一个接口，又表现的形态不一样的现象，就叫做多态( *Polymorphism* )。

[![img](E:\Development\Typora\images\20200731002329410.png)](https://img-blog.csdnimg.cn/20200731002329410.png)

```
class Animal:
    def sound(self):
        print("叫唤呀...")

class Cat:
    def sound(self):
        print("喵喵喵...")

class Dog:
    def sound(self):
        print("汪汪汪...")




animal = Animal()
dog = Dog()
cat = Cat()

animals = (animal, dog, cat)
for a in animals:
    a.sound()
```

 输出

```
叫唤呀...
汪汪汪...
喵喵喵...
```



### 类方法

 类方法通过[@classmethod](https://github.com/classmethod)装饰器实现，类方法和普通方法的区别是， 类方法只能访问类变量，不能访问实例变量

```
class Dog:

    name = "狗爷"

    # 类方法
    @classmethod
    def eat2(cls):
        print("dog %s is eating..." % cls.name)
        
Dog.eat2()
dog.eat2()
```

 输出

```
dog 狗爷 is eating...
dog 狗爷 is eating...
```



### 静态方法

 **通过**@staticmethod装饰器即可把其装饰的方法变为一个静态方法，什么是静态方法呢？其实不难理解，普通的方法，可以在实例化后直接调用，并且在方法里可以通过self.调用实例变量或类变量，但静态方法是不可以访问实例变量或类变量的，一个不能访问实例变量和类变量的方法，其实相当于跟类本身已经没什么关系了，它与类唯一的关联就是需要通过类名来调用这个方法

```
class Dog:

    name = "狗爷"

    # 静态方法
    @staticmethod
    def eat3():
        print("Dog")

Dog.eat3()
dog.eat3()
```

 输出

```
Dog
Dog
```



### 属性方法

 属性方法的作用就是通过[@property](https://github.com/property)把一个方法变成一个静态属性

```
class Dog:

    name = "狗爷"

    def __init__(self, name):
        self.name = name

    def eat(self):
        print("dog %s is eating..." % self.name)

    # 属性方法
    @property
    def eat4(self):
        print("dog %s is eating..." % self.name)

    @eat4.setter
    def eat4(self, status):
        print("dog", status)

    @eat4.getter
    def eat4(self):
        return self.name

    @eat4.deleter
    def eat4(self):
        print("删除")
  


dog = Dog("小黑")


dog.eat4
dog.eat4 = "已吃完"
print(dog.eat4)
del dog.eat4
```

 输出

```
dog 已吃完
小黑
删除
```

> 注意当存在getter方法时，调用属性方法其实就是执行getter方法
>
> 当不存在getter方法时，调用属性犯法才是执行属性本身的方

 **使用场景**

 好吧，把一个方法变成静态属性有什么卵用呢？既然想要静态变量，那直接定义成一个静态变量不就得了么？well, 以后你会需到很多场景是不能简单通过 定义 静态属性来实现的， 比如 ，你想知道一个航班当前的状态，是到达了、延迟了、取消了、还是已经飞走了， 想知道这种状态你必须经历以下几步:

1. 连接航空公司API查询
2. 对查询结果进行解析
3. 返回结果给你的用户

 因此这个status属性的值是一系列动作后才得到的结果，所以你每次调用时，其实它都要经过一系列的动作才返回你结果，但这些动作过程不需要用户关心， 用户只需要调用这个属性就可以，明白 了么？

 总觉得跟Vue中的computed属性有点像



### 反射

#### 介绍

 反射的概念是由Smith在1982年首次提出的，主要是指程序可以访问、检测和修改它本身状态或行为的一种能力（自省）。这一概念的提出很快引发了计算机科学领域关于应用反射性的研究。它首先被程序语言的设计领域所采用,并在Lisp和面向对象方面取得了成绩。

 python面向对象中的反射**：通过字符串的形式操作对象相关的属性。**python中的一切事物都是对象（都可以使用反射）

 四个可以实现自省的函数

 下列方法适用于类和对象（一切皆对象，类本身也是一个对象）

```
def hasattr(*args, **kwargs): 
    """
    Return whether the object has an attribute with the given name.
    This is done by calling getattr(obj, name) and catching AttributeError.
    """
    pass
def getattr(object, name, default=None): 
    """
    getattr(object, name[, default]) -> value
    Get a named attribute from an object; getattr(x, 'y') is equivalent to x.y.
    When a default argument is given, it is returned when the attribute doesn't
    exist; without it, an exception is raised in that case.
    """
    pass
def setattr(x, y, v):
    """
    Sets the named attribute on the given object to the specified value.
    setattr(x, 'y', v) is equivalent to ``x.y = v''
    """
    pass
def delattr(x, y): 
    """
    Deletes the named attribute from the given object.
    delattr(x, 'y') is equivalent to ``del x.y''
    """
    pass
```



#### 例子

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.__age = age


person = Person("黑子", 20)


print("-----属性反射-----")
# 反射的增删改查
if hasattr(person, "name"):
    print(getattr(person, "name"))  # 获取属性
    delattr(person, "name")   # 删除属性
    setattr(person, "name", "白子")  # 新增或者修改属性
else:
    print("没有name这个属性")

print(person.name)



print("-----方法反射-----")
# 方法反射
def walk(name):
    print("你好呀，我们来散步吧！！！我叫", name)

# 新增
print(hasattr(person, "walk"))
setattr(person,"walk", walk)
print(hasattr(person, "walk"))

# 获取
person.walk("武大")
walk_p = getattr(person, "walk")
walk_p("武二")


# 删除
delattr(person, "walk")
print(hasattr(person, "walk"))
```

 输出

```
-----属性反射-----
黑子
白子
-----方法反射-----
False
True
你好呀，我们来散步吧！！！我叫 武大
你好呀，我们来散步吧！！！我叫 武二
False
```



#### 跨模块反射

 在做跨模块导入的时候，先看一个双下划线方法

```
print(__name__)
# __name__获取模块名
if __name__ == "__main__":
    print("当前模块输出")
```

 当前模块输出

```
__main__
当前模块输出
```

 其他模块导入

```
import test_name
```

 输出

```
test_name
```

> __name__可以输出当前模块的名称

 **当前模块反射**

```
mod = sys.modules[__name__]
if hasattr(mod, "person"):
    print(getattr(mod, "person").name)
    getattr(mod, "person").walk("武大")
```

 **跨模块导入反射：**

- 被导入的模块

  ```
  class Person:
      def __init__(self, name, age):
          self.name = name
          self.__age = age
  
      def walk(self, name):
          print("你好呀，我们来散步吧！！！我叫", name)
  
  person = Person("小黑", 20)
  
  
  print(__name__)
  # __name__获取模块名
  if __name__ == "__main__":
      print("当前模块输出")
  ```

- 导入模块

  ```
  import test_name
  import sys
  
  mod = sys.modules["test_name"]
  if hasattr(mod, "person"):
      print(getattr(mod, "person").name)
      getattr(mod, "person").walk("武大")
  ```

  输出

  ```
  test_name
  小黑
  你好呀，我们来散步吧！！！我叫 武大
  ```

### 动态加载模块

```
__import__("test_name")
__import__("basicGrammar.module.my_module")

--------

import importlib
importlib.import_module("test_name")
importlib.import_module("basicGrammar.module.my_module")
```

 输出

```
test_name
hello alex
```



### 类的双下线方法

#### len方法

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __len__(self):
        return 5
    
    
person = Person("小贵", 25)
print(len(person))
```

 输出

```
5
```



#### **hash**方法

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __len__(self):
        return 5

    def __hash__(self):
        return 55555
    
person = Person("小贵", 25)

print("-----len-----")
print(len(person))

print("-----hash-----")
print(hash(person))
```

 输出

```
-----len-----
5
-----hash-----
55555
```



#### eq方法

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __len__(self):
        return 5

    def __hash__(self):
        return 55555

    def __eq__(self, other):
        return True
    
    
person = Person("小贵", 25)

print("-----len-----")
print(len(person))

print("-----hash-----")
print(hash(person))

print("-----eq-----")
print(person.__eq__("你好"))
```

 输出

```
-----len-----
5
-----hash-----
55555
-----eq-----
True
```



#### **item**系列

可以把一个对象变成dict， 可以像dict一样增删改查

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age


    names = {"name": "小龟龟"}
    
    def __getitem__(self, item):
        return Person.names[item]

    def __setitem__(self, key, value):
        Person.names[key] = value

    def __delitem__(self, key):
        Person.names.pop(key)
        

person = Person("小贵", 25)

print("-----getitem-----")
print(person["name"])

print("-----setitem-----")
person["age"] = 20
print(Person.names)

print("-----delitem-----")
del person["age"]
print(Person.names)
```

 输出

```
-----getitem-----
小龟龟
-----setitem-----
{'name': '小龟龟', 'age': 20}
-----delitem-----
{'name': '小龟龟'}
```



#### **str** & **repr**

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __repr__(self):
        return "Person %s %s" % (self.name, self.age)

    def __str__(self):
        return "%s %s" % (self.name, self.age)


person = Person("小贵", 25)

# repr&str
print("-----repr&str-----")
'''
str函数或者print函数调用时--->obj.__str__()
repr或者交互式解释器中调用时--->obj.__repr__()
如果__str__没有被定义,那么就会使用__repr__来代替输出
注意:这俩方法的返回值必须是字符串,否则抛出异常
'''
print(repr(person), person.__repr__())
print(str(person), person.__str__(), person)
```

 输出

```
Person 小贵 25 Person 小贵 25
小贵 25 小贵 25 小贵 25
```



#### **del** 析构方法

 析构方法，当对象在内存中被释放时，自动触发执行。

 注：此方法一般无须定义，因为Python是一门高级语言，程序员在使用时无需关心内存的分配和释放，因为此工作都是交给Python解释器来执行，所以，析构函数的调用是由解释器在进行垃圾回收时自动触发执行的。

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __repr__(self):
        return "Person %s %s" % (self.name, self.age)

    def __str__(self):
        return "%s %s" % (self.name, self.age)

    def __del__(self):
        print("销毁对象(析构)...")
        
        
        
        
person = Person("小贵", 25)

# repr&str
print("-----repr&str-----")
print(repr(person), person.__repr__())
print(str(person), person.__str__(), person)



print("-----del(析构)-----")
# 析构方法
del person
```

 输出

```
-----repr&str-----
Person 小贵 25 Person 小贵 25
小贵 25 小贵 25 小贵 25
-----del(析构)-----
销毁对象(析构)...
```



#### new方法

##### 使用

 我们知道实例化**init**会自动执行， 其实在**init**方法之前，还有一个**new**方法也会自动执行，你可以在**new**里执行一些实例化前的定制动作

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age
        print("init")

    def __new__(cls, *args, **kwargs):
        print(cls, args, kwargs)


person = Person("小贵", 15)
```

 输出

```
<class '__main__.Person'> ('小贵', 15) {}
```

> 为什么init没有执行？
>
> 因为init方法是由new方法调用的，我们重写了new方法，此时并没有执行init方法

```
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age
        print("init")

    def __new__(cls, *args, **kwargs):
        print(cls, args, kwargs)
        return super().__new__(cls)


person = Person("小贵", 15)
```

 输出

```
<class '__main__.Person'> ('小贵', 15) {}
init
```



##### 用**new**方法实现单例模式

 单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。**通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，**从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。

**什么情况下用单例？**

 对于系统中的某些类来说，只有一个实例很重要，例如，一个系统中可以存在多个打印任务，但是只能有一个正在工作的任务；一个系统只能有一个窗口管理器或文件系统；一个系统只能有一个计时工具或ID(序号)生成器。如在Windows中就只能打开一个任务管理器。如果不使用机制对窗口对象进行唯一化，将弹出多个窗口，如果这些窗口显示的内容完全一致，则是重复对象，浪费内存资源；如果这些窗口显示的内容不一致，则意味着在某一瞬间系统有多个状态，与实际不符，也会给用户带来误解，不知道哪一个才是真实的状态。因此有时确保系统中某个对象的唯一性即一个类只能有一个实例非常重要

 如何保证一个类只有一个实例并且这个实例易于被访问呢？定义一个全局变量可以确保对象随时都可以被访问，但不能防止我们实例化多个对象。一个更好的解决办法是让类自身负责保存它的唯一实例。这个类可以保证没有其他实例被创建，并且它可以提供一个访问该实例的方法。这就是单例模式的模式动机。

```
class Person:

    instance = None

    def __init__(self, name, age):
        self.name = name
        self.age = age
        print("init")



    def __new__(cls, *args, **kwargs):
        if cls.instance is None:
             instance = super().__new__(cls)
             cls.instance = instance
        return cls.instance



person1 = Person("小贵", 15)
person2 = Person("小贵", 15)
person3 = Person("小贵", 15)
person4 = Person("小贵", 15)
print(id(person1))
print(id(person2))
print(id(person3))
print(id(person4))
```

 输出

```
init
init
init
init
1260870865056
1260870865056
1260870865056
1260870865056
```



#### call方法

 对象后面加括号，触发执行。

 注：构造方法**new**的执行是由创建对象触发的，即：对象 = 类名() ；而对于 **call** 方法的执行是由对象后加括号触发的，即：对象() 或者 类()()

```
class School:

    def __init__(self, name):
        self.name = name

    def __call__(self, *args, **kwargs):
        print(self, args, kwargs)

school = School("呼呼呼")
school("好学校", flag="世界一流")
```

 输出

```
<__main__.School object at 0x0000029FB38A8460> ('好学校',) {'flag': '世界一流'}
```



### 动态生成一个类

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age


person = Person("小贵", 25)
print(type(person))
print(type(Person))
```

 输出

```
<class '__main__.Person'>
<class 'type'>
```

 上述代码中，person 是通过 Person 类实例化的对象，其实，不仅 person 是一个对象，Person类本身也是一个对象，因为在**Python中一切事物都是对象**。

 如果按照一切事物都是对象的理论：person对象是通过执行Person类的构造方法创建，那么Person类对象应该也是通过执行某个类的 构造方法 创建。

 所以，person**对象是Person类的一个实例**，**Person类对象是 type 类的一个实例**，即：Person类对象 是通过type类的构造方法创建。



 **type是如何创建一个类的呢？**

```
def __init__(self, age):
    self.age = age

dog_class = type("Dog", (), {"role": "dog","__init__": __init__})
print(dog_class.role)

dog = dog_class(25)
print(dog.age)
```

 输出

```
dog
25
```

 了解类本身是通过type创建的还是挺重要的，以后我们在代码中就可以动态的生成类了，而不是提前必须先定义好。 以后学到django web框架时，生成动态表单就用到这个知识点。



### isinstance&issubclass

```
class A:
    pass

class B(A):
    pass

b = B()
print(isinstance(b, A))
print(isinstance(b, B))

print(issubclass(B, A))
print(issubclass(B, B))
```

 输出

```
True
True
True
True
```



### 异常

#### 捕获异常

 在编程过程中为了增加友好性，在程序出现bug时一般不会将错误信息显示给用户让用户蒙逼，而是显示一个更友好的提示信息。

```
while True:
    try:
        num1 = int(input("n1>"))
        num2 = int(input("n2>"))

        res = num1+num2
        print("result：", res, name)
    except ValueError as e:
        print("您输入的值不合法哦~", e)
    except NameError as e:
        print("name没有定义哦~", e)
```

 输出

[![img](E:\Development\Typora\images\20200731210903609.png)](https://img-blog.csdnimg.cn/20200731210903609.png)



#### 常见异常

1. AttributeError 试图访问一个对象没有的属性，比如foo.x，但是foo没有属性x
2. IOError 输入/输出异常；基本上是无法打开文件
3. ImportError 无法引入模块或包；基本上是路径问题或名称错误
4. IndentationError 语法错误（的子类） ；代码没有正确对齐
5. IndexError 下标索引超出序列边界，比如当x只有三个元素，却试图访问x[5]
6. KeyError 试图访问字典里不存在的键
7. KeyboardInterrupt Ctrl+C被按下
8. NameError 使用一个还未被赋予对象的变量
9. SyntaxError Python代码非法，代码不能编译(个人认为这是语法错误，写错了）
10. TypeError 传入对象类型与要求的不符合
11. UnboundLocalError 试图访问一个还未被设置的局部变量，基本上是由于另有一个同名的全局变量，导致你以为正在访问它
12. ValueError 传入一个调用者不期望的值，即使值的类型是正确的



#### 其它异常结构

```
try:
    # 主代码块
    pass
except KeyError,e:
    # 异常时，执行该块
    pass
else:
    # 主代码块执行完，若未触发任何异常，执行该块
    pass
finally:
    # 无论监测的代码是否发生异常，都执行该处代码
    pass
```



#### 主动触发异常

```
try:
    raise Exception("错误了...")
except Exception as e:
    print(e)
```



#### 自定义异常

```
class UserNotFoundException(BaseException):  # BaseException是所有异常的基类
    def __init__(self, msg):
        self.msg = msg

    def __str__(self):
        return self.msg


try:
    raise UserNotFoundException("未找到用户...")
except UserNotFoundException as e:
    print(e)
```

 输出

```
未找到用户...
```



### 断言(asset)

assert语法用于判断代码是否符合执行预期

```
def test_asset(name, age):
    assert type(name) is str
    assert type(age) is int
    print(name, age)

test_asset("黑子", 15)

print("-----------------")

test_asset(15, "黑子")
```

 输出

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706139.png)](https://img-blog.csdnimg.cn/20200731233554894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)





## 网络编程

### 引子

 我们已经知道，假设我现在要写一个程序，给另一台计算机发数据，必须通过tcp/ip协议 ，但具体的实现过程是什么呢？我应该怎么操作才能把数据封装成tcp/ip的包，又执行什么指令才能把数据发到对端机器上呢? 不能只有世界观，没有方法论呀。。。此时，socket隆重登场，简而言之，socket这个东东干的事情，就是帮你把tcp/ip协议层的各种数据封装啦、数据发送、接收等通过代码已经给你封装好了，你只需要调用几行代码，就可以给别的机器发消息了。

### Socket介绍

#### 什么是socket?

 Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706140.png)](https://img-blog.csdnimg.cn/20200801151114775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

 socket起源于Unix，而Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式 来操作。Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）

 你想给另一台计算机发消息，你知道他的IP地址，他的机器上同时运行着qq、迅雷、word、浏览器等程序，你想给他的qq发消息，那想一下，你现在只能通过ip找到他的机器，但如果让这台机器知道把消息发给qq程序呢？答案就是通过port,一个机器上可以有0-65535个端口，你的程序想从网络上收发数据，就必须绑定一个端口，这样，远程发到这个端口上的数据，就全会转给这个程序啦

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706141.png)](https://img-blog.csdnimg.cn/20200801151239348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### Socket通信套路

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706142.png)](https://img-blog.csdnimg.cn/20200801151338856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### Socket套接字方法

#### socket 实例类

```
cket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

1. family(地址家族)
   - socket.AF_UNIX：用于本机进程间通讯，为了保证程序安全，两个独立的程序(进程)间是不能互相访问彼此的内存的，但为了实现进程间的通讯，可以通过创建一个本地的socket来完成
   - socket.AF_INET:(还有AF_INET6被用于ipv6，还有一些其他的地址家族，不过，他们要么是只用于某个平台，要么就是已经被废弃，或者是很少被使用，或者是根本没有实现，所有地址家族中，AF_INET是使用最广泛的一个，python支持很多种地址家族，但是由于我们只关心网络编程，所以大部分时候我么只使用AF_INET)
     socket type类型
2. type
   - socket.SOCK_STREAM #for tcp
   - socket.SOCK_DGRAM #for udp
   - socket.SOCK_RAW #原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK_RAW可以；其次，SOCK_RAW也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过IP_HDRINCL套接字选项由用户构造IP头。
   - socket.SOCK_RDM #是一种可靠的UDP形式，即保证交付数据报但不保证顺序。SOCK_RAM用来提供对原始协议的低级访问，在需要执行某些特殊操作时使用，如发送ICMP报文。SOCK_RAM通常仅限于高级用户或管理员运行的程序使用。
   - socket.SOCK_SEQPACKET #废弃了
     (Only SOCK_STREAM and SOCK_DGRAM appear to be generally useful.)
3. proto=0 请忽略，特殊用途
4. fileno=None 请忽略，特殊用途



#### 服务端套接字函数

- s.bind() 绑定(主机,端口号)到套接字
- s.listen() 开始TCP监听
- s.accept() 被动接受TCP客户的连接,(阻塞式)等待连接的到来



#### 客户端套接字函数

- s.connect() 主动初始化TCP服务器连接
- s.connect_ex() connect()函数的扩展版本,出错时返回出错码,而不是抛出异常



#### 公共用途的套接字函数

- s.recv() 接收数据
- s.send() 发送数据(send在待发送数据量大于己端缓存区剩余空间时,数据丢失,不会发完，可后面通过实例解释)
- s.sendall() 发送完整的TCP数据(本质就是循环调用send,sendall在待发送数据量大于己端缓存区剩余空间时,数据不丢失,循环调用send直到发完)
- s.recvfrom() Receive data from the socket. The return value is a pair (bytes, address)
- s.getpeername() 连接到当前套接字的远端的地址
- s.close() 关闭套接字
- socket.setblocking(flag) #True or False,设置socket为非阻塞模式，以后讲io异步时会用
- socket.getaddrinfo(host, port, family=0, type=0, proto=0, flags=0) 返回远程主机的地址信息，例子 socket.getaddrinfo(‘[luffycity.com](http://luffycity.com/)’,80)
- socket.getfqdn() 拿到本机的主机名
- socket.gethostbyname() 通过域名解析ip地址



### 建立基础服务端和客户端

#### 服务端

```
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(("127.0.0.1", 50000))
s.listen(2)  # 允许挂起多少个连接
client_conn, client_addr = s.accept()
print(client_conn, client_addr)

client_conn.send("大爷，您竟来了！！！".encode())
data = client_conn.recv(1024)
print(data.decode())

client_conn.close()
s.close()
```



#### 客户端

```
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))

data = client.recv(1024)
print(data.decode())
client.send("你说呢？？？".encode())

client.close()
```



#### 测试

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706143.png)](https://img-blog.csdnimg.cn/20200801152002854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 循环发送数据

 第一次接触就这么交待了，只说了一句话，感觉不够过瘾，如何实现更多的交互呢？简单，只需要让客户端不断的发，服务端不断的收就可以了，写个循环搞定

#### 服务端

```
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("127.0.0.1", 50000))
server.listen(2)
print("-----开始等待客户端连接-----")
while True:
    client_conn, client_addr = server.accept()
    print(f"got a new connetcion ：{client_addr}")
    while True:
        data = client_conn.recv(1024)   # 阻塞的
        if not data:  # 客户端断开了
            print(f"close a connection ：{client_addr}")
            break
        print(f"from client {client_addr}:{data.decode()}")
        client_conn.send(data.decode().upper().encode())

client_conn.close()
server.close()
```

#### 客户端

```
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))
while True:
    msg = input(">>>: your msg：").strip()
    client.send(msg.encode())
    if not msg: break

    server_response = client.recv(1024)
    print(f"from server:{server_response.decode()}")

client.close()
```

#### 测试

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706144.png)](https://img-blog.csdnimg.cn/20200801155746263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### ssh

#### 服务端

```
import socket
import subprocess  # 使用指令
import json

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("127.0.0.1", 50000))
server.listen(2)

count = 0
while True:
    conn, client_addr = server.accept()
    count+=1
    print(f"[{count}] - got a new connection : {client_addr}")

    while True:
        cmd_byte = conn.recv(1024)
        cmd = cmd_byte.decode()
        if cmd.__eq__("exit"):
            print(f"{client_addr} close the connection")
            break
        print(f"from client {client_addr} : {cmd}")
        # 执行指令
        cmd_res = subprocess.getstatusoutput(cmd)

        # 先告诉客户端，要发的数据长度
        res_data = json.dumps(cmd_res).encode()
        res_size = str(len(res_data))

        # 强行将两次发送数据分隔开，解决粘包问题
        conn.send(res_size.encode())

        conn.send(res_data)


server.close()
```



#### 客户端

```
import socket
import json

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))

while True:
    cmd = input(">>>")
    if not cmd:
        continue
    client.send(cmd.encode())
    if cmd.__eq__("exit"):
        print("exit...")
        break

    # 获取命令的执行结果大小
    cmd_res_size = int(client.recv(1024).decode())
    client.send("ok".encode())
    # 循环接受消息
    receiver_size = 0  # 接受到的数据大小
    print(receiver_size, cmd_res_size)
    cmd_result = b""  # 接受到的数据
    while receiver_size < cmd_res_size:
        data = client.recv(1024)
        cmd_result += data
        # cmd_result += json.loads(client.recv(1024).decode())[1]
        # print(cmd_result)
        receiver_size += len(data)

    print(rf"{json.loads(cmd_result.decode())}")

client.close()
```



#### 测试

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706145.png)](https://img-blog.csdnimg.cn/2020080118303979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### 粘包问题

 多个数据一起发送，一定要注意粘包问题。这里不细说了，讲一下解决方法

1. 让缓冲区超时(time.sleep(1))
2. 通过一个recv让2条信息强行隔开(上面例子)
3. 约定一个固定长度的消息头，消息头里包含指令结果的长度(下面例子)



#### 服务端

```
import socket
import subprocess  # 使用指令
import json

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("127.0.0.1", 50000))
server.listen(2)

count = 0
while True:
    conn, client_addr = server.accept()
    count+=1
    print(f"[{count}] - got a new connection : {client_addr}")

    while True:
        cmd_byte = conn.recv(1024)
        cmd = cmd_byte.decode()
        if cmd.__eq__("exit"):
            print(f"{client_addr} close the connection")
            break
        print(f"from client {client_addr} : {cmd}")
        # 执行指令
        cmd_res = subprocess.getstatusoutput(cmd)

        # 获取数据的长度
        res_data = json.dumps(cmd_res).encode()
        cmd_res_size = len(res_data)
        # 生成固定的消息头
        msg_head = f"|{cmd_res_size}".zfill(16)

        # 先告诉客户端，要发的数据长度
        conn.send(msg_head.encode())

        conn.send(res_data)


server.close()
```



#### 客户端

```
import socket
import json
import time
time.sleep(0.5)

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))

while True:
    cmd = input(">>>")
    if not cmd:
        continue
    client.send(cmd.encode())
    if cmd.__eq__("exit"):
        print("exit...")
        break

    # 获取命令的执行结果大小
    msg_header = client.recv(1024).decode()
    cmd_res_size = int(msg_header.split("|")[1])
    # 循环接受消息
    receiver_size = 0  # 接受到的数据大小
    print(receiver_size, cmd_res_size)
    cmd_result = b""  # 接受到的数据
    while receiver_size < cmd_res_size:
        data = client.recv(1024)
        cmd_result += data
        receiver_size += len(data)

    print(rf"{json.loads(cmd_result.decode())[1]}")



client.close()
```



### 传送文件

 收发收发文件与远程执行命令的程序原理是一摸一样的，比如下载文件的过程：

#### 服务端

```
import socket
import os

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("127.0.0.1", 50000))
server.listen(2)

count = 0
while True:
    conn, client_addr = server.accept()
    count += 1
    print(f"[{count}] - got a new connection : {client_addr}")

    while True:
        cmd_byte = conn.recv(1024)
        cmd = cmd_byte.decode()
        if cmd.__eq__("exit"):
            print(f"{client_addr} close the connection")
            break
        print(f"from client {client_addr} : {cmd}")
        # 执行指令

        # 1.服务端收到 客户端指令，解析，判断get开头，代表下载文件
        if cmd.startswith("get"):
            # 2.提取文件名，判断文件是否在服务器上存在，如果存在就下载
            file_name = cmd.split()[1]
            file_path = os.path.join("F:\\Animation\\", file_name)
            if os.path.isfile(file_path):
                file_size = os.path.getsize(file_path)
                # 3.拿到文件的大小，生成消息头，发给客户端
                msg_head = f"|{file_size}".zfill(16).encode()
                conn.send(msg_head)
                # 4.打开文件，发送文件内容
                file = open(file_path, "rb")
                for line in file:
                    conn.send(line)
                file.close()
                print(f"file {file_name} has sent to client, size {file_size}")
        else:
            conn.send("|0".zfill(16).encode())


server.close()
```



#### 客户端

```
import socket
import uuid
import os

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))

while True:
    file_name = input("download>>>").strip()
    if file_name.__eq__("exit"):
        break
    client.send(file_name.encode())

    # 获取文件的大小
    file_size = int(client.recv(32).decode().split("|")[1])
    if file_size == 0:
        print("获取文件格式不对：get 文件名")
        continue

    suffix = file_name.split(".")[1]
    file_name_download = os.path.join("F:\\Animation\\", str(uuid.uuid1())+"."+suffix)
    file = open(f"{file_name_download}", "ab")

    # 循环获取数据
    receive_size = 0

    while receive_size < file_size:
        file_info = client.recv(1024)
        # 更新获取的数据
        receive_size += len(file_info)
        file.write(file_info)
        print(file_size, file_info)
    file.close()
    print(f"file download done...size{file_size}")


client.close()
```



#### 测试

[![img](E:\Development\Typora\images\20200801225439480.png)](https://img-blog.csdnimg.cn/20200801225439480.png)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706146.png)](https://img-blog.csdnimg.cn/20200801225008797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

[![img](E:\Development\Typora\images\20200801225532459.png)](https://img-blog.csdnimg.cn/20200801225532459.png)



### 进度条使用

#### 客户端修改

 客户端相关代码

```
import socket
import uuid
import os

# 封装的进度条
def process_bar(total_size):
    old_receive_percent = 0
    new_receive_percent = 0

    while new_receive_percent <= 100:
        receive_size = yield
        new_receive_percent = int((receive_size / total_size) * 100)
        # 只有有增长时才增加进度条
        if new_receive_percent > old_receive_percent:
            print(f"\r{new_receive_percent * '>'}{new_receive_percent}%", end="")
            old_receive_percent = new_receive_percent


client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("127.0.0.1", 50000))

while True:
    file_name = input("download>>>").strip()
    if file_name.__eq__("exit"):
        break
    client.send(file_name.encode())

    # 获取文件的大小
    file_size = int(client.recv(32).decode().split("|")[1])
    if file_size == 0:
        print("获取文件格式不对：get 文件名")
        continue

    suffix = file_name.split(".")[1]
    file_name_download = os.path.join("F:\\Animation\\", str(uuid.uuid1()) + "." + suffix)
    file = open(f"{file_name_download}", "ab")

    # 循环获取数据
    receive_size = 0
    # 使用进度条
    file_process_bar = process_bar(file_size)
    file_process_bar.__next__()

    while receive_size < file_size:
        file_info = client.recv(1024)
        # 更新获取的数据
        receive_size += len(file_info)
        file.write(file_info)
        # 更新进度条
        file_process_bar.send(receive_size)

    file.close()
    print(f"file download done...size{file_size}")

client.close()
```



#### 测试

[![img](E:\Development\Typora\images\20200802002742675.gif)](https://img-blog.csdnimg.cn/20200802002742675.gif)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706147.png)](https://img-blog.csdnimg.cn/20200802003014332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### UDP

#### 服务端

```
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server.bind(("127.0.0.1", 50000))

while True:
    data, addr = server.recvfrom(1024)
    print(f"接收到{addr}的数据：{data.decode()}")
    server.sendto(data.decode().upper().encode(), addr)
```



#### 客户端

```
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

while True:
    msg = input(">>").strip()
    client.sendto(msg.encode(),("127.0.0.1", 50000))
    data, addr = client.recvfrom(1024)
    print(f"接收到服务端的消息：{data.decode()}")
```



#### 测试

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-1662829443706148.png)](https://img-blog.csdnimg.cn/20200803081040544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### TCP VS UDP

tcp基于链接通信

- 基于链接，则需要listen（backlog），指定连接池的大小
- 基于链接，必须先运行的服务端，然后客户端发起链接请求
- 对于mac系统：如果一端断开了链接，那另外一端的链接也跟着完蛋recv将不会阻塞，收到的是空(解决方法是：服务端在收消息后加上if判断，空消息就break掉通信循环)
- 对于windows/linux系统：如果一端断开了链接，那另外一端的链接也跟着完蛋recv将不会阻塞，收到的是空(解决方法是：服务端通信循环内加异常处理，捕捉到异常后就break掉通讯循环)

udp无链接

- 无链接，因而无需listen（backlog），更加没有什么连接池之说了
- 无链接，udp的sendinto不用管是否有一个正在运行的服务端，可以己端一个劲的发消息，只不过数据丢失
- recvfrom收的数据小于sendinto发送的数据时，在mac和linux系统上数据直接丢失，在windows系统上发送的比接收的大直接报错
- 只有sendinto发送数据没有recvfrom收数据，数据丢失