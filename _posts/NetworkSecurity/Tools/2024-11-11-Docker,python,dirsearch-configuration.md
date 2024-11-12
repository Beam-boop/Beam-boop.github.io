---

title: Docker,python,dirseach configuration

date: 2024-11-11 10:00:00 +0800

categories: Tools

tags: [tools, docker, python, network security]

pin: true

---

## Docker, python, dirseach configuration

Dirseach 是一款web路径扫描工具，[git地址](https://github.com/maurosoria/dirsearch)，必须要使用python3.9以上版本才能运行

### 1. motivation

   刷题刚好遇到，就来安装玩玩。至于为啥用docker呢，因为不想占用本地空间，最好就是在服务器上创建docker啦。

### 2. solution 

   - 直接拉取docker image，run就好了，but现在docker hub被封了，docker pull一直会出现网络错误，那就自己动手安装吧

     - 创建基于ubuntu的docker

     ```bash
     sudo docker run -itd --privileged --name xxxx -p xxx:22 ubuntu:latest /bin/bash
     ```

     - 进入容器，安装python3.12

     ```bash
     apt update
     apt install python3.12 -y
     ```

     -   设置软连接
     
     ```bash
     update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 312 
     update-alternatives --config python3 
     ```
     
     - 测试python版本
     
     ```bash
     python --version
     ```
     
     - 安装pip3
     
     ```bash
     apt install pip3
     ```
     
     但是！直接pip3 安装会出错，所以我直接粗暴的去掉这个错误，具体原因可[参考](https://www.yaolong.net/article/pip-externally-managed-environment/)
     
     ```bash
     sudo mv /usr/lib/python3.12/EXTERNALLY-MANAGED /usr/lib/python3.12/EXTERNALLY-MANAGED.bk
     ```
     
     - 克隆dirsearch仓库，克隆之前要配置一下git 的ssh连接，具体可以参考[Server_Docker_Github_SSH](https://country-if.github.io/posts/Server_Docker_Github_SSH/)
     
     ```bash
     git clone git@github.com:maurosoria/dirsearch.git
     ```
     
     - 进入dirsearch目录下，然后安装依赖
     
     ```bash
     pip install -r requirements.txt
     ```
     
     接下来就可以愉快的使用啦～

​	