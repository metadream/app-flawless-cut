# Flick-Cut

开发 Flick-Cut 的起因是想改善另一个无损分割工具
[LosslessCut](https://github.com/mifi/lossless-cut) 的用户界面，该项目的 Issue
中也曾提到新 UI 的计划，但始终未见升级。适逢近日正学习
Electron，便以此练手，开发了这款与之功能类似的应用。与 LosslessCut
相比，Flick-Cut
除了界面优化外，也去除了一些不常用的功能以及允许用户选择的设置，以保持简洁易用。尽管
Flick-Cut 在代码方面几乎完全重写，但依然借鉴了 LosslessCut
作者的诸多实践，在此表示感谢。

![软件界面](https://raw.githubusercontent.com/metadream/app-flick-cut/master/screenshot.png)

## 主要功能

- 无损切割常见格式的视频和音频（快速）
- 无损合并相同编码格式的视频片段（快速）
- 有损切割视频和音频并自动转为 MP4 格式（慢速）
- 有损切割提取音频并自动转为 MP3 格式（慢速）
- 以最小文件最高质量将视频帧截取为图片
- 录制屏幕及麦克风输入
- 音频声波可视化
- 支持 Windows/Linux 平台

## 快捷键

| 符号  | 键名   | 功能      |
| ----- | ------ | --------- |
| SPACE | 空格键 | 播放/暂停 |
| ->    | 右箭头 | 前进一秒  |
| <-    | 左箭头 | 后退一秒  |

## 支持格式

由于 Flick-Cut 基于 Chromium 核心和 HTML5 视频播放器，因此并非所有 ffmpeg
支持的格式都将直接受支持。为更加快速流畅地使用本软件，通常应导入以下格式/编解码器：MP4，MOV，WebM，MKV，OGG，WAV，MP3，AAC，H264，Theora，VP8，VP9。有关
Chromium 受支持的格式/编解码器的更多信息，请参见
https://www.chromium.org/audio-video。

对于 Chromium 不支持的格式，本软件采用快速实时转码播放技术，即允许播放 ffmpeg
能够解码的所有视频文件，而实际分割操作的依然是无损原始文件。但遗憾的是，尤其遇到大视频文件，该方法在效率上（准确地说是寻轨流畅性方面）还是和原生支持格式不可同日而语。

## 开发构建

### 1. 安装 NODE & NPM

请自行安装。

### 2. 安装 Electron 及其他依赖

推荐的安装方法是把它作为项目中的开发依赖项，以便能在不同的 app 中使用不同的
Electron 版本。 在项目所在目录中运行下面的命令（使用淘宝 Electron 镜像）：

```
export ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/"
npm install
```

### 3. 调试运行

```
npm start
```

### 4. 构建打包

```
npm run build:linux
npm run build:win
```

## 遇过的坑

- 打包成 asar 档案后，fluent-ffmpeg 无法运行档案中的 exe 命令（即
  ffmpeg.exe），查遍相关资料发现，fluent-ffmpeg 中启动子进程使用的 `spawn`
  方法无法读取 asar 档案中的二进制文件，但 `execFile` 方式可以，于是弃用
  fluent-ffmpeg 模块，自己改写 ffmpeg.js。

- 在 ffmpeg 子进程通过 pipe 管道输出视频流的情况下，必须设置
  `encoding: 'buffer'` 选项，且必须指定足够大的 `maxBuffer`
  数值（通常是视频文件大小），否则客户端无法接收到数据。

- 用 ffmpeg 设置录屏或录音设备时，几乎所有网上资料都显示参数形如
  `audio="microphone device"`，该形式可通过命令行执行成功，但在代码中报错“找不到设备”，经无数次查找和尝试（开始以为是设备名的编码格式问题），最后终于在仅有的一篇文章中找到答案：去掉双引号。

## 已知问题

- 在没有找到更好的方法之前，无损切割视频的时间准确性是个很大的问题（重编码有损切割则不存在），即切割后的视频的起止时间点可能和预期的并不完全一致，这时就需要进行一次选择：是否使用关键帧切割。如果不用，时间会相对准确，但视频的前后几秒可能是空白；如果用，则切割后的视频时长会相差数秒甚至9秒。两害相权取其轻，本软件选择按关键帧切割。或许会有其他折衷的办法解决，例如仅仅对关键帧以外的部分重新编码，但终究比较复杂，暂时没时间研究。

- 搞不清楚什么类型的编码可以直接 copy 成 mp4，在增加 `-vcodec copy -acodec copy`
  之后命令会报错，若能解决的话，可以根据不同类型选择重新编码还是复制编码生成
  mp4，以节约转码时间。

- 由于并未实施严格测试，存在的问题亟待发现。
