---
title: "scrot使用imlib2的截图工具"
date: 2018-01-01 00:00:00 +0800
categories: [Blog, Example]
tags: [example]
comments: true
toc: true
---

scrot使用imlib2的幕截图工具.
用xlib和imlib2,而imlib2 还包含作者自己的工具库giblib.

giblib is a simple library which wraps imlib2. It provides a wrapper to imlib2's context API，avoiding all the context_get/set calls, adds fontstyles to the truetype renderer and supplies a generic doubly-linked list and some string functions

#### main.c主逻辑分析
初始化xlib与imlib配置
结构主要分为解析options部分,imlib图形使用,和主要逻辑,
作者使用全局opt变量用来保存全局配置,方便其他函数调用修改.
代码根据程序执行时的配置参数执行不同的逻辑分支.
通过image镜像生成图片文件保存

虽然程序参数选择中含有多窗口的截图,但是由于程序初始化是只初始化一个window所以-m无用

程序-s选项提供给用户区域截图或者截取某个windows的功能

代码的主要框架也是顺序流程,从程序执行初始化到后面交互再到以后的保存图像,并没有多少逻辑分支

#### options.c 解析配置参数
包含opt的声明.通过getopt_long解析参数.

#### imlib.c 初始化x与imlib库
初始化了一个windows


只初步了解了一些xlib的函数使用,具体原理还需了解
源码中涉及libx11的内容较多,后续学习时候可以参考使用,在此就不做分析



#### 参考文献
[xlib](http://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html)
[imlib2](http://adesklets.sourceforge.net/doc/imlib2/imlib2_8c.html#a14)
[giblib](http://www.cs.tut.fi/lintula/manual/gtk/glib/glib-string-utility-functions.html)
