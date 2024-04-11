---
title: ffmpeg实用命令
date: 2024-04-03 02:25:37
tags:
---
## 介绍
ffmpeg是一个`媒体转换器`，可以把媒体源文件转换成多种格式。

## 转码
ffmpeg用`-i`参数读取源文件，比如读取input.avi，转码输出output.mp4：

> ffmpeg -i input.avi output.mp4

这种方式转码速度较慢，因为会对源文件重新编码。

ts转MP4：
> ffmpeg -i input.ts -c copy -bsf:a aac_adtstoasc output.mp4

`-c`参数指定`copy`的意思是流复制，这样省去了重新编码的时间，格式转换将十分迅速。

`-bsf:a`参数指定`aac_adtstoasc`的意思是，指定`aac_adtstoasc`过滤器，
该过滤器创建 MP4 音频配置， `-bsf`表示选择过滤器，冒号后面的a表示audio。
输入`ffmpeg -bsfs`命令会列出所有可用过滤器。

## 截取视频
截取从00：3：00开始到结尾的视频
> ffmpeg -ss 00:03:00 -i video.mp4 -c copy pieces.mp4

截取指定时间，从00：3：00到00：20：00
> ffmpeg -ss 00:03:00 -i video.mp4 -to 00:20:00 -c copy pieces.mp4

## 合并视频
1.写入first file.mp4和second file.mp4到list.txt中
> (echo file 'first file.mp4' & echo file 'second file.mp4' )>list.txt

2.以list.txt作为输入源
> ffmpeg -safe 0 -f concat -i list.txt -c copy output.mp4
