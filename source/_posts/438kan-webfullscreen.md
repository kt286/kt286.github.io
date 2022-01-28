---
title: 使用 UserScript 给 438kan 添加网页全屏支持
date: 2022-01-28 19:26:15
categories: UserScript
tags: [dplayer,userscript]
---

## 前言
最近一直在 [438kan](http://438kan.com) 上追剧，但是 438kan 自带的播放器嵌套在一个 `iframe` 中，不能使用网页全屏功能。而使用全屏播放视频时遇到要回复QQ、微信消息的情况则需要不断在全屏/非全屏之间切换，非常麻烦。所以决定研究研究，实现 438kan 的网页全屏功能。

## 初次分析
我们首先在浏览器中打开播放器地址 [http://438kan.com/static/player/dplayer.html](http://438kan.com/static/player/dplayer.html)，右键-查看网页源代码，找到了其中的关键代码

```javascript
var dplayer = new DPlayer({
	element: document.getElementById("player"),
	autoplay: true,
	video: {
		url: parent.MacPlayer.PlayUrl,
		type:'hls',
	}
});
```
通过 `parent.MacPlayer.PlayUrl` 这句我们可以发现，视频播放地址是在 `iframe` 外加载的，根本没必要使用 `iframe` ，可以把 `iframe` 整个替换掉。这里我们直接用这个 `iframe` 的父级div `MacPlayer` 作为播放器的容器，通过 `UserScript` 中的 `@require` 加载 `dplayer` 及其依赖 `hls`，直接加载 `MacPlayer.PlayUrl` 这个地址的视频

```javascript
// ==UserScript==
// @name             438kan网页全屏
// @description      438kan网页全屏
// @version          1.0
// @author           kt286
// @include          *://*.438kan.com/*
// @include          *://438kan.com/*
// @require          https://cdn.jsdelivr.net/npm/hls.js@1.1.3/dist/hls.min.js
// @require          https://cdn.jsdelivr.net/npm/dplayer@1.26.0/dist/DPlayer.min.js
// @grant            none
// @run-at           document-end
// ==/UserScript==
var dplayer = new DPlayer({
    element: document.querySelector(".MacPlayer"),
    autoplay: true,
    video: {
        url: MacPlayer.PlayUrl
    }
});
```
这里需要注意一个问题，`iframe` 中播放器地址是被 [http://438kan.com/static/player/dplayer.js](http://438kan.com/static/player/dplayer.js) 这个 js 加载的，为了防止不必要的资源加载，我们使用 `uBlock Origin` 或 `Adblock Plus` 将其拦截。

## 第二次分析
脚本写完后，复制到 [Violentmonkey](https://violentmonkey.github.io/) 中，刷新网页。音乐声响起，但是视频没有出现。通过 F12 我们可以发现视频已经加载出来了，但是高度异常

![视频高度异常](/img/posts/438kan-webfullscreen/1.png)

继续分析我们发现 `MacPlayer` 这个元素的 `padding-bottom` 居然有 `500` 多，把视频高度都给挤没了。

![播放器 padding-bottom](/img/posts/438kan-webfullscreen/2.png)

既然知道问题出在哪里了，解决方案也就有了，那就是设置 `padding-bottom` 为 `0px`。现在我们就有两种实现方案：`UserCSS` 和 `UserScript`。这里我们继续使用 `UserScript`。`UserScript`中添加自定义样式的方法是 `GM_addStyle`。

```javascript
// ==UserScript==
// @name             438kan网页全屏
// @description      438kan网页全屏
// @version          1.0
// @author           kt286
// @include          *://*.438kan.com/*
// @include          *://438kan.com/*
// @require          https://cdn.jsdelivr.net/npm/hls.js@1.1.3/dist/hls.min.js
// @require          https://cdn.jsdelivr.net/npm/dplayer@1.26.0/dist/DPlayer.min.js
// @grant            GM_addStyle
// @run-at           document-end
// ==/UserScript==
GM_addStyle(".MacPlayer { padding:0!important; }");
var dplayer = new DPlayer({
    element: document.querySelector(".MacPlayer"),
    autoplay: true,
    video: {
        url: MacPlayer.PlayUrl
    }
});
```

## 第三次分析
增加自定义样式后，视频出来了，网页全屏功能也正常了。但还有一点点小瑕疵，就是导航栏悬浮在播放器上方，遮挡了一部分视频。这个就简单了，只要把播放器 `z-index` 值设置的大于导航栏的值就可以了，完整代码如下：

```javascript
// ==UserScript==
// @name             438kan网页全屏
// @description      438kan网页全屏
// @version          1.0
// @author           kt286
// @include          *://*.438kan.com/*
// @include          *://438kan.com/*
// @require          https://cdn.jsdelivr.net/npm/hls.js@1.1.3/dist/hls.min.js
// @require          https://cdn.jsdelivr.net/npm/dplayer@1.26.0/dist/DPlayer.min.js
// @grant            GM_addStyle
// @run-at           document-end
// ==/UserScript==
GM_addStyle(".MacPlayer { padding:0!important; z-index:999!important; }");
var dplayer = new DPlayer({
    element: document.querySelector(".MacPlayer"),
    autoplay: true,
    video: {
        url: MacPlayer.PlayUrl
    }
});
```

刷新视频，网页全屏一切正常。这下舒服了。