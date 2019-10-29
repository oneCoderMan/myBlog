---
title: IISFTP
date: 2019-10-24 16:21:00
tags: [linux, 字符串格式化漏洞]
categories: [tool]
---
# win10搭建ftp服务实践
win10下面用自带的iis搭建ftp服务用于文件传输
<!-- more -->
## 1.开启IIS服务
![pic](/images/iisftp/01.png)
![pic](/images/iisftp/02.png)

## 2.使用搜索功能打开IIS，添加ftp站点
![pic](/images/iisftp/03.png)
![pic](/images/iisftp/04.png)
<br>经过上面的步骤在本机上是可以访问到文件的，在局域网或者其它网络的机器是无法访问的。接下来继续配置。
```
ftp://ip
```

## 3.配置防火墙
一个简单的方法是关闭防火墙，这里配置其他方法不用关闭防火墙。
### 3.1 放行21端口
![pic](/images/iisftp/05.png)
点击高级设置<br>
![pic](/images/iisftp/06.png)
入站规则-> 新建规则<br>
![pic](/images/iisftp/07.png)
选择端口，下一步<br>
![pic](/images/iisftp/08.png)
然后一直下一步。

### 3.2 设置ftp允许通过防火墙
![pic](/images/iisftp/09.png)
![pic](/images/iisftp/10.png)
然后点击允许其他应用
![pic](/images/iisftp/11.png)
大功告成。