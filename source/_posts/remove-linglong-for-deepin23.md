---
title: Deepin V23 去除玲珑计划
date: 2024-01-04 17:04:52
updated: 2024-01-04 17:04:52
categories: Deepin
tags: [deepin,c++]
---

使用了一段时间 V23 版本，饱受玲珑应用的毒害，很多原来功能正常的应用上了玲珑就各种小问题，但是仓库里不提供 deb 版，所以打算在玲珑应用彻底稳定之前，先去除玲珑这个毒瘤。

目前本人使用的软件有音乐、影院、归档管理器、看图、邮箱、截图录屏、文本编辑器、文档查看器，本计划只针对这几个软件。

### 音乐

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-music.git --depth=1
cd deepin-music
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.music
# 安装 deb 版
sudo apt install ../deepin-music_7.0.3_amd64.deb
```

### 文本编辑器

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-editor.git --depth=1
cd deepin-editor
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.editor
# 安装 deb 版
sudo apt install ../deepin-editor_6.0.15_amd64.deb
```

### 影院

影院需要修改 control 文件中依赖版本

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-movie-reborn.git --depth=1
cd deepin-movie-reborn
# 修改依赖版本
# 修改 deepin-movie-reborn/debian/control 中 libavcodec60、libavformat60、libavutil58 为 libavcodec58、libavformat58、libavutil56
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.movie
# 安装 deb 版（xxx为版本号）
sudo apt install ../deepin-movie_xxx.deb ../libdmr_xxx.deb
```

### 看图

看图需要额外编译一下依赖包

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-ocr-plugin-manager.git --depth=1
cd deepin-ocr-plugin-manager
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 安装
sudo apt install ../libdeepin-ocr-plugin-manager-dev_1.0.0_amd64.deb ../libdeepin-ocr-plugin-manager_1.0.0_amd64.deb


# 下载源码
git clone https://github.com/linuxdeepin/deepin-image-viewer.git --depth=1
cd deepin-image-viewer
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.image.viewer
# 安装 deb 版
sudo apt install ../deepin-image-viewer_6.0.3_amd64.deb
# 删除玲珑中目目录
sudo rm -rf /persistent/linglong/entries/share/deepin-ocr-plugin-manager
```

### 文档查看器

文档查看器编译时测试可能报错，这里先去掉测试

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-reader.git --depth=1
cd deepin-reader
# 安装依赖
sudo apt build-dep .

# 移除测试代码
# 打开 deepin-reader/tests/tests.pro 文件，找到以下代码，删除 main.cpp \ 后边的内容
# SOURCES += \
#     main.cpp \
# ============下边开始删除===========
#     ut_mainwindow.cpp \
#     ut_common.cpp \
#     ut_application.cpp \
#     ............
#     uiframe/ut_centraldocpage.cpp \
#     uiframe/ut_docsheet.cpp \
#     uiframe/ut_doctabbar.cpp
# ============删除到这里===========


# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.reader
# 安装 deb 版
sudo apt install ../deepin-reader_6.0.7_amd64.deb
```

### 归档管理器

归档管理器需要修改部分代码

```bash
# 下载源码
git clone https://github.com/linuxdeepin/deepin-compressor.git --depth=1
cd deepin-compressor
# 安装依赖
sudo apt build-dep .

# 修改代码，参考下边链接
# https://github.com/linuxdeepin/deepin-compressor/pull/156/files

# 编译
dpkg-buildpackage -b -us -uc
# 卸载玲珑版
ll-cli uninstall org.deepin.compressor
# 安装 deb 版
sudo apt install ../deepin-compressor_6.0.0_amd64.deb
```

### 截图录屏

这个比较特殊，opencv 安装时有依赖问题，需要重新编译
```bash
# 下载源码
apt source libopencv-dev
cd opencv_4.5.4+dfsg.1-1+dde
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 安装
sudo apt install ../libopencv-imgcodecs4.5_4.5.4+dfsg.1-1+dde_amd64.deb


# 下载源码
git clone https://github.com/linuxdeepin/deepin-screen-recorder.git --depth=1
cd deepin-screen-recorder
# 安装依赖
sudo apt build-dep .
# 编译
dpkg-buildpackage -b -us -uc
# 有可能需要将 deepin-screen-recorder/src/Makefile 中 std=gnu-11 替换为 std=gnu-14 (共发现两处)
dpkg-buildpackage -b -us -uc -nc
# 卸载玲珑版
ll-cli uninstall org.deepin.screen-recorder
# 安装 deb 版
sudo apt install ../deepin-screen-recorder_6.0.1_amd64.deb
```

### 邮箱

邮箱没有源码，需要复制文件到指定目录
```bash
sudo cp -r /persistent/linglong/layers/org.deepin.mail/5.4.28/x86_64/files/bin/deepin-mail /usr/bin/deepin-mail
sudo cp -r /persistent/linglong/layers/org.deepin.mail/5.4.28/x86_64/files/share/applications/* /usr/share/applications/
sudo cp -r /persistent/linglong/layers/org.deepin.mail/5.4.28/x86_64/files/share/deepin-mail/* /usr/share/deepin-mail/
sudo cp -r /persistent/linglong/layers/org.deepin.mail/5.4.28/x86_64/entries/deepin-manual/* /usr/share/deepin-manual/ 
sudo cp -r /persistent/linglong/layers/org.deepin.mail/5.4.28/x86_64/entries/icons/* /usr/share/icons/

cp -r ~/.linglong/org.deepin.mail/share/deepin/deepin-mail/* ~/.local/share/deepin/deepin-mail/
```

### 最后卸载玲珑
```bash
sudo apt purge linglong*
sudo rm -rf /persistent
sudo rm -rf ~/.linglong
```