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



