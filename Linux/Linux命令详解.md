## 1.系统管理

### 1.终止进程（kill）

> ​		kill 从字面来看，就是用来杀死进程的命令，但事实上，这个或多或少带有一定的误导性。从本质上讲，kill 命令只是用来向进程发送一个信号，至于这个信号是什么，是用户指定的。
>
> ​		也就是说，kill 命令的执行原理是这样的，kill 命令会向操作系统内核发送一个信号（多是终止信号）和目标进程的 PID，然后系统内核根据收到的信号类型，对指定进程进行相应的操作。
>
> kill 命令的基本格式如下：



```bash
[root@localhost ~]# kill [信号] PID
```

kill 命令是按照 PID 来确定进程的，所以 kill 命令只能识别 PID，而不能识别进程名。Linux 定义了几十种不同类型的信号，读者可以使用 kill -l 命令查看所有信号及其编号，这里仅列出几个常用的信号，如表 1 所示。

| 信号编号 | 信号名 | 含义                                                         |
| -------- | ------ | ------------------------------------------------------------ |
| 0        | EXIT   | 程序退出时收到该信息。                                       |
| 1        | HUP    | 挂掉电话线或终端连接的挂起信号，这个信号也会造成某些进程在没有终止的情况下重新初始化。 |
| 2        | INT    | 表示结束进程，但并不是强制性的，常用的 "Ctrl+C" 组合键发出就是一个 kill -2 的信号。 |
| 3        | QUIT   | 退出。                                                       |
| 9        | KILL   | 杀死进程，即强制结束进程。                                   |
| 11       | SEGV   | 段错误。                                                     |
| 15       | TERM   | 正常结束进程，是 kill 命令的默认信号。                       |

**需要注意的是，表中省略了各个信号名称的前缀 SIG，也就是说，SIGTERM 和 TERM 这两种写法都对，kill 命令都可以理解。**

【例 1】 标准 kill 命令。

```bash
[root@localhost ~】# service httpd start
\#启动RPM包默认安装的apache服务
[root@localhost ~]# pstree -p 丨 grep httpd | grep -v "grep"
\#查看 httpd 的进程树及 PID。grep 命令査看 httpd 也会生成包含"httpd"关键字的进程，所以使用“-v”反向选择包含“grep”关键字的进程，这里使用 pstree 命令来查询进程，当然也可以使用 ps 和 top 命令
|-httpd(2246)-+-httpd(2247)
|  |-httpd(2248)
|  |-httpd(2249)
|  |-httpd(2250)
|  |-httpd(2251)
[root@localhost ~]# kill 2248
\#杀死PID是2248的httpd进程，默认信号是15，正常停止
\#如果默认信号15不能杀死进程，则可以尝试-9信号，强制杀死进程
[root@localhost ~]# pstree -p | grep httpd | grep -v "grep"
|-httpd(2246>-+-httpd(2247)
|  |-httpd(2249)
|  |-httpd(2250)
|  |-httpd(2251)
\#PID是2248的httpd进程消失了
```


【例 2】使用“-1”信号，让进程重启。

```bash
[root@localhost ~]# kill -1 2246
使用“-1 (数字1)”信号，让httpd的主进程重新启动
[root@localhost ~]# pstree -p | grep httpd | grep -v "grep"
|-httpd(2246)-+-httpd(2270)
|  |-httpd(2271)
|  |-httpd(2272)
|  |-httpd(2273)
|  |-httpd(2274)
\#子httpd进程的PID都更换了，说明httpd进程已经重启了一次
```


【例 3】 使用“-19”信号，让进程暂停。

```bash
[root@localhost ~]# vi test.sh #使用[vi命令](http://c.biancheng.net/vi/)编辑一个文件，不要退出
[root@localhost ~]# ps aux | grep "vi" | grep -v "grep"
root 2313 0.0 0.2 7116 1544 pts/1 S+ 19:2.0 0:00 vi test.sh
\#换一个不同的终端，查看一下这个进程的状态。进程状态是S（休眠）和+（位于后台），因为是在另一个终端运行的命令
[root@localhost ~]# kill -19 2313
\#使用-19信号，让PID为2313的进程暂停。相当于在vi界面按 Ctrl+Z 快捷键
[root@localhost ~]# ps aux | grep "vi" | grep -v "grep"
root 2313 0.0 0.2 7116 1580 pts/1 T 19:20 0:00 vi test.sh
```

\#注意2313进程的状态，变成了 T（暂停）状态。这时切换回vi的终端,发现vi命令已经暂停，又回到了命令提示符，不过2313进程就会卡在后台。如果想要恢复，可以使用"kill -9 2313”命令强制中止进程，也可以利用后续章节将要学习的工作管理来进行恢复

### 2.终止特定的一类进程(killall)

> killall 也是用于关闭进程的一个命令，但和 kill 不同的是，killall 命令不再依靠 PID 来杀死单个进程，而是通过程序的进程名来杀死一类进程，也正是由于这一点，该命令常与 ps、pstree 等命令配合使用。
>
> killall 命令的基本格式如下：

