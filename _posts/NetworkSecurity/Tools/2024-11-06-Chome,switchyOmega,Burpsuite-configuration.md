---

title:  Chome,switchyOmega,Burpsuite configuration

date: 2024-11-06 10:00:00 +0800

categories: Tools

tags: [tools]

pin: true

---

## Chome,switchyOmega,Burpsuite 配置 

switchyOmega是chome的一款插件，非常有意思，特别是用于定向抓包，可以设置针对靶机定向抓包，再也不用傻傻去配置系统代理，不用在刷题的时候要查资料的时候重新设置。

1. chome商店直接查找下载[即可](https://chromewebstore.google.com/search/Proxy%20SwitchyOmega?hl=zh-CN&utm_source=ext_sidebar)

   ![image-20241107234733096](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241107234733096.png)

2. 安装后打开界面，点击代理Proxy。设置代理服务器地址以及端口，记得要和BP对应起来，不然抓不到包；不代理的地址列表中的匹配规则可以全部删除，可能会造成影响。

   ![image-20241107235008157](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241107235008157.png)

3. 使用很简单，pin一下插件在网页扩展栏。假设现在使用ctfhub.com，点击proxy就可以使用代理；点击系统代理就可以切换回来，是不是很方便呢～

   ![image-20241107235620872](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241107235620872.png)

后面发现如果用网页需要vpn才能，就访问不了，于是重新设置了规则。

1. 首先设置了通过BP代理的情景模式；

   ![image-20241108110124116](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241108110124116.png)

2. 接着设置proxy，是通过vpn进行代理的，其实也就是系统代理（如果电脑常开vpn），端口号可通过自己的科学上网软件查找
   ![image-20241108110354244](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241108110354244.png)

3. auto switch设置，设置靶机走bp情景模式，其他默认是走proxy，这样子就ok啦！

   ![image-20241108111619428](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241108111619428.png)