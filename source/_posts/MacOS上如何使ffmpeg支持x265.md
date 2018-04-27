---
title: MacOS上如何使ffmpeg支持x265
date: 2018-04-27 21:50:56
tags: [MacOS, ffmpeg, x265]
---


使用brew install ffmpeg 安装ffmpeg默认是没有支持x265的， 使用brew info ffmpeg 获取安装选项帮助，

使用brew reinstall ffmpeg --with-x265 重新安装即可。

 

[参考](http://trac.ffmpeg.org/wiki/CompilationGuide/MacOSX)
