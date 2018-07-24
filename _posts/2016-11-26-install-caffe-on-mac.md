---
title: mac安装caffe过程
date: 2016-11-26 13:51:29
tags:
    - caffe
    - 深度学习
categories:
    - 深度学习
---

# 0. 关于Caffe

昨天看到amazon将mxnet做的其深度深度算法的消息，突然觉得应该开始学习一些深度学习的知识了。
本来想试试mxnet的，但是觉得还是先从caffe开始吧。caffe是深度学习在图像领域广泛使用的框架，有很多现在的model可以直接使用。

# 1. 安装依赖

使用brew安装依赖，首先更新一下brew，不然可能出现意想不到的各种错误。
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

然后安装依赖

```shell
brew install -vd snappy leveldb gflags glog szip lmdb
brew tap homebrew/science
brew install hdf5 opencv protobuf boost
```

# 2. 编译

下载caffe源码

```shell
git clone https://github.com/bvlc/caffe.git
cd caffe
cp Makefile.config.example Makefile.config
```

修改编译模式开关

```shell
CPU_ONLY := 1
```
编译
```shell
make -j
```

编译完成，可以开始动手运行例子了！

