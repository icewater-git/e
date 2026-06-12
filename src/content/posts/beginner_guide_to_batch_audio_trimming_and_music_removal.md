---
title: 小白也能学会：批量剪辑音频，自动去除前后背景音乐
published: 2026-06-12
pinned: false
description: 　　本文使用 FFmpeg 的无损剪辑方式，不会重新压缩音频，因此处理速度快，也不会造成额外的音质损失。
tags: []
category: 
licenseName: "Unlicensed"
author: 冰水之源
sourceLink: ""
draft: false
image: 'https://m.wbiao.cn/mallapi/wechat/picReverseUrl?url=https://mmbiz.qpic.cn/mmbiz_png/mngWTkJEOYJOwD4NQEWPibZAbfmGiapC1DzdAw4DSOnHxnkATGVRQn2yy8mQRfpZAmuf4n0Xc3qmmVG0ibjfgb4Sg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=8'
---

> ✨转载请注明出处

　　平时在网上下载讲座时，尤其是一些佛学讲座、传统文化课程、有声书或者音频开示，经常会遇到一个问题：音频的开头和结尾都会附带一段背景音乐。

　　刚开始听的时候还好，但如果习惯晚上睡觉时连续播放音频，问题就来了。讲座结束后，突然响起音量较大的片尾音乐；下一段音频开始前，又播放一段开场音乐。反反复复，不仅影响睡眠，也容易让人从专注听讲的状态中被打断。

　　如果只是处理一两个音频，手动剪辑并不麻烦。但如果有几十个、上百个音频都存在同样的问题，一个个打开软件去操作，就会变得十分耗时。

　　今天介绍一种适合新手的方法：利用 Python 和 FFmpeg，一次性批量处理整个文件夹中的音频，自动删除开头和结尾的背景音乐。

