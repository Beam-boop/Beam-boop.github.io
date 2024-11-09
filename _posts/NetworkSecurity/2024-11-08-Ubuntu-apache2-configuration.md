---

title: Ubuntu apache2 configuration

date: 2024-11-08 10:00:00 +0800

categories: Environment

tags: [environment]

pin: true

---

## Ubuntu apache2 的配置

### 1. Motivation

   docker下安装apache，在网上有很多办法，比如说

```bash
docker pull httpd:latest
```

​	但是很多都是基于debain的linux系统，而没有基于ubuntu，本人比较习惯ubuntu，并且为了更好的学习网安攻防的	基础知识，所以决定自己搭建一个本地的apache环境，以供安全学习。

### 2. Prerequisites

   docker有ubuntu的镜像

### 3. Solution

   - 创建docker容器

   ```bash
   sudo docker run -itd --privileged --name [xxx_apache] -p xxx:22 -p xxx:80 -p xxx:443 [images] /bin/bash
   ```

   ```bash
   sudo docker exec -it [name] bash
   ```

   解释：22是ssh链接的端口，主要是为了方便后面在本地用vscode直接连接服务器进行代码创造；80端口是apache2 http的默认端口，443是https的默认端口，我全都映射的宿主机的端口

   - 安装与开启apache2

     以下都不需要用root，因为docker下一般都在root下

     ```bash
     apt update
     apt install apache2
     service apache2 status
     ```

     查看完apache2的状态后，如果是没有开启，可以开启

     ```bash
     service apache2 start
     # some order of apache2
     service apache2 restart
     service apache2 stop
     service apache2 enable
     service apache2 reload
     ```

     这个时候其实可以直接访问到apache2的页面，本地浏览器打开[ip地址:端口(刚才docker run映射80端口的宿主机的端口)]

   - 设置虚拟主机

     直接[参考](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04)即可，写得很好，以下为翻译。

     Ubuntu 20.04 上的 Apache 默认启用一个服务器块，该服务器块配置为提供 /var/www/html 目录中的文档。虽然这对于单个站点来说效果很好，但如果托管多个站点，它可能会变得笨拙。我们不修改 `/var/www/html`，而是在 /var/www 中为 your_domain 站点创建一个目录结构。

     ```bash
     mkdir /var/www/your_domain
     ```

     接下来，使用 `$USER` 环境变量分配目录的所有权(`$USER`我给的是root)：

     ```bash
     chown -R $USER:$USER /var/www/your_domain
     ```

     如果没有修改设置默认文件权限的 umask 值，则Web 根目录的权限应该是正确的。为了确保您的权限正确并允许所有者读取、写入和执行文件，同时只授予组和其他人读取和执行权限，可以输入以下命令：

     ```bash
     chmod -R 755 /var/www/your_domain
     ```

     接下来，使用 nano 或vim创建示例 index.html 页面：

     ```bash
     apt install vim -y
     vim /var/www/your_domain/index.html
     ```

     在里面添加以下示例 HTML：

     ```html
     <html>
         <head>
             <title>Welcome to Your_domain!</title>
         </head>
         <body>
             <h1>Success!  The your_domain virtual host is working!</h1>
         </body>
     </html>
     ```

     完成后保存并关闭文件。 为了让 Apache 提供此内容，有必要使用正确的指令创建一个虚拟主机文件。我们不要直接修改 /etc/apache2/sites-available/000-default.conf 中的默认配置文件，而是在 /etc/apache2/sites-available/your_domain.conf 中创建一个新配置文件：

     ```bash
     vim /etc/apache2/sites-available/your_domain.conf
     ```

     粘贴以下配置块，该配置块与默认配置块类似，但针对我们的新目录和域名进行了更新：

     ```shell
     <VirtualHost *:80>
         ServerAdmin webmaster@localhost
         ServerName your_domain
         ServerAlias www.your_domain
         DocumentRoot /var/www/your_domain
         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
     ```

     请注意，我们已将 DocumentRoot 更新为新目录，并将 ServerAdmin 更新为 your_domain 站点管理员可以访问的电子邮件。我们还添加了两个指令：ServerName，它建立应与此虚拟主机定义匹配的基本域；ServerAlias，它定义应匹配的ServerName。

     完成后保存并关闭文件。 使用 a2ensite 工具启用该文件：

     ```bash
     a2ensite your_domain.conf
     ```

     禁用 000-default.conf 中定义的默认站点：

     ```bash
     a2dissite 000-default.conf
     ```

     接下来，测试一下配置是否错误，有这个就算ok，`Syntax OK`：

     ```bash
     apache2ctl configtest
     ```

     重新启动 Apache 以实施更改：

     ```bash
     systemctl apache2 restart
     ```

     重新打开网页，就会发现！

     ![image-20241109105354776](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241109105354776.png)