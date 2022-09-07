# Linux中的符号命令 

## 一、总结

在shell中常用的特殊符号罗列如下：
\# ; ;; . , / \ 'string' | ! $ ${} $? $$ $* "string" * ** ? : ^ $# $@ `command` {} [] [[]] () (()) || && {xx,yy,zz,...}~ ~+ ~- & \<...\>  + - %= == !=

## 二、详解

### **# 井号 (comments)**

这几乎是个满场都有的符号，除了先前已经提过的"第一行"
\#!/bin/bash
井号也常出现在一行的开头，或者位于完整指令之后，这类情况表示符号后面的是注解文字，不会被执行。
\# This line is comments.
由于这个特性，当临时不想执行某行指令时，只需在该行开头加上 # 就行了。这常用在撰写过程中。
如果被用在指令中，或者引号双引号括住的话，或者在倒斜线的后面，那他就变成一般符号，不具上述的特殊功能。

1. [root@RHEL6 scripts]# echo "a=$a" # a= 0
2. a=
3. [root@RHEL6 scripts]# #echo "a=$a" # a= 0
4. [root@RHEL6 scripts]#

### **~ 帐户的 home 目录**

算是个常见的符号，代表使用者的 home 目录：cd ~；也可以直接在符号后加上某帐户的名称：cd ~user或者当成是路径的一部份：~/bin
~+ 当前的工作目录，这个符号代表当前的工作目录，她和内建指令 pwd的作用是相同的。
\# echo ~+/var/log
~- 上次的工作目录，这个符号代表上次的工作目录。
\# echo ~-/etc/httpd/logs

### **; 分号 (Command separator)**

1、在 shell 中，担任"连续指令"功能的符号就是"分号"。
例子：cd ~/backup ; mkdir startup ;cp ~/.* startup/.
在命令与命令中间利用分号（；）来隔开，分号前的命令执行完成（无论成功与否）后就会立刻接着执行后面的命令

### **;; 连续分号 (Terminator)**

专用在 case 的选项，担任 Terminator 的角色。

1. case "$fop"
2. inhelp)
3. echo "Usage: Command -help -version filename"
4. ;;
5. version)
6. echo "version 0.1"
7. ;;
8. esac



### **. 点号 (dot,就是“点”)**

1、在 shell 中，使用者应该都清楚，一个 dot 代表当前目录，两个 dot 代表上层目录。
CDPATH=.:~:/home:/home/web:/var:/usr/local
在上行 CDPATH 的设定中，等号后的 dot 代表的就是当前目录的意思。
2、如果档案名称以 dot 开头，该档案就属特殊档案，用 ls 指令必须加上 -a 选项才会显示。
3、在 regular expression 中，一个 dot 代表一定有一个任意字符，空格字符也是一个字符。



### **'string' 单引号 (single quote)**

被单引号用括住的内容，将被视为单一字串。在引号内的代表变数的$符号，没有作用，也就是说，他被视为一般符号处理，防止任何变量替换。
heyyou=homeecho '$heyyou' # We get $heyyou

### **"string" 双引号 (double quote)**

被双引号用括住的内容，将被视为单一字串。它防止通配符扩展，但允许变量扩展。这点与单引数的处理方式不同。
heyyou=homeecho "$heyyou" # We get home

### **`command` 倒引号/反单引号(backticks)**

在前面的单双引号，括住的是字串，但如果该字串是一列命令列，会怎样？答案是不会执行。要处理这种情况，我们得用倒单引号来做。

1. [root@awake scripts]# fdv=`date +%F`
2. [root@awake scripts]# echo "Today $fdv"
3. Today 2015-06-19
4. [root@awake scripts]# 

在倒引号内的 date +%F 会被视为指令，执行的结果会带入 fdv 变量中。
fdv=`date +%F`还有另外一种写法就是fdv=$(date +%F)，两个命令是等价的，只是反单引号(``)容易被看穿('')单引号而已。

