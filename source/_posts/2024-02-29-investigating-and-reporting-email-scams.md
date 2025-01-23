---
title: 面对骚扰诈骗邮件我做了什么，还能做什么？
date: 2024-02-29 11:19:03
categories: 
- 有趣
---

前几个月的某一天，在我想办法解决bug的时候qq邮箱突然发来了一条提示：**通知：因您本人2023年还款行为超出预期，现为您更新本年个人专属消费额度，请前往确认(AD)** ，我寻思我信用卡有什么新状况了，然后就点进去看了下，果不其然上当了，是一条骚扰信息，然后刚才解决bug的思路全断了

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/6078debe-4b92-4d38-98b5-552446eb1c3b)

这时候我想起来，为什么这种类似的邮件标题名我见过好几次呢，为此我翻了下之前的收件箱，果不其然有五六条。你发广告可以接受，你冒充信用卡 支付宝发短信骗人真是是可忍，孰不可忍，为此，我深入调查收集证据并进行了12315市场监督局 12377网信办 12321网络不良与垃圾信息举报受理中心的投诉，下面分享一下我的经历

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/f178057f-a500-4d98-b691-a09ae7ec9043)


## 收集证据

首先骚扰邮件主要分为三个部分：发件人地址，邮件内容图片地址和跳转地址，这三块的内容都可能是不同的。作为程序员我们应该都清楚，知道域名可以通过whois查询域名的注册信息，有的注册信息可能只有域名注册商知道，只显示给你一串id，你拿到这串id向注册商请求才可能找到真实信息，也有直接显示本身的注册信息的。为此我查了下这几封邮件的所有相关地址和whois信息，详细结果可以直接点击这个语雀文档查看 https://www.yuque.com/xuetengfei/kb/ggvd2bf2w66kklg4

就比如说2024-01-01这封骚扰邮件，新年第一天就收到骚扰邮件，真晦气
![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/e2d11275-2ecc-4355-beed-d786ea6595fd)

### 发件人

从图中可以看到发件人为**system@newsletter.cosemail.cn**，通过whois查询这个域名 https://whois.chinaz.com/cosemail.cn ，发现域名注册商是 **广州稳流互动网络有限公司**

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/4ee8504c-5ddb-4019-8071-dfc5580f132e)

### 邮件图片地址

图片的地址是 zwank.com，whois查询出来是域名注册商的信息，但是没关系，我国有条特殊政策就是域名需要备案，通过icp备案查询 https://icp.chinaz.com/zwwank.com ， 发现备案主办单位也是 **广州稳流互动网络有限公司** ，巧了么不是

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/334a310c-4c4b-4617-8be3-fc9df4798bfc)

### 跳转地址

跳转地址是从joipee.com 跳转到 https://wat.wodeyxbb.cn/ ，joipee.com 备案主办单位也是 **广州稳流互动网络有限公司** ，至于这个 wodeyxbb.cn 则是个人注册的 名叫**陈妙**，

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/b9c972fe-f151-4a74-a0b3-cf35ab19d03b)

通过whois查询**陈妙**，发现竟然还有qq

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/50cf5e62-7acf-4325-ab33-5bb53e491708)

但是站长之家没有完整显示qq，没事，我用命令行自己查也行  **whois wodeyxbb.cn** 

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/0ace250b-abad-4d05-b8d5-ea2679fa45bd)

原来是你啊

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/3222c788-a9ab-4f72-b764-36887574715a)

都一个多月过去了，发消息不回我，呵呵😄

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/16c0432c-3240-4195-ab6f-9f3dda10cbda)


## 举报

我把上面几封邮件的证据都写在了 https://www.yuque.com/xuetengfei/kb/ggvd2bf2w66kklg4 这个文档里，然后向12315 12377 12321举报，主要有两点，第一：涉嫌非法搜集我的个人信息邮箱账号，第二：骚扰、虚假信息，并涉嫌诈骗，因为我去黑猫投诉上搜了下，很多被这种邮件跳转的三方链接诈骗到的，为了响应国家反诈号召，净化网络空间，自己怎么也要出份力不是

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/fd8cd13c-ce99-460c-951c-21d8e503b248)

### 12315市场监督管理局

企查查查到 广州稳流互动网络有限公司 位于 广东省广州市天河区， 我就在12315小程序上进行了相关投诉

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/48fbc8ae-f658-45ff-9b90-5d1053ab5cbb)

昨天12315打电话通知我，说他联系了广州稳流互动，对方说邮件内容是深圳豪斯莱提供的，邮箱账号是北京品世信息提供的，让我去找另外两家公司。

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/3a0b0b07-c69b-47ff-82e2-8afe7148cc7f)

真是奇了，第一：邮件内容地址都是稳流的服务器。第二：邮箱就算是北京品世信息提供的，广州稳流是不是也涉嫌买卖公民个人信息。第三：最主要是的骚扰、诈骗邮件发件人备案主办单位就是广州稳流，这个怎么不敢提了。我问12315的工作人员，他说核实到的就是他说的反馈的信息，至于我提供的证据，他说他不管这一块没法核实，让我去12321投诉

不得不说12315真的是和稀泥的一把好手，2021年我在杭州一家名叫猫舍犬舍的店买了只猫，一周内就猫瘟花了五六千没救回来，找12315投诉要求退钱，也是一直和稀泥。我昨天看了下这个个店竟然还在，只不过又换了一个名字piupiu宠物家，这个店也是1818黄金眼的常客了，但是没办法

### 12377网信办不良信息举报中心

举报一个月了，并且这个只能填网址，证据都没有上传的地方，当然也是石沉大海了
![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/9b16b88e-8e93-44a0-b9df-3c078456a4b5)

### 12321网络不良与垃圾信息举报受理中心

就算成功了也只能封ip，可以说基本上没什么用，换个vps服务器或者代理ip，都不带重样的

![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/38f485a2-9177-44ab-98b6-02f315fc665e)
![image](https://github.com/XueMeijing/xuemeijing.github.io/assets/35559153/c39d5750-5174-4684-8040-de9b02fe5b0c)

## 总结

可以说折腾了这么久，也没有什么进展，该be evil的还是be evil