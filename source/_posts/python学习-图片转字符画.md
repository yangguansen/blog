---
title: python学习-图片转字符画
date: 2017-10-14 10:30:29
tags:
	- python
---

> 前言

最近在学习python，惊讶于其处理数据方面的方便，以及目前机器学习潮流python占了不可动摇的地位，未来也是一股潮流，于是在熟悉python基本语法之后，决定先从小工具做起，慢慢练习，熟练掌握python的用法，目前觉得python在做一些日常小工具上还是挺方便的，这样对于自己来说，也不会只局限于“前端”，要往“软件工程师”方向发展。


> 原理

整个代码量不多，其中原理便是：颜色的色值区间为0-255，使用PIL的image模块读取图片的颜色值，例如`（255,255,255，1）`,前三个数表示颜色值，第四个表示透明度。我们创建一个字符串数组，例如

```list("$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~<>i!lI;:,\"^`'.")```

把读取到的色值按从小到大取数组中的字符串，然后拼凑字符串，写入文件。把图片的像素转为色值的核心算法在于：
```gray ＝ 0.2126 * r + 0.7152 * g + 0.0722 * b```

我们需要获取在shell中输入的图片名，所以用`argparse`包来获取shell中的输入的变量。

以下是全部代码：
```
from PIL import Image
import argparse

# 灰度值小（暗）的用列表开头的符号，灰度值大（亮）的用列表末尾的符号。
# gray ＝ 0.2126 * r + 0.7152 * g + 0.0722 * b

ascii_char = list("$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~<>i!lI;:,\"^`'. ")
length = len(ascii_char)

# 颜色色值  0-255
MaxValue = 255

def get_char(r,g,b,alpha=256):
    if alpha == 0:
        return ' '

    gray = 0.2126 * r + 0.7152 * g + 0.0722 * b
    return ascii_char[int(gray / MaxValue * length )]

parser = argparse.ArgumentParser()

parser.add_argument('file')
parser.add_argument('--output')
parser.add_argument('--width', type = int, default=80)
parser.add_argument('--height', type = int, default = 80)

args = parser.parse_args()

IMG = args.file
WIDTH = args.width
HEIGHT = args.height
OUTPUT = args.output or 'output.txt'


if __name__ == '__main__':

    im = Image.open(IMG)
    im = im.resize((WIDTH, HEIGHT), Image.NEAREST)

    txt = ""

    for i in range(HEIGHT):
        for j in range(WIDTH):
            txt += get_char(*im.getpixel((j, i)))
        txt += '\n'

    print(txt)

    with open(OUTPUT, 'w') as f:
        f.write(txt)

```


（完）