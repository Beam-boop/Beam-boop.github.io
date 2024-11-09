---

title: Ctfhub Study 1

date: 2024-11-08 10:00:00 +0800

categories: NetworkSecurity

tags: [network security]

pin: true

---

## ctfhub刷题笔记

### 1. motivation

​	听师兄说得刷刷ctf题目才能更好掌握知识，马上来学！学的时候刚好看到一个[大佬的刷题笔记](https://wuuconix.link/2021/08/18/ctfhub/)，除了从attacker的角度解题之外，还从编写程序的角度考虑了该漏洞是如何实现的。没办法像大佬一样去申请域名，然后建一个自己的网站，所以就基于服务器搭建了一个基于ubuntu的apache环境，来学习php，以及复现漏洞，具体的搭建方法可以[参考](http://10.2.34.87:10251/posts/Ubuntu-apache2-configuration/)。

### 2. web前置技能-Http协议（TODO）

   - 请求方式
   - 302跳转
   - Cookie
   - 基础认证
   - 响应包源代码

### 3. 信息泄漏

   - 目录遍历

     ![image-20241109211202721](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109211202721.png)

     其实只需要在里面找到flag就好了，so easy～

     主要是如何php实现网页上遍历目录，那就按照规矩，去apache下实现一下吧

     其实比较简单，就在目录下新建一个文件夹b，然后里面添加.htaccess文件，通在里面写入以下内容

     ```bash
     <Files *> Options Indexes </Files>
     ```

     ![image-20241109213232232](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109213232232.png)

     - phpinfo

       这题目的解法比较简单，flag藏在了phpinfo里面，直接ctrl f就可以搜出来了

       ![image-20241109213804383](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109213804383.png)

       还是老规矩，得看看怎么实现的哈～

       好吧，我以为可以通过phpinfo()函数的输入去更改，结果不行；后面又尝试了修改'php.ini'，也不太行，所以得按照大佬的[办法](https://wuuconix.link/2021/08/18/ctfhub/#phpinfo)去做了，这里就不复现了