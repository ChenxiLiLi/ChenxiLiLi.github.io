---
title: Git和Github学习笔记
date: 2020-02-20 06:26:29
tags:
---

### 第一步创建个人的仓库目录`repository`，git的所有相关命令都会在该目录下进行；第二步，在当前目录下右键`Git Bash here` 打开命令行，开始测试相关命令。

#### 1、在repository下创建版本库

```bash
$ git init
```

命令结果：生成`.git`等文件，将该目录变成Git可管理的仓库。

#### 2、添加文件进版本库

需要两步操作，第一步add，第二步commit

```bash
$ git add <文件名>
$ git commit -m "提交说明"
```

命令结果：第一步将相应的文件添加进暂存区；第二步将暂存区的文件提交到仓库。

**注意**：`git add`可以多次使用来添加多个文件，`git commit`一次性提交所有文件。

#### 3、查看仓库(工作区)的状态

```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
//现阶段working tree为空，没有可提交事务
```

命令结果：实时的工作区状态，如是否有文件被修改，工作区是否有可提交文件等。

#### 4、版本问题

```bash
$ git reset --hard commit_id  //主要命令
$ git log
$ git reflog
```

命令结果：`git reset --hard commit_id`可以使版本回退到commit_id所指定的版本；`git log`用来查看提交历史，此时前面的提交说明就尤为重要，能让你快速定位到需要回退的版本；`git reflog`用来查看命令历史，通过命令历史可以查到相应版本的commit_id，从而进行版本变更工作。

#### 5、工作区和暂存区

![git-add](D:\git\repository\MyBlog\source\images\gitadd.png)

​		***：  git add命令将文件从工作区添加到版本库的stage暂存区。**

![git-commit](D:\git\repository\MyBlog\source\images\gitcommit.png)

​		 ***：git commit命令提交更改的内容，将暂存区的所有文件提交到当前的分支上。**

#### 6、查看文件内容

```bash
$ cat HelloWorld.java
public class HelloWorld{
	public static void main(String[] args){
		System.out.println("welcome to the world!");
    }
}
```

#### 7、删除文件

```bash
//删除本地文件
$ rm <文件名> / 在文件管理器中手动删除文件
//删除版本库中的文件
$ git rm <文件名>		
$ git commit -m "提交说明"
```

*：如果误删了文件，可以使用`git checkout -- <文件名>`，用版本库里的版本文件替换工作区的，达到恢复的效果。

#### 8、将本地库推送到远程

```bash
//将本地的master分支推送到远程origin主机的master分支
$ git push -u origin master 
//不带任何参数，默认只推送到当前分支
$ git push	
```

#### 9、克隆远程库

```bash
$ git clone <远程库地址> <本地库目录>
eg:
$ git clone https://github.com/ChenxiLiLi/ChenxiLiLi.github.io.git     Clone
$ git clone git@github.com:ChenxiLiLi/ChenxiLiLi.github.io.git Clone
```

*：`GitHub`提供了`ssh`和`https`两种地址，推荐使用`ssh`支持的原生git协议，下载速度更快。



https://chenxilili.github.io

