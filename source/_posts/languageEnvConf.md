---
title: languageEnvConf
date: 2019-10-28 21:51:16
tags: [语言环境安装配置]
categories: [tool]
---
# 语言环境的安装和配置
<!-- more -->
## ubuntu16.04下的python3.7安装
>ubuntu16.04默认安装有python3.5和python2.7
安装路径在`/usr/local/lib`下

可以通过如下命令查看python的指向
```
ll /usr/bin | grep python
```
可以知道python的默认指向时python2.7
* step1： 下载python3.7
```
wget https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tgz
```

* step2: 解压安装包
```
tar -zxvf Pyhon-3.7.1.tgz
```

* step3: 编译安装
```
cd Python-3.7.1
./configure --prefix=/usr/local/python3.7.1
make
make install
```
经过上面步骤有
文件|路径
---:|:--
可执行文件|/usr/local/python3.7.1/bin
库文件|/usr/local/python3.7/lib
配置文件|/usr/local/python3.7/share

* step4：配置环境变量
```
#查看环境变量
echo $PATH
vim ~/bashrc
#添加如下的语句
PATH=$PATH:$HOME/bin:/usr/local/python3.7.1/bin
source ~/bashrc
```

* step5: 测试：
输入`python3.7`可进入python解释器即可
`python -V`查看版本。
或者
```
import sys
print(sys.version)
print(sys.version_info)

```

> **zlib模块缺失的解决方法**：
官网下载zlib`wget http://zlib.net/zlib-1.2.11.tar.gz`，可以查看最新版本
然后解压，配置，安装
tar -xvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
最后重新编译安装python3.7

> **No module named '_ctypes'解决方案**
`sudo apt-get install libffi-dev`
最后重新编译安装python3.7

* step6: 修改系统默认指向
```
删除原有链接备份
mv /usr/bin/python /usr/bin/python.bak
建立新的链接
ln -s /usr/local/python3.7.1/bin/python3.7 /usr/bin/python
修改pip
mv /usr/bin/pip /usr/bin/pip.bak
建立新的pip
ln -s /usr/local/python3.7.1/bin/pip3 /usr/bin/pip
```
# 参考文献
[1] https://blog.csdn.net/u014775723/article/details/85213793

