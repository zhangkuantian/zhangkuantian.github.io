---
author: Akiba Arisa
author_gh_user: zhanbao2000
read_time: 3 min
tags:
    - ida
title: IDA 替换二进制文件
---

最近在修改某个🇬🇧游戏，涉及到一些 IDA 在替换方面的使用，这里记录一下，免得每次都去麻烦群友问 IDA 怎么用。

## 文本搜索

``` mermaid
graph LR
  A[Search] --> B[text];
```

输入想要搜索的文本即可。

## 搜索字节（十六进制、十进制和八进制）

``` mermaid
graph LR
  A[Search] --> B[sequence of bytes];
```

然后选择 `Hex`、`Decimal` 或 `Octal`，输入你需要搜索的字节，每个字节使用空格分隔。

## Hex 替换

首先在 `IDA View` 选项卡中选择到你想要替换的内容所在的字节（以字符串为例），然后切换到 `Hex View` 选项卡（十六进制编辑器）。选择一个字节，你接下来要会从这里开始编辑。

!!! 如果没有这个选项卡

    ``` mermaid
    graph LR
      A[View] --> B;
      B[Open subviews] --> C[Hex dump];
    ```

    从这里打开 `Hex View` 选项卡。

有两种方式可以编辑：

1. 在十六进制编辑器中右键选择 `Edit`（或者按 ++f2++），然后直接编辑十六进制或者直接编辑文本内容，编辑完成后右键选择 `Apply Changes`（或者按 ++f2++）。

2. 选择 `Edit` -> `Patch Program` -> `Change Byte..`，在弹出的对话框中对应填写你要替换的十六进制值（一次性替换16个字节），然后点 `OK`。

## 搜索字符串

``` mermaid
graph LR
  A[View] --> B;
  B[Open subviews] --> C[Strings];
```

在出来的字符串列表中直接使用 ++ctrl+f++ 搜索即可。

## 生成伪代码

在 `IDA View` 选项卡中选择到你生成的函数，按 ++f5++ 生成伪代码。

## 保存二进制文件

``` mermaid
graph LR
  A[Edit] --> B;
  B[Patch Program] --> C[Apply patches to input file..];
```
从这里保存二进制文件。
