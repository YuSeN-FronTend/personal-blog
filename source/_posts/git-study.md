---
title: Git
date: 2022-12-9 10:12
categories: git
---

![](http://106.55.171.176:9000/yusen/Snipaste_2022-12-09_11-58-19.png)

<!-- more -->

## Git简介

- Git是一个开源的分布式版本控制系统，用于敏捷高效的处理或大或小的项目。

- Git是 Linus Torvalds 为了帮助管理 Linux内核开发的一个开放源码的版本控制软件。
- Git与常用的版本控制工具CVS，Subversion等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

## Git和SVN的区别

Git不仅仅是个版本控制系统，它也是个内容管理系统（CMS），工作管理系统等。如果你熟悉SVN，你需要做一些思想转换来适应Git的一些概念和特征。

### Git和SVN的区别点

- **Git是分布式的，SVN不是：**这是Git和其他非分布式的版本控制系统如SVN、CVS等最核心的区别。
- **Git把内容按元数据方式存储，而SVN是按文件：**所有的资源控制都是把文件的元信息隐藏在一个类似.svn、.cvs等的文件夹里。
- **Git分支和SVN的分支不同：**分支在SVN中一点都不特别，其实它就是版本库中的另外一个目录。
- **Git没有一个全局的版本号，而SVN有：**目前为止这是跟SVN相比Git缺少的最大一个特征。
- **Git的内容完整性要由于SVN：**Git的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

![](http://106.55.171.176:9000/yusen/Snipaste_2022-12-09_10-48-11.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=sttch%2F20221209%2F%2Fs3%2Faws4_request&X-Amz-Date=20221209T024847Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=441ad3d388c3c871e016cd34da165ef90167d88658e90b024203b7e5dcddb7ef)

## Git基本操作

Git的工作就是创建和保存你项目的快照及与之后的快照进行对比。

Git常用的6个命令：**git clone**、**git push**、**git add**、**git commit**、**git checkout**、**git pull**。

![](http://106.55.171.176:9000/yusen/Snipaste_2022-12-09_11-35-45.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=sttch%2F20221209%2F%2Fs3%2Faws4_request&X-Amz-Date=20221209T033559Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=d44e55ef1bfe37cd619bfb219d7e79c96cc8bb2b561c89b319a880ab853c5268)

**说明：**

- workspace：工作区
- staging area：暂存区/缓存区
- local repository：版本库或本地仓库
- remote repository：远程仓库

一个简单的操作步骤

```bash
$ git init
$ git add .
$ git commit
```

- git init        --初始化仓库
- git add .     --添加文件到暂存区
- git commit --将暂存区内容添加到仓库中

#### 创建仓库的命令

下表列出了git创建仓库的命令：

| 命令      | 说明                                   |
| --------- | -------------------------------------- |
| git init  | 初始化仓库                             |
| git clone | 拷贝一份远程仓库，也就是下载一个项目。 |

### 提交和修改

Git的工作就是创建和保存你的项目的快照及与之后的快照进行对比。

下表列出了有关创建与提交你的项目的快照的命令：

| 命令       | 说明                                   |
| ---------- | -------------------------------------- |
| git add    | 添加文件到暂存区                       |
| git status | 查看仓库当前的状态，显示有变更的文件。 |
| git diff   | 比较文件的不同，即暂存区和工作区的差异 |
| git commit | 提交暂存区到本地仓库                   |
| git reset  | 回退版本                               |
| git rm     | 将文件从暂存区和工作区中删除           |
| git mv     | 移动或重命名工作区文件                 |

#### 提交日志

| 命令             | 说明                                   |
| ---------------- | -------------------------------------- |
| git log          | 查看历史提交记录                       |
| git blame <file> | 以列表形式查看指定该文件的历史修改记录 |

#### 远程操作

| 命令       | 说明               |
| ---------- | ------------------ |
| git remote | 远程仓库操作       |
| git fetch  | 从远程获取代码库   |
| git pull   | 下载远程代码并合并 |
| git push   | 上传远程代码并合并 |















