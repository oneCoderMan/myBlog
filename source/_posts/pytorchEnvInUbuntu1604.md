---
title: pytorchEnvInUbuntu1604
date: 2019-10-07 10:53:23
tags: [linux, PyTorch]
categories: tool
---
# ubuntu16.04中搭建PyTorch环境
[官网链接](https://pytorch.org/get-started/locally/)
<!-- more -->
## 1.选择的环境
* Anaconda
原因：python搭建torch，numpy等模块太费劲，需要下载源代码编译安装等。直接选择anaconda比较简单。不需要处理复杂的包依赖关系
## 2.安装过程
* step1: 安装Anaconda
```
# The version of Anaconda may be different depending on when you are installing`
curl -O https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
sh Anaconda3-5.2.0-Linux-x86_64.sh
```
* step2: 使配置文件生效
```
source ~/.bashrc
```
> 还可以通过重启来使之生效reboot
* step3: 安装pyTorch
```
conda install pytorch torchvision cpuonly -c pytorch
```
## 3.安装过程的一些麻烦事

Anaconda安装后默认使用它带的python环境，为了不影响其他python程序的运行，如（VNCServer），会出现**终端调不出来**的情况。
需要重新配置环境变量。<br>
找到~/.bashrc文件中的如下语句（路径可能随系统不同而变化）：
```
export PATH="/root/anaconda2/bin:$PATH"
```
修改为下面的语句：
```
export PATH="$PATH:/root/anaconda2/bin"
```
> 这个时候在终端运行`python --version`是系统配置的python环境了，需要anaconda的python环境，只需要找到anaconda安装目录下的bin文件即可。