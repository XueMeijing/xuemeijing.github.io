---
title: 2023年河南省大学生国家安全知识竞赛网页技术解密
date: 2023-05-17 16:19:03
categories: 
- 有趣
---

昨天一个后端朋友给我发了一张截图，问我这个[2023年河南省大学生国家安全知识竞赛网页](https://2023gjaqzsjs.haedu.cn/gajs/#/questionBank)f12内DOM节点显示韩文，但是页面显示是中文，复制出来的结果是韩文，问我怎么回事。我也是第一次见这种情况，便研究了起来。本文主要记录我调试和发现答案的流程，涉及两部分，一个是该网页字体解密，一个是JSON数据AES解密

**刚写完这篇文章，刷新就打不开这个网页了，发现竞赛学习时间过了🥲**

# 网页字体解密

**注意：以下截图均为github链接，不用vpn很大可能打不开**

## 定位原因
通过调试发现，这类字体都有.secret类名，该类名指定使用一个名叫ddjdt的font-family，去掉该font-family属性页面正常显示为韩文，因此确定是字体原因

![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/15cef2a3-607c-46d1-b121-cd81522cc36b)

## 寻找资料 了解原理

在网络上搜索字体加密有很多文档，大部分都是拿猫眼的网页当做例子，但是由于每个网站的加密方式不一样，所以那些教程不一定适用于该网页，但是整体的流程大概明白了。

简单来说字体加密主要是为了反爬虫和防复制，比如猫眼这些网站就是为了防止别人爬数据，对于这个知识竞赛网页主要是为了防止复制搜答案。

对于汉字 **我** 来说，他可以有宋体、黑体、微软雅黑等不同的字形展示，但是他们对应的Unicode编码是同一个，同理也可以创造一个新的字体库叫animal，把 **我** 对应的字形描绘成老虎，这样浏览器在碰到 **我** 的Unicode时，就会把它渲染成老虎的样式。字体库内部使用TTGlyph来存储字形轮廓信息，它和SVG一样都是矢量图形格式。

所以该网页DOM内部是韩文，这些韩文的Unicode编码对应的字形实际上是微软雅黑或者其他字体库的汉字字形TTGlyph，导致页面渲染出来是汉字。

## 字体库xml分析验证

打开css发现该字体使用的是 https://2023gjaqzsjs.haedu.cn/fonts/wryh2.ttf 这个字体文件，下载文件，然后用python把ttf转换成xml文件看到字体库内部结构，代码如下

```python
from fontTools.ttLib import TTFont, TTCollection

# 某些字体库是集合，TTFont会报错，需要用TTCollection
font = TTFont("wryh2.ttf")
font.saveXML("wryh2.xml")
```
解析后的xml文件首先看到的是GlyphOrder，内部有很多GlyphID
```xml
<GlyphOrder>
  <GlyphID id="0" name="glyph00000"/>
  <GlyphID id="1" name="glyph00001"/>
  <GlyphID id="2" name="glyph00002"/>
  ...
</GlyphOrder>
```
搜索 glyph00002，可以看到对应一个Unicode编码 0xb460
```xml
<cmap_format_4 platformID="0" platEncID="3" language="0">
  ...
  <map code="0xb45c" name="glyph00157"/><!-- HANGUL SYLLABLE DULS -->
  <map code="0xb460" name="glyph00002"/><!-- HANGUL SYLLABLE DUM -->
  ...
</cmap_format_4>
```
转换0xb460打印出来是韩文，也证实了上面的说法
```javascript
String.fromCharCode('0xb460') // '둠'
```
同时与glyph00002相关的还有这个TTGlyph，也就是它的字形文件了
```xml
<TTGlyph name="glyph00002" xMin="0" yMin="0" xMax="1077" yMax="1582">
  <contour>
    <pt x="1077" y="0" on="1"/>
    <pt x="189" y="0" on="1"/>
    <pt x="189" y="169" on="1"/>
    <pt x="536" y="169" on="1"/>
    <pt x="536" y="1345" on="1"/>
    <pt x="180" y="1242" on="1"/>
    <pt x="180" y="1422" on="1"/>
    <pt x="731" y="1582" on="1"/>
    <pt x="731" y="169" on="1"/>
    <pt x="1077" y="169" on="1"/>
  </contour>
  <instructions/>
</TTGlyph>
```

## 操作、预览字体库

在查找的文档中，很多人说可以用FontCreator加载字体库，但是它只适用于windows，并且有试用期，故放弃。问了chatGPT后说可以用BirdFont和FontDrop，

- [BirdFont](https://birdfont.org/)：免费，有mac客户端，可以根据Unicode搜索
- [FontDrop](https://fontdrop.info/)：免费，网页端，没有搜索功能
- [百度字体编辑器](https://kekee000.github.io/fonteditor/)：免费，网页端，可以根据Unicode搜索(吐个槽，百度好像把这个产品下架了，这个链接还是github的)

我使用的是FontDrop，很方便，把字体文件拖进来就能解析，可以看到字体名称和字体作者等信息
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/b977cedc-77aa-4ac2-88a3-a2559f670d46)
下面是字体展示，鼠标滑过会展示该字形的Unicode
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/06594a36-c206-43e7-9a6c-0181eac5a31c)
分析DOM结构，获取全部字形的Unicode
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/f7f31764-7173-4b49-a20a-3aad1f8077c8)
```javascript
[...document.querySelector('#glyph-list-end').children].forEach(item => {
  const regex = /Unicode:\s+(\w+)/;
  const match = item.children[0].textContent.match(regex);
  
  if (match) {
    const unicode = match[1];
    console.log(unicode); // 输出: BFEE
  } else {
    console.log('未找到匹配的内容');
  }
})
```
执行结果如下
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/51659aee-f59d-4d0f-ad0a-e5047460432d)
emmmm。竟然还有未找到匹配的内容，调试后发现该字形的Unicode是鼠标滑过字形后动态插入到DOM的，鼠标没有滑过的字形就是未找到
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/62b3ef20-8838-4228-a4aa-3ccae7af8c72)
简单，给全部字形节点加mouseover事件，手动触发，再获取全部Unicode就正常了
```javascript
[...document.querySelector('#glyph-list-end').children].forEach(item => {
  var event = new MouseEvent('mouseover');
  item.dispatchEvent(event);
})
```
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/747cc3db-a27f-40f3-873e-8e777b95df66)

