---
title: 深度文件管理器改造小记
date: 2022-05-19 16:22:24
updated: 2022-05-19 16:22:24
categories: Deepin
tags: deepin
---

## 前言
使用深度文件管理器的时候，遇到一些小问题。翻了翻代码，发现修改起来不是很复杂，就顺手改了。下面记录一下修改内容。本次修改基于最新内测版5.6.3。

## 环境准备
```bash
apt source dde-file-manager
cd dde-file-manager-5.6.3
sudo apt build-dep .
```

## 修改内容
环境配置好后就可以修改了，本次修改的内容如下：

### 1.列表模式显示太密集
这个问题很简单，就是改一下列表项高度。

```diff
# src/dde-file-manager-lib/interfaces/dlistitemdelegate.cpp

void DListItemDelegate::updateItemSizeHint()
{
    Q_D(DListItemDelegate);

    d->textLineHeight = parent()->parent()->fontMetrics().lineSpacing();
-    d->itemSizeHint = QSize(-1, qMax(int(parent()->parent()->iconSize().height() * 1.1), d->textLineHeight));
+    d->itemSizeHint = QSize(-1, qMax(int(parent()->parent()->iconSize().height() * 1.5), d->textLineHeight));
}
```
![修改前](/img/posts/dde-file-manager-beautify/1.png)
![修改后](/img/posts/dde-file-manager-beautify/2.png)

### 2.浅色主题下，计算机里我的目录中文件夹选中后文字看不清楚
这个问题比较有意思，文字高亮颜色为白色，浅色主题中看不清楚，深色主题反而效果很好。这里有两种修改思路
1. 判断一下使用的主题，分别设置不同的文字高亮颜色
2. 去掉文字高亮
为了和其他地方保持一致，这里选择方案2

```diff
# src/dde-file-manager-lib/views/computerviewitemdelegate.cpp

void ComputerViewItemDelegate::paint(QPainter *painter, const QStyleOptionViewItem &option, const QModelIndex &index) const
{

# 省略部分代码

        painter->setFont(par->font());
-        painter->setPen(qApp->palette().color((option.state & QStyle::StateFlag::State_Selected) ? QPalette::ColorRole::HighlightedText : QPalette::ColorRole::Text));
+        painter->setPen(qApp->palette().color(QPalette::ColorRole::Text));
        painter->drawText(option.rect.x() + (option.rect.width() - fstw) / 2, option.rect.y() + topmargin + iconsize + text_topmargin, elided_text);
        return;

# 省略部分代码
    
}
```
![修改前](/img/posts/dde-file-manager-beautify/3.png)
![修改后](/img/posts/dde-file-manager-beautify/4.png)

### 3.开启保险箱功能
保险箱功能实际上代码里有，只是判断了仅在 UOS 专业版中开放，我们只需要去掉这些判断即可。
```diff
# src/dde-file-manager-lib/vault/vaulthelper.cpp

bool VaultHelper::isVaultEnabled()
{
+    return true;
-    if (!VaultController::ins()->isVaultVisiable())     // 判断域管策略,是否显示保险箱
-        return false;
-
-    // 获取系统类型
-    DSysInfo::UosType uosType = DSysInfo::uosType();
-    // 获取系统版本
-    DSysInfo::UosEdition uosEdition = DSysInfo::uosEditionType();
-    if (DSysInfo::UosServer == uosType) {   // 如果是服务器类型
-        // 如果是企业版/行业版/欧拉版
-        if (DSysInfo::UosEnterprise == uosEdition
-                || DSysInfo::UosEnterpriseC == uosEdition
-                || DSysInfo::UosEuler == uosEdition) {
-            return true;
-        }
-    } else if (DSysInfo::UosDesktop == uosType) {   // 如果是桌面类型
-        // 如果是专业版
-        if (DSysInfo::UosProfessional == uosEdition) {
-            return true;
-        }
-    }
-    return false;
}
```
![修改后](/img/posts/dde-file-manager-beautify/5.png)

## 编译安装
```bash
mkdir build
cd build
qmake ../filemanager.pro
make -j4
sudo cp src/dde-file-manager-lib/libdde-file-manager.so.1.8.2 /usr/lib/x86_64-linux-gnu/
```

## 后记
文件管理器还有另一个问题：文件夹中文件数量过多时，需要大量时间才可以全部加载完成。能力有限，就等待官方修改吧。