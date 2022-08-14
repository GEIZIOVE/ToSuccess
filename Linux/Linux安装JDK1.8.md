# Linux安装JDK1.8

## 前言

在 Linux系统上安装 JDK的时候，基本上所有的资料都要你检查并卸载掉系统上原生的 Open JDK，然后再进行 JDK的安装。那么 Open JDK和 JDK有什么区别呢？

其实，Open JDK是 JDK的原始开放代码，JDK7就是在 Open JDK7的基础上发布的。可以简单的认为，Open JDK 是基础版，而 JDK是发行版。

我们不使用Open JDK，这其中最重要的有两点：

- Open JDK不包含 Deployment（部署）功能
  - 部署的功能包括：Browser Plugin、Java Web Start、以及Java控制面板，这些功能在Open JDK中是找不到的
- Open JDK源代码不完整
  - JDK的一部分源代码因为产权的问题无法开放 Open JDK使用，导致Open JDK进行了一些源代码的替换
  - 而且Open JDK只包含最精简的 JDK，没有其他的软件包，得自己去下载，环境的配置相对更麻烦
  - 

## 卸载Open JDK

首先，我们先检查系统是否自带了 JDK。输入命令

```bash
java –version
```



rpm -qa | grep java

rpm -e --nodeps java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64

rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64

 

## 开始安装：

mkdir /usr/local/src/java

rz 上传jdk tar包

tar -xvf jdk-7u71-linux-i586.tar.gz 

 

yum install glibc.i686

 

## 配置环境变量：

① vi /etc/profile

 

② 在末尾行添加

​    \#set java environment

​    JAVA_HOME= /usr/jdk/jdk1.8.0_241

​    CLASSPATH=.:$JAVA_HOME/lib.tools.jar

​    PATH=$JAVA_HOME/bin:$PATH

​    export JAVA_HOME CLASSPATH PATH

 

CentOS8或者这个

export JAVA_HOME= /usr/jdk/jdk1.8.0_241 #指定自己的jdk文件路径

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin

 

“=”两边都不能出现空格，直接复制该环境变量配置代码至  /etc/profile中 时，”=”自动产生了空格，导致路径识别不出来。

 

保存退出

③source /etc/profile 使更改的配置立即生效

④java -version 查看JDK版本信息，如果显示出1.7.0证明成功

 

export JAVA_HOME= /usr/jdk/jdk1.8.0_241 #指定自己的jdk文件路径

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin