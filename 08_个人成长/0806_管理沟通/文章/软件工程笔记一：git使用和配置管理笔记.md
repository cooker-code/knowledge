---
title: 软件工程笔记一：git使用和配置管理笔记
author: 聿文笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwOTU4NzQyOQ==&mid=2247485156&idx=1&sn=f1da81901c34a7f070ddf325390de968&chksm=c039cb1fb0006ac78e0454851b0777a34ead6e4f089db93ba171ce9a8a8b5e077218efdc207e&mpshare=1&scene=24&srcid=01023oBrEJ8JGxC8Iydpk6PP&sharer_shareinfo=ff9ab9fdde61257efa3c72aa4809acee&sharer_shareinfo_first=ff9ab9fdde61257efa3c72aa4809acee#rd
---

Git是世界上最先进的分布式版本控制系统之一，使用git能较好的进行团队协作并能看见每次修改的版本。本文总结了git常用的一些基础指令，包括如何绑定自己的信息、创建一个仓库、在github上进行协作等基础知识。

参考教程为：https://www.liaoxuefeng.com/wiki/896043488029600

1.安装git并检查版本：

直接从Git官网直接下载安装程序，然后按默认选项安装即可。安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功。

在windows安装：

```
C:\> scoop install git
```

在mac系统上安装：

```
brew install git
```

输入命令检查git是否安装成功并查看版本：

```
git -v
```

配置用户名和邮箱号：

```
git config --global user.name "Your Name"git config --global user.email "email@example.com"
```

2.进行本地仓库创建和配置：

选择一个合适的文件夹位置打开cmd（命令控制系统），然后输入：

```
mkdir learngitcd learngit
```

通过init命令将目录变成可管理的仓库：

```
git init
```

创建文件并进行上传（此处以简历readme.text文件为例）：

```
git add readme.txt
```

```
git commit -m "wrote a readme file"
```

其中使用commit可以一次性提交很多个add的文件

```
git add file1.txtgit add file2.txt file3.txtgit commit -m "add 3 files."
```

3.在本地创建一个C语言或者C++项目，将所有的项目源程序文件提交到本地库中。

使用git进行进行c文件创建：

```
git add main.c
```

创建c文件并插入内容：

用vs code在文中进行修改：

4.多次修改某一个源代码文件，进行多次提交。

查看当前仓库的状态：

```
git status
```

查看修改内容以及提交修改后版本：

```
git commit -m
```

进行多次修改和提交：

5.查看文件的历史记录，并且查看不同版本之间的差别

```
git log
```

6.将文件恢复到前面的某个版本，并查看详细的恢复过程

进行最新一次代码修改的提交：

```
git revert
```

7.删除某一个提交文件，并且查看项目状态。

查看状态

```
git status
```

查看项目记录：

```
git log --online
```

8.进入Github，尝试clone一个开源项目

选择一个自己需要的开源项目，复制网页链接：

在本地查看完成克隆的文件：

9.在本地创建一个C语言或者C++项目，将所有的项目源程序文件提交到本地库中：

1）添加所有文件：

```
git add .
```

2）只添加某个文件：

```
git add main.py
```

3）添加某个文件夹：

```
git add src/
```

4）提交到本地库：

```
git commit -m "提交说明"
```

10.为某个文件创建分支，在分支中提交和对比文件

1）创建分支：

```
git checkout -b feature-fileA
```

2）在分支中提交和对比文件

进行提交：

```
git add config.yamlgit commit -m "修改 config.yaml 的配置逻辑"
```

比较差异：

```
git diff main -- config.yaml
```

```
git diff branchA branchB -- config.yaml
```

11.将修改的内容合并到主分支

```
# 1. 切换到主分支git checkout main# 或老项目git checkout master  
# 2. 合并（10）所在分支git merge feature-fileA
```

12.与项目组同学合作，各自将本地代码提到同一个远程版本库

1）登录到本地的一个Github案例(这个是之前的所以有一些初始数据)，查看当前内容与URL链接：

2）通过git remote add origin URL链接将该Github项目与本地仓库进行绑定：

```
git remote add origin https://github.com/username/project-name.git
```

检查是否绑定成功：

```
git remote -v
```

此时可以看见在github上上传的内容