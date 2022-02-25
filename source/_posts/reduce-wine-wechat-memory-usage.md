---
title: 大幅降低 Wine 微信内存占用
date: 2022-02-25 11:20:18
updated: 2022-02-25 11:20:18
categories: Wine
tags: [Wine,微信]
---

## 前言
每次打开 Wine 版微信，内存使用率就会涨上一大截。平时没怎么在意，今天随意看了一眼就惊呆了：微信刚打开就已经占用了 1G 左右，而且随着使用时间推移甚至能涨到 2G 。不能忍，今天一定要好好治一治这个内存大户。
![Wine 微信内存占用](/img/posts/reduce-wine-wechat-memory-usage/1.png)

## 分析
系统监视器里搜索 `wechat` ，发现了华点：微信本体只占用了不到 100M 内存，剩下的都被 `WeChatWebBrower.exe` 和 `WeChatApp.exe` 这两个进程占走了。纳尼？？我用的可是微信，不是微信浏览器。精简掉这两个进程应该就可以减少内存占用了。

## 精简
右键属性里找到这两个进程所在的目录，发现两个都在一起，刚好可以一网打尽。
```bash
rm -rf ~/.deepinwine/Deepin-WeChat/drive_c/users/$(whoami)/Application\ Data/Tencent/WeChat/XPlugin/Plugins/XWeb/
touch ~/.deepinwine/Deepin-WeChat/drive_c/users/$(whoami)/Application\ Data/Tencent/WeChat/XPlugin/Plugins/XWeb
chmod 000 ~/.deepinwine/Deepin-WeChat/drive_c/users/$(whoami)/Application\ Data/Tencent/WeChat/XPlugin/Plugins/XWeb 
```
精简掉这两个进程后再次打开微信，只占用 200M 左右内存，舒服了。
![精简后的微信内存占用](/img/posts/reduce-wine-wechat-memory-usage/2.png)

## 后记
当然了，精简之后肯定会有一些功能异常，目前发现的只有收到的链接、公众号文章无法打开，小程序无法使用。反正我也不在电脑上使用小程序，可以忍。至于公众号文章查看，可以通过一些设置来继续使用，只是有些繁琐。
![关闭`使用系统默认浏览器打开网页`](/img/posts/reduce-wine-wechat-memory-usage/3.png)
![用默认浏览器打开](/img/posts/reduce-wine-wechat-memory-usage/4.png)