### **, 逗点 (comma，标点中的逗号)**

这个符号常运用在运算当中当做"区隔"用途。如下例
\#!/bin/bashlet "t1 = ((a = 5 + 3, b = 7 - 1, c = 15 / 3))"echo "t1= $t1, a = $a, b = $b"

### **/ 斜线 (forward slash)**

在路径表示时，代表目录。
cd /etc/rc.d
cd ../..
cd /
通常单一的 / 代表 root 根目录的意思；在四则运算中，代表除法的符号。
let "num1 = ((a = 10 / 2, b = 25 / 5))"

### **\ 倒斜线**

在交互模式下的escape 字元，有几个作用；放在指令前，有取消 aliases的作用；放在特殊符号前，则该特殊符号的作用消失；放在指令的最末端，表示指令连接下一行。
\# type rmrm is aliased to `rm -i'# \rm ./*.log
上例，我在 rm 指令前加上 escape 字元，作用是暂时取消别名的功能，将 rm 指令还原。
\# bkdir=/home# echo "Backup dir, \$bkdir = $bkdir"Backup dir,$bkdir = /home
上例 echo 内的 \$bkdir，escape 将 $ 变数的功能取消了，因此，会输出 $bkdir，而第二个 $bkdir则会输出变数的内容 /home。

### **| 管道 (pipeline)**

pipeline 是 UNIX 系统，基础且重要的观念。连结上个指令的标准输出，做为下个指令的标准输入。
who | wc -l
善用这个观念，对精简 script 有相当的帮助。

### **!** **惊叹号(negate or reverse)**

** 逻辑运算意义上的非（not）的意思** 
1、通常它代表反逻辑的作用，譬如条件侦测中，用 != 来代表"不等于"

1. if [ "$?" != 0 ]then
2. echo "Executes error"
3. exit 1
4. fi

2、在规则表达式中她担任 "反逻辑" 的角色



1. ls a[!0-9]

上例，代表显示除了a0, a1 .... a9 这几个文件的其他文件。
3、在history命令中“!”号的用法

1. [root@RHEL6 ~]#history
2. ......
3.  1036 echo $$
4.  1037 ll
5.  1038 ll | grep '^d'
6.  1039 ll | grep "^d"
7.  1040 history
8.  1041 ll | grep '^d'
9.  1042 history
10. [root@RHEL6 ~]# !1038 //执行第1038条命令
11. ll | grep '^d'
12. drwxr-xr-x. 2 root root 4096 Jun 12 15:23 bin
13. drwxr-xr-x. 2 root root 4096 Jun 16 15:33 scripts
14. drwxr-xr-x. 7 root root 4096 Nov 21 2014 vmware-tools-distrib
15. [root@RHEL6 ~]# !! //执行上一个命令也就是刚刚的!1038
16. ll | grep '^d'
17. drwxr-xr-x. 2 root root 4096 Jun 12 15:23 bin
18. drwxr-xr-x. 2 root root 4096 Jun 16 15:33 scripts
19. drwxr-xr-x. 7 root root 4096 Nov 21 2014 vmware-tools-distrib
20. [root@RHEL6 ~]# !echo //执行最近一echo为开头的命令，也就是1036那条命令
21. echo $$
22. 4316
23. [root@RHEL6 ~]#

!$的用法

1. mkdir /var/www/html/upload
2. chown -R apache !$ #这时的!$表示上一条命令 mkdir /var/www/html/upload,也就是chown将/var/www/html/upload的所有者权限分配给apache这个用户



### **: 冒号**

