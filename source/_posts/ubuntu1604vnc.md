---
title: ubuntu1604vnc
date: 2019-10-05 19:40:54
tags: [Linux VNC]
categories: tool
---
# ubuntu16.04 VNC远程桌面
## 1. VNC简介
<!-- more -->
* VNC(Virtual Network Console):虚拟网络控制台，一个远程控制的工具软件 <br>
* VNC包括四个命令：**vncserver**, **vncviewer**, vncpasswd, vncconnect.
* VNC由两部分组成，客户端的应用软件(vncviewer)&emsp;+&emsp;服务器端的应用程序(vncserver)

## 2. ubuntu16.04中安装服务端程序
使用SSH登录到系统<br>
* step1 
```
sudo apt-get update
```
* step 2
```
sudo apt-get install vnc4server
```
* step 3
启动vncserver
```
vncserver
```
步骤三中会出现如下界面
![pic01](/images/ubuntu1604vnc/01.png)
输入密码即可，这是客户端登陆密码。<br>
出现如下界面说明服务端安装成功
![pic02](/images/ubuntu1604vnc/02.png)
> 此时使用vncviewer登录出现的界面如下所示，对此需要安装一个图形化桌面
![pic03](/images/ubuntu1604vnc/03.png)

## 3. ubuntu16.04安装图形化桌面
### 3.1 全ubuntu16.04桌面(不推荐)gnome环境
> 缺点：占用资源多
* step1&emsp;安装x-windows
```
sudo apt-get install x-window-system-core
```
* step2&emsp;安装登录管理器(可不选)
```
sudo apt-get install gdm
```
* step3&emsp;安装ubuntu桌面
```
sudo apt-get install ubuntu-desktop
```
* step4&emsp;安装gnome配套软件
```
sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```
* step5&emsp;修改VNC配置文件，使桌面生效
```
vim ~/.vnc/xstartup
```
变成如下文件配置
```
#!/bin/sh
# Uncomment the following two lines for normal desktop:
export XKL_XMODMAP_DISABLE=1
 unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc
unset DBUS_SESSION_BUS_ADDRESS
gnome-panel &
gnmoe-settings-daemon &
metacity &
nautilus &
gnome-terminal &
```
重新启动就可以了
```
vncserver -kill :1  #杀掉原桌面进程（:1)就是
vncserver :1 #重新启动
```
### 3.2 ubuntu16.04剪切版(推荐)&emsp;gnome环境
> 优点：占用系统资源少，仅安装核心组件：不安装例如 office、浏览器、等等的额外组件
* step1&emsp;安装x-windows
```
sudo apt-get install x-window-system-core
```
* step2&emsp;安装登录管理器(可不选)
```
sudo apt-get install gdm
```
* step3&emsp;安装ubuntu桌面(剪切版)
```
apt-get install --no-install-recommends ubuntu-desktop
```
* step4&emsp;安装gnome配套软件
```
sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```
* step5&emsp;修改VNC配置文件，使桌面生效
```
vim ~/.vnc/xstartup
```
**在文件最后添加如下记录**
```
gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
```
### 3.3 安装xfce4桌面
> 优点：Xfce占用的内存和CPU使用量非常小；Xfce桌面很简单，而且没有乱七八糟的东西[参考文献](#re3)
* step 1:
```
sudo apt-get install xfce4
```
* step 2:修改配置文件
该文件路径一般在下面
```
vi ~/.vnc/xstartup
```
```
#x-window-manager &
sesion-manager & xfdesktop & xfce4-panel &
xfce4-menu-plugin &
xfsettingsd &
xfconfd &
xfwm4 &
```
## 4. 卸载桌面程序
### 4.1 卸载gnome桌面
* step1: 卸载掉gnome-shell主程序及其配套软件
```
sudo apt-get remove gnome-shell
sudo apt-get remove ubuntu-desktop
sudo apt-get remove gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```
* step2: 清理安装gnome时候留下的缓存程序软件包
```
sudo apt-get autoclean
sudo apt-get clean
```
### 4.2 卸载xfce4桌面
* step1: 卸载xfce4
```
sudo apt-get remove xfce4 
```
* step2: 卸载相关软件
```
sudo apt-get remove xfce4*
```
* step3: 自动卸载不必要的软件
```
sudo apt-get  autoremove
```
* step4: 系统清理
```
apt-get  clean
```
## 5. vnc相关的重要命令
1. 启动vncserver
> :1是桌面号
```
vncserver :1
```
2. 关闭vncserver
```
vncserver -kill :1
```
3. 客户端连接vncserver
```
ip:1
ip:5901
```
# 参考文献
[1] https://blog.csdn.net/u014389734/article/details/79513517
[2] http://dblab.xmu.edu.cn/blog/1998-2/
[3] https://www.cnblogs.com/chenmingjun/p/8506995.html <span id="re3">Linux的桌面环境gnome、kde、xfce、lxde 等等使用比较</span>
