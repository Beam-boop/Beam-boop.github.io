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

   - #### 目录遍历

   ![image-20241109211202721](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109211202721.png)

   其实只需要在里面找到flag就好了，so easy～

   主要是如何php实现网页上遍历目录，那就按照规矩，去apache下实现一下吧

   其实比较简单，就在目录下新建一个文件夹b，然后里面添加.htaccess文件，通在里面写入以下内容

   ```bash
   <Files *> Options Indexes </Files>
   ```

   ![image-20241109213232232](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109213232232.png)

   - #### phpinfo

     这题目的解法比较简单，flag藏在了phpinfo里面，直接ctrl f就可以搜出来了

     ![image-20241109213804383](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109213804383.png)

     还是老规矩，得看看怎么实现的哈～

     好吧，我以为可以通过phpinfo()函数的输入去更改，结果不行；后面又尝试了修改'php.ini'，也不太行，所以得按照大佬的[办法](https://wuuconix.link/2021/08/18/ctfhub/#phpinfo)去做了，这里就不复现了
     
     - #### 备份文件下载
     
       - ##### 网站源码
     
       ![image-20241111222529136](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241111222529136.png)
     
       看这个题目，并且通过之前apache的学习，感觉备份文件应该在地址的/目录下，所以可以尝试用脚本撞一下哪个组合才是对的，以下是我的测试代码：
       
       ```python
       import requests
       
       a_list = ["web", "website", "backup", "back", "www", "wwwroot", "temp"]
       b_list = ["tar", "tar.gz", "zip", "rar"]
       for a in a_list:
           for b in b_list:
               url = "http://challenge-a6e7a6b7b0ef7729.sandbox.ctfhub.com:10800/" + a + "." + b
               print("test:", url)
               res = requests.get(url)
               if(res.status_code != 404):
                   print("find it!")
               else:
                   print("return 404")
       ```
     
       结果如下：
     
       ![image-20241111225509163](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241111225509163.png)
     
       所以我们可以看到 `www.zip`就是备份文件，输入进去后，直接下载了一个文件，但是！怎么这样子！
     
       ![image-20241111225642156](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241111225642156.png)
     
       百思不得其解，那到底去哪里拿呢？去上了个厕所，就知道了哈哈，可以直接输入浏览器，至于为什么，不懂，纯靠经验吧
       
       ![image-20241111225807958](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241111225807958.png)
       
       - ##### bak文件
       
         其实就是寻找index.php的备份文件，有一个很简单的办法，就是从标题入手，既然是bak文件，那后缀应该是.bak，输入/index.php.bak，结果还真是。打开下载的文件就有flag了，但是！我们不能这么简单去解决，毕竟比赛的时候可不会这么简单告诉你的，于是我们需要了解.php文件的备份后缀有哪些，以下就是我列举出来的(肯定还有，先列举这些)
       
         ```txt
         index.php.git
         index.php.svn
         index.php.swp
         index.php.bak
         index.php.bkf
         index.php.rar
         index.php.zip
         index.php.7z
         index.php.txt
         ```
       
         那么，我们可以通过web路径扫描仪[dirseach](https://github.com/maurosoria/dirsearch)来实现，进入`/dirsearch`目录下，输入命令
       
         ```bash
         python3 dirsearch.py -u http://challenge-ff6ff29ba8eee467.sandbox.ctfhub.com:10800/ -w ../php_backup.txt
         ```
       
         这就尝试出了结果，应该来说上面那题目也可以这么做，dirsearch的下载和安装可以[参考](https://beam-boop.github.io/posts/Docker,-python,-dirsearch-configuration/)
       
         ![image-20241112124815447](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241112124815447.png)