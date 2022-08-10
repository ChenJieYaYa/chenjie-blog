# Git

## 一、什么是Git？

Git是**分步式版本控制系统**，集中式版本控制和分步式版本控制的区别是什么？？

* 集中式版本控制存在**中央服务器**，**中央服务器用于保存修改过的所有版本**，而程序员的开始是在自己的电脑上，所以干活前先从中央服务器获取最新版本，干完活后再推送版本到中央服务器，综上**程序员的电脑上只会保存最新的版本**，一旦中央服务器故障，则整个版本库丢失，存在单点故障的问题

![1660032776682](assets/1660032776682.png)

* 分步式版本控制不存在中央服务器，**程序员的电脑上就是一个版本库**，但这怎么实现版本控制呢？利用**远程库**，程序员A在自己的电脑上修改A文件，同时程序员B也修改A文件，那么只需程序员AB将修改的文件推送到远程库，那么就可以看到对方的修改，若远程库故障，则程序员只是无法将更新推送到远程库，但本地还是存在版本控制的，所以远程库恢复之后同样可以推送，无需担心单点故障

![1660033109331](assets/1660033109331.png)

## 二、Git代码托管中心

### 1.Git分区

![1660033841176](assets/1660033841176.png)

### 2.远程库

远程库就是代码托管中心，可将本地库`push`到远程库，远程库存在以下几种

| 局域网 |          互联网           |
| :----: | :-----------------------: |
| GitLab | GitHub(外网)、Gitee(国内) |

## 三、Git安装

