---
title: numpyPractice
date: 2019-10-05 20:08:05
tags: [Numpy, Python]
categories: Numpy
---
# numpy (Numerical Python)数值计算基础
[官网链接](https://numpy.org/)
<!-- more -->
# 一. 简介
* python 语言数值计算扩充库
* 强大的高维数组处理和矩阵运算能力
* NumPy 的主要对象是多维数组 Ndarray

# 二. 应用demo

##  1.导入模块与查看版本信息
```
import numpy as np
print(np.__version__)
```
## 2.创建数组
> 主要创建数组的途径<br>
> * 从 Python 数组结构列表，元组等转换。
> * 使用 np.arange、np.ones、np.zeros 等 NumPy 原生方法。
> * 从存储空间读取数组。
> * 通过使用字符串或缓冲区从原始字节创建数组。
> * 使用特殊函&emsp;&emsp;数，如 random。
### 2.1 通过列表创建数组
```
# 一维数组
a = np.array([1, 2, 3])
print(a)
# 二维数组
b = np.array([(1, 2, 3), (4, 5, 6)])
print(b)
```
### 2.2 使用原生方法
```
#创建全为0的数组
a = np.zeros((3, 3))
print(a)

#创建全为1的数组
b = np.ones((2, 2, 4))
print(b)
```
> **说明** : numpy.ones(shape, dtype=None, order='C')，shape指定数组的形状,每个轴的长度

```
#创建一维等差数组
a = np.arange(5)
print(a)
# 结果：[0 1 2 3 4]

#创建二维等差数组
b = np.arange(6).reshape(2, 3)
print(b)
# 结果：[[0 1 2]
#       [3 4 5]]
```
> **arange说明** <br>
>  * 格式： numpy.arange(start, stop, step, dtype=None)<br>
>  * [start, stop) 半开半闭区间内创建一系列均匀间隔的值

```
# 创建二维单位阵
a = np.eye(3)
print(a)
# 结果：
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]
```
> **eye说明** <br>
> * 格式： numpy.eye(N, M=None, k=0, dtype=<type 'float'>) <br>
> * k：对角线索引：0（默认）是指主对角线，正值是指上对角线，负值是指下对角线

```
# 创建等间隔的一维数组
a = np.linspace(1, 10, num=7)
print(a)
# 结果 [ 1 2.5 4 5.5 7 8.5 10. ]
```
> **linspace说明** <br>
> * 格式： numpy.linspace(start, stop, num=50, endpoint=True, retstep=False, dtype=None) 同arange<br>
> * endpoint 指示是否最后一个样本包含在序列内

### 2.3 使用特殊函数创建数组
```
# 创建二维随机数组
a = np.random.rand(2, 3)
print(a)

# 结果： [[0.72448631 0.19235668 0.00701753]
        [0.05801567 0.9642479  0.22793563]]
```
> **rand()说明** <br>
> 括号内指定数组的形状 <br>
> 使用[0, 1)之间的数随机填充
```
# 创建二维随机数组(数值<5)
a = np.random.randint(5, size=(2, 4))
print(a)
# 结果：
[[3 1 2 0]
 [1 4 2 2]]
```
> **randint()说明** <br>
> 格式： randint(low, high, size, dtype) <br>
> 生成[low, high) 的随机数
> 一维数组的时候size参数可直接填数字  eg:`randint(1, 3, 10)`

### 2.4 从已知数据创建
```
# 使用lambda创建数组
a = np.fromfunction(lambda i, j: i + j, (3, 3))
print(a)
# 结果：
#[[0. 1. 2.]
#[1. 2. 3.]
#[2. 3. 4.]]
```
> **fromfunction说明**<br>
> 格式： fromfunction（function，shape）：通过函数返回值来创建多维数组

## 3. 数组运算
### 3.1 一维数组的四则运算
生成两个Ndarry<br>
```
a = np.array([10, 20, 30, 40, 50])
b = np.arange(1, 6)
#b数组：[1 2 3 4 5]
```

```
c = a + b
print("c: ", end="")
#结果： [11 22 33 44 55]
c = a - b
c = a * b
c = a / b
```
**注意** ：**数组大小不一样会发生异常**
### 3.2 二维数组（矩阵）的运算
生成两个Ndarry <br>
```
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])
```
同一维数组一样，可以进行个元素分别的四则运算
```
c = a + b
c = a * b
```
但是矩阵乘法是不一样的： 矩阵m\*n与矩阵n\*p的矩阵是m\*p
```
#矩阵乘法
c = np.dot(a, b)

c = np.mat(a) * np.mat(b)
```
> **注意**<br>
>这里的a, b都是二维数组，所以用dot进行运算<br>
> 可以使用np.mat(a)将二维数组转换为矩阵，直接使用*
```
# 矩阵数乘
c = 2 * a

# 矩阵转置
c = a.T

# 矩阵求逆
c = np.linalg.inv(a)
```
> np.linalg中涉及到矩阵的计算方法，求解特征值、特征向量、逆矩阵等

## 4. 数组索引和切片
### 4.1 一维数组
一个一维数组
```
a = np.array([1, 2, 3, 4, 5])
```
一维数组索引
```
print(a[0], a[-1])
# 结果： 1 5
```
一维数组切片
```
a = np.array([1, 2, 3, 4, 5])

print(a[0:2])
# 结果 [1 2]

print(a[:-1])
# 结果 [1 2 3 4]
```
### 4.2 二维数组
一个二维数组
```
a = np.array([(1, 2, 3), (4, 5, 6), (7, 8, 9)])
```
二维数组索引
```
a = np.array([(1, 2, 3), (4, 5, 6), (7, 8, 9)])
print(a[0])
# 结果： [1 2 3]

print(a[-1])
# 结果： [7 8 9]
```
二维数组切片
```
a = np.array([(1, 2, 3), (4, 5, 6), (7, 8, 9)])

print(a[0:3, 1]) #等价于a[:, 1]
# 结果： [2 5 8]  取第2列

# 取第二三行
print(a[1:3, :])
# 结果：
# [[4 5 6]
# [7 8 9]]
```
> (a, c)括号前面控制的是选取的行， 后面控制的是列。

## 5. 数组形状操作
* 查看数组形状
```
a = np.random.random((3, 2))
print(a)
print(a.shape)
# result: (3, 2)
```
* 更改数组形状（不改变原始数组）
```
a = np.random.randint(1, 10, 6).reshape(3, 2)
print(a)
c = a.reshape((2, 3))
print(c)
```
> **说明** <br>
> 1. numpy.reshape() 等效于 ndarray.reshape()。<br>
> 2. numpy.reshape(a, newshape)。 其中newshape 用于指定新的形状(整数或者元组)。

* 更改数组形状（改变原始数组）
```
a = np.random.randint(1, 10, 6).reshape(3, 2)
print(a)
a.resize(2, 3)
print(a)
```
* 数组扁平化(变为一维数组)
```
a = np.random.randint(1, 10, 6).reshape(3, 2)
print(a)
c = np.ravel(a)
d = a.ravel()
```
> numpy.ravel(a, order='C') 按行读取
> numpy.ravel(a, order='F') 按列读取

* 垂直方向堆叠数组
```
a = np.random.randint(10, size=(2, 2))
b = np.random.randint(10, size=(2, 2))
print(a)
print(b)
c = np.vstack((a, b))
print(c)
```
> 输出 <br>
a: [[2 9] [8 1]] <br>
b: [[1 5] [0 1]]
c: [[2 9] [8 1] [1 5] [0 1]] <br>
 拓展 d = np.hstack((a, b))  水平方向上堆叠数组

 ## 6. 数组排序与统计
 * 返回每列最大值
 ```
a = np.array(([1, 4, 3], [6, 2, 9], [4, 7, 2]))
c = np.max(a, axis=0)
print(c)
# result： [6 7 9]
 ```
 * 返回每行最小值
 ```
a = np.array(([1, 4, 3], [6, 2, 9], [4, 7, 2]))
c = np.min(a, axis=1)
print(c)
# result: [1 2 2]
 ```
* 返回每列最大值索引
```
a = np.array(([1, 4, 3], [6, 2, 9], [4, 7, 2]))
c = np.argmax(a, axis=0)
print(c)
# result: [1 2 1]
```
* 返回每行最小值索引
```
a = np.array(([1, 4, 3], [6, 2, 9], [4, 7, 2]))
c = np.argmin(a, axis=1)
print(c)
# result: [0 1 2]
```
* 统计数组各列的中位数
```
a = np.array(([1, 4, 3], [6, 2, 9], [4, 7, 2]))
c = np.median(a, axis=0)
print(c)
# result: [4. 4. 3.]
```
* 其他
>  np.mean(a, axis=1)统计数组各行的算术平均值 <br>
> np.average(a, axis=0)统计数组各列的加权平均值 <br>
> np.var(a, axis=1)统计数组各行的方差 <br>
> np.std(a, axis=0) 统计数组各列的标准偏差<br>

