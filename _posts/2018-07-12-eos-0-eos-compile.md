---
title: 【EOS源码阅读】1.EOS源码编译运行
date: 2018-07-12 18:14:33
tags:
  - 区块链
  - EOS
categories:
  - 区块链
---

# 前言

EOS是当下最火的区块链技术，被社会广泛看好为下一代区块链3.0。
和以太坊不同，EOS的主语言是C++，最近开始关注区块链相关的技术，接下来的学习将从EOS源码开始着手一步步的展开。

# EOS

EOS，可以理解为Enterprise Operation System，即为商用分布式应用设计的一款区块链操作系统。
EOS是引入的一种新的区块链架构，旨在实现分布式应用的性能扩展。

EOS是BM设计的，BM是与小神童齐名的天才，只是人比较低调，较少人知道，整个EOS项目，BM只负责技术，商业方面是由BLOCK.ONE团队运作的。
EOS是针对区块链存在的扩容困难、交易费高昂、不能跨链连接、公链不适合企业使用等问题进行设计的。

EOS最近发布了EOS.IO DAWN 2.0，同时也发布了一个公开的测试网络，大家也可以开始体验EOS的各种功能。

# 编译环境
```shell
Mac OS 
版本 10.13.4
8 GB 1867 MHz DDR3
```

在编译前，需要将硬盘整理出20G空间，编译要求7G内存以及最少20G空间，主要是安装的各种依赖会比较多，而且比较占用磁盘空间。

# 构建源码

```shell
git clone https://github.com/EOSIO/eos --recursive
cd eos && ./eosio_build.sh
```

# 错误处理集合

## LLVM没有安装
```shell
Could not find a package configuration file provided by "LLVM" (requested

  version 4.0) with any of the following names:

    LLVMConfig.cmake

    llvm-config.cmake

  Add the installation prefix of "LLVM" to CMAKE_PREFIX_PATH or set

  "LLVM_DIR" to a directory containing one of the above files.  If "LLVM"

  provides a separate development package or SDK, be sure it has been

  installed.
```

处理方案：
使用brew安装llvm
```shell
brew install llvm@4
# 按实际安装目录export, 选择4.0.x的版本
export export LLVM_DIR=/usr/local/opt/llvm@4
```


## cmake版本过低

处理方案：

```shell
brew upgrade cmake
brew link cmake
```

# 编译完成
源码中的的eosio_build.sh脚本可以一键安装，但是构建时间较长，
中途还会出现各种依赖安装错误，可能还需要输入sudo密码。
最终构建成功的页面如下：

```shell

	 _______  _______  _______ _________ _______
	(  ____ \(  ___  )(  ____ \\__   __/(  ___  )
	| (    \/| (   ) || (    \/   ) (   | (   ) |
	| (__    | |   | || (_____    | |   | |   | |
	|  __)   | |   | |(_____  )   | |   | |   | |
	| (      | |   | |      ) |   | |   | |   | |
	| (____/\| (___) |/\____) |___) (___| (___) |
	(_______/(_______)\_______)\_______/(_______)

	EOSIO has been successfully built. 00:32:59

	To verify your installation run the following commands:

	/usr/local/bin/mongod -f /usr/local/etc/mongod.conf &
	cd /Users/linzhe/blockchain/eos/build; make test

	For more information:
	EOSIO website: https://eos.io
	EOSIO Telegram channel @ https://t.me/EOSProject
	EOSIO resources: https://eos.io/resources/
	EOSIO Stack Exchange: https://eosio.stackexchange.com
	EOSIO wiki: https://github.com/EOSIO/eos/wiki

```
