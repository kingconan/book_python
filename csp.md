## CSP

###### constraint satisfaction problem

```
CSP问题的描述

#1 a set of variables X={x1,...,xn},
#2 for each variable xi, a finite set Di of possible values (its domain),
#3 and a set of constraints restricting the values that the variables can simultaneously take.

#1 n个变量
#2 每个变量的阈值D
#3 约束C，一组变量之间不能出现的情况
```

```
例子
#1 8皇后问题，变量8个，每个变量的Domain为8个位置，限制C是任意两组不能在横线，斜线
#2 地图着色问题，变量n个区域，变量取值4（4色问题），限制C相邻区域颜色不同
```

```
BP暴力搜索
```

```
Forward Checking

当给某个变量赋值后，检查所有含有该变量的C，更新其中对应变量的D。
例如Ci = (x1=v1,x2=v2)，假设x1=v1了，那么x2的D2中，备选变量里要剔除v2了，根据Ci。
该检查如果发现某个xi的Di为空，则表示无解，停止继续搜索，回溯

不足：只考虑了当前被赋值变量与其他变量（current variable & future variables)之间的约束关系。
没有考虑到全部约束关系，why not
```

```
Maitaining Arc Consistency（弧一致性。。。什么名字。。）

检查所有约束
当对某一变量xi赋值后，导致xk的D发生变化的话，那就要重新检查含xk的约束，是否满足条件。
如果不满足则可停止当前xi的赋值，回溯别的值

严肃定义
arc表示的是consistency graph里的有向箭头，例如arc(x1->x2)表示对于x1可取的所有值，x2中都能找到满足约束的值
也就是说，如果不一致，一定有一个取值x1=value导致x2取任何值都不合法，那么value在当前状态下，不该在x1的domain中，
可以删除。然而删除后，指向x1的arc可能会出现不一致，所以继续检查。

AC-3初始将所有arc放进队列，如果队列不为空，就检查每组arc，看看是否一致，如果不一致，通过删除导致不一致的值，
继续检查下，如果某个domain为空，则说明当前状态无解，否则当前状态可能有解。

所以整个算法就是在BP搜索过程中，用AC-3判断一下当前的约束是否合法（domain为空）




```



