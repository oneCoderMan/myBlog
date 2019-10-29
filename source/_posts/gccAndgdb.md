---
title: gccAndgdb
date: 2019-10-09 14:12:03
tags: [linux, gcc]
categories: tool
---
# linux下的c编程工具的使用
<!-- more -->
test.c的代码如下:
```
#include<stdio.h>
main()
{
    printf("Hello, World\n");
}
```
## 1.gcc工具的使用
* 不带任何参数
```
gcc test.c
```
会在同一个目录下面生成一个`a.out`文件，输入命令`./a.out`就可直接运行。
> 直接经历了四个步骤：预处理Preprocessing，编译Compilation，汇编Assembly，链接Linking。

* 指定生成的文件名
```
gcc test.c -o test
```
不会使用默认的a.out。

* gcc保留中间文件
gcc编译过程中，会删除中间的文件，可通过下面命令参数来保留各个阶段的文件
```
gcc -save_temps test.c
```

* gcc生成调试信息
```
gcc -g test.c -o test
```

* gcc编译生成32bit代码
```
 gcc -m32 hello.c -o hello
```
如果`-m32`无法运行，请安装下载相关包
```
sudo apt-get install build-essential module-assistant  
sudo apt-get install gcc-multilib g++-multilib
```

* gcc版本信息查看
```
gcc -v
```
## 2.gcc的各个阶段分析
### 2.1 预处理
```
gcc -E test.c -o test.i
```
功能：主要处理源代码中以 “#” 开始的预编译指令
生成文件：`test.i`
指定参数: `-E`
### 2.2 编译
```
gcc -S test.c -o test.s
#或者
gcc -S test.i -o test.s
```
输入的文件可以是源文件，也可以是预处理后的文件
功能：对预处理完的文件进行一系列**词法分析、语法分析、语义分析**及优化后生成相应的**汇编代码**文件
生成文件：`test.s`
指定参数：`-S`
### 2.3 汇编
```
gcc -c test.c -o test.o
#或者
gcc -c test.s -o test.o
```
输入的文件可以是源文件，也可以是编译后的文件
功能：汇编器将汇编代码转变成机器可以执行的指令
生成文件：`test.o`(**不能直接执行**)
指定参数：`-c`
> objdump：一种可阅读的格式让你更多地了解二进制文件（反汇编）

`objdump -sd hello.o`可查看test.o的信息
![objdump](/images/gccAndgdb/objdump.png)
### 2.4 链接
```
gcc test.o -o test
```
输入文件可以是源文件也可以是汇编后的文件
指定参数：无
生成文件：`test`(**可以执行**)
目标文件需要链接一大堆文件才能得到最终的可执行文件
通过以下命令可查看反汇编代码
```
objdump -d -j .text text
```

## 3.gdb的使用
测试程序如下`test.c`:
```
#include <stdio.h>
int main(){
   int a = 1;
   int b = a;
   printf("a = %d, b =%d\n", a, b);
   return 0;
}
```
GDB主要帮助你完成下面四个方面的功能：
1. 启动你的程序，可以按照你的自定义的要求随心所欲的运行程序。
2. 可让被调试的程序在你所指定的调置的断点处停住。
3. 当程序被停住时，可以检查此时你的程序中所发生的事。
4. 你可以改变你的程序，将一个BUG产生的影响修正从而测试其他BUG。
### 3.1 gbd调试程序的简易入门操作
* 安装
```
sudo apt install gdb
```
* 启动调试
```
#编译文件，加上调试信息。不加参数-g可能无法调试
gcc -g test.c -o test
#开始启动调试
gdb test
```
> 拓展: 启动调试的时候输入`gdb -tui test`会有一个漂亮的交互窗口

* 调试的操作
**注意**  :括号内的东西不用输入

1. 显示当前的代码，一次10行，继续输入`l`继续显示。
```
(gdb) l
```
2. 单步
```
(gdb) start
```
这个时候会进入程序运行状态，单步运行需要输入`n`，输入一次执行一条语句。`s`单步会进入子程序中去。

3. 断点+单步
启动的时候`start`程序会在第一行停止。
这个时候可以设置断点，如下所示：
```
(gdb) b 5
(gdb) c #运行程序到断点处，一般在设置完断点后使用
(gdb) r #重新运行程序 run
```
>表示在第5行设置断点，输入命令`c`会直接运行到第5行；其中b后面的参数也可以是一个函数名。可以按照单步的方法继续运行。
可以同时设置多个断点
```
(gdb) display b
```
这个display 可以查看变量的值

4. 查看所有断点和删除
查看已经设置的断点
```
(gdb) info breakpoints
#可以简写成
(gdb) i b
```
![gdbInfo](/images/gccAndgdb/gdbInfo.png)
注意到每一个断点有编号，使用编号可删除断点
```
(gdb) delete 2
```