![](https://m.wbiao.cn/mallapi/wechat/picReverseUrl?url=https://mmbiz.qpic.cn/mmbiz_jpg/mngWTkJEOYJOwD4NQEWPibZAbfmGiapC1D3sRvnqYjh7BnWsuB1Ge3RKNPMZYJ0Gu9Blhovia53calORjibBOtyiaMw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

### 实现效果

　　假设每个音频的结构如下：

　　原音频：

　　【开场音乐 30 秒】+【讲座内容】+【结束音乐 60 秒】

　　处理后：

　　【讲座内容】

　　如果一个文件夹中有 100 个音频，程序会自动全部处理，无需逐个操作。

---

### 第一步：安装 Python

　　首先安装 Python。

　　打开官方网站：https://www.python.org/

　　下载安装最新版即可。

![](https://m.wbiao.cn/mallapi/wechat/picReverseUrl?url=https://mmbiz.qpic.cn/mmbiz_jpg/mngWTkJEOYJOwD4NQEWPibZAbfmGiapC1DHumufT8SgpjEOjTpzJGNeeuEYUgjOnib60eiaukAicoLDPicjLWcB9nUGg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

　　安装时一定要勾选：“Add Python to PATH”，这样后续在命令行中才能直接使用 Python。

　　安装完成后，打开命令提示符，输入：

```bash
python --version
```

　　如果能够显示版本号，说明安装成功。

---

### 第二步：安装 FFmpeg

　　FFmpeg 是一个非常强大的音视频处理工具。

　　下载地址：https://ffmpeg.org/download.html

　　下载后解压。

![](https://m.wbiao.cn/mallapi/wechat/picReverseUrl?url=https://mmbiz.qpic.cn/mmbiz_jpg/mngWTkJEOYJOwD4NQEWPibZAbfmGiapC1DAUX48phuNGtIRgaWMlcClgxapBAeW6MbjibrthbeqERHMgN1A4iaibC1g/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

　　找到：

```text
ffmpeg.exe
ffprobe.exe
```

　　将其所在目录添加到系统环境变量 PATH 中。

　　配置完成后，打开命令提示符输入：

```bash
ffmpeg -version
```

　　以及：

```bash
ffprobe -version
```

　　如果都能显示版本信息，则安装成功。

---

### 第三步：准备音频文件

　　例如建立一个文件夹：

```text
讲座音频
├─ 音频1.mp3
├─ 音频2.mp3
├─ 音频3.mp3
├─ 音频4.mp3
└─ ...
```

　　将所有需要处理的 MP3 文件放进去。

---

### 第四步：创建 Python 脚本

　　在该文件夹中，新建一个文件：

```text
trim_mp3.py
```

　　将下面代码复制进去：

```python
import os
import subprocess

head_cut = float(input("请输入删除前几秒："))
tail_cut = float(input("请输入删除后几秒："))

output_dir = "output"
os.makedirs(output_dir, exist_ok=True)

def get_duration(file):
    result = subprocess.run(
        [
            "ffprobe",
            "-v", "error",
            "-show_entries", "format=duration",
            "-of", "default=noprint_wrappers=1:nokey=1",
            file
        ],
        capture_output=True,
        text=True
    )
    return float(result.stdout.strip())

for filename in os.listdir("."):
    if filename.lower().endswith(".mp3"):
        try:
            duration = get_duration(filename)

            if duration <= head_cut + tail_cut:
                print(f"跳过：{filename}")
                continue

            keep_duration = duration - head_cut - tail_cut

            output_file = os.path.join(output_dir, filename)

            cmd = [
                "ffmpeg",
                "-y",
                "-ss", str(head_cut),
                "-i", filename,
                "-t", str(keep_duration),
                "-c", "copy",
                output_file
            ]

            subprocess.run(
                cmd,
                stdout=subprocess.DEVNULL,
                stderr=subprocess.DEVNULL
            )

            print(f"完成：{filename}")

        except Exception as e:
            print(f"失败：{filename} -> {e}")

print("全部处理完成！")
```

---

### 第五步：运行脚本

　　在当前文件夹地址栏输入：

```text
cmd
```

　　回车打开命令窗口。

　　然后输入：

```bash
python trim_mp3.py
```

　　程序会提示：

```text
请输入删除前几秒：
```

　　例如输入：

```text
30
```

　　接着提示：

```text
请输入删除后几秒：
```

　　例如输入：

```text
60
```

　　此时程序便会自动处理当前文件夹中的所有 MP3 文件。

---

### 处理完成后在哪里查看？

　　程序运行结束后，会自动生成一个新的文件夹：

```text
output
```

　　结构如下：

```text
讲座音频
├─ 音频1.mp3
├─ 音频2.mp3
├─ 音频3.mp3
├─ trim_mp3.py
└─ output
    ├─ 音频1.mp3
    ├─ 音频2.mp3
    └─ 音频3.mp3
```

　　原文件不会被修改。

　　处理后的文件全部保存在 output 文件夹中。

---

### 为什么选择这种方法？

　　对于大量音频来说，批量处理比手工剪辑效率高得多。

　　例如：

* 佛学讲座
* 传统文化课程
* 有声书
* 念佛录音
* 网络课程
* 播客节目

　　很多资源都会统一加入片头和片尾音乐。

![](https://m.wbiao.cn/mallapi/wechat/picReverseUrl?url=https://mmbiz.qpic.cn/mmbiz_jpg/mngWTkJEOYJOwD4NQEWPibZAbfmGiapC1Dlk6BDTmSTiaVIj353msiaWWzLGGSpEhicNFtva9Eg56zS5nicGiaBXrSXFA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=7)

　　如果有几十个甚至上百个文件，手动操作可能需要数小时，而脚本只需运行一次即可全部完成。

　　更重要的是，本文使用 FFmpeg 的无损剪辑方式，不会重新压缩音频，因此处理速度快，也不会造成额外的音质损失。

　　对于经常整理讲座资料、制作个人学习音频库的朋友来说，这是一种简单而实用的方法。
