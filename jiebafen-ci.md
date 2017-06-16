# jieba

```
https://github.com/fxsjy/jieba
```

### 分词

三种模式，自定义字典，动态调整字典

```py
# -*- coding: utf-8 -*-
import jieba


seg = "苏梅，啊苏梅，我要去海岛，路的山使我们的嘎嘎";
seg_list = jieba.cut(seg)  # 默认是精确模式
print(", ".join(seg_list))

jieba.load_userdict("mydict")
seg_list = jieba.cut(seg)  # 默认是精确模式
print(", ".join(seg_list))
```

### 关键词提取

```py
import jieba.analyse
```

> TF : 词频，越多表示重要
>
> IDF ：逆文档频率， log\(语料库中文档总数 / \(包含该词的文档数 + 1\) \)，当所有文档都出现过该词，为0，表示该词常见
>
> TF-IDF = TF x IDF

> TextRank :  基于PageRank，节点用词表示，边如何定义？首先定义窗口大小为k，文章由m个句子组成，将每个句子分成窗口为k的词组序列\[w1,w2,...,wk\]，\[w2,w3,...,wk+1\]如是，如果词wi和wj在一个窗口内，则表示wi与wj无相无权边。然后进行迭代即可。无向边就相当双向有权边处理。貌似窗口为2时效果可以。

```
PageRank : 基于一个假设，网页A的重要性记为PR(A)，如果它链接到m个网站，则每个被链接的网站接受到来自A的重要性为PR(A)/m
基于以上原则可以写出一个迭代算法
P = M x P，其中P是每个网页的初始重要性向量，M是根据PageRank链接原则得到的矩阵，不断的迭代计算左侧P然后再带入右侧继续，
最终希望收敛。

然而实际会有问题，所以修正算法如下
S(Vi) = (1-d) + d*Σ（j∈In(Vi)） S(Vj)/Out(Vj)，d表示阻尼系数，一般0.85。意思是所有指向Vi的网页中，贡献的权重之和再乘以
阻尼系数d，d可理解为点击到Vi的概率，依据这个得到矩阵M，然后进行迭代计算

```

```
推荐系统
基于内容：item A和item B相似，user 1 喜欢 item A，推荐 item B 给user 1。相似指的是属性类别相似
协同过滤：基于用户的，如果user 1和user 2相似，则可以把user 2喜欢的推荐给user 1，基于项目的，原则与基于内容一致，但是相似定义不同，
是指喜欢item A的一般也喜欢item B，则A与B相似。 基于模型，训练一个模型出来。
```

### 词云WordCloud

* 注意中文问题显示乱码，通过设置中文字体路径来解决
* 图片mask，背景要纯白才可以，然后配合word\_cloud.recolor函数，可以自定义云图颜色
* 比较好的参考 `https://zhuanlan.zhihu.com/p/20436642`

![](/assets/屏幕快照 2017-06-14 下午5.38.32.png)

### code

```py
# -*- coding: utf-8 -*-
import matplotlib.pyplot as plt
from wordcloud import WordCloud, ImageColorGenerator
from scipy.misc import imread
import jieba.analyse

seg = "苏梅，啊苏梅，我要去海岛，路的山使我们的嘎嘎";
seg_list = jieba.cut(seg)  # 默认是精确模式
print(", ".join(seg_list))

jieba.load_userdict("mydict")
seg_list = jieba.cut(seg)  # 默认是精确模式
print(", ".join(seg_list))


content = open("test.txt").read() #SCI小说
color_mask = imread("img1.jpg")
wordlist_after_jieba = jieba.cut(content, cut_all = True)
wl_space_split = " ".join(wordlist_after_jieba)

wc = WordCloud(font_path="/Library/Fonts/Xingkai.ttc",
                         background_color='white',
                         max_words=2000,
                         max_font_size=40,
                         mask=color_mask
                         ).generate(wl_space_split)

image_colors = ImageColorGenerator(color_mask)

plt.imshow(wc.recolor(color_func=image_colors, random_state=3))
plt.axis("off")
plt.show()
```

```py
#分词结果
Building prefix dict from the default dictionary ...
Loading model from cache /var/folders/q8/g5z2qg1140s8fb1gjnrfbhn40000gn/T/jieba.cache
Loading model cost 0.378 seconds.
苏梅, ，, 啊, 苏梅, ，, 我要, 去, 海岛, ，, 路, 的, 山, 使, 我们, 的, 嘎嘎
Prefix dict has been built succesfully.
苏梅, ，, 啊, 苏梅, ，, 我要, 去, 海岛, ，, 路的山, 使, 我们, 的, 嘎嘎
```



