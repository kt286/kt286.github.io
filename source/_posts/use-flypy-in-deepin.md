---
title: 在 deepin 中使用小鹤音形
date: 2022-03-14 20:56:25
updated: 2022-04-10 21:59:07
categories: Deepin
tags: [deepin,flypy,rime]
---

## 前言
之前一直在使用全拼输入法，但是全拼有一个众所周知的问题：重码率高。虽然这类输入法大都有词频动态调整、云输入等功能。但是每个用户习惯不同，换台设备后打字效率就会降低。而五笔输入法需要记忆大量字根，上手难度大。所以在网上看到小鹤音形的时候眼前一亮，决定尝试一下。

## 介绍
小鹤音形分双拼和双形两部分，简单理解就是先告诉输入法这个字怎么读，再告诉输入法怎么写。通过两个维度确定一个字，极大减少了重码问题。详细介绍见小鹤音形官网[^1]。下边放一下键位图和字根图备忘。

### 双拼
![小鹤双拼键位图](/img/posts/use-flypy-in-deepin/hejp.png)

### 双形
![小鹤鹤形字根图](/img/posts/use-flypy-in-deepin/xhup.png)

## 使用

### 方案1：RIME 挂接（官方）
小鹤官方只提供了 Windows 和 Android 版，其他平台可以用 RIME 挂接。但是 deepin 仓库中未提供 fcitx5-rime 包。所以我们需要自己编译一下。
```bash
sudo apt-get install -y build-essential cmake-extras extra-cmake-modules appstream libecm-dev libfcitx5core-dev fcitx5-modules-dev librime-dev
mkdir -p ~/workspace/fcitx5-rime
git clone https://github.com/fcitx/fcitx5-rime.git ~/workspace/fcitx5-rime
mkdir -p ~/workspace/fcitx5-rime/build
cd ~/workspace/fcitx5-rime/build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j4
sudo make install
```
编译安装完成后，下载小鹤网盘[^2]中挂接文件 `3.1.挂接——音形码/小鹤音形“鼠须管”for macOS.zip`。解压后，将 rime 目录中所有文件复制到 `~/.local/share/fcitx5/rime` 目录中。重启 fcitx5 ，添加 `中州韻` 输入法后即可使用。
![fcitx5-rime](/img/posts/use-flypy-in-deepin/fcitx5-rime.png)

### 方案2：fcitx5-flypy
fcitx5-flypy 是基于公开码表打造的适配 fcitx5 的输入法，无需安装 rime 即可使用。编译安装方法如下
```bash
sudo apt-get install build-essential cmake-extras extra-cmake-modules gettext appstream libecm-dev libfcitx5core-dev libboost-dev libimecore-dev libimetable-dev
mkdir -p ~/workspace/fcitx5-flypy
git clone https://github.com/kt286/fcitx5-flypy.git ~/workspace/fcitx5-flypy
mkdir -p ~/workspace/fcitx5-flypy/build
cd ~/workspace/fcitx5-flypy/build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j4
sudo make install
```
目前发现几个小问题：
1. 使用万能键 ` 时，会提示词语，而不仅仅是单字
2. 码表中记录条数没有官方码表多（Windows 版有 67656 条数据，但本项目只有 57655 条数据）
3. 使用 orq、ouj 无法输入当前日期和时间

![Windows 版码表](/img/posts/use-flypy-in-deepin/mabn.png)

如在使用过程中遇到其他问题，欢迎到 [github](https://github.com/kt286/fcitx5-flypy/issues) 中提 issue。

## 参考
[^1]: [小鹤入门](https://help.flypy.com/#/xh)
[^2]: [小鹤网盘](http://flypy.ys168.com)