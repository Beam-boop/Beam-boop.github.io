---
title:  Docker,python,GitHack configuration

date: 2024-11-13 10:00:00 +0800

categories: Tools

tags: [tools,network Security,docker]

pin: true


---

## Docker,python,GitHack  配置 

Githack是一个用于利用Git漏洞的工具，通过泄露的.git文件夹下的文件，可以重建还原工程源代码

### 1. motivation

   ctfhub刷题的时候，需要用到githack工具，本以为安装很简单，但是实际上还是踩了不少坑

### 2. solution

​	github上比较流行的版本有两个，[lijiejie的Githack](https://github.com/lijiejie/GitHack)和[BugScanTeam的Githack](https://github.com/BugScanTeam/GitHack)，前者能够支持python3，后者只能支持python2，那安装前者不就好了？但是我安装完后,重建源代码后发现目录下没有`.git`，经过一些查找，网上有人说lijiejie的githack不是很好，遂放弃！但是后者需要用到python2，那一个docker既有python3，又有python2，运行的时候需要来回切换的话，那岂不是很麻烦，python3主要用dirsearch。先安装了再说

- 安装python2

  因为python2现在没办法直接`apt install python2`了，所以只能通过下载压缩包，解压来进行安装

  ```bash
  wget https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tgz
  tar xzf Python-2.7.9.tgz
  cd Python-2.7.9
  ./configure --enable-optimizations
  make altinstall
  ```

  有warning不用管！继续安装就好，后面还需要连接软链接

  ```bash
  python2.7 -V
  ln -sfn '/usr/local/bin/python2.7' '/usr/bin/python2'
  update-alternatives --install /usr/bin/python python /usr/bin/python2 1
  ```

- 安装GitHack

  ```bash
  git clone git@github.com:BugScanTeam/GitHack.git
  cd GitHack/
  ```

  使用方法就是，需要注意的是，一定要用python2，而且记得url后面加上`/.git`，才能够使用此工具

  ```bash
  python2 GitHack.py http://www.example.com/.git/
  ```

  