---

title: vulnhub study

date: 2025-04-08 10:00:00 +0800

categories: NetworkSecurity

tags: [network security]

pin: true

---
# vulnhub刷题笔记

#### oscp

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