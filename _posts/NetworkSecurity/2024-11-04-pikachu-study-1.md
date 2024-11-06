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

- ##### 验证码绕过

  设计验证码的初衷是为了防止暴力破解

  1. on-client(在浏览器前端设计的，很容易被绕过)，通过F12查看网页源码，可以发现验证码是通过js代码实现的生成和验证，那么就可以重复基于表单的暴力破解来找出账号密码；
  
     ![image-20241106103842729](https://cdn.jsdelivr.net/gh/Beam-boop/cloudimages/imagesimage-20241106103842729.png)