在 bash 中，这是一个内建指令："什么事都不干"，但返回状态值 0。
:
echo $? # 回应为 0
: > f.
上面这一行，相当于cat/dev/null>f.
。不仅写法简短了，而且执行效率也好上许多。
有时，也会出现以下这类的用法
: ${HOSTNAME?} ${USER?} ${MAIL?}
这行的作用是，检查这些环境变数是否已设置，没有设置的将会以标准错误显示错误讯息。像这种检查如果使用类似 test 或 if这类的做法，基本上也可以处理，但都比不上上例的简洁与效率。
除了上述之外，还有一个地方必须使用冒号
PATH=$PATH:$HOME/fbin:$HOME/fperl:/usr/local/mozilla
在使用者自己的HOME 目录下的 .bash_profile或任何功能相似的档案中，设定关于"路径"的场合中，我们都使用冒号，来做区隔。

### *** 星号 (wild card)**

相当常用的符号。
1、在文件名扩展(Filename expansion)上，她用来代表0到无穷多个任意字符。

1. [root@RHEL6 ~]# ls a*
2. aaa anaconda-ks.cfg
3. [root@RHEL6 ~]#

2、在正则表达式（Regular Expressions）中，*代表重复零个到无穷多个的前一个字符，如:grep -n 'ess* file.txt ，则可能会匹配es、ess、esss等等。正则表达式中的0到无穷多个字符使用的是“.*”表示。
3、在运算时，它则代表 "乘法"。
let "fmult=2*3"
除了内建指令 let，还有一个关于运算的指令expr，星号在这里也担任"乘法"的角色。不过在使用上得小心，他的前面必须加上escape 字元。

### ***\* 次方运算**

两个星号在运算时代表 "次方" 的意思。
let "sus=2**3"echo "sus = $sus" # sus = 8

### **$及$$ 钱号(dollar sign)**

1、使用变量的前导符，即变量之前需要加的变量替代值  
变量替换(Variable Substitution)的代表符号。

1. [root@RHEL6 ~]# vrs=123
2. [root@RHEL6 ~]# echo "vrs = $vrs"
3. vrs = 123
4. [root@RHEL6 ~]#

2、在 Regular Expressions 里被定义为 "行" 的最末端 (end-of-line)。这个常用在grep、sed、awk 以及 vim(vi) 当中。

1. [root@RHEL6 ~]# ll | grep "txt$" //列出行末是txt结尾的行
2. -rw-r--r--. 1 root root 1700 May 21 10:50 1.txt
3. -rw-r--r--. 1 root root 650 May 31 18:11 123.txt
4. -rw-r--r--. 1 root root 1700 May 21 10:50 2.txt
5. -rw-r--r--. 1 root root 923 May 27 09:20 network.txt
6. -rw-r--r--. 1 root root 96 Jun 1 17:58 printf.txt
7. -rw-r--r--. 1 root root 673 Jun 1 12:24 regular_express.txt

3、在bash中$本身也是个变量。代表的是目前这个shell的进程代码，即所谓的PID（Process ID）想要知道我们当前的shell的PID，可以这样

1. [root@RHEL6 ~]# echo $$
2. 4316
3. [root@RHEL6 ~]#

出现的数字就是你的PID号码
$! 
Shell最后运行的后台Process的PID 

### **? 问号** 

1、在文件名扩展(Filename expansion)上扮演的角色是匹配一个任意的字元，但不包含 null 字元。

1. [root@RHEL6 ~]# ls m?n*
2. man.1 man.test
3. [root@RHEL6 ~]#

善用她的特点，可以做比较精确的档名匹配。
2、在bash中“？”问号也是一个特殊的变量。在bash里面这个变量很重要。这个变量是上一个执行的命令所回传的值。当我们执行某些命令时，这些命令都会回传一个执行后的代码，一般说，如果成功执行该命令，则会回传一个0值，如果执行过程发生错误，就会回传错误代码。一般以非0的数值来替代。
3、在Regular Expressions 正则表达式中(扩展的正则表达式，需要grep -E或者是egrep)“?”代表匹配无和?号前面单一字符，或者类型的实例如4(th)?等于4th或者4

### **$? 状态值 (status variable)**