[Git下载地址](https://git-scm.com/download)，基本上无脑`Next`即可

![1660034825367](assets/1660034825367.png)

![1660034870940](assets/1660034870940.png)

![1660034938868](assets/1660034938868.png)

![1660035042773](assets/1660035042773.png)

![1660035091368](assets/1660035091368.png)

![1660035176853](assets/1660035176853.png)

![1660035195098](assets/1660035195098.png)

![1660035209759](assets/1660035209759.png)

![1660035262998](assets/1660035262998.png)

![1660035276863](assets/1660035276863.png)

![1660035297573](assets/1660035297573.png)

![1660035320585](assets/1660035320585.png)

![1660035341029](assets/1660035341029.png)

![1660035354858](assets/1660035354858.png)

![1660035373362](assets/1660035373362.png)

下载成功后随便对文件夹右键，如图即为安装成功

![1660023450677](../%E5%85%B6%E4%BB%96/assets/1660023450677.png)

点击`Git Bash Here`，查看Git版本

![1660035480241](assets/1660035480241.png)

## 四、Git常用命令

### 1.设置用户签名

用户签名用于区分不同的操作者身份，Git首次安装需要设置一次用户签名，否则无法提交代码，此处设置的签名和将来的远程库没有任何关系

* 设置用户名

```
git config --global user.name "用户名"
```

* 设置用户邮箱，可设置成虚拟邮箱

```
git config --global user.email "邮箱"
```

* 验证是否设置成功

![1660035784868](assets/1660035784868.png)

### 2.初始化本地库 

初始化本地库后Git就拥有该本地库的权限，初始化命令`git init `

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First
$ git init
Initialized empty Git repository in F:/系统默认/桌面/First/.git/

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
```

初始化后生成`.git`隐藏文件

![1660036353956](assets/1660036353956.png)

### 3.查看本地库状态 

查看命令`git status`，工作区没有任何文件时查看结果如下

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

新建文件再次查看结果如下

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ vim hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)
```

### 4.添加到暂存区

添加命令`git add 文件名`，可通过通配符`.`添加所有

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git add .

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   hello.txt
```

###  5.提交到本地库

提交命令`git commit -m "日志信息" 文件名`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git commit -m "hello"
[master (root-commit) 3f5a1b5] hello
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master
nothing to commit, working tree clean
```

 修改`hello.txt`文件后再次查看

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ vim hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

 再次提交`hello.txt`文件到本地库

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git add .

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git commit -m "222"
[master 9d6c8ed] 222
 1 file changed, 1 insertion(+), 1 deletion(-)

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git status
On branch master
nothing to commit, working tree clean
```

### 6.查看历史记录

查看版本信息命令`git reflog`，查看版本详细信息命令`git log`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git reflog
9d6c8ed (HEAD -> master) HEAD@{0}: commit: 222
3f5a1b5 HEAD@{1}: commit (initial): hello

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git log
commit 9d6c8ed5631d732dd6b1ad046424b6e47f6b87c7 (HEAD -> master)
Author: ChenJieYaYa <1243833281@qq.com>
Date:   Tue Aug 9 17:22:22 2022 +0800

    222

commit 3f5a1b59879483dec0393a4d69d15278c41aa523
Author: ChenJieYaYa <1243833281@qq.com>
Date:   Tue Aug 9 17:18:15 2022 +0800

    hello
```

### 7.版本穿梭

穿梭命令`git reset --hard 版本号 `，其原理其实就是移动`HEAD`指针，版本并不会被删除

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git reflog
9d6c8ed (HEAD -> master) HEAD@{0}: commit: 222
3f5a1b5 HEAD@{1}: commit (initial): hello

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git reset --hard 3f5a1b5
HEAD is now at 3f5a1b5 hello

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git reflog
3f5a1b5 (HEAD -> master) HEAD@{0}: reset: moving to 3f5a1b5
9d6c8ed HEAD@{1}: commit: 222
3f5a1b5 (HEAD -> master) HEAD@{2}: commit (initial): hello

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ cat hello.txt
hello git!
```

## 五、Git分支

### 1.什么是分支？

版本控制过程中，同时推进多个任务，可为每个任务建立单独的分支，使用分支意味着**程序员可以把自己的工作从开发主线上分离开来**，开发自己分支时不会影响主线分支的运行，分支底层其实也是指针的引用

分支可以**同时并行推进多个功能开发，提高开发效率**，若某个分支开发失败，则不会对其他分支有任何影响，失败的分支删除重新开始即可

### 2.分支命令

查看分支命令`git branch -v`，`*`代表当前所在分支

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git branch -v
* master 3f5a1b5 hello
```

创建分支命令`git branch 分支名`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git branch hhh

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git branch -v
  hhh    3f5a1b5 hello
* master 3f5a1b5 hello
```

修改`master`分支，发现`hhh`分支并未改变

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ vim hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git add hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$  git commit -m "master branch" hello.txt
[master 2a7bf62] master branch
 1 file changed, 4 insertions(+), 1 deletion(-)

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git branch -v
  hhh    3f5a1b5 hello
* master 2a7bf62 master branch

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ cat hello.txt
hello git!


hhhhhhhh
```

切换分支命令`git checkout 分支名`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$  git checkout hhh
Switched to branch 'hhh'

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (hhh)
$ cat hello.txt
hello git!
```

合并分支命令`git merge 分支名`，实现在`master`分支上合并`hhh`分支

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git checkout hhh
Switched to branch 'hhh'

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (hhh)
$ vim hello.txt

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (hhh)
$ git add .

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (hhh)
$ git commit -m "s"
[hhh a7ae8f9] s
 1 file changed, 2 insertions(+)

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (hhh)
$ git checkout master
Switched to branch 'master'

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ cat hello.txt
hello git!


hhhhhhhh

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git merge hhh
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed; fix conflicts and then commit the result.

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master|MERGING)
$ cat hello.txt
<<<<<<< HEAD
hello git!


hhhhhhhh
=======
hello git!

hhh update
>>>>>>> hhh
```

以上代码内可以看到产生了冲突`(master|MERGING)`，这种冲突是因为合并分支时，在**同一个文件的同一个位置**有两套完全不同的修改，Git无法决定使用哪个，必须**人为决定**新代码内容

* 修改`hello.txt`文件内容，`<<<<<<< HEAD` 当前分支内容 `=======` 合并分支内容 `>>>>>>> hhh`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master|MERGING)
$ vim hello.txt

