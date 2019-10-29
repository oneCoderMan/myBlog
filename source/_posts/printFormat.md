---
title: printFormat
date: 2019-10-22 14:20:24
tags: [linux, 字符串格式化漏洞]
categories: [二进制漏洞]]
---
# 字符串格式漏洞入门案例
介绍一个格式化字符串漏洞
<!-- more -->
## 1. 运行环境
ubuntu16.04<br>
gcc5.4.0<br>
gcc-peda

## 2. 准备工作
* 关闭ASLR(地址空间布局随机化)
```
echo 0 > /proc/sys/kernel/randomize_va_space
```
> 地址空间布局随机化：<br>每次加载程序到内存中时，进程地址空间的堆栈起始地址动态变化。用来阻止缓冲区溢出的攻击的**linux内核参数**<br>
0：关闭
2：打开

* 禁用栈保护措施和关闭PIE保护
```
gcc -m32 -fno-stack-protector -no-pie fmt.c
```
> PIE:<br>
PIE全称是position-independent executable，中文解释为地址无关可执行文件，该技术是一个针对代码段（.text）、数据段（.data）、未初始化全局变量段（.bss）等固定地址的一个防护技术，如果程序开启了PIE保护的话，在每次加载程序时都变换加载地址，从而不能通过ROPgadget等一些工具来帮助解题[1]

# 参考文献
[1] https://www.anquanke.com/post/id/177520