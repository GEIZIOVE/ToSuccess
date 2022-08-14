# 1.Git安装

## windows安装

下载安装包后需要设置用户名和邮箱

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。

此外

查看用户名和邮箱地址：

```
$ git config user.name

$ git config user.email
```

修改用户名和邮箱地址

```
$  git config --global user.name  "xxxx"

S  git config --global user.email  "xxxx"
```



# 2.创建仓库

仓库又叫 版本库，英文名**repository**，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。



## **在现有目录中初始化仓库**

使用Git对现有的项目进行管理，只需进入**该项目目录**并输入：

```
$ git init
```

以上命令将在该项目目录下创建一个.git的子目录，包含该Git仓库中所有的必须文件。

![image-20220507144441959](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220507144441959.png)

## 把文件添加到版本库

如果该项目目录（初始化Git仓库的文件夹）已经存在文件，可以开始跟踪这些文件并提交。使用git add命令实现对指定文件或文件夹的跟踪

**第一步**，用命令`git add`告诉Git，把文件添加到仓库：

```java
$ git add *.c
```

**第二步**，用命令`git commit`告诉Git，把文件提交到仓库：

```java
$ git commit -m '初始化项目版本' //`-m`后面输入的是本次提交的说明，可以输入任意内容
```



Q:第一步提交后若出现

```
$ git add readme.txt

warning: LF will be replaced by CRLF in readme.txt.
The file will have its original line endings in your working directory
```

A：设置**git config --global core.autocrlf false**，不允许自动更换或者Vscode右下角LF设置自己调整为CRLF

### 问题

Q:为什么Git添加文件需要`add`，`commit`一共两步呢？

A:因为`commit`可以一次提交很多文件，所以你可以多次`add`不同的文件，比如：

```java
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."	
```



Q：输入`git add readme.txt`，得到错误：`fatal: not a git repository (or any of the parent directories)`。

A：Git命令必须在Git仓库目录内执行（`git init`除外），在仓库目录外执行是没有意义的。



Q：输入`git add readme.txt`，得到错误`fatal: pathspec 'readme.txt' did not match any files`。

A：添加某个文件时，该文件必须在当前目录下存在，用`ls`或者`dir`命令查看当前目录的文件，看看文件是否存在，或者是否写错了文件名。

## 删除本地仓库

1. 删除.git文件夹

2. 打开Git Bash命令行界面，输入rm -rf仓库地址即可





# 3.时光机穿梭

##  掌握工作区的状态

当通过git add命令提交的文件发送变动时，可运行git status命令查看结果

```
$ git status
```

​	若一个文件发生变化

```c
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt  //git status告诉我们，将要被提交的修改包括readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

​	上面的命令输出告诉我们，`readme.txt`被修改过了，但还没有准备提交的修改。

- 要随时掌握工作区的状态，使用`git status`命令。
- 如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

## 版本回退

 s在Git中，我们用`git log`命令查看 文 件一共有几个版本被提交到Git仓库里:

```c
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800

    append GPL

commit e475afc93c209a690c39c13a46716e8fa000c366
Author: MichaelLiao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800

    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800

    wrote a readme file
```



`git log`命令显示从最近到最远的提交日志，我们可以看到3次提交:

最近的一次是	`append GPL`，上一次是	`add distributed`，最早的一次是	`wrote a readme file`。

​	

如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：

```
$ git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
e475afc93c209a690c39c13a46716e8fa000c366 add distributed
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file
```

- 看到的一大串类似`1094adb...`的是`commit id`（版本号），他是一个SHA1计算出来的一个非常大的数字，用十六进制表示。
- 每提交一个新版本，实际上Git就会把它们自动串成一条时间线。



准备把`readme.txt`回退到上一个版本，也就是`add distributed`的那个版本，怎么做呢？

首先，Git必须知道当前版本是哪个版本，在Git中，用`HEAD`表示当前版本，也就是最新的提交`1094adb...`，上一个版本就是`HEAD^`，

> 上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

使用`git reset`命令：

```c
$ git reset --hard HEAD^
HEAD is now at e475afc add distributed
```

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。



## 工作区和暂存区

#### 工作区（Working Directory）

就是你在电脑里能看到的目录，比如我的`learngit`文件夹就是一个工作区：

![working-dir](https://www.liaoxuefeng.com/files/attachments/919021113952544/0)

#### 版本库（Repository）

工作区有一个<u>隐藏目录</u>`.git`，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（windows叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

![git-repo](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)

前面讲了我们把文件往Git版本库里添加的时候，是分两步执行的：

**第一步**是用`git add`把文件添加进去，**实际上就是把文件修改添加到暂存区**；

第二步是用`git commit`提交更改，**实际上就是把暂存区的所有内容提交到当前分支**。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。

你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

我们再对`readme.txt`做个修改，比如加上一行内容：

```c
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
```

然后，在工作区新增一个`LICENSE`文本文件（内容随便写）。

先用`git status`查看一下状态：

```c
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	LICENSE

no changes added to commit (use "git add" and/or "git commit -a")
```

Git非常清楚地告诉我们，`readme.txt`被修改了，而`LICENSE`还从来没有被添加过，所以它的状态是`Untracked`。

​	Untracked表示文件未被git管理或

add

现在，使用两次命令`git add`，把`readme.txt`和`LICENSE`都添加后，用`git status`再查看一下：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   LICENSE
	modified:   readme.txt
```

现在，暂存区的状态就变成这样了：