一般来说，UNIX(linux) 系统的进程以执行系统调用exit()来结束的。这个回传值就是status值。回传给父进程，用来检查子进程的执行状态。
一般指令程序倘若执行成功，其回传值为 0；失败为 1。

1. [root@RHEL6 ~]# tar cvzf backup.tar.gz scripts/ > /dev/null
2. [root@RHEL6 ~]# echo $?
3. 0
4. [root@RHEL6 ~]#

由于进程的ID是唯一的，所以在同一个时间，不可能有重复性的PID。有时，script会需要产生临时文件，用来存放必要的资料。而此script亦有可能在同一时间被使用者们使用。在这种情况下，固定文件名在写法上就显的不可靠。唯有产生动态文件名，才能符合需要。$$符号或许可以符合这种需求。它代表当前shell 的 PID。

1. [root@RHEL6 ~]# echo "$HOSTNAME, $USER, $MAIL" > ftmp.$$
2. [root@RHEL6 ~]# ll ftm*
3. -rw-r--r--. 1 root root 39 Jun 17 09:50 ftmp.4316
4. [root@RHEL6 ~]# echo $$
5. 4316
6. [root@RHEL6 ~]#

使用它来作为文件名的一部份，可以避免在同一时间，产生相同文件名的覆盖现象。
ps: 基本上，系统会回收执行完毕的 PID，然后再次依需要分配使用。所以 script 即使临时文件是使用动态档名的写法，如果script 执行完毕后仍不加以清除，会产生其他问题。



### **${} 变量的正规表达式**

