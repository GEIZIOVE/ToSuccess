# Linux后台执行命令并获取pid，使用pid文件杀死进程

有一些脚本或命令在执行的时候会占据屏幕，当关掉终端时该命令就被杀掉了，有时候我们想让它们在后台执行，甚至想要在执行之后获得他的pid以便于追踪它的执行状况，可以使用下面的方式：

**后台执行command命令：** 

```bash
nohup command > cmd.out 2>&1 &
```

 **后台执行command命令并获取pid：**



```bash
#后台执行命令并打印它的pid
nohup command > cmd.out 2>&1 & echo $!
 
#后台执行命令并将它的pid保存在cmd.pid文件内
nohup command > cmd.out 2>&1 & echo $! > cmd.pid
```

 **根据pid文件杀死进程：**

```bash
kill -9 `cat cmd.pid`
```

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzZGMwNTIx,size_16,color_FFFFFF,t_70.png)

 

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzZGMwNTIx,size_16,color_FFFFFF,t_70-16622952338481.png)