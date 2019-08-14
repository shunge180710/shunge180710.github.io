---
layout:     post
title:      "如何使用dateFns或原生js进行时间戳运算"
# subtitle:   "欢迎使用"
date:       2019-02-01 12:00:00
author:     "zhishun"
header-img: "img/bg/hello_world.jpg"
catalog: true
tags:
    - 说明
---

## date-fns ( JavaScript日期实用程序库)
[官方文档](https://date-fns.org/docs/Getting-Started)
进去之后大概是这个样子，这里可以选择自己的安装方式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201105620391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTYwNDk5Mg==,size_16,color_FFFFFF,t_70)
## vue/nuxt
公司项目用的是nuxt，所以也发一下出来，感兴趣的朋友可以稍微了解一下
[vue.js文档](https://cn.vuejs.org/v2/guide/)
[nuxt文档](https://zh.nuxtjs.org/guide/installation)


## 直接进入正题
我需要增加个显示时间的功能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201123513509.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201123724776.png)
```javascript
//首先引入date-fns
import dateFns from 'date-fns'
```
[关于format](https://date-fns.org/v1.30.1/docs/format)
```javascript
//运用dateFns的format,迅速得出进店和离店时间
  computed: {
    ...mapState({
      actionInfo: state => state.rawReportData.actionInfo
    }),
    EntranceTime() {
      return dateFns.format(this.actionInfo.enterTime, 'HH:mm:ss')
    },
    DepartureTime() {
      return dateFns.format(this.actionInfo.leaveTime, 'HH:mm:ss')
    },
```
接下来本以为在店时间也是如此滴顺利

[时间戳转换](http://tool.chinaz.com/tools/unixtime.aspx)
[Math.floor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor)
```javascript
//把13位时间戳-毫秒（还有种10位的时间戳代表秒数），/1000转化成秒数，Math.floor向下取整，相减后*1000转化为毫秒
//当时间戳为0，代表北京时间1970/1/1 8:0:0；时间戳为1，代表1970/1/1 8:0:1，以此类推
 InstoreTime() {
      return dateFns.format(
        (Math.floor(this.actionInfo.leaveTime / 1000) -
          Math.floor(this.actionInfo.enterTime / 1000)) *
          1000,
        'HH:mm:ss'
      )
```
**在店时间 = 离店 - 进店**
<font face="微软雅黑" size=4 color="deeppink"> 划重点！！这一步是为了避免显示在店时间多一秒或者小一秒的情况！！！</font>
本还在感叹我考虑得如此周到，如此天衣无缝，简直是天才，然鹅。。。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201130654238.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201130703972.png)
<font face="微软雅黑" size=4 color="deeppink"> 我谷歌搜了一圈，貌似是时差的原因，所以会差8小时。强行减去8小时？？？不知道会的朋友们能不能提供一下解决得思路</font>

只好用原生js手撸一份
```javascript
//中间发现了一个bug,在店时间应该要能显示大于24小时得，而不是到24小时重新归零
//所以之前的方法是不妥的 let hours = parseInt(msec / (1000 * 60 * 60)) 就很好的解决了这个问题
 InstoreTime() {
      let msec =
        (Math.floor(this.actionInfo.leaveTime / 1000) -
          Math.floor(this.actionInfo.enterTime / 1000)) *
        1000
      let hours = parseInt(msec / (1000 * 60 * 60))
      let minutes = parseInt((msec % (1000 * 60 * 60)) / (1000 * 60))
      let seconds = parseInt((msec % (1000 * 60)) / 1000)
      hours < 10 ? (hours = '0' + hours) : hours
      minutes < 10 ? (minutes = '0' + minutes) : minutes
      seconds < 10 ? (seconds = '0' + seconds) : seconds
      return hours + ':' + minutes + ':' + seconds
    }
```
比如要求小时，想着如何把毫秒转化为小时，emmm，/1000变成秒，/60变成分钟，再/60变成小时，就做出来了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190201131800526.png)
