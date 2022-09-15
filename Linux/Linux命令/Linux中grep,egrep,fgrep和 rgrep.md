# Linux中grep,egrep,fgrep和 rgrep

> Linux 系统上的grep 命令最常见的命令之一。功能概括起来就是查找文件中的制定字符串.。使用花样技巧很多。

> `grep`命令还有一些近亲，适应不同场景。像`egrep`,`fgrep`和`rgrep`。这些命令的工作方式都与`grep`类似，但扩展了它的功能，有时还简化了它的语法。看起来有点混乱。看完下面的介绍想必就会清楚一些。

> 在本教程中，我们将介绍`grep`、`egrep`、`fgrep`和`rgrep`的各种命令示例。

**egrep–支持更强的正则表达式匹配**

**fgrep–更快，但是不能正则匹配**

**rgrep--可以递归搜索**

### grep

> 为了便于演示我们创建了一个名为的简单文本文档`rumenz.txt`，其中包含一堆 Linux 发行版的名称。

1.`grep`可用于在文件中搜索字符串。我们搜索`Ubuntu`试试：

```bash
$ grep Ubuntu rumenz.txt 
Ubuntu
```

2.与 Linux 中的其他所有内容一样，`grep`它也区分大小写。要忽略大小写，我们需要使用`-i`参数：

```bash
$ grep -i ubuntu rumenz.txt 
Ubuntu
Kubuntu
Xubuntu
```

3.`-n`参数会显示找到每个匹配项的行号。

```bash
$ grep -i -n ubuntu rumenz.txt 
3:Ubuntu
8:Kubuntu
9:Xubuntu
```

4.我们还可以使用`-v`(invert) 选项来显示与我们的搜索模式`不匹配的行`

```bash
$ grep -iv ubuntu rumenz.txt
Arch Linux
AlmaLinux
Fedora
Red Hat Enterprise Linux
CentOS
Linux Mint
Debian
Manjaro
openSUSE
```

8.使用`-c`参数，grep 可以计算文件中字符串出现的次数。所以这里 grep 将打印 Ubuntu 没有出现在文件中的次数：

```bash
$ grep -ivc ubuntu rumenz.txt
9
```

9.`-x`参数将仅打印明确匹配项。

```bash
$ grep -ix ubuntu rumenz.txt
Ubuntu
```

10.系统管理员在搜索日志文件时肯定用到这个示例。`-B3`（在匹配前显示 3 行)和`-A3`(在匹配后显示 3 行)。

```bash
$ grep -B3 -A3 command /var/log/dmesg
[201120] kernel: pcpu-alloc: [0] 0 
[201186] kernel: Built 1 zonelists, mobility grouping on.  Total pages: 515961
[201188] kernel: Policy zone: DMA32
`[201191] kernel: Kernel command line: BOOT_IMAGE=/boot/vmlinuz-0-59-generic root=UUID=a80ad9d4-90ff-4903-b34d-ca70d82762ed ro quiet splash`
[201563] kernel: Dentry cache hash table entries: 262144 (order: 9, 2097152 bytes, linear)
[201648] kernel: Inode-cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
[201798] kernel: mem auto-init: stack:off, heap alloc:on, heap free:off
```

### grep 和正则表达式

> grep 和正则表达式组合使用,功能更加强大。

1.`grep`只返回包含数字的行，我们将使用以下命令：

```bash
$ grep [0-9] file.txt
```

2.`grep`返回所有空行：

```bash
$ grep -ch ^$ file.txt
```

3.让我们看看哪一行以`L`开头并以数字结尾。`^`用于匹配行首，`$`用于匹配行尾：

```bash
$ grep ^L.*[0-9]$ file.txt
```

4.要`grep`只匹配单词中`b`是第三个字符的行，我们可以使用以下命令：

```bash
$ grep ..b file.txt`
```

### egrep

> `egrep`是`grep` 的扩展版本。换句话说，`egrep`等于`grep -E`。egrep 支持更多的正则表达式模式。

1.让我们搜索恰好包含两个连续`p`字符的行：

```bash
$ egrep p{2} file.txt
OR
$ grep pp file.txt
OR
$ grep -E p{2} file.txt
```

2.`egrep`让我们得到以`S`或`A`结尾的所有行的命令输出：

```bash
$ egrep "S$|A$" file.txt
```

### fgrep

> `fgrep`是一个更快的版本，`fgrep`等于`grep -F`。fgrep 查询速度比grep命令快，但是不够灵活：它只能找固定的文本,不支持正则表达式。

1.你只能使用此工具进行简单的模式搜索,例如：

```bash
$ fgrep Fedora rumenz.txt 
Fedora
```

2.表达式将不起生效，只会返回空白输出。

```bash
$ fgrep -i linux$ rumenz.txt 
$ grep -i linux$ rumenz.txt 
Arch Linux
AlmaLinux
Red Hat Enterprise Linux
```

### rgrep

> `rgrep`是`grep`的递归版本。递归意味着 `rgrep` 可以递归地遍历目录，因为它对指定的模式进行`grep`。`rgrep`类似于`grep -r`。

**搜索所有文件，递归搜索字符串`linux`**

```bash
$ rgrep -i linux *
dir1/rumenz.txt:AlmaLinux
dir1/rumenz.txt:Red Hat Enterprise Linux
dir2/rumenz.txt:Linux Mint
```



# 问题记录

## [grep命令提示"binary file matches **.log"解决方法](https://www.cnblogs.com/amyzhu/p/11160736.html)

仔细想想，这个问题遇到很多次了，之前一直以为很复杂，一搜索发现解决这么简单，记录一下做备忘。

```bash
grep test XXX.log
Binary file app.log matches
```

此时使用`-a`参数接口。

```bash
grep -a test XXX.log
```

- `-a, --text equivalent to --binary-files=text`，即让二进制文件等价于文本。

注：`zgrep`遇到同样问题，解决方法也是类似

