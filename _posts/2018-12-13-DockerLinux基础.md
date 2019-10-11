---
layout:     post

title:      Docker--Linux--基础

date:       2018-12-13

author:     Augustine Tong

header-img: img/steve.jpg

catalog: true

tags:
    - Docker
    - Linux
---

# Docker--Linux--基础
**代码均为自己手打过一次, 输出结果均为自己的输出结果**.  

## 获取镜像

```bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

- Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。  
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

docker pull ubuntu:18.04

docker run -it --rm ubuntu:18.04 bash


-it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。  
--rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。  
ubuntu:18.04：这是指用 ubuntu:18.04 镜像为基础来启动容器。  
bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。  



docker image ls

docker image rm [选项] <镜像1> [<镜像2> ...]

```



## 操作容器

```bash
docker run ubuntu:14.04 /bin/echo 'Hello world'

这跟在本地直接执行 /bin/echo 'hello world' 几乎感觉不出任何区别。  

下面的命令则启动一个 bash 终端，允许用户进行交互。  


docker run -t -i ubuntu:14.04 /bin/bash

其中，-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上，-i 则让容器的标准输入保持打开。  
```

## Linux 操作
以ls指令列出"~"目录下的所有隐藏档与相关的文件属性，要达到这一要求需要加入 -al 这样的选项

```bash
ls -al ~

ls       -al    ~

ls -a -l ~

// 上面这三个指令的下达方式是一模一样的执行结果  

date // 显示日期和时间
Date // 找不到命令 
DATE // 找不到命令 

date
Tue Dec 11 11:46:20 UTC 2018

date +%Y/%m/%d
2018/12/11

date +%H:%M
11:46

注意，仅CentOS中
显示日历

cal


    December 2018   
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31


cal 2018

       October               November               December      
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6                1  2  3                      1
 7  8  9 10 11 12 13    4  5  6  7  8  9 10    2  3  4  5  6  7  8
14 15 16 17 18 19 20   11 12 13 14 15 16 17    9 10 11 12 13 14 15
21 22 23 24 25 26 27   18 19 20 21 22 23 24   16 17 18 19 20 21 22
28 29 30 31            25 26 27 28 29 30      23 24 25 26 27 28 29
                                              30 31



cal [month] [year]

cal 2 2019

    February 2019   
Su Mo Tu We Th Fr Sa
                1  2
 3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28

```

简单好用的计算器bc

bc

坑：Docker中的Centos镜像最开始并没有bc，需要自己安装

```bash
yum install man
yum install bc

bc计算器里面，运算符

+
-
*
/
^ 指数
% 余数

预设10 / 100 = 0
可以先执行
scale = 3
这样结果均为三位小数
```


### Linux 重要热键


```bash
ca [tab][tab]

发现会显示所有以ca开头的指令  
两次tab如果在command后面，那就代表着【命令补齐】  

ls -al ~/.bash  [tab][tab]  
在该目录下面所有以 .bash 为开头的文件名都会被显示出来  
两次tab如果在文件名后面，那就是【文件补齐】  

坑：Docker中的Centos镜像最开始并没有bash-completion ，需要自己安装  

yum install bash-completion 

两次tab也可能是【参数补齐】


[Ctrl]-c  
中断目前程序  

[Ctrl]-d  
键盘输入结束EOF  
可取代exit  

```



## Docker 容器操作  

```bash
例如刚才已经yum install 了，如果不保存结果，那么下次会要同样的流程再来一遍 install    

最好的解决方法是保存    

docker ps -l  
获取最后一次更改的容器id  

docker ps -a  
获取所有的容器id  

无需拷贝完整的id，通常来讲最开始的三至四个字母即可区分  
但是全拷贝更稳一点  
docker commit 3625ff5e4825 augu/centos  

但我之前已有过一个augu/centos
为了更好地区分，最好还是去掉原来的那一个
怎么删除容器呢
docker container ls -a
查看所有已创建的容器

清理所有处于终止状态的容器  
docker container prune

删除单独一个容器  
docker container rm 容器名/id  

列出本地image镜像  
docker image ls  

删除镜像  
docker image rm 容器名/id  

```

## Linux


```bash
date --help
可以把使用的指令的用法做一个大致的了解

回到Mac的$界面后
可以输入
man date

进入manual后
输入/date
即可搜索所有的date关键字

cd [相对路径/绝对路径]
变换目录

cd ~augu
去到augu这个用户的家目录，即/home/augu

cd
cd ~
回到自己的家目录，即/root 这个目录

cd ..
回到上层的目录

cd /var/spool/mail

cd ../postfix

pwd
显示当前目录

cd /var/mail

pwd
/var/mail

pwd -P
/var/spool/mail

加上 pwd -P 的选项后，会不以连结文件的数据显示，而是显示正确的完整路径

mkdir 
mkdir [-mp] 目录名称
-m :配置文件案的权限，直接设定，不需要看预设权限
-p :帮助你直接将所需要的目录(包含上层目录)递归建立起来

cd /tmp
mkdir test
mkdir test1/test2/test3/test4
mkdir: cannot create directory ‘test1/test2/test3/test4’: No such file or directory

mkdir -p test1/test2/test3/test4
加了这个 -p 的选项，可以自行帮你建立多层目录


mkdir -m 711 test2
如果没有加上 -m 来强制设定属性，系统会使用默认属性
我们给予 -m 711 来给予新的目录 drwx--x--x 的权限
ls -ld test*
来查看mkdir的权限


rmdir [-p] 目录名称
删除一个空目录

ls -ld test*
查看有多少目录存在

rmdir test2
可以，test2为空

rmdir test1
不行，里面非空

rmdir -p test1/test2/test3/test4

ls -ld test*
```