hello git!

hhh update
hhhhhhhh
```

* 添加到本地库后`|MERGING`消失，注意提交到本地库时不能指定文件名

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master|MERGING)
$ git add .

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master|MERGING)
$ git commit -m "m"
[master aeb939d] m

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
```

## 六、团队协作

### 1.团队内协作

![1660040368916](assets/1660040368916.png)

### 2.跨团队协作

![1660040643680](assets/1660040643680.png)

## 七、GitHub

### 1.什么是GitHub？

[GitHub网址](https://github.com/)，相当于远程库，歪果仁写的，GitHub创建远程仓库的步骤如下

![1660051697900](assets/1660051697900.png)

![1660052343811](assets/1660052343811.png)

![1660052921467](assets/1660052921467.png)

### 2.GitHub命令

查看当前所有远程库别名：`git remote -v`

创建远程库别名：`git remote add 别名 远程地址`，不写别名默认`origin`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git remote add HelloGit https://github.com/ChenJieYaYa/HelloGit.git

CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git remote -v
HelloGit        https://github.com/ChenJieYaYa/HelloGit.git (fetch)
HelloGit        https://github.com/ChenJieYaYa/HelloGit.git (push)
```

推送本地分支到远程仓库：`git push 别名 分支名`

```
CJ@DESKTOP-JB5EHNO MINGW64 /f/系统默认/桌面/First (master)
$ git push HelloGit master
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (12/12), 875 bytes | 97.00 KiB/s, done.
Total 12 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/ChenJieYaYa/HelloGit.git
 * [new branch]      master -> master
```

![1660053871267](assets/1660053871267.png)

将远程仓库的内容克隆到本地：`git clone 远程地址`

将远程仓库对应分支最新内容拉下来后与当前本地分支直接合并：`git pull 远程库地址别名 远程分支名` 

****

接下来使用三个账号模拟团队协作的过程

|        账号        |   姓名    |            邮箱            |
| :----------------: | :-------: | :------------------------: |
|   atguiguyueyue    |  岳不群①  |  atguiguyueyue@aliyun.com  |
| atguigulinghuchong |  令狐冲②  | atguigulinghuchong@163.com |
|  atguigudongfang1  | 东方不败③ |  atguigudongfang@163.com   |

### 3.团队内协作

①岳不群`push`本地文件`git-shTest`到远程仓库

* 设置别名：`git remote add ori https://github.com/atguiguyueyue/git-shTest.git`

* `push`到远程仓库：`git push ori master`

![1660098737246](assets/1660098737246.png)

②令狐冲`clone`到本地：`git clone https://github.com/atguiguyueyue/git-shTest.git`

③岳不群邀请令狐冲加入团队

![1660098057409](assets/1660098057409.png)

![1660098081290](assets/1660098081290.png)

![1660098134249](assets/1660098134249.png)

④令狐冲在地址栏复制连接地址，并接受岳不群的邀请
![1660098254086](assets/1660098254086.png)

![1660098374248](assets/1660098374248.png)

⑤令狐冲修改内容并`push`到远程仓库

* 编辑`clone`下来的文件

```
$ vim hello.txt

$ cat hello.txt
hello git! hello atguigu! 2222222222222
hello git! hello atguigu! 33333333333333
hello git! hello atguigu!
hello git! hello atguigu!
hello git! hello atguigu! 我是最帅的，比岳不群还帅
hello git! hello atguigu!
hello git! hello atguigu!
```

* 将编辑好的文件添加到暂存区：`git add hello.txt`
* 将暂存区的文件上传到本地库：`git commit -m "lhc commit" hello.txt`
* 将本地库的内容`push`到远程仓库：`git push origin master`
* 回到岳不群的GitHub远程仓库中可以看

![1660099017527](assets/1660099017527.png)

![1660099149463](assets/1660099149463.png)

⑥岳不群将远程仓库对应分支最新内容拉下来后与当前本地分支直接合并：`git pull ori master`

### 4.跨团队协作

①将远程仓库的地址复制发给邀请跨团队协作的人，比如东方不败

![1660099262788](assets/1660099262788.png)

②东方不败复制收到的链接，然后点击`Fork`将项目叉到自己的本地仓库

![1660099324463](assets/1660099324463.png)

![1660099350879](assets/1660099350879.png)

③东方不败修改`hello.txt`内容，并`commit`

![1660099393661](assets/1660099393661.png)

![1660099406033](assets/1660099406033.png)

![1660099430096](assets/1660099430096.png)

④创建新的`pull`请求

![1660099597592](assets/1660099597592.png)

![1660099609911](assets/1660099609911.png)

![1660099633439](assets/1660099633439.png)

⑤岳不群账号内可看到一个`pull`请求

![1660099693622](assets/1660099693622.png)

![1660099702077](assets/1660099702077.png)

⑥进入聊天室可以讨论代码相关内容

![1660099754157](assets/1660099754157.png)

![1660099763808](assets/1660099763808.png)

⑦检查代码没问题可合并

![1660099801616](assets/1660099801616.png)

![1660099813402](assets/1660099813402.png)

### 5.SSH免密登录

①进入当前用户的家目录：`cd`

②删除`.ssh`目录：`rm -rvf .ssh`

③运行命令生成`.ssh`秘钥目录，输入命令后回车就好：`ssh-keygen -t rsa -C atguiguyueyue@aliyun.com`

④进入`.ssh`目录查看文件列表：`cd .ssh `、`ll -a`

⑤查看`id_rsa.pub`文件内容，复制秘钥：` cat id_rsa.pub`

⑥GitHub设置SSH远程登录

![1660100144039](assets/1660100144039.png)

![1660100159527](assets/1660100159527.png)

![1660100181351](assets/1660100181351.png)

## 八、IDEA集成Git

### 1.忽略某些文件

在IDEA中，存在`.idea`和`target`等文件是与实际功能无关的，所以此时需要忽略，以下是忽略步骤

* 新建`git.ignore`文件

![1660053947299](assets/1660053947299.png)

* `git.ignore`文件内容如下

```
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*

.classpath
.project
.settings
target
.idea
*.iml
```

* 配置`.gitconfig`文件

```
[user]
	email = 1243833281@qq.com
	name = ChenJieYaYa
[core]
	autocrlf = false
	excludesfile = C:/Users/CJ/git.ignore
```

### 2.定位Git程序

![1660054556366](assets/1660054556366.png)

![1660054676529](assets/1660054676529.png)

### 3.初始化本地库

![1660054865425](assets/1660054865425.png)

![1660055033676](assets/1660055033676.png)

### 4.添加到暂存区

![1660055139149](assets/1660055139149.png)

### 5.提交到本地库

![1660055216051](assets/1660055216051.png)

![1660055291830](assets/1660055291830.png)

### 6.切换版本

①在IDEA左下角，点击Version Control(现在变成Git)，然后点Log查看版本

![1660100885783](assets/1660100885783.png)

②右键选择要切换的版本，然后在菜单里点击Checkout Revision

![1660100924666](assets/1660100924666.png)

### 7.创建分支

![1660100962896](assets/1660100962896.png)

![1660100971360](assets/1660100971360.png)

![1660100981415](assets/1660100981415.png)

![1660100994297](assets/1660100994297.png)

### 8.切换分支

![1660101026798](assets/1660101026798.png)

![1660101039149](assets/1660101039149.png)

### 9.合并分支

如果代码没有冲突，分支直接合并成功，分支合并成功以后，代码自动提交，无需手动提交本地库

![1660101066989](assets/1660101066989.png)

### 10.解决冲突

![1660101128492](assets/1660101128492.png)

![1660101138206](assets/1660101138206.png)

![1660101152232](assets/1660101152232.png)

![1660101176167](assets/1660101176167.png)

![1660101187219](assets/1660101187219.png)

![1660101259635](assets/1660101259635.png)

## 九、IDEA集成GitHub

### 1.设置GitHub账号

![1660101397991](assets/1660101397991.png)

如果出现401等连接不上的问题是因为网络，可以使用以下方式连接

![1660101444238](assets/1660101444238.png)

GitHub上设置

![1660101476362](assets/1660101476362.png)

![1660101505976](assets/1660101505976.png)

![1660101517043](assets/1660101517043.png)

### 2.分享工程到GitHub

![1660101576541](assets/1660101576541.png)

![1660101585771](assets/1660101585771.png)

### 3.推送本地库到远程库

`push`是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致`push`操作会被拒绝，也就是说，要想`push`成功，一定要保证本地库的版本要比远程库的版本高！因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！**如果本地的代码版本已经落后，切记要先`pull`拉取一下远程库的代码**，将本地代码更新到最新以后，然后再修改，提交，推送！ 

![1660101648167](assets/1660101648167.png)

![1660101672696](assets/1660101672696.png)

![1660101700492](assets/1660101700492.png)

### 4.拉取远程库到本地库

`pull`是拉取远端仓库代码到本地，如果远程库代码和本地库代码不一致，会自动合并，如果自动合并失败，还会涉及到手动解决冲突的问题

![1660101818707](assets/1660101818707.png)

![1660101844585](assets/1660101844585.png)

### 5.克隆远程库到本地

![1660101903789](assets/1660101903789.png)

![1660101917519](assets/1660101917519.png)

![1660101931930](assets/1660101931930.png)

## 十、IDEA集成Gitee码云

### 1.什么是Gitee码云？

GitHub服务器在国外，使用GitHub作为项目托管网站，如果网速不好的话，严重影响使用体验，甚至会出现登录不上的情况，针对这个情况，可以使用国内的项目托管网站-[码云](https://gitee.com)，操作和GitHub大同小异，不讲解，接下来进入IDEA集成Gitee码云

### 2.IDEA安装码云插件

默认IDEA是不安装码云插件的，需手动安装，进入插件库后搜索Gitee手动安装，安装后重启IDEA

![1660102496221](assets/1660102496221.png)

### 3.设置Gitee账号

![1660102611982](assets/1660102611982.png)

### 4.IDEA连接码云

Idea连接码云和连接GitHub几乎一样，首先在Idea里面创建一个工程，初始化Git工程，然后将代码添加到暂存区，提交到本地库，这些步骤上面已经讲过，此处不再赘述

➢ **将本地代码** **push** **到码云远程库** 

![1660102771386](assets/1660102771386.png)

![1660102809150](assets/1660102809150.png)

码云服务器在国内，用HTTPS链接即可，没必要用SSH免密链接

![1660102843758](assets/1660102843758.png)

 ![1660102872036](assets/1660102872036.png)

![1660102932752](assets/1660102932752.png)

只要码云远程库链接定义好以后，对码云远程库进行pull和clone的操作和Github一致，此处不再赘述

### 5.码云复制GitHub项目

![1660103011660](assets/1660103011660.png)

![1660103023678](assets/1660103023678.png)

![1660103049743](assets/1660103049743-1660103054448.png)

![1660103154392](assets/1660103154392.png)

![1660103169024](assets/1660103169024.png)

![1660103213266](assets/1660103213266.png)

## 十一、GitLab

[尚硅谷视频链接地址](https://www.bilibili.com/video/BV1vy4y1s7k6?p=27&spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=8811945bf338927db8b1ca45a8f75a87)

































































