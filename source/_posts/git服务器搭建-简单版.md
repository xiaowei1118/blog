---
title: git服务器搭建-简单版
date: 2016-04-24 20:47:13
tags: [Git]
categories: coding
---

### 前言
今天我们来讲讲Git服务器的搭建。Git是目前很火的一款版本管理工具，相对于svn集中式的版本管理，Git基于分布式的管理形式大受欢迎，而且GitHub作为Git官方的托管平台，为开源事业做出了巨大的贡献。GitHub上托管开源项目是免费的，但是作为私有仓库是收费的。如果你想拥有自己的代码仓库的话，就一起来搭建属于自己的Git服务器吧。

### 步骤详解
1. 安装Git
```
#在centos中，安装Git
yum install git
#如果是debian系列的linux，如ubuntu,安装git
sudo apt-get install
```

2. 创建git用户
我们为git服务器创建一个专门的用户（一般为git）,以下命令需要在root权限下执行
```bash
adduser git
```

3. 初始化Git仓库
```bash
#切换成git用户
su git
cd ~
mkdir src
cd src
#创建Git裸创库example.git
git init --bare example.git
#将所有example.git的拥有者改为git用户
chown -R git:git example.git
```

4. 收集公钥
在本地通过以下命令行，生成一堆`rsa密钥`对，然后将其中的`id_rsa.pub`中的内容放置到`/home/git/.ssh/authorized_keys`文件中,如果有新增的伙伴，同样将他的公钥放到`authorized_keys`中，一个用户的公钥放一行。(如果没有authorized_keys文件，请手动生成`touch authorized_keys`)。
剩下的私钥放在本地用户家目录下的.ssh文件夹下即可。
```bash
ssh-kengen -t rsa
```

5. 克隆仓库
```
git clone git@ip:~/src/example.git
Cloning into 'example'...
warning: You appear to have cloned an empty repository
```