![git-stage](https://www.liaoxuefeng.com/files/attachments/919020074026336/0)

所以，`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

```
$ git commit -m "understand how stage works"
[master e43a48b] understand how stage works
 2 files changed, 2 insertions(+)
 create mode 100644 LICENSE
```

一旦提交后，如果你又没有对工作区做任何修改，那么工作区就是“干净”的：

```
$ git status
On branch master
nothing to commit, working tree clean
```

现在版本库变成了这样，暂存区就没有任何内容了：

![git-stage-after-commit](https://www.liaoxuefeng.com/files/attachments/919020100829536/0)



## 撤销修改

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考[版本回退](https://www.liaoxuefeng.com/wiki/896043488029600/897013573512192)一节，不过前提是没有推送到远程库。

## 删除文件

一是确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`：

```c
$ git rm test.txt //
rm 'test.txt'

$ git commit -m "remove test.txt" // 现在，文件就从版本库中被删除了。
...
```

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

```c
$ git checkout -- test.txt
```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

 **注意：从来没有被添加到版本库就被删除的文件，是无法恢复的！**



# 4.远程仓库

## 添加远程仓库

在本地的仓库下运行命令关联远程仓库：

```c
$ git remote add origin https:xxx		
```

添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

```
$ git push -u origin master
```

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。



此后，从现在起，只要本地作了提交，就可以通过命令：

```c
$ git push origin master
```

把本地`master`分支的最新修改推送至GitHub或者Gitee



## 取消远程仓库

1. 删除本地文件夹下的.git 文件夹即可 

2. git remote remove origin



### 问题

Q:遇到了

```
fatal: Could not read from remote repository.
Please make sure you have the correct access rights and the repository exists.
```



A:廖老师是通过 git@github.com:michaelliao/learngit.git 关联的，我在win7和win10上测试推送时都不能成功，https的才可以。如果已经用git@关联，则可以在.git目录下的config文件中，把 url = 后面的内容改为https类型的即可。 https类型的格式为： $ git remote add origin https://github.com/daoke0818/testGit2.git



Q:如果在第一步中创建时已经初始化过项目，则这时会提醒 

```
error: failed to push some refs to 'https://github.com/daoke0818/testGit2.git' 
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing 
hint: to the same ref. You may want to first integrate the remote changes 
hint: (e.g., 'git pull ...') before pushing again. 
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```



A:因为远程库中已经存在readme文件了，所以需要先pull下来。命令如下：

```c
 $ git pull origin master
```

这时又会报错： 

```java
From https://github.com/daoke0818/testGit

branch master -> FETCH_HEAD fatal: refusing to merge unrelated histories 
```

说这两个库有不相干的历史记录而无法合并，这时我们可以加上一个参数 --allow-unrelated-histories 即可成功

```java
$ git pull origin master --allow-unrelated-histories
```

但是这时会可能会提示必须输入提交的信息，默认会打开vim编辑器，先按 i 切换到插入模式，写完后 Esc→：→wq 即可保存退出编辑器。如果不进入vim编辑器，则会自动生成一个合并代码的commit。然后再使用前面的命令push将本地提交推送到远程仓库。

后面如果本地还有commit，就可以直接用 git push origin master 推送。



## 克隆远程仓库

要克隆一个仓库，首先必须知道仓库的地址，然后使用`git clone`命令克隆。

```java
$ git clone git@github.com:michaelliao/gitskills.git
Cloning into 'gitskills'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
Receiving objects: 100% (3/3), done.
```









由于Git好像不支持一个远程仓库连接到多个本地仓库，只能是一个本地仓库连接到一个或者多个远程仓库，但是又想要将不同的文件夹push到同一个远程仓库进行管理，所以只好在一个本地仓库新建不同的分支对其进行管理了。

1.首先在github新建远程仓库：



2.在需要push的文件夹首先进行本地提交：

git init
git add .
git commit -m "your commit"
3.将本地仓库与远程仓库关联：

git remote add origin git@github.com:WeilongXia/Debug-static_in_class.git
4.将本地默认分支master推送到远程默认分支origin/master

git push -u origin master



# Git将不同的文件夹push到同一个远程仓库

由于Git好像不支持一个远程仓库连接到多个本地仓库，只能是一个本地仓库连接到一个或者多个远程仓库，但是又想要将不同的文件夹push到同一个远程仓库进行管理，所以只好在一个本地仓库新建不同的分支对其进行管理了。

1.首先在github新建远程仓库：

2.在需要push的文件夹首先进行本地提交：

git init
git add .
git commit -m "your commit"
3.将本地仓库与远程仓库关联：

git remote add origin git@github.com:WeilongXia/Debug-static_in_class.git
4.将本地默认分支master推送到远程默认分支origin/master

git push -u origin master




若要推送别的文件夹到此远程仓库，在本地仓库新建分支并切换到该分支上：

git checkout -b another_version
或者

git branch another_version
git checkout another_version
然后再将当前文件夹内容替换为需要提交的文件夹内容，进行本地提交（略）

之后再

git push origin another_version
这样本地分支another_version便被推送到了远程origin/another_version！





![image-20220530225222459](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220530225222459.png)



# git 添加所有新文件

对于项目而言, 所有增加的代码 文件,或者把项目的变化提交到仓库,

都离不开 git add -A、git add -u、git add .

```
git add -A  添加所有变化
git add -u  添加被修改(modified)和被删除(deleted)文件，不包括新文件(new)
git add .   添加新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
```