```bash
[root@localhost ~]# killall [选项] [信号] 进程名
```

注意，此命令的信号类型同 kill 命令一样，因此这里不再赘述，此命令常用的选项有如下 2 个：

- -i：交互式，询问是否要杀死某个进程；
- -I：忽略进程名的大小写；



接下来，给大家举几个例子。

【例 1】杀死 httpd 进程。

```bash
[root@localhost ~]# service httpd start
\#启动RPM包默认安装的apache服务
[root@localhost ~]# ps aux | grep "httpd" | grep -v "grep"
root 1600 0.0 0.2 4520 1696? Ss 19:42 0:00 /usr/local/apache2/bin/httpd -k start
daemon 1601 0.0 0.1 4520 1188? S 19:42 0:00 /usr/local/apache2/bin/httpd -k start
daemon 1602 0.0 0.1 4520 1188? S 19:42 0:00 /usr/local/apache2/bin/httpd -k start
daemon 1603 0.0 0.1 4520 1188? S 19:42 0:00 /usr/local/apache2/bin/httpd -k start
daemon 1604 0.0 0.1 4520 1188? S 19:42 0:00 /usr/local/apache2/bin/httpd -k start
daemon 1605 0.0 0.1 4520 1188? S 19:42 0:00 /usr/local/apache2/bin/httpd -k start
\#查看httpd进程
[root@localhost ~]# killall httpd
\#杀死所有进程名是httpd的进程
[root@localhost ~]# ps aux | grep "httpd" | grep -v "grep"
\#查询发现所有的httpd进程都消失了
```


【例 2】交互式杀死 sshd 进程。

```bash
[root@localhost ~]# ps aux | grep "sshd" | grep -v "grep"
root 1733 0.0 0.1 8508 1008? Ss 19:47 0:00/usr/sbin/sshd
root 1735 0.1 0.5 11452 3296? Ss 19:47 0:00 sshd: root@pts/0
root 1758 0.1 0.5 11452 3296? Ss 19:47 0:00 sshd: root@pts/1
\#查询系统中有3个sshd进程。1733是sshd服务的进程，1735和1758是两个远程连接的进程
[root@localhost ~]# killall -i sshd
\#交互式杀死sshd进程
杀死sshd(1733)?(y/N)n
\#这个进程是sshd的服务进程，如果杀死，那么所有的sshd连接都不能登陆
杀死 sshd(1735)?(y/N)n
\#这是当前登录终端，不能杀死我自己吧
杀死 sshd(1758)?(y/N)y
\#杀死另一个sshd登陆终端
```

### 3.查看正在运行的进程（ps)

**我这里只记录BS格式**

> ps 命令是最常用的监控进程的命令，通过此命令可以查看系统中所有运行进程的详细信息。

​		ps 命令有多种不同的使用方法，这常常给初学者带来困惑。在各种 Linux 论坛上，询问 ps 命令语法的帖子屡见不鲜，而出现这样的情况，还要归咎于 UNIX 悠久的历史和庞大的派系。在不同的 Linux 发行版上，ps 命令的语法各不相同，为此，Linux 采取了一个折中的方法，即融合各种不同的风格，兼顾那些已经习惯了其它系统上使用 ps 命令的用户。

ps 命令的基本格式如下：

```bash
[root@localhost ~]# ps aux
\#查看系统中所有的进程，使用 BS 操作系统格式
[root@localhost ~]# ps -le
\#查看系统中所有的进程，使用 Linux 标准命令格式
```

选项：

> - a：显示一个终端的所有进程，除会话引线外；
> - u：显示进程的归属用户及内存的使用情况；
> - x：显示没有控制终端的进程；
> - -l：长格式显示更加详细的信息；
> - -e：显示所有进程；

可以看到，ps 命令有些与众不同，它的部分选项不能加入"-"，比如命令"ps aux"，其中"aux"是选项，但是前面不能带“-”。

大家如果执行 "man ps" 命令，则会发现 ps 命令的帮助为了适应不同的类 UNIX 系统，可用格式非常多，不方便记忆。所以，我建议大家记忆几个固定选项即可。比如：

> - "ps aux" 可以查看系统中所有的进程；
> - "ps -le" 可以查看系统中所有的进程，而且还能看到进程的父进程的 PID 和进程优先级；
> - "ps -l" 只能看到当前 Shell 产生的进程；


有这三个命令就足够了，下面分别来查看。

【例 1】

```bash
[root@localhost ~]# ps aux
#查看系统中所有的进程
USER PID %CPU %MEM  VSZ  RSS   TTY STAT START TIME COMMAND
root   1  0.0  0.2 2872 1416   ?   Ss   Jun04 0:02 /sbin/init
root   2  0.0  0.0    0    0   ?    S   Jun04 0:00 [kthreadd]
root   3  0.0  0.0    0    0   ?    S   Jun04 0:00 [migration/0]
root   4  0.0  0.0    0    0   ?    S   Jun04 0:00 [ksoftirqd/0]
…省略部分输出…
```

表 1 中罗列出了以上输出信息中各列的具体含义。

