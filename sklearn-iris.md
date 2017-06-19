### 数据集介绍

> 数据四个维度：Sepal花萼长，宽，Petal花瓣长，宽
>
> 分类target：Iris Setosa（山鸢尾）、Iris Versicolour（杂色鸢尾），Iris Virginica（维吉尼亚鸢尾）



```py
#KNN
#
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



