---
title: shell-auto
date: 2019-09-11 16:32:38
tags: shell
categories: Linux Shell
---
# shell脚本实现Linux系统监控
***
代码见 https://github.com/oneCoderMan/myCodes/blob/master/shell/auto.sh
***

<!--more-->

## 一. 预备知识
### 1.1 getopts

> 作用: 处理参数复杂的命令行选项和参数<br>
>格式：getopts option_string args<br>
>说明： option_string中是参数列表， args从命令行中接受的值

示例代码：option.sh<br>
```
#! /bin/bash
while getopts f:e:p: option
do
   case "${option}" in
       f) file_name=${OPTARG};;
       e) editor=${OPTARG};;
       p) page=${OPTARG};;
       *) echo "invalid";;
   esac
done
echo "${file_name}"
echo "${editor}"
echo ${page}
```
运行命令：`./option.sh -f bool.pdf -e YiJun -p HUANAN` <br>
option_string是匹配的参数,字符后面有：表示该命令中必须加上参数，如-e yijun。否则会出现错误。参数的值会保存在**OPTARG**变量中。<br>
>检测到非法参数就会停止
### 1.2 $+sign各含义
>1. **$#**   :Stores the number of command-line arguments that
      were passed to the shell program.<br>
>2. **$?**    :Stores the exit value of the last command that was
      executed.<br>
>3. **$0**   :Stores the first word of the entered command (the
      name of the shell program).<br>
>4. **$\***   :Stores all the arguments that were entered on the
      command line ($1 $2 ...).<br>
>5. **$@**  :Stores all the arguments that were entered
      on the command line, individually quoted ("$1" "$2" ...). <br>
>6. **$>** : 1>是标准输出重定向， 可以省去1(变为 > )； 2>错误重定向。$>是两种的结合，标准与错误重定向<br>
>7. **>$1** : 将重定向到1管道， 即定向到标准输出

