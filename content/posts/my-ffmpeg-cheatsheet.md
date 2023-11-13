---
title: "My FFmpeg Cheatsheet"
date: 2023-11-05T15:45:31+08:00
draft: false
---

# ffmpeg

## 格式转换

```
ffmpeg -i A.mkv A.mp4
```
## 复制流

```
ffmpeg -i input.mkv -codec copy output.mp4
```
## 视频转换音频

```
$ ffmpeg -i .\Revenge.webm demo.mp3 #将 Revenge.webm 这个 MV 转换成了 mp3 歌曲
```

## 抽取音频命令
```
ffmpeg -i 3.mp4 -vn -y -acodec copy 3.aac
ffmpeg -i 3.mp4 -vn -y -acodec copy 3.m4a
```

## 剪切
```
ffmpeg -ss 00:00:00 -t 00:00:30 -i test.mp4 -vcodec copy -acodec copy output.mp4
* -ss 指定从什么时间开始（hh:mm:ss）
* -t 指定需要截取多长时间
* -i 指定输入文件
```

## 合并（非 MPEG-1, MPEG-2 PS, DV)

Use this method when you want to avoid a re-encode and your format does not support file-level concatenation (most files used by general users do not support file-level concatenation).

```
$ cat mylist.txt
file '/path/to/file1'
file '/path/to/file2'
file '/path/to/file3'
    
$ ffmpeg -f concat -safe 0 -i mylist.txt -c copy output.mp4
```

For Windows:

```
(echo file 'first file.mp4' & echo file 'second file.mp4' )>list.txt
ffmpeg -safe 0 -f concat -i list.txt -c copy output.mp4
```

## 合并 (MPEG-1, MPEG-2 PS, DV)

Use this method with formats that support file-level concatenation (MPEG-1, MPEG-2 PS, DV). Do not use with MP4.

```
ffmpeg -i "concat:input1|input2" -codec copy output.mkv
```

This method does not work for many formats, including MP4, due to the nature of these formats and the simplistic concatenation performed by this method.

```
ffmpeg -i input.flv -q:a 5 out.mp3
```
flv 提取mp3

## merge m4s video and audio
```
ffmpeg -i video.m4s -i audio.m4s -c:v copy -c:a copy output.mp4
```

## yuv录屏
```
ffmpeg -f gdigrab -t 15 -framerate 30 -i desktop -pix_fmt yuv420p -vcodec rawvideo -s 1920x1080 -y output.yuv
```