5. 设置条件断点
```
(gdb) b 5 if a == 2
```

6. 观察变量改变的地方
```
(gdb) watch b
(gdb) c
```
变量内存值被修改的时候会展示出来

7. 反汇编命令
```
(gdb) disassemble main
#可简写成
(gdb) disas main
```
其中的main是一个函数名称。

### 3.2 gdb插件---gdb-peda
PEDA(**Python Exploit Development Assistance for GDB**)是一个强大的 gdb 插件。它提供了高亮显示反汇编代码、寄存器、内存信息等人性化的功能。同时，PEDA 还有一些实用的新命令，比如checksec 可以查看程序开启了哪些安全机制等等。
* 安装
```
$ git clone https://github.com/longld/peda.git ~/peda
$ echo "source ~/peda/peda.py" >> ~/.gdbinit
```
## 4. Linux代码执行漏洞分析实战
主要工作：出发一个NoIP本地运行的栈溢出漏洞，用gdb调试工具找出漏洞发生的地点。
### 4.1 准备
NoIP本地栈溢出漏洞
* step1 注册NO-IP账号
```
https://my.noip.com/#!/account
https://www.noip.com
```
* step2 在Linux下安装No-ip
[官网](https://www.noip.com/support/knowledgebase/installing-the-linux-dynamic-update-client/)
```
cd /usr/local/src
wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
tar xzf noip-duc-linux.tar.gz
cd no-ip-2.1.9
make
make install
```
* 由PoC出发漏洞
```
gdb noip2
run -i $(python -c 'print ("\x41"*500)') 
```
> 如果没有触发漏洞，将数值可以加大为1000，直到有漏洞触发

![图片](/images/gccAndgdb/noipPoc.png)
图1

* 使用静态反汇编工具idaPro
win系统下远程连接linux编译文件调试参考文献[1](#r01)

### 4.2实验过程
由图1可知，漏洞发生之前，打印输出了字符串`IP address detected on command line.`我们先定位到这个字符串输出的位置。
* step1: IDAPro远程连接到Noip程序
打开搜索框，输入上面的那个字符串进行搜索。
![pic](/images/gccAndgdb/findIPString.png)
> 可以看到这里有一个mov操作，之后调用了`call Msg`。

因此我们在这个地点下一个断点，红色框框对应的地址()。
* step2: 切换回linux，设置断点
退出之前的gdb, `quit`
重新进入`gdb noip2`
通过第一步得到的地址设置断点
```
b *0x00000000004021AB
```
![pic](/images/gccAndgdb/bAddress.png)
可观察到在源码中对应的行数为line 607
* step3: 执行得到漏洞的命令
```
run -i $(python -c 'print ("\x41"*1000)')
```
![pic](/images/gccAndgdb/againRun1.png)
可以看到在`0x4021ab`处发生了中断，即我们设置的断点的地方。
* step4: 单步执行
```
ni
#或者
n
```
在这个过程中，可以看到运行到了`call Msg`，并且打印出了字符串
![pic](/images/gccAndgdb/callMsg.png)
继续单步执行，可以看到输出另一串`Running in single use mode.`
如果之前进行了`push`操作，可以使用命令这可以查看栈顶的的值
```
(gdb) x/10x $esp
# 查看内存的值
(gdb) x/10x 0x4083b5(视情况)
(gdb) x/100x 0x4083b5
# 也可以以字符串形式输出内存的值
(gdb) x/10s 0x4083b5
```
![pic](/images/gccAndgdb/xsStr.png)
* step5 继续单步，一直到出现漏洞
![pic](/images/gccAndgdb/findError.png)

可以发现，当程序调用`dynamic_update`的前中后出现了问题。**有可能是在这个函数里面出现问题，也有可能是该函数嵌套的函数出现了问题。**
得到调用`dynamic_update`函数的地址为：`0x40233c`
* step6: 重新设置断点
切换回IDAPro，以此选择`jump`->`jump address`输入上面得到的函数地址。切换到`graph view`按住**F5**可查看c伪代码。
找到调用`dynamic_update`函数之前的一个地址`0x402337`，重新设置断点
```
(gdb) b *0x0x402337
(gdb) run -i $(python -c 'print ("\x41"*1000)')
```
* step7 进入到函数内部
这里使用`si`或者`s`进行单步。
进入到`dynamic_update`函数后，使用`n`来单步，确定程序崩溃的地方。


[1] <span id="r01">https://blog.csdn.net/lacoucou/article/details/71079552</span> windows下使用IDA远程调试linux(ubuntu)下编译的程序