## **〇、序**

Python是一种面向对象的解释型计算机程序设计语言，其使用，具有跨平台的特点，可以在Linux、macOS以及Windows系统中搭建环境并使用，其编写的代码在不同平台上运行时，几乎不需要做较大的改动，使用者无不受益于它的便捷性。

此外，Python的强大之处在于它的应用领域范围之广，遍及人工智能、科学计算、Web开发、系统运维、大数据及云计算、金融、游戏开发等。实现其强大功能的前提，就是Python具有数量庞大且功能相对完善的标准库和第三方库。通过对库的引用，能够实现对不同领域业务的开发。然而，正是由于库的数量庞大，对于管理这些库以及对库作及时的维护成为既重要但复杂度又高的事情。

**官网[Getting started with conda — conda 4.14.0.post32+7c5e2cd70 documentation](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html)**





## **一、什么是Anaconda？**

### **1. 简介**

> Anaconda（[官方网站](https://link.zhihu.com/?target=https%3A//www.anaconda.com/download/%23macos)）就是可以便捷获取包且对包能够进行管理，同时对环境可以统一管理的发行版本。Anaconda包含了conda、Python在内的超过180个科学包及其依赖项。

### **2. 特点**

`Anaconda`具有如下特点：

​	▪ 开源

​	▪ 安装过程简单

​	▪ 高性能使用Python和R语言

​	▪ 免费的社区支持

其特点的实现主要基于`Anaconda`拥有的：

​	▪ conda包

​	▪ 环境管理器

​	▪ 1,000+开源库

如果日常工作或学习并不必要使用1,000多个库，那么可以考虑安装Miniconda（[下载界面请戳](https://link.zhihu.com/?target=https%3A//conda.io/miniconda.html)），这里不过多介绍Miniconda的安装及使用。



### **3. Anaconda、conda、pip、virtualenv的区别**

#### **① Anaconda**

Anaconda是一个包含180+的科学包及其依赖项的发行版本。其包含的科学包包括：`conda, numpy, scipy, python notebook`等。



#### **② conda**

conda是包及其依赖项和环境的管理工具。

▪ 适用语言：`Python, R, Ruby, Lua, Scala, Java, JavaScript, C/C++, FORTRAN`。

▪ 适用平台：Windows, macOS, Linux

▪ 用途：

① 快速安装、运行和升级包及其依赖项。

② 在计算机中便捷地创建、保存、加载和切换环境。

> 如果你需要的包要求不同版本的Python，你无需切换到不同的环境，因为conda同样是一个环境管理器。仅需要几条命令，你可以创建一个完全独立的环境来运行不同的Python版本，同时继续在你常规的环境中使用你常用的Python版本。——[Conda官方网站](https://link.zhihu.com/?target=https%3A//conda.io/docs/)

▪ conda为Python项目而创造，但可适用于上述的多种语言。

▪ conda包和环境管理器包含于`Anaconda`的所有版本当中。



#### **③ pip**

pip是用于安装和管理软件包的包管理器。

▪ pip编写语言：`Python`。

▪ Python中默认安装的版本：

① Python 2.7.9及后续版本：默认安装，命令为 ***pip***

② Python 3.4及后续版本：默认安装，命令为 ***pip3***

▪ pip名称的由来：pip采用的是**递归缩写**进行命名的。其名字被普遍认为来源于2处：

① “Pip installs Packages”（“pip安装包”）

② “Pip installs Python”（“pip安装Python”）



#### **④ virtualenv**

virtualenv是用于创建一个**独立的**Python环境的工具。

▪ 解决问题：

1. 当一个程序需要使用Python 2.7版本，而另一个程序需要使用Python 3.6版本，如何同时使用这两个程序？如果将所有程序都安装在系统下的默认路径，如：***/usr/lib/python2.7/site-packages\***，当不小心升级了本不该升级的程序时，将会对其他的程序造成影响。
2. 如果想要安装程序并在程序运行时对其库或库的版本进行修改，都会导致程序的中断。
3. 在共享主机时，无法在全局 ***site-packages\*** 目录中安装包。

▪ virtualenv将会为它自己的安装目录创建一个环境，这并**不与**其他virtualenv环境共享库；同时也可以**选择性**地不连接已安装的全局库。



#### **⑤ pip 与 conda 比较**

##### **→ 依赖项检查**

▪ pip：

① **不一定**会展示所需其他依赖包。

② 安装包时**或许**会直接忽略依赖项而安装，仅在结果中提示错误。

▪ `conda`：

① 列出所需其他依赖包。

② 安装包时自动安装其依赖项。

③ 可以便捷地在包的不同版本中自由切换。



##### **→ 环境管理**

▪ pip：维护多个环境难度较大。

▪ `conda`：比较方便地在不同环境之间进行切换，环境管理较为简单。



##### **→ 对系统自带Python的影响**

▪ pip：在系统自带Python中包的更新/回退版本/卸载将影响其他程序。

▪ `conda`：不会影响系统自带Python。



##### **→ 适用语言**

▪ pip：仅适用于Python。

▪ `conda`：适用于Python, R, Ruby, Lua, Scala, Java, JavaScript, C/C++, FORTRAN。



#### **⑥ conda与pip、virtualenv的关系**

▪ `conda`**结合**了`pip`和`virtualenv`的功能。



## 二、快速入门

### 启动conda

window

- 从“开始”菜单中，搜索并打开“`Anaconda Prompt`”。

[![../_images/anaconda-prompt.png](E:\Development\Typora\images\anaconda-prompt.png)](https://conda.io/projects/conda/en/latest/_images/anaconda-prompt.png)

**在 Windows 上，下面的所有命令都键入 Anaconda Prompt 窗口。**



**苹果操作系统**

- 打开快速启动板，然后单击终端图标。

在 macOS 上，下面的所有命令都键入到终端窗口中。

**Linux**

- 打开终端窗口。

在 Linux 上，下面的所有命令都键入到终端窗口中。



### 管理conda

通过键入以下内容来验证 conda 是否已在系统上安装并运行：

> ```bash
> conda --version
> ```

Conda 显示已安装的版本号。您不需要导航到 Anaconda 目录。

例：`conda 4.7.12`

注意

如果收到错误消息，请确保在安装后关闭并重新打开终端窗口，或者立即执行此操作。然后验证您是否登录到用于安装 Anaconda 或 Miniconda 的同一用户帐户。

将 conda 更新到当前版本。键入以下内容：

> ```bash
> conda update conda
> ```

Conda 比较版本，然后显示可供安装的版本。

如果有较新版本的 conda 可用，请键入以更新：`y`

> ```bash
> Proceed ([y]/n)? y
> ```

提示

我们建议您始终将 conda 更新到最新版本。



### 管理环境

Conda 允许您创建单独的环境，其中包含不会与其他环境交互的文件、包及其依赖项。

当您开始使用 conda 时，您已经有一个名为 的默认环境。但是，您不想将程序放入基本环境中。创建单独的环境以保持程序彼此隔离。`base`

1. ### 创建一个新环境并在其中安装包。

   我们将命名环境并安装BioPython软件包。在 Anaconda 提示符下或在终端窗口中，键入以下内容：`snowflakes`

   ```bash
   conda create --name snowflakes biopython
   ```

   Conda检查BioPython需要哪些其他软件包（“依赖项”），并询问您是否要继续：

   ```bash
   Proceed ([y]/n)? y
   ```

   **键入“y”，然后按回车键继续。**

2. 要使用或“激活”新环境，请键入以下内容：

   - windows：`conda activate snowflakes`
   - macOS 和 Linux：`conda activate snowflakes`

   注意

   `conda activate`仅适用于 conda 4.6 及更高版本。

   对于 4.6 之前的 conda 版本，请键入：

   - windows：`activate snowflakes`
   - macOS 和 Linux：`source activate snowflakes`

   现在您已进入您的环境，您键入的任何 conda 命令都将转到该环境，直到您将其停用。`snowflakes`

3. 若要查看所有环境的列表，请键入：

   ```bash
   conda info --envs
   ```

   此时将显示一个环境列表，类似于以下内容：

   ```bash
   conda environments:
   
       base           /home/username/Anaconda3
       snowflakes   * /home/username/Anaconda3/envs/snowflakes
   ```

   > 提示
   >
   > 活动环境是带有**星号 （*）** 的环境。

4. 将当前环境更改回默认值（基本）：`conda activate`

   注意

   对于 conda 4.6 之前的版本，请使用：

   > - windows：`activate`
   > - macOS， Linux：`source activate`

   > 提示
   >
   > 停用环境后，其名称将不再显示在提示中，并且星号 （*） 将返回到基数。要进行验证，您可以重复该命令。`conda info --envs`



### 管理Python

创建新环境时，conda 会安装您下载并安装 Anaconda 时使用的相同 Python 版本。如果你想使用不同版本的Python，例如Python 3.5，只需创建一个新环境并指定所需的Python版本。

1. Create a new environment named "snakes" that contains Python 3.9:

   ```bash
   conda create --name snakes python=3.9
   ```

   When conda asks if you want to proceed, type "y" and press Enter.

2. Activate the new environment:

   - Windows: `conda activate snakes`
   - macOS and Linux: `conda activate snakes`

   注意

   `conda activate`仅适用于 conda 4.6 及更高版本。

   对于 4.6 之前的 conda 版本，请键入：

   - 窗户：`activate snakes`
   - macOS 和 Linux：`source activate snakes`

3. 验证 snakes 环境是否已添加且处于活动状态：

   ```
   conda info --envs
   ```

   Conda 显示所有环境的列表，并在活动环境的名称后加上星号 （*）：

   ```
   # conda environments:
   #
   base                     /home/username/anaconda3
   snakes                *  /home/username/anaconda3/envs/snakes
   snowflakes               /home/username/anaconda3/envs/snowflakes
   ```

   活动环境也显示在提示符（括号）或 [括号] 的前面，如下所示：

   ```
   (snakes) $
   ```

4. 验证当前环境中的 Python 版本：

   ```
   python --version
   ```

5. 停用蛇的环境，回到基地环境：`conda activate`

   注意

   对于 conda 4.6 之前的版本，请使用：

   > - 窗户：`activate`
   > - macOS， Linux：`source activate`



### 管理包

在本节中，您将检查已安装哪些包，检查哪些包可用，然后查找特定包并进行安装。

1. 若要查找已安装的包，请首先激活要搜索的环境。在上面查找[激活蛇环境的](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-envs)命令。

2. 检查您尚未安装的名为“beautifulsoup4”的软件包是否可从 Anaconda 存储库获得（必须连接到 Internet）：

   ```
   conda search beautifulsoup4
   ```

   Conda 在 Anaconda 存储库中显示具有该名称的所有软件包的列表，因此我们知道它是可用的。

3. 将此软件包安装到当前环境中：

   ```
   conda install beautifulsoup4
   ```

4. 检查新安装的程序是否在此环境中：

   ```
   conda list
   ```

