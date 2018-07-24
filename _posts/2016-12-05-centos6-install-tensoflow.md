---
title: centos6安装tensoflow
date: 2016-12-05 20:48:57
tags:
    - tensoflow
categories:
    - 深度学习
---

# 0 升级python2.6

系统默认安装的Python是2.6.6的，我们需要升级到Python2.7，用wget命令从官方下载源文件，然后解压进行编译

```shell
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
tar xvf Python-2.7.12.tgz
cd Python-2.7.12
make && make install
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
```

然后编辑/usr/bin/yum，将第一行的#!/usr/bin/python修改成#!/usr/bin/python2.6.6
现在执行yum命令已经不会出现之前的错误信息了。

# 1 安装pip

```shell
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py

whereis pip
ln -s /usr/local/bin/pip2.7 /usr/bin/pip
```

# 2 安装依赖的lib

```shell
sudo yum -y install epel-release
sudo yum -y install gcc gcc-c++ python-pip python-devel atlas atlas-devel gcc-gfortran openssl-devel libffi-devel
pip install --upgrade numpy scipy wheel cryptography
```

由于CentOS6的系统安装了epel-release-latest-7.noarch.rpm 导致在使用yum命令时出现Error: xz compression not available问题。
解决方法：

```shell
1.到http://ftp.riken.jp/Linux/fedora/epel/下载epel-release-latest-6.noarch.rpm
2.卸载epel-release-latest-7.noarch.rpm：yum remove epel-release
3.清空epel目录：rm -rf/var/cache/yum/x86_64/6/epel/
4.安装epel6:rpm -ivhepel-release-latest-6.noarch.rpm
```

# 3 安装tensorflow

```shell
sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.0rc0-cp27-none-linux_x86_64.whl
```

# 4 test

```shell
ImportError: /lib64/libc.so.6: version `GLIBC_2.16' not found (required by /usr/local/lib/python2.7/site-packages/tensorflow/python/_pywrap_tensorflow.so)
Error importing tensorflow.  Unless you are using bazel,
you should not try to import tensorflow from its source directory;
please exit the tensorflow source tree, and relaunch your python interpreter
from there.

## 解决方法
下载新版本glibc安装
```
