---
title: 分布式版本控制系统github
date: 2019-11-05 14:50:25
tags: ['分布式','git']
---

***简介：*** GitHub是一个面向开源及私有软件项目的托管平台 ，简单理解可以认为是，世界上没有卖后悔药的但是有了github就可以吃后悔药了。
<!--more-->

## 1.github的注册和使用

##### （1)初始化本地仓库git

1.去git官网下载安装git到本机 [下载链接](https://git-for-windows.github.io/) 
2.安装成功后即可以在桌面上右键看到 **Git  Bash Here**  的目录选项
3.配置文件为 `~/.gitconfig` ，执行任何Git配置命令后文件将自动创建。
第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录：

```java
git config --global user.name "danglea" //你的名称随意
git config --global user.emaiil "2415414232@qq.com" //你的邮箱
```

用于标识自己

##### （2）将本地git和GitHub关联起来

1.创建GitHub账号  [创建地址](https://github.com/) 

2.生成本机的密钥Ssh Key 一直回车

```java
ssh -keygen -t rsa  
```

在你操作的目录下就可以找到一下 **.ssh**的文件夹

![密钥](http://pztpuk0kp.bkt.clouddn.com/git1.png)

打开**id_rsa**文档复制里面内容粘贴到

github中----点击setting---选择SSH and GPG Keys----New SSH Key

##### （5）将自己的仓库和github上的仓库连接起来

1.在GitHub上创建仓库，粘贴SSH Key

```Java
git init //初始化一个git仓库
git add .//将所有内容从工作区添加到缓存区
git commit  -m "介绍内容"//提交到本地仓库
git remote add origin 密钥 //添加远程仓库连接
git push -u origin master //将数据推送到远程仓库 
```

## 2.git的常用命令

```java
git log --pretty=oneline //查看日志
git reflog //查看详细日志
git clone 地址 //克隆项目从GitHub远程仓库    


git init //交给git管理
git add 内容 //添加内容到git从工作区到缓存区
git status //查看当前状态    
git commit -m"介绍内容"//将缓存区的内容提交到本地库
git branch //查看分支
git branch ask //生成一个叫sak的分支
git checkout ask //切换到ask分支
git checkout -b ask//生成一个ask分支并切换到ask分支
git merge ask //合并ask分支    
git branch -d ask //删除本地已经合并的分支 -D强制删除
git branch --merged  //查看已经合并的分支
git branch --no --merged //查看没有合并的分支

ssh -Keygen  -t rsa//生成密钥
git remote add origin 地址//添加远程仓库连接
git remote -v //查看已经关联的远程仓库
git push -u origin master //将数据推送到远程仓库
git pull origin ask:ask //将远程的ask分支剪切到我的本地ask分支   
git push origin --delete ask//删除远程仓库的ask分支       
```

