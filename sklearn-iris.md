#### 决策树

用信息熵为度量，构造一棵树熵值下降最快的树。目前主流的算法有

* ID3 : 信息熵增益最大
* C4.5  :  克服ID3的缺点，信息增益比最大
* CART ：回归树与分类树

```
熵的定义，我们引用信息学的定义比较好理解，信息熵表示对一个随机变量最有效编码时，所需要的最短编码长度。
举个知乎上的例子。考试选择题作弊，ABCD四个选项，如何设计？
假设ABCD的四个选项的概率一样1/4
则根据熵的定义
H = -Σ p * log(p)，H = -4*(1/4)*log(1/4) = 2，2位即可，和直觉一直 00,01,10,11表示ABCD
然后改变定义，假设ABCD的概率分布式 1/8, 1/8, 4/8, 2/8
根据定义可得 H = 1.75，平均编码长度是1.75bit，其实就是答案C的概率高，编码可以用短点的哦。
所以给定一个数据集D，由于分类结果已标注，我们可以计算H(D)，然后决策树如何建立呢？
我们依据是，对于特征Fi，我们计算H(D|Fi)条件熵，然后得到熵的增益，越大表示该特征使得信息熵下降最快！
或者说，熵增表示越混乱，我们的目的是不混乱，就是决策树到达叶子时希望都是一个类别，所以必然选择下降最快的方式。
条件熵的计算
H(Y|X) = Σ p(i)*H(Y|X=x(i))
举个例子，X是天气，Y是开花情况，1表示开花
晴天开花 1 1 0 0 0
阴天开花 1 1 1 1 
雨天开花 1 1 1 0 0
H(Y) = (-)9/14*log(9/14) + (-)5/14*log(5/14)
H(Y|X) = 5/14*( -2/5*log(2/5)-3/5*log(3/5) ) + 4/14*( -4/4*log(4/4) - 0/4*log(0/4) ) + ...
g(Y|X) = H(Y) - H(Y|X) 信息增益，这个就是天气这个特征的信息增益。

C4.5的要修正的是，如果一个特征的取值够多，则一定能把分类分开的其实。。。但是这样就木有意义了，比如上面开花的例子，
我们增加一个维度，叫ID。。。ID = [1 14]
H(Y|ID) = 1/14*(1*log1) + ... = 0， g(Y|ID) = H(Y) 最大了。。。但是这个特征明显无意义。。。所以。。。
要对分支过多进行惩罚，引入属性内部信息
IntI(D,F) = Σ |Di|/|D| * log(|Di|/|D|)，特征数越多，该值越大
信息增益比 gr(D|F) = g(D|F) / IntI(D,F)

以上两个方法都是基于信息熵的，涉及了对数运算，慢。。。所以CART选择一些近似模型，保留信息熵模型的同时减少计算
基尼数 Gini(p) = Σ p(k)*( 1 - p(k) ) = Σp(k) - Σp(k)^2 = 1 - Σp(k)^2 ，预算变得简单了
回归树 ，待定。。。是处理回归模型，使用方差来度量的
```

### 集成学习

```
核心思想就是多个分类器组合在一起
1. 训练集合如何划分，可以有放回抽样，均等，根据上一个分类器的学习结果来划分，分错的怎么处理等等
2. 投票规则，均等，有权重

随机森林：在随机森林中，我们将生成很多的决策树CART。当在基于某些属性对一个新的对象进行分类判别时，
随机森林中的每一棵树都会给出自己的分类选择，并由此进行“投票”，森林整体的输出结果将会是票数最多的分类选项；
而在回归问题中，随机森林的输出将会是所有决策树输出的平均值。
```

### !! 降维算法

```
寻找映射函数，改变维度，或者说寻找有意义的特征
x -> y

PCA 主成分分析，线性降维
```

### 深度学习



### 数据集介绍

> 数据四个维度：Sepal花萼长，宽，Petal花瓣长，宽
>
> 分类target：Iris Setosa（山鸢尾）、Iris Versicolour（杂色鸢尾），Iris Virginica（维吉尼亚鸢尾）

```py
#KNN - k-nearest neighbors
#Pros : 无需训练
#Cons : 计算大，评分慢，可解释性差
#step 1 : 找出预测对象A的距离最近的k个邻居
#step 2 : k个邻居投票决定，对象A属于哪个分类
from sklearn import neighbors, datasets

#训练数据
iris = datasets.load_iris()
target = iris.target
data = iris.data

#训练
knn = neighbors.KNeighborsClassifier(n_neighbors=5,weights="uniform")
knn.fit(data,target)

#预测
X_pred = [3, 5, 4, 2]
result = knn.predict([X_pred, ])

print(iris.target_names[result])
#['versicolor']
print(knn.predict_proba([X_pred, ]))
#[[ 0.   0.8  0.2]]
```

```py
#Kmeans
#Step 1 : 初始化k个中心
#Step 2 : 计算所有训练集到k中心距离并标号
#Step 3 : 根据2中数据重新计算k个中心
#Step 4 : 反复执行2，3步骤，直到收敛条件

from sklearn.datasets.samples_generator import make_blobs
X, y = make_blobs(n_samples=300, centers=4,
                  random_state=0, cluster_std=0.60)
plt.scatter(X[:, 0], X[:, 1], s=50);

from sklearn.cluster import KMeans
est = KMeans(4)  # 4 clusters
est.fit(X)
y_kmeans = est.predict(X)
plt.scatter(X[:, 0], X[:, 1], c=y_kmeans, s=50, cmap='rainbow');
```

![](/assets/下载.png)![](/assets/下载 %281%29.png)

#### 



