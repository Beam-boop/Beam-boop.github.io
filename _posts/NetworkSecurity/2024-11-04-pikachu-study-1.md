---

title: Pikachu Study 1

date: 2024-11-04 10:00:00 +0800

categories: NetworkSecurity

tags: [network security]

pin: true

---



## pikachu靶场通关—自学

### 暴力破解

尝试性破解+字典+自动化

- #### 基于表单的暴力破解

  设置payload，进行重放攻击。

  攻击类型包括：

  1. Sniper: payload只能替换一个变量位置，另外的位置按照原先的变量；

  2. Battering ram: payload同时替换所有变量位置；

  3. Pitchfork: 可以在每个变量位置设置一个payload，同时替换多个，但是是一一对应的替换，比如payloads的数据条数都是6，那么最后组合的结果就是6；

  4. Cluster bomb（**实战中常用**）： 可以在每个变量位置设置一个payload，同时替换多个，属于多对一对应的替换，比如payloads的数据条数都是6，那么最后组合的结果就是36。

     流程：配置代理，抓包后发送到测试器，设置payload，最后的结果可以通过返回数据包的长度or option—Grep-match来过滤

- #### 验证码绕过

  设计验证码的初衷是为了防止暴力破解

  1. on-client(在浏览器前端设计的，很容易被绕过)，通过F12查看网页源码，可以发现验证码是通过js代码实现的生成和验证，那么就可以重复基于表单的暴力破解来找出账号密码；
  
     ![image-20241106103842729](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241106103842729.png)

  2. on-server：
  
     正常的流程应该是后台生成验证码的图片，然后把验证码字符串存储在session中，将验证码图片传到前端显示。用户输入验证码后，后台拿到前端的post中的验证码字段进行比较。
  
     1. 后台验证码过期，php的验证码是24mins才过期
  
     2. 验证码校验不合格，出现逻辑漏洞。正常的验证码校验分为三类：没有验证码/验证码错误/验证码正确
  
     3. 验证码设计有规律，很容易被猜中
  
  解题的关键是判断可以从以上哪一种情况入手来进行攻击，通过前端直接输入可以看出校验逻辑没有问题，那就很有可能是验证码长期可以使用。使用bp，通过repeater模块，输入正确的验证码（要刷新页面重新获得验证码图片），测试不同账号密码，go后返回的提示来进行判断
  
  ![](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241107170030284.png)
  
  ![](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241107170121352.png)

发现可以用相同的验证码来进行暴力破解，那就好办了，重复之前的办法，拿到用户名和密码
