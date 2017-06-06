# 程序基本框架

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