bash 对 ${} 定义了不少用法。以下是取自线上说明的表列
  ${parameter:-word}  ${parameter:=word}  ${parameter:?word}  ${parameter:+word}  ${parameter:offset}  ${parameter:offset:length}  ${!prefix*}  ${#parameter}  ${parameter#word}  ${parameter##word}  ${parameter%word}  ${parameter%%word}  ${parameter/pattern/string}  ${parameter//pattern/string}

### **$\*** 

$* 引用script的执行引用变量，引用参数的算法与一般指令相同，指令本身为0，其后为1，然后依此类推。引用变量的代表方式如下：
$0, $1, $2, $3, $4, $5, $6, $7, $8, $9, ${10}, ${11}.....
个位数的，可直接使用数字，但两位数以上，则必须使用 {} 符号来括住。
$* 则是代表所有引用变量的符号。使用时，得视情况加上双引号。
echo "$*"
还有一个与 $* 具有相同作用的符号，但效用与处理方式略为不同的符号。

### **$@**

$@ 与 $* 具有相同作用的符号，不过她们两者有一个不同点。
符号 $* 将所有的引用变量视为一个整体。但符号 $@ 则仍旧保留每个引用变量的区段观念。

### **$#**

这也是与引用变量相关的符号，她的作用是告诉你，引用变量的总数量是多少。
echo "$#"

### **$(())与declare -i**

declare -i total=$firstnu*$secnu
total=$(($firstnu*secnu))
两个例子是一个意思，就是做整数乘法运算，也可以是+-*/%加减乘除，%号是求余数。
区别就是小方括号内可以加上空格符，也是合法的写法，而declare -i 不可以。

### **(  ) 指令群组 (command group)**

用括号将一串连续指令括起来，这种用法对 shell 来说，称为指令群组。如下面的例子：(cd ~ ; vcgh=`pwd` ;echo $vcgh)，指令群组有一个特性，shell会以产生 subshell来执行这组指令。因此，在其中所定义的变数，仅作用于指令群组本身。我们来看个例子
\# cat ftmp-01#!/bin/basha=fsh(a=incg ; echo -e "\n $a \n")echo $a#./ftmp-01incgfsh
除了上述的指令群组，括号也用在 array 变数的定义上；另外也应用在其他可能需要加上escape字元才能使用的场合，如运算式。

### **((  ))**

这组符号的作用与 let 指令相似，用在算数运算上，是 bash 的内建功能。所以，在执行效率上会比使用 let指令要好许多。
\#!/bin/bash(( a = 10 ))echo -e "inital value, a = $a\n"(( a++))echo "after a++, a = $a"

### **{  } 大括号 (Block of code)**

有时候 script 当中会出现，大括号中会夹着一段或几段以"分号"做结尾的指令或变数设定。
\# cat ftmp-02#!/bin/basha=fsh{a=inbc ; echo -e "\n $a \n"}echo $a#./ftmp-02inbcinbc
这种用法与上面介绍的指令群组非常相似，但有个不同点，它在当前的 shell 执行，不会产生 subshell。
大括号也被运用在 "函数" 的功能上。广义地说，单纯只使用大括号时，作用就像是个没有指定名称的函数一般。因此，这样写 script也是相当好的一件事。尤其对输出输入的重导向上，这个做法可精简 script 的复杂度。


此外，大括号还有另一种用法，如下
**{xx,yy,zz,...}**
这种大括号的组合，常用在字串的组合上，来看个例子
mkdir {userA,userB,userC}-{home,bin,data}
我们得到 userA-home, userA-bin, userA-data, userB-home, userB-bin,userB-data, userC-home, userC-bin,userC-data，这几个目录。这组符号在适用性上相当广泛。能加以善用的话，回报是精简与效率。像下面的例子
chown root /usr/{ucb/{ex,edit},lib/{ex?.?*,how_ex}}
如果不是因为支援这种用法，我们得写几行重复几次呀！

### **[  ] 中括号**

1、在通配符和正则表达式中[]代表一定有一个在中括号内的字符，例如[abcd]代表一定有一个字符，可能是a、b、c、d这四个任何一个；
2、流程控制中，扮演括住判断式的作用。
if [ "$?" != 0 ]then
echo "Executes error"
exit1
fi

**[-]** 
在通配符和正则表达式中都表示范围如[0-9]、[a-z]、[A-Z]，需要注意的是字母的范围与语系有关
**[^]**
在通配符和正则表达式中都表示“非”之意如[^A-Z]，表示非大写字符。

### **[[   ]]**

这组符号与先前的 [] 符号，基本上作用相同，但她允许在其中直接使用 || 与&& 逻辑等符号。
\#!/bin/bashread akif [[ $ak > 5 || $ak< 9 ]]thenecho $akfi

### **|| 逻辑符号**

这个会时常看到，在中括号中[]代表 or 逻辑的符号。
在命令的行中如下
cmd1||cmd2
若cmd1执行完毕且正确执行($?=0)，则cmd2不执行
若cmd1执行完毕且为错误($?≠0)，则开始执行cmd2

### **&& 逻辑符号**

这个也会常看到，在中括号中[]代表 and 逻辑的符号。
在命令行中如下
cmd1&&cmd2 
若cmd1执行完毕且正确执行（$?=0）,则开始执行cmd2
若cmd1执行完毕且为错误（$?≠0），则cmd2不执行

1. [root@RHEL6 ~]# ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe //如果/tmp/abc目录不存在则创建这个目录，成功后在目录下创建hehe文件
2. ls: cannot access /tmp/abc: No such file or directory //这个就是ls /tmp/abc标准错误输出
3. [root@RHEL6 ~]# ll /tmp/abc/hehe
4. -rw-r--r--. 1 root root 0 Jun 17 11:00 /tmp/abc/hehe //已经创建了文件
5. [root@RHEL6 ~]# ls /tmp/bcd || echo "not exist" && echo "exist" //这个实例告诉我们||与&&的使用是要注意顺序的。
6. ls: cannot access /tmp/bcd: No such file or directory
7. not exist
8. exist
9. [root@RHEL6 ~]# ls /tmp/bcd && echo "exist" || echo "not exist" //呈上，这个是正确的顺序
10. ls: cannot access /tmp/bcd: No such file or directory
11. not exist
12. [root@RHEL6 ~]#
13. 一般来说，假设判断式有三个，也就是cmd1 && cmd2 || cmd3，而且顺序通常不会变，因为一般说cmd2与cmd3会放置肯定可以执行成功的命令。


**& 后台工作**
单一个& 符号，且放在完整指令列的最后端，即表示将该指令列放入后台中工作。
tar cvfz data.tar.gz data > /dev/null&


**\<...\> 单字边界**
这组符号在规则表达式中，被定义为"边界"的意思。譬如，当我们想找寻 the 这个单字时，如果我们用
grep the FileA
你将会发现，像 there 这类的单字，也会被当成是匹配的单字。因为 the 正巧是 there的一部份。如果我们要必免这种情况，就得加上 "边界" 的符号
grep '\' FileA

**+ 加号 (plus)**
在运算式中，她用来表示 "加法"。
expr 1 + 2 + 3
此外在规则表达式中，用来表示"很多个"的前面字元的意思。
\# grep '10\+9' fileB109100910000910000931010009#这个符号在使用时，前面必须加上escape 字元。

**- 减号 (dash)**
在运算式中，她用来表示 "减法"。
expr 10 - 2
此外也是系统指令的选项符号。
ls -expr 10 - 2
在 GNU 指令中，如果单独使用 - 符号，不加任何该加的文件名称时，代表"标准输入"的意思。这是 GNU指令的共通选项。譬如下例
tar xpvf -
这里的 - 符号，既代表从标准输入读取资料。
不过，在 cd 指令中则比较特别
cd -
这代表变更工作目录到"上一次"工作目录。

**% 除法 (Modulo)**
在运算式中，用来表示 "除法"。
expr 10 % 2
此外，也被运用在关于变量的规则表达式当中的下列
${parameter%word}${parameter%%word}
一个 % 表示最短的 word 匹配，两个表示最长的 word 匹配。

**= 等号 (Equals)**
常在设定变数时看到的符号。
vara=123echo " vara = $vara"
或者像是 PATH 的设定，甚至应用在运算或判断式等此类用途上。

**== 等号 (Equals)**
常在条件判断式中看到，代表 "等于" 的意思。
if [ $vara == $varb ]
...下略

**!= 不等于**
常在条件判断式中看到，代表 "不等于" 的意思。
if [ $vara != $varb ]
...下略


**^**
1、这个符号在正则表达式中，代表行的 "开头" 位置

1. [root@RHEL6 ~]# ll | grep "^d" 
2. drwxr-xr-x. 2 root root 4096 Jun 12 15:23 bin
3. drwxr-xr-x. 2 root root 4096 Jun 16 15:33 scripts
4. drwxr-xr-x. 7 root root 4096 Nov 21 2014 vmware-tools-distrib
5. [root@RHEL6 ~]#

2、在[]中也与"!"(叹号)一样表示“非”
如[^A-Z]，表示非大写字符，[^abc]表示非a、b、c这3个字符，[^]的用法在通配符中和正则表达式中意思是一样的。

**输出/输入重导向**
\>    >>  <  <<  :>  &>  2&>  2<>>&  >&2  

文件描述符(File Descriptor)，用一个数字（通常为0-9）来表示一个文件。
常用的文件描述符如下：
文件描述符      名称     常用缩写   默认值
   0        标准输入    stdin       键盘
   1        标准输出    stdout     屏幕
   2       标准错误输出  stderr      屏幕
我们在简单地用<或>时，相当于使用 0< 或 1>（下面会详细介绍）。
\* cmd > file
把cmd命令的输出重定向到文件file中。如果file已经存在，则清空原有文件，使用bash的noclobber选项可以防止复盖原有文件。
\* cmd >> file
把cmd命令的输出重定向到文件file中，如果file已经存在，则把信息加在原有文件後面。
\* cmd < file
使cmd命令从file读入
\* cmd << text
从命令行读取输入，直到一个与text相同的行结束。除非使用引号把输入括起来，此模式将对输入内容进行shell变量替换。如果使用<<- ，则会忽略接下来输入行首的tab，结束行也可以是一堆tab再加上一个与text相同的内容，可以参考後面的例子。
\* cmd <<< word
把word（而不是文件word）和後面的换行作为输入提供给cmd。
\* cmd <> file
以读写模式把文件file重定向到输入，文件file不会被破坏。仅当应用程序利用了这一特性时，它才是有意义的。
\* cmd >| file
功能同>，但即便在设置了noclobber时也会复盖file文件，注意用的是|而非一些书中说的!，目前仅在csh中仍沿用>!实现这一功能。
: > filename    把文件"filename"截断为0长度.# 如果文件不存在, 那么就创建一个0长度的文件(与'touch'的效果相同).
cmd >&n 把输出送到文件描述符n
cmd m>&n 把输出到文件符m的信息重定向到文件描述符n
cmd >&- 关闭标准输出
cmd <&n 输入来自文件描述符n
cmd m<&n m来自文件描述各个n
cmd <&- 关闭标准输入
cmd <&n- 移动输入文件描述符n而非复制它。
cmd >&n- 移动输出文件描述符 n而非复制它。
注意： >&实际上复制了文件描述符，这使得cmd > file 2>&1与cmd 2>&1 >file的效果不一样。

 



## 三、部分详详解

### 1.$符号命令

#### `$、$()、${}`的区别

#### 2.1$

$ 在linux里是用来指明变量。
在Shell 脚本中向脚本传递参数也会用到 $，例如脚本内获取参数的格式为：$n，n 代表一个数字，$1 为执行脚本的第一个参数，$2 为执行脚本的第二个参数，以此类推。

#### 2.2 $()

小括号里面是linux命令
比如
cat ( p w d ) > a a a 等 价 于 c a t ‘ p w d ‘ > a a a 其 实 是 要 执 行 里 面 的 p w d 然 后 用 输 出 代 替 (pwd)>aaa 等价于 cat `pwd`>aaa 其实是要执行里面的pwd然后用输出代替(pwd)>aaa等价于cat‘pwd‘>aaa其实是要执行里面的pwd然后用输出代替()内容。

#### 2.3 ${}

大括号里面则是数组变量
举个例子
$A = (hello linux shell)
$echo ${A[0]}
则会输出hello

${0} 代表命令本身
${1} 代表命令后输入的第1个参数
${2} 代表命令后输入的第2个参数

\#====== 示例 ================
./main.sh -f xxx
${0} 代表 ./main.sh
${1} 代表 -f
${2} 代表 xxx

### 3 $组合命令

$$
Shell本身的PID（ProcessID，即脚本运行的当前进程ID号）
$!
Shell最后运行的后台Process的PID(后台运行的最后一个进程的进程ID号)
$?
最后运行的命令的结束代码（返回值）即执行上一个指令的返回值 (显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误)
$-
显示shell使用的当前选项，与set命令功能相同
∗ 所 有 参 数 列 表 。 如 " * 所有参数列表。如"∗所有参数列表。如"*“用「”」括起来的情况、以"$1 $2 … $n"的形式输出所有参数，此选项参数可超过9个。
@ 所 有 参 数 列 表 。 如 " @ 所有参数列表。如"@所有参数列表。如"@“用「”」括起来的情况、以"$1" “2 " … " 2" … "2"…"n” 的形式输出所有参数。
@ 跟 @ 跟@跟*类似，但是可以当作数组用
$#
添加到Shell的参数个数
$0
Shell本身的文件名
1 ～ 1～1～n
添加到Shell的各参数值。$1是第1参数、$2是第2参数…。