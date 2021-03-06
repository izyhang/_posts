---
layout: post
title: 支付宝小程序指定页面跳转
date: 2020-03-11 13:25:37
updated: 2020-03-11 13:25:37
tags:
  - alipay
  - tinyapp
  - route
categories: Android
---

支付宝小程序支持从外部调起，具体做法是通过`scheme`，如下

alipays://platformapi/startapp?appId=[appId]&page=[pagePath]&query=[params]

- appId 是小程序唯一 Id
- pagePath 是页面路径，也就是本文要讲的，不带则跳首页
- params 是额外参数

> 更多信息请前往官网
> [https://opensupport.alipay.com/support/knowledge/31867/201602383690?ant_source=zsearch](https://opensupport.alipay.com/support/knowledge/31867/201602383690?ant_source=zsearch)

<!-- More -->

## 例子 - 答答星球

### 如何拿到答答星球的 appId

- 最简单的方法就是通过小程序分享功能，分享到钉钉，再打开，复制链接，可以拿到这么一串 url
  https://render.alipay.com/p/s/i/?scheme=alipays%3A%2F%2Fplatformapi%2Fstartapp%3FappId%3D77700189%26page%3Dpages%252Findex%252Findex%26enbsv%3D0.1.2003090940.1%26chInfo%3Dch_share__chsub_DingTalkSession
- decode 一下 scheme 部分
  alipays://platformapi/startapp?appId=77700189&page=pages%2Findex%2Findex&enbsv=0.1.2003090940.1&chInfo=ch_share\_\_chsub_DingTalkSession
- 其中的 77700189 就是答答星球的 appId

> page 部分再 decode 则是 pages/index/index，首页的意思，但是我想跳到其他页面那就得知道路径

### 拿路径

- 一台 root 的手机
- 进入到`data/data/com.eg.android.AlipayGphone/files/nebulaInstallApps`
  ![](1.jpeg)
- 找到 77700189
  ![](2.jpeg)
- 其中的 tar 文件就是答答星球打包后的压缩包
- 传到电脑解压拿到所有页面路径
  ![](3.jpeg)

### 跳转

比方说想要跳转到答答星球的天天涨知识页面`pages/domain/home/index`

```kotlin
val scheme = "alipays://platformapi/startapp?appId=77700189&page=pages%2Fdomain%2Fhome%2Findex"
val intent = Intent(Intent.ACTION_VIEW, Uri.parse(scheme))
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
startActivity(intent)
```