## 获取Unicode和字形的映射，创建密码表

没办法，这里只能手动建立映射了，或者有OCR接口的话可以优化一下
```javascript
var unicodeArr = ["none","BFEE","B460","BA67"...]
var decryptArr = ["","0","1","2"...]
var unicodeMap = {}
unicodeArr.forEach((item, index) => {
  unicodeMap[item] = decryptArr[index]
})
console.log(unicodeMap)
```
映射结果如下
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/d471864a-d53f-4052-ae19-2897f9bcd6bb)

## 解密

复制一句密文来测试下效果
```javascript
var encryptStr = '욜춼헵풱쐋뒈킧젋쉍쯃总쉫욜춼헵풱칌，以(      )为宗旨。'
var decryptSte = encryptStr.split('')
  .map(item => {
    const HEX = item.charCodeAt().toString(16).toUpperCase()
    return unicodeMap[HEX] || item
  })
  .join('')
console.log('encryptStr', encryptStr)
console.log('decryptSte', decryptSte)
```
解密效果如下
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/eeddd7e1-6d6f-4f4d-aaf4-a9a6bee733f7)
看起来解密效果还可以，一部分字体没有解密出来是因为字体文件的问题，多个Unicode对应同一个字体文件TTGlyph，用上面说的百度字体编辑器可以看到 **国** 这个字形文件有两个Unicode，把这些特殊Unicode手动加到之前的密码表就好了
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/7e795ae6-fb06-415c-bfe5-2eacc5117642)


# JSON数据解密

## 确认接口
这些加密字体很明显是后端接口传过来的，但是这个分页接口返回的竟然是加密后的字符串，真有你的
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/6f04b6d7-f7cf-466d-9eb6-125b1feaa2ab)

## 定位代码
在返回的js文件内搜索这个接口地址 2023gjaqzsjs.haedu.cn 和 /json/，找到相关代码，看到返回的加密字符串被$decrypt方法解密
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/4c23698e-da90-4e69-abc8-1eb1b1351c88)

搜索$decrypt方法发现是用AES解密
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/01a7fce6-d1a0-46c1-a4f8-518ed9c5bcbc)

通过其他的一些变量名我猜用的很可能是**crypto-js**这个库，但是看crypto-js源码的decrypt函数第三个参数是一个配置对象啊，这里怎么是个字符串，难道是用的其他解密库？我在这里卡了半天，打断点才发现是作者创建了一个AES对象，把crypto-js的decrypt又封装了一下

这里的o方法就是把16进制转为10进制index，然后获取t[index], 比如r["AES"][o("0xa")]实际就是r["AES"]["decrypt"]

![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/41319712-5058-456f-a54a-095c55be2e4c)

引入crypto-js的cdn文件，然后复制相关解密变量到本地测试
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/f75438b8-8e9c-4a8a-a321-d376ab2f68f0)

结果如下
![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/970c662f-fbb8-4882-b05e-babb6140a4f6)

![image](https://github.com/XueMeijing/dingtalk-chatgpt-node/assets/35559153/3d8c92c6-07ca-46e0-82b0-d1d2fcf0fccb)

和知识竞赛题目内的加密字体一样，JSON解密完成

# 总结

学习了字体加密解密的一些知识，加深了对字体文件的理解。而且在我以往的印象里反爬虫就是ip限制，请求频率限制，没想到还有JSON加密和字体加密。但是目前还有个问题：后端怎么生成的加密字体文件和字体JSON，通过密码表还是加密规则，这个字体文件只加密了一部分常用字
