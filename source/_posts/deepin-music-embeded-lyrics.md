---
title: 为深度音乐添加内嵌歌词支持
date: 2022-01-30 09:09:09
updated: 2022-01-30 12:27:09
categories: Deepin
tags: [deepin,music,c++]
---

## 前言
一个风和日丽的晚上，结束了一天的紧张工作（摸鱼）后打算在睡前听一会网抑云音乐。刚要打开播放器的时候突然想到之前看到的深度音乐的截图，决定试用一下。

将下载好的音乐导入深度音乐后点击播放，兴冲冲的打开歌词显示，大大的“没有找到歌词”映入眼帘
![没有找到歌词](/img/posts/deepin-music-embeded-lyrics/1.png)

不对呀，我记得我下载歌曲的时候明明勾选了嵌入歌词的。掏出祖传的 `ffprobe` 查看了一下，发现确实是有歌词的。看来是深度音乐不支持了。与其慢慢等官方实现，不如自己动手、丰衣足食。
![ffprobe 结果](/img/posts/deepin-music-embeded-lyrics/2.png)

## 思路
既然知道歌词已经写到歌曲中了。就先看看写到了歌曲的什么位置，打开 [洛雪音乐助手](https://github.com/lyswhut/lx-music-desktop) 的源码，全局搜索 `lyrics`，找到如下代码：

```javascript
// src/main/utils/mp3Meta.js

const handleWriteMeta = (meta, filePath) => {
  if (meta.lyrics) {
    meta.unsynchronisedLyrics = {
      language: 'zho',
      text: meta.lyrics,
    }
    delete meta.lyrics
  }
  NodeID3.write(meta, filePath)
}
```
根据代码我们可以知道歌词被写入到 ID3 标签中的 `unsynchronisedLyrics` 中。Google 了一下，发现这个是 ID3v2 标准专门用来存储文本歌词[^1]的，而洛雪音乐助手写入的是 lrc 格式的歌词，所以我们只需要读取这个属性中内容，直接提供给深度音乐显示就可以了。

## 开发
歌词的提取我们可以参考深度音乐对歌曲封面图的处理方法：添加歌曲后开始读取歌词，读取到歌词后，写入缓存文件夹里的 lyrics 文件夹。播放时先使用原来的逻辑获取歌曲同目录下同名歌词文件，如果没找到就去缓存文件夹里 lyrics 文件夹中查找歌词文件，如果还没找到再显示“没有找到歌词”。

核心代码如下，代码参考了 Stack Overflow 的一个回答[^2] 和 TagLib 文档[^3]
```cpp
// src/libdmusic/metadetector.cpp

#include <taglib/mpegfile.h>
#include <taglib/id3v2tag.h>
#include <taglib/id3v2frame.h>
#include <taglib/unsynchronizedlyricsframe.h>

// 获取歌词数据，写到缓存目录
void MetaDetector::getLyricData(const QString &path, const QString &tmpPath, const QString &hash)
{
    QString lyricsDirPath = tmpPath + "/lyrics";
    QString lyricName = hash + ".lrc";
    QDir lyricsDir(lyricsDirPath);
    if (!lyricsDir.exists()) {
        bool isExists = lyricsDir.cdUp();
        isExists &= lyricsDir.mkdir("lyrics");
        isExists &= lyricsDir.cd("lyrics");
    }

    if (!path.isEmpty() && !tmpPath.isEmpty() && !hash.isEmpty()) {
        if (!lyricsDir.exists(lyricName)) {
            QFile lyric(lyricsDirPath + "/" + lyricName);

            TagLib::MPEG::File f1(path.toStdString().c_str());
            TagLib::ID3v2::FrameList frames = f1.ID3v2Tag()->frameListMap()["USLT"];
            if (!frames.isEmpty()) {
                TagLib::ID3v2::UnsynchronizedLyricsFrame *frame = dynamic_cast<TagLib::ID3v2::UnsynchronizedLyricsFrame *>(frames.front());
                if (frame) {
                    if (lyric.open(QIODevice::WriteOnly)) {
                        QString str = TStringToQString(frame->text());
                        lyric.write(str.toUtf8());
                    }
                    lyric.close();
                }
            }   
        }
    }

    return;
}

```

```cpp
// src/music-player/widget/lrc/musiclyricwidget.cpp

void MusicLyricWidget::onMusicPlayed(MediaMeta meta)
{
    Q_UNUSED(meta)
    QFileInfo fileInfo(Player::getInstance()->getActiveMeta().localPath);
    QString lrcPath = fileInfo.dir().path() + QDir::separator() + fileInfo.completeBaseName() + ".lrc";
    QFile file(lrcPath);
    if (!file.exists()) {
        // 读取根目录中歌词文件失败后读取媒体文件内嵌的歌词
        MediaMeta meta = Player::getInstance()->getActiveMeta();
        QFileInfo lyricInfo(Global::cacheDir() + "/lyrics/" + meta.hash + ".lrc");
        if (!lyricInfo.exists()) {
            m_nolyric->show();
            m_lyricview->hide();
        } else {
            m_nolyric->hide();
            m_lyricview->show();
            m_lyricview->getFromFile(lyricInfo.filePath());
        }
    } else {
        m_nolyric->hide();
        m_lyricview->show();
        m_lyricview->getFromFile(lrcPath);
    }
}
```

完整代码见：[https://github.com/linuxdeepin/deepin-music/pull/111/](https://github.com/linuxdeepin/deepin-music/pull/111/)

![效果图](/img/posts/deepin-music-embeded-lyrics/3.png)

## 后记
本次修改只实现了对 ID3v2 标签中非同步歌词[^1]的读取。至于另一种同步歌词[^4]，手头上没有测试文件，就暂时搁置，以后有时间再来实现。


## 参考
[^1]: [Unsychronised_lyrics_text_transcription - id3v2.3.0 - ID3.org](https://id3.org/id3v2.3.0#Unsychronised_lyrics.2Ftext_transcription)
[^2]: [macos - How to get the lyrics of a song from the song file - Stack Overflow](https://stackoverflow.com/questions/11051055/how-to-get-the-lyrics-of-a-song-from-the-song-file)
[^3]: [TagLib API Documentation](https://taglib.org/api/namespaceTagLib_1_1ID3v2.html)
[^4]: [Synchronised_lyrics_text - id3v2.3.0 - ID3.org](https://id3.org/id3v2.3.0#Synchronised_lyrics.2Ftext)