如下例子：<br>
`./command -yes -no /home/username`
```
$# = 3
$* = -yes -no /home/username
$@ = array: {"-yes", "-no", "/home/username"}
$0 = ./command, $1 = -yes etc.
```
### 1.3 tput与颜色转义
[参考链接](https://www.cnblogs.com/technologylife/p/8275044.html)
 >**tput作用**：更改终端功能<br>
```
tput sgr0       # 恢复默认终端
tput blink      # 文本闪烁
```
一个例子1-3：<br>
```
tecreset=$(tput sgr0)
echo -e '\E[32m'"Operating System Type :" $tecreset $os
```
结果：<br>
![结果](/assets/shell/p1.png)
>**说明:** <br>
>1. 如果把1-3中的$tecreset去掉，都会是绿色。<br>
>2. 方便操作，不用每次都使用命令替换$(), \`\`， 因此用一个变量tecreset存储命令。

**linux终端的颜色由转义序列控制<br>**
>转义序列由控制符 `esc`开头，控制序列引导符为：`\033`或者`\e[`，m为转义结束。<br>
```
格式1：\033[ Param {;Param;...} m

格式2：\e[ Param {;Param;...}m

```
例子1-3   [参考资料](http://c.biancheng.net/linux/echo.html)  <br>
```
echo -e "\033[颜色1;颜色2m 要展示的文字 \033[0m"
\033[0m：表示将颜色恢复回原来的颜色
```
### 1.4 各种括号
[参考文献](https://blog.csdn.net/x1269778817/article/details/46535729)
* $()与\`\`
> 用来做命令替换的，简化shell脚本的编写
* ${}
>变量替换的，即变量引用
* $[]与 $(())
> 都是用来做数学运算的<br>
>**注意** : $(())中的变量前面可以加$,也可不用
* []
> test命令的一种形式 左右需留空格，即判断true或者false
* (())与[[]]
> 分别是数学表达式和字符表达式[]的加强版
## 二.监控系统脚本各模块
### 2.1 脚本安装 -i
代码如下：<br>
```
if [[ $iopt -eq 1 ]]; then   #对变量的引用要加上$  then如果是同一行的话要加上;
  {
    #获得它的路径
    wd=`pwd`
    #获得它的文件名
    basename `echo $0` > /tmp/script
    #拼接
    scriptname=`echo -e -n $wd/ && cat /tmp/script`
    # 加载到环境变量中去
    su -c "cp $scriptname /usr/bin/monitor" root && echo "congratulations! install successful!" || echo "install failed!"
  }
fi
```
>**代码说明**<br>
>1. \`\`(反引号)是执行shell命令，可将结果保存在变量中。同$(),这个有些shell不支持。<br>
>2. basename 命令是将前缀和最后的/删除，保留最后一个字符串显示出来<br>
>3. $0 是指shell本生的文件名，`sh test.sh`得到 test.sh。`./test.sh` 得到的是./test.sh <br>
>4. echo中的-e 用来打印转义符， -n不换行打印<br>
>5. su -c command USER 切换用户USER执行命令command，执行完之后返回原用户<br>
>6. ||  && 在命令组合中具有短路左右，可以看成if else结构<br>
>7. [[ ]] (())这个是进行数学表达式和字符表达式运算的加强版。推荐使用 ,**左右需留空格**<br>
>8. -eq 是数字等于判断， 对于字符串等于使用=

### 2.2 关于模块 -v
代码如下:<br>
```
if [[ $vopt -eq 1]]
then
  {
    echo -e "monitor version 1\nreleased Under ...."
  }
fi
```
### 2.3 帮助模块 -h
代码如下： <br>
```
if [[ $hopt -eq 1 ]]
then
  {
    echo -e "  -i\tinstall the scripts "
    echo -e "  -v\tprint version "
    echo -e " -h\t print help info"
  }
fi
```
### 2.4 查看操作系统类型
>**注意** ：不同操作系统对应的代码不一样啊， 本文中只讨论centOS系统.不同版本的代码见附录1

代码如下：
```
#查看系统版本和名称
OS=`uname -s`
REV=`uname -r`
MACH=`uname -m`

#查看操作系统类型，这里适用于redHat
if [[ ${OS} = "Linux" ]]; then
{
  KERNEL=`uname -r`  #内核发布版本
  if [ -f /etc/redhat-release ]; then  # test -f filename, 测试是否为普通文件
  {
    DIST="RedHat"
    Psuedoname=`cat /etc/redhat-release | sed s/.*\(// | sed s/\)//`
    REV=`cat /etc/redhat-release | sed s/.*release\ // | sed s/\ .*//`
  }
  fi
  OSSTR="${OS} ${DIST} ${REV}(${Psuedoname} ${KERNEL} ${MACH})"
}
fi
echo ${OSSTR}
```
代码说明：
>1. uname是查看系统版本的命令 <br>
>2. `[ -f /etc/redhat-release ]` 文件测试指令<br>
>3. 可以查看 http://coolshell.cn/articles/9104.html 获取sed帮助

### 2.5 监控系统的各种信息
* 查看DNS<br>
`cat /etc/resolv.conf | sed '1d' | awk '{print $2}'`
>**解释说明** <br>
>读取配置文件，删除第一行， 打印第二个字段
* 查看系统负载
```
loadverage=$(top -n 1 -b | grep "load average:" | awk '{print $10 $11 $12}')
echo -e "\E[32m load average: \E[0m "${loadverage}
```
>**说明** <br>
>top -n 1 只迭代一次，不会动态刷新， top -n 1 -b 示非动态打印系统资源使用情况
* 查看系统运行时间
```
tecuptime=$(uptime | awk '{print $3,$4}' | cut -f1 -d',')
echo -e "\E[32m System Uptime Days: \E[0m "${loadverage}
```
>**说明** <br>
> cut -f1 -d',':   -f1选定第一列， -d\'* \' 表示用*分割

来源：https://superuser.com/questions/247127/what-is-and-in-linux

## 附录
### 附录1： 不同操作系统查看操作系统类型

```
if [ "${OS}" = "SunOS" ] ; then
    OS=Solaris
    ARCH=`uname -p`
    OSSTR="${OS} ${REV}(${ARCH} `uname -v`)"
# uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。
elif [ "${OS}" = "AIX" ] ; then
    OSSTR="${OS} `oslevel` (`oslevel -r`)"
#AIX是IBM开发的一套类UNIX操作系统，关于它更多的指令可以查看http://www.cnblogs.com/sbaicl/articles/2947795.html
elif [ "${OS}" = "Linux" ] ; then
    KERNEL=`uname -r`
    if [ -f /etc/redhat-release ] ; then
        DIST='RedHat'
        PSUEDONAME=`cat /etc/redhat-release | sed s/.*\(// | sed s/\)//`
        REV=`cat /etc/redhat-release | sed s/.*release\ // | sed s/\ .*//`
#sed通常用来匹配一个或多个正则表达式的文本进行处理,可以查看http://coolshell.cn/articles/9104.html
    elif [ -f /etc/SuSE-release ] ; then
        DIST=`cat /etc/SuSE-release | tr "\n" ' '| sed s/VERSION.*//`
        REV=`cat /etc/SuSE-release | tr "\n" ' ' | sed s/.*=\ //`
    elif [ -f /etc/mandrake-release ] ; then
        DIST='Mandrake'
        PSUEDONAME=`cat /etc/mandrake-release | sed s/.*\(// | sed s/\)//`
        REV=`cat /etc/mandrake-release | sed s/.*release\ // | sed s/\ .*//`
    elif [ -f /etc/debian_version ] ; then
        DIST="Debian `cat /etc/debian_version`"
        REV=""

    fi
    if ${OSSTR} [ -f /etc/UnitedLinux-release ] ; then
        DIST="${DIST}[`cat /etc/UnitedLinux-release | tr "\n" ' ' | sed s/VERSION.*//`]"
    fi

    OSSTR="${OS} ${DIST} ${REV}(${PSUEDONAME} ${KERNEL} ${MACH})"

fi
```
