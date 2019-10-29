---
title: configurationARGS
date: 2019-10-07 15:24:00
tags: [linux, conf]
categories: tool
---
# 常用配置参数
## 1. vim的配置参数
<!-- more -->
针对每一个用户修改，在/etc/vimrc中是所有用户生效
```
vim ~/.vimrc
```
输入如下配置：
```
"关闭vim一致性原则
set nocompatible
"显示行号
set number
"设置在编辑过程中右下角显示光标的行列信息
set ruler
"在状态栏显示正在输入的命令
set showcmd
"设置历史记录条数
set history=1000
"设置取消备份 禁止临时文件的生成
set nobackup
set noswapfile
"设置匹配模式
set showmatch
"设置C/C++方式自动对齐
set autoindent
set cindent
"开启语法高亮功能
syntax on
"指定配色方案为256色
set t_Co=256
"设置搜索时忽略大小写
set ignorecase
"配置backspace的工作方式
set backspace=indent,eol,start
"设置在vim中可以使用鼠标
set mouse=a
"设置tab宽度
set tabstop=4
"设置自动对齐空格数
set shiftwidth=4
"设置退格键时可以删除4个空格
set smarttab
set softtabstop=4
"将tab键自动转换为空格
set expandtab
```
最后使配置生效
```
source ~/.vimrc
```
