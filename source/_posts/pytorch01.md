---
title: pytorch01
date: 2019-09-24 15:57:24
tags: [PyTorch, 线性回归]
categories: deepLearing
---
# PyTorch入门之线性回归

[torch参考链接](http://pytorch.org/docs/0.3.0/tensors.html)
<!-- More -->
## 1. 安装
> * python 3.7
> * Torch 1.2.0 用于计算
> * matplotlib 用于画图

**踩坑点** <br>
Torch安装需要用到pip命令，直接在pycharm中会失败 <br>
**去官网获得pip的安装的命令** [Torch官网](https://pytorch.org/get-started/locally/)

## 2. PyTorch基础知识
* Tensor是PyTorch的基础数据结构
* 另一个重要的变量是Variable变量，该变量进行自动微分运算
> 反向传播算法随时进行<br>
> 该变量三个重要的值： 数据（data），父节点（creator），以及梯度（grad）
* 使用之前需要导入两个包

创建变量方法如下<br>
```
import torch
#导入自动梯度的运算包，主要用Variable这个类
from torch.autograd import Variable

#创建一个Variable，包裹了一个2*2张量，将需要计算梯度属性置为True
x = Variable(torch.ones(2, 2), requires_grad=True)
```
x变量如下所示 &emsp;&emsp;**注意** `.data` 可以返回一个 Variable 所包裹的 Tensor<br>
![x变量](/images/pytorch01/t1.png)

* `torch.rand(5, 3)`产生[0,1]的均匀分布的随机数值，参数指定形状(一维时第二个参数可不要)
* `torch.randn(100)`产生均值为0，方差为1，正态分布随机数值(同样可以指定形状)
* `torch.linspace(start, end, steps=100, out=None) → Tensor`线性间隔的数值，返回一个一维张量
* `y.t()`完成张量的转置
* Tensor可以与numpy.ndarray进行转换
> ndarray转换为Tensor<br>
> ```
a = np.ones([5, 3]) #建立一个5*3全是1的二维数组（矩阵）
b = torch.from_numpy(a) #利用from_numpy将其转换为tensor
```
Tensor转换为ndarry<br>
```
c = b.numpy() #b为Tensor
```
 **区别** ：Tesnsor可以在GPU上运算
* 使用backward进行反向传播，计算一些导数信息，可通过`.grad`获得梯度

## 3. 单变量线性回归
> y = ax + b
### 3.1 产生数据
```
# linspace可以生成0-100之间的均匀的100个数字
x = Variable(torch.linspace(0, 100).type(torch.FloatTensor))

# 随机生成100个满足标准正态分布的随机数，均值为0，方差为1.
# 将这个数字乘以10，标准方差变为10
rand = Variable(torch.randn(100)) * 10

# 将x和rand相加，得到伪造的标签数据y。
# 所以(x,y)应能近似地落在y=x这条直线上
y = x + rand
```
使用画图工具画图
```
plt.figure(figsize=(10,8)) #设定绘制窗口大小为10*8 inch
# 绘制数据，考虑到x和y都是Variable，
# 需要用data获取它们包裹的Tensor，并专成numpy
plt.plot(x.data.numpy(), y.data.numpy(), 'o')
plt.xlabel('X') #添加X轴的标注
plt.ylabel('Y') #添加Y轴的标注
plt.show() #将图形画在下面
```
结果如下所示<br>
![pic](/images/pytorch01/t2.png)

### 3.2 递归下降
step 1: 初始化参数（随机初始）
```
#创建a变量，并随机赋值初始化
a = Variable(torch.rand(1), requires_grad = True)
#创建b变量，并随机赋值初始化
b = Variable(torch.rand(1), requires_grad = True)
```
step 2: 训练1000次
```
learning_rate = 0.0001 #设置学习率
for i in range(1000):
    ### 下面这三行代码非常重要，这部分代码，清空存储在变量a，b中的梯度信息，
    ### 以免在backward的过程中会反复不停地累加
    #如果a和b的梯度都不是空
    if (a.grad is not None) and (b.grad is not None):
        a.grad.data.zero_() #清空a的数值
        b.grad.data.zero_() #清空b的数值
    #计算在当前a、b条件下的模型预测数值
    predictions = a.expand_as(x) * x + b.expand_as(x)
    #通过与标签数据y比较，计算误差
    loss = torch.mean((predictions - y) ** 2)
    print('loss:', loss.data.numpy())
    loss.backward() #对损失函数进行梯度反传
    #利用上一步计算中得到的a的梯度信息更新a中的data数值
    a.data.add_(- learning_rate * a.grad.data)
    #利用上一步计算中得到的b的梯度信息更新b中的data数值
    b.data.add_(- learning_rate * b.grad.data)
```
> a.expand_as(x) : 将a升级为同x一样维度的张量

step 3: 将训练后的结果展示出来
```
x_data = x.data.numpy() # 获得x包裹的数据
plt.figure(figsize = (10, 7)) #设定绘图窗口大小
xplot, = plt.plot(x_data, y.data.numpy(), 'o') # 绘制原始数据
yplot, = plt.plot(x_data, a.data.numpy() * x_data + b.data.numpy())  #绘制拟合数据
plt.xlabel('X') #更改坐标轴标注
plt.ylabel('Y') #更改坐标轴标注
str1 = str(a.data.numpy()[0]) + 'x +' + str(b.data.numpy()[0]) #图例信息
plt.legend([xplot, yplot],['Data', str1]) #绘制图例
plt.show()
```
以下是拟合结果<br>
![pic](/images/pytorch01/t3.png)
## 3.3 测试
```
x_test = Variable(torch.FloatTensor([1, 2, 10, 100, 1000])) #随便选择一些点1，2，……，1000
predictions = a.expand_as(x_test) * x_test + b.expand_as(x_test) #计算模型的预测结果
print(predictions) #输出
```  
## 附录
[完整代码](https://github.com/oneCoderMan/myCodes/blob/master/PyTorch/linearRegression.py)

source: https://www.shiyanlou.com/courses/1073/learning/?id=5821
