---
title: Hexo使用APlayer插入音乐
date: 2019-06-26 01:13:28
categories: HEXO
tags: hexo

---
<blockquote class="blockquote-center">带着感恩的心启程，学会爱，爱父母，爱自己，爱朋友，爱他人。</blockquote>


<div class="aplayer" data-id="108740" data-server="netease" data-type="song" data-mode="single"></div>


## 背景:

以前一直使用网易云音乐去生成外链播放器,但因为好多的歌存在着版权问题,导致每次找背景歌曲都非常麻烦~无意在网上看到APlayer,Aplayer是一个html5的嵌入式播放器,将其用在博客中去插入音乐链接,非常的好用.

**APlayer支持：**

- 媒体格式
  - MP4 H.264（AAC或MP3）
  - WAVE PCM
  - Ogg Theora Vorbis
- 特征
  - 播放列表
  - 歌词

## 开始安装:

我用的是 next 主题，这里直接使用官网提供的[CDN](https://aplayer.js.org/docs/#/?id=cdn)进行引入APlayer。也可以使用github的方式去安装<https://github.com/MoePlayer/hexo-tag-aplayer/blob/master/docs/README-zh_cn.md>

我们去编辑 `/themes/next/layout/_partials/` 目录下的 `header.swig`，引入 Aplayer.js加入以下三行代码,插入到文件最后面即可

```
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer@1.7.0/dist/APlayer.min.css">
```

```
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.7.0/dist/APlayer.min.js"></script>
```

```
<script src="https://cdn.jsdelivr.net/npm/meting@1.1.0/dist/Meting.min.js"></script>
```

## 如何使用

使用方法很简单， 在markdown格式的博文中，在需要插入音乐的地方加入以下`div`即可：

```
<div class="aplayer" data-id="108740" data-server="netease" data-type="song" data-mode="single"></div>
```

## 效果图

<div class="aplayer" data-id="108740" data-server="netease" data-type="song" data-mode="single"></div>

## 常用参数

| 主要参数      |                              值                              |
| :------------ | :----------------------------------------------------------: |
| data-id       |                      歌曲/专辑/歌单 ID                       |
| data-server   | netease（网易云音乐）tencent（QQ音乐） xiami（虾米） kugou（酷狗） |
| data-type     | song （单曲） album （专辑） playlist （歌单） search （搜索） |
| data-mode     | random （随机） single （单曲） circulation （列表循环） order （列表） |
| data-autoplay |              false（手动播放） true（自动播放）              |

更多的参数可参考[官方文档](https://aplayer.js.org/#/zh-Hans/?id=%E5%8F%82%E6%95%B0)

