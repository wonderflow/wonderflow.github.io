---
title: "使用 Goland 设置 cuelang 的自动格式化"
date: 2020-11-02T17:39:57+08:00
---

Goland 可以方便的通过自定义 file watchers 设置自动化 format culang 文件。

设置方法：

【Preferences】 => 【File Watchers】

![undefined](/images/2020-11-01-cue/cue1.jpg)

添加：

![undefined](/images/2020-11-01-cue/2.jpg)

1. File type 选择 Any
2. Scope Pattern设置：  `file[*]:*.cue`  。
3. Program 是 cue，这也要求你的 PATH 环境下有 cue 这个可执行文件。
4. Arguments参数设置： `fmt $FilePath$`
5. 输出文件： `$FilePath$`
6. 工作目录： `$ProjectFileDir$`

然后点击 `apply` 生效就好了。