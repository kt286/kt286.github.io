---
title: 在 deepin 中使用小鹤音形
date: 2022-03-14 20:56:25
updated: 2022-04-17 19:03:46
categories: Deepin
tags: [deepin,flypy,rime]
---

## 前言
之前一直在使用全拼输入法，但是全拼有一个众所周知的问题：重码率高。虽然这类输入法大都有词频动态调整、云输入等功能。但是每个用户习惯不同，换台设备后打字效率就会降低。而五笔输入法需要记忆大量字根，上手难度大。所以在网上看到小鹤音形的时候眼前一亮，决定尝试一下。

## 介绍
小鹤音形分双拼和双形两部分，简单理解就是先告诉输入法这个字怎么读，再告诉输入法怎么写。通过两个维度确定一个字，极大减少了重码问题。详细介绍见小鹤音形官网[^1]。下边放一下双拼键位图和鹤形字根图备忘。

### 双拼
![小鹤双拼键位图](/img/posts/use-flypy-in-deepin/hejp.png)

### 双形
![小鹤鹤形字根图](/img/posts/use-flypy-in-deepin/xhup.png)

## 使用

### 方案1：rime 挂接（官方）
小鹤官方只提供了 Windows 和 Android 版安装包，其他平台需要使用 rime 挂接。但是 deepin 仓库中未提供 fcitx5-rime 包，需要我们自己编译安装。
```bash
# 安装编译工具
sudo apt-get install -y build-essential cmake-extras extra-cmake-modules

# 编译支持 lua 的 librime 库
sudo apt-get install -y libcapnp-dev libgoogle-glog-dev libyaml-cpp-dev libleveldb-dev libmarisa-dev capnproto
git clone https://github.com/rime/librime.git ~/workspace/librime
cd ~/workspace/librime
./install-plugins.sh hchunhui/librime-lua
make merged-plugins
sudo make install

# 编译 fcitx5-rime
sudo apt-get install -y appstream libecm-dev libfcitx5core-dev fcitx5-modules-dev
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
# 安装编译工具
sudo apt-get install -y build-essential cmake-extras extra-cmake-modules

# 编译 fcitx5-lua (orq 输入当前日期功能使用 lua 实现，如不需要，可以跳过此步)
sudo apt-get install -y gettext appstream libfcitx5core-dev fcitx5-modules-dev liblua5.3-dev 
mkdir -p ~/workspace/fcitx5-lua
git clone https://github.com/fcitx/fcitx5-lua.git ~/workspace/fcitx5-lua
mkdir -p ~/workspace/fcitx5-lua/build
cd ~/workspace/fcitx5-lua/build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j4
sudo make install

# 编译 fcitx5-flypy
sudo apt-get install -y gettext appstream libecm-dev libfcitx5core-dev libboost-dev libimecore-dev libimetable-dev
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
3. ~~使用 orq、ouj 无法输入当前日期和时间~~

![Windows 版码表](/img/posts/use-flypy-in-deepin/mabn.png)

如在使用过程中遇到其他问题，欢迎到 [github](https://github.com/kt286/fcitx5-flypy/issues) 中提 issue。

## 后记
1. deepin 20 仓库里虽然提供了 fcitx5，但是只提供了一些基础包。fcitx5-rime、fcitx5-lua 等都没有提供。希望后续可以补全。
2. 使用了一段时间小鹤音形后，输入人名拼音时会不自觉输入双拼。而且由于之前过于依赖拼音输入法，导致很多字只会读，不会写。打字速度反而明显降低。需要一段时间去适应。算是有得有失吧。

## 参考
[^1]: [小鹤入门](https://help.flypy.com/#/xh)
[^2]: [小鹤网盘](http://flypy.ys168.com)
