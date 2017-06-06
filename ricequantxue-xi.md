# 介绍

* 策略研究下可以随便试各种python，提供了注释，代码和结果展示的混合视图，可以用来教学

* 我的策略是真正执行回测的地方，有些库的使用受到限制，比如plot之类的



# RQAlpha

搭建本地开发环境

```py
pip install rqalpha #http://rqalpha.readthedocs.io/zh_CN/latest/intro/overview.html

```

# 回测程序基本框架

```py
#Ricequant使用的是python3语法
def init(context):
    #初始化函数，context会在每个方法中传递

def before_trading(context, bar_dict):
    #每天交易开始前调用，当天仅调用一次

def handle_bar(context, bar_dict):
    #算法在这里，bar数据变化会触发此函数

def after_trading(context, bar_dict):
    #每天收盘后被调用，15:30
```

## numpy

```py
#numpy数组是一个存储了同一类型的数值网络
import numpy as np


a = np.array([1, 2, 3], [21, 22, 23]) #创建 2x3 的矩阵
b = np.eye(2) #单位矩阵
c = np.random.random(2,2) #随机的2x2矩阵
d = a[:2, 1:3] #切片，行选择0,1,列选择1，2的子矩阵
#+ - * / 是对应元素的四则元素，不涉及矩阵运算
#矩阵的乘法使用 x.dot(y) 或者 np.dot(x, y)

#好吧，原来有单独矩阵的构造方法

m1 = np.mat("1 2 3; 4 5 6"] #使用字符串，mat是matrix别名
m2 = np.matrix([1, 2, 3], [4, 5, 6]) #使用序列
```

## pandas

```py
from pandas import Series, DataFrame
import pandas as pd

#pandas的series是一维数组的对象，和numpy的区别在于扩展了类型，不要求相同
s = Series(np.random.randn(5),index =['a','b','c','d','e']) #get_price返回的是Series
#输出s
a   -2.895114
b   -1.231825
c    0.471328
d   -1.287756
e    1.475353
dtype: float64

#data frame是表类型数据结构，get_price返回多个股票信息时时表结构，纵坐标为时间，横坐标为股票代码
d = {'one' : [1., 2., 3., 4.],
     'two' : [4., 3., 2., 1.]}
df = pd.DataFrame(d)
#输出df
      one     two
0     1.0     4.0
1     2.0     3.0
2     3.0     2.0
3     4.0     1.0
```

## Scipy

```
#提供了比较高级的函数，用于统计，数值计算
```