| 表头    | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| USER    | 该进程是由哪个用户产生的。                                   |
| PID     | 进程的 ID。                                                  |
| %CPU    | 该进程占用 CPU 资源的百分比，占用的百分比越高，进程越耗费资源。 |
| %MEM    | 该进程占用物理内存的百分比，占用的百分比越高，进程越耗费资源。 |
| VSZ     | 该进程占用虚拟内存的大小，单位为 KB。                        |
| RSS     | 该进程占用实际物理内存的大小，单位为 KB。                    |
| TTY     | 该进程是在哪个终端运行的。其中，tty1 ~ tty7 代表本地控制台终端（可以通过 Alt+F1 ~ F7 快捷键切换不同的终端），tty1~tty6 是本地的字符界面终端，tty7 是图形终端。pts/0 ~ 255 代表虚拟终端，一般是远程连接的终端，第一个远程连接占用 pts/0，第二个远程连接占用 pts/1，依次増长。 |
| STAT    | 进程状态。常见的状态有以下几种：-D：不可被唤醒的睡眠状态，通常用于 I/O 情况。-R：该进程正在运行。-S：该进程处于睡眠状态，可被唤醒。-T：停止状态，可能是在后台暂停或进程处于除错状态。-W：内存交互状态（从 2.6 内核开始无效）。-X：死掉的进程（应该不会出现）。-Z：僵尸进程。进程已经中止，但是部分程序还在内存当中。-<：高优先级（以下状态在 BSD 格式中出现）。-N：低优先级。-L：被锁入内存。-s：包含子进程。-l：多线程（小写 L）。-+：位于后台。 |
| START   | 该进程的启动时间。                                           |
| TIME    | 该进程占用 CPU 的运算时间，注意不是系统时间。                |
| COMMAND | 产生此进程的命令名。                                         |

【例 3】如果不想看到所有的进程，只想查看一下当前登录产生了哪些进程，那只需使用 "ps -l" 命令就足够了：

```bash
[root@localhost ~]# ps -l
#查看当前登录产生的进程
F S UID   PID  PPID C PRI NI ADDR SZ WCHAN TTY       TIME CMD
4 S 0   18618 18614 0  80  0 - 1681  -     pts/1 00:00:00 bash
4 R 0   18683 18618 4  80  0 - 1619  -     pts/1 00:00:00 ps
```



### 4.后台命令脱离终端运行(nohup)

我们需要在远程终端执行某些后台命令,有以下 3 种方法：

1. 把需要在后台执行的命令加入 /etc/rc.local 文件，让系统在启动时执行这个后台程序。这种方法的问题是，服务器是不能随便重启的，如果有临时后台任务，就不能执行了。
2. 使用系统定时任务，让系统在指定的时间执行某个后台命令。这样放入后台的命令与终端无关，是不依赖登录终端的。
3. 使用 nohup 命令。

**nohup 命令的作用就是让后台工作在离开操作终端时，也能够正确地在后台执行。**

此命令的基本格式如下：

```bash
[root@localhost ~]# nohup [命令] &
```

注意，这里的‘&’表示此命令会在终端后台工作；反之，如果没有‘&’，则表示此命令会在终端前台工作。



#### 1、简介

1. nohup 命令运行由 Command参数和任何相关的Arg参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 &（ 表示“and”的符号）到命令的尾部。
2. nohup 是 no hang up 的缩写，就是不挂断的意思。
3. nohup命令：如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。
4. 在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中。

#### 2、语法

```js
nohup command > myout.file 2>&1 &
```

复制

在上面的例子中，0 – stdin (standard input)，1 – stdout (standard output)，2 – stderr (standard error) ；

2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到myout.file文件中。

```js
0 22 * * * /usr/bin/python /home/test.py > /home/test.log 2>&1
```

复制

这是放在crontab中的定时任务，晚上22点时候怕这个任务，启动这个python的脚本，并把日志写在test.log文件中。

#### 3、nohup和&的区别

- &

指在后台运行，但当用户推出(挂起)的时候，命令自动也跟着退出。

- nohup

不挂断的运行，注意并没有后台运行的功能就是指，用nohup运行命令可以使命令永久的执行下去，和用户终端没有关系，例如我们断开SSH连接都不会影响他的运行，注意了nohup没有后台运行的意思；&才是后台运行。

nohup COMMAND &

这样就能使命令永久的在后台执行。

#### 4、举例

```js
sh test.sh &
```

复制

将sh test.sh任务放到后台 ，即使关闭xshell退出当前session依然继续运行，但标准输出和标准错误信息会丢失（缺少的日志的输出）。将sh test.sh任务放到后台 ，关闭xshell，对应的任务也跟着停止。

```js
nohup sh test.sh
```

复制

将sh test.sh任务放到后台，关闭标准输入，终端不再能够接收任何输入（标准输入），重定向标准输出和标准错误到当前目录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。

```js
nohup sh test.sh &
```

复制

将sh test.sh任务放到后台，但是依然可以使用标准输入，终端能够接收任何输入，重定向标准输出和标准错误到当前目录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。