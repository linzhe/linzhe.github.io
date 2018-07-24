---
title: word2vec系列：入门基础
date: 2016-12-18 12:37:52
tags:
    - word2vec
categories:
    - 深度学习
---

# 快速入门

1、从http://word2vec.googlecode.com/svn/trunk/ 下载所有相关代码：
一种方式是使用svn Checkout，可加代理进行check。
另一种就是export to github，然后再github上下载，我选择第二种方式下载。

2、运行make编译word2vec工具：（如果其中makefile文件后有.txt后缀，将其去掉）在当前目录下执行make进行编译，生成可执行文件(编译过程中报出很出Warning，暂且不管)；

3、运行示例脚本：./demo-word.sh 看一下./demo-word.sh的内容，大致执行了3步操作

从http://mattmahoney.net/dc/text8.zip 下载了一个文件text8 ( 一个解压后不到100M的txt文件，可自己下载并解压放到同级目录下)；
使用文件text8进行训练，训练过程比较长；
执行word2vec生成词向量到 vectors.bin文件中，（速度比较快，几分钟的事情）
在demo-word.sh中有如下命令

运行结果如图：