## 执行文件路径的变量

```bash
su -
转到管理员权限

echo $PATH
输出PATH

PATH="${PATH}:/root"
增加一条PATH

echo $PATH
输出PATH

每个目录中间用冒号(:)来隔开， 每个目 录是有『顺序』之分的


ls -al ~
将家目录下的所有文件列出来(含属性与隐藏文件)

ls -alF --color=never ~
不显示颜色，但在文件名末显示出该文件名代表的类型(type)

ls -al --full-time ~
完整的呈现文件的修改时间
```

### Copy

```bash

cp [-adfilprsu] source destination
cp [options] source1 source2 source3 source4 ... directory

-a : 相当于 -dr --preserve=all 的意思(常用)
-d : 若来源文件为链接文件(link file), 则复制链接文件属性而非文件本身
-i : 若目标文件已经存在，在覆盖时会先询问动作的进行(常用)
-p : 连同文件的属性(权限，用户，时间)一起复制过去，而非使用默认属性(备份常用)
-r : 递归持续复制，用于目录的复制行为(常用)

1 用 root 身份，将家目录下的 .bashrc 复制到 /tmp 下，并更名为 bashrc
cp ~/.bashrc /tmp/bashrc
cp -i ~/.bashrc /tmp/bashrc
重复两次动作，由于/tmp底下已经存在bashrc了，再加上 -i 选项后，
则在覆盖前会询问使用者是否确认，可按下n 或者 y来二次确定

2 变换目录到/tmp，并将/var/log/wtmp 复制到/tmp 且观察属性:
cd /tmp
cp /var/log/wtmp .  // 想要复制到当前目录，最后的 . 不要忘
ls -l /var/log/wtmp wtmp
Output:
-rw-rw-r-- 1 root utmp 0 Dec  5 01:36 /var/log/wtmp
-rw-r--r-- 1 root root 0 Dec 12 02:36 wtmp

// 注意上面的不同之处，在不加任何选项的情况下，文件的某些属性/权限会改变
// 这是个很重要的特性! 还有，连文件建立的时间也不一样
// 如果你想要将文件的所有特性都一起复制过来，可以加上 -a

cp -a /var/log/wtmp wtmp_2
ls -l /var/log/wtmp wtmp_2

-rw-rw-r-- 1 root utmp 0 Dec  5 01:36 /var/log/wtmp
-rw-rw-r-- 1 root utmp 0 Dec  5 01:36 wtmp_2

两个资料一模一样

3 复制 /etc/ 这个目录下所有内容到 /tmp 底下

cp /etc/ /tmp
报错，目录不能直接复制

cp -r /etc/ /tmp

// -r 可以复制目录，但是文件和目录的权限可能会改变，
// 所以也可以利用 cp -a /etc/ /tmp 来下达指令，尤其在备份的情况下

4 将范例1复制的 bashrc 建立一个连接档
ls -l bashrc

-rw-r--r-- 1 root root 176 Dec 12 02:31 bashrc

cp -s bashrc bashrc_slink
cp -l bashrc bashrc_hlink
ls -l bashrc*

-rw-r--r-- 2 root root 176 Dec 12 02:31 bashrc
-rw-r--r-- 2 root root 176 Dec 12 02:31 bashrc_hlink
lrwxrwxrwx 1 root root   6 Dec 12 02:57 bashrc_slink -> bashrc

-l 和 -s 都会建立连接档 link file, 
-l 建立实体链接 hard link ， 
-s symbolic link ， bashrc_slink 是一个快捷方式，这个快捷方式会连接到 bashrc 上去，所以你看到档名右侧有个指向->的符号

bashrc_hlink 文件与 bashrc 的属性与权限完全一模一样
第二栏的link数从1变为2

5 若 ~/.bashrc 比 /tmp/bashrc 新才复制过来

cp -u ~/.bashrc /tmp/bashrc

-u 的特性，是在目标文件与来源文件有差异时，才会复制的
所以常用于 备份 的工作当中

6 将范例四造成的 bashrc_slink 复制成为 bashrc_slink_1 与 bashrc_slink_2

cp bashrc_slink bashrc_slink_1
cp -d bashrc_slink bashrc_slink_2
ls -l bashrc bashrc_slink*

-rw-r--r-- 2 root root 176 Dec 12 02:31 bashrc
lrwxrwxrwx 1 root root   6 Dec 12 02:57 bashrc_slink -> bashrc
-rw-r--r-- 1 root root 176 Dec 12 03:12 bashrc_slink_1
lrwxrwxrwx 1 root root   6 Dec 12 03:13 bashrc_slink_2 -> bashrc

原本复制的是连结档，但是却将连结档的实际文件复制过来了
如果没有加上任何选项时，cp 复制的是源文件，而非链接文件的属性
若要复制链接文件的属性，就得要使用 -d 的选项了!如 bashrc_slink_2 所示


7 将家目录的 .bashrc 及 .bash_history 复制到 /tmp 底下

cp ~/.bashrc ~/.bash_history /tmp

可以将多个数据一次复制到同一个目录去!最后面一定是目录


Example:
能否使用 dmtsai 的身份，完整的复制/var/log/wtmp 文件到/tmp 底下，并更名为 dmtsai_wtmp ?

cp -a /var/log/wtmp /tmp/dmtsai_wtmp
ls -l /var/log/wtmp /tmp/dmtsai_wtmp





```



