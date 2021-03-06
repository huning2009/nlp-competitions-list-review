## WSDM2019-虚假新闻检测比赛

前言: 这篇文章主要以第二名为讨论对象，来自美团NLP团队。同时会对比第一名和第三名的方案。此外，给出了SemEval2019的答案分类任务上的第一名方案，和该比赛联系较多。

### 一.背景

从标题来看，做成一个二分类问题更加地直接，而本届比赛的思路则不同。前者二分类问题的输入是一个文本(新闻标题/新闻文本/新闻标题+新闻内容) ，而比赛的数据输入是两个文本(新闻标题)，输出是三分类的标签(一致/不一致/无关)。这样的话，显然自然语言推理(NLI)的任务中的方法自然适合用于该比赛。

### 二.数据介绍

训练样本量为32万，测试样本量为8万。由于输入是新闻标题，长度在20-100词之内。既然是分类问题，多数情况下要考察不平衡现象。三类样本的占比如下：

|一致|不一致|无关|
|------|------|------|
|29.0%|2.6%|68.4%|

由上表可以得出结论：类别严重不平衡。

### 三.数据预处理和数据增强

#### 1.数据预处理

结合数据特点，使用各种数据预处理方法。例如繁简转换，停用词过滤等，相关技术可以参考[博客](https://zhpmatrix.github.io/2019/03/08/preprocess-augmentation-in-nlp/)。

#### 2.数据增强

##### (a)标签传播

标签传播的思想作为一种数据增强手段，用处较多。在拍拍贷-问题相似度比赛中，仍旧可以采用该方法做数据增强。

假设A和B是一致的，A和C是一致的，显然B和C应该是一致的；

假设A和B是一致的，A和D是不一致的，则B和D也是不一致的；

##### (b)位置交换

A和B是一致的，则B和A也是一致的。

### 四.模型选择

BERT为主，辅助SVM，LR，KNN，NB

### 五.策略设计

模型融合，设计三层。第一层：25个BERT基模型；第二层：SVM/KNN/NB等传统数据挖掘模型；第三层：LR模型

### 六.评估指标

带有权重的分类准确率。其中，具体权重分配如下表：

|一致|不一致|无关|
|------|------|------|
|1/15|1/5|1/16|

结论：少数类样本，权重大。通过这种方式，引导模型去关注少数类样本或者说希望选择一个对少数类关注度较高的模型。

### 七.线上结果

|Model|Weighted Acc on Private LB|
|------|------|
|单模型|0.86750|
|25个BERT平均|0.87700|
|25个BERT加权平均|0.87702|
|三层模型融合|0.88156|

### 八.反思

官方提供的中文BERT是在中文维基百科语料上训练得到的，语料数据和新闻语料是有区别的。能够将中文BERT继续在新闻数据上训练，提升中文BERT对新闻数据的表征能力。实际上，就在写这篇文章的当日，百度放出了ERNIE，或许基于ERNIE可以在该比赛基础上进一步提升。关于ERNIE的讨论可以参照知乎的一个讨论，[如何评价百度新发布的NLP预训练模型ERNIE？
](https://www.zhihu.com/question/316140575/answer/624096104)，其中自己给出了一个回答如下：

```
还没来得及读代码，从官方README文件，PaddlePaddle/LARK，读到的信息如下：

改进：

（1）mask的粒度：字(BERT)->词(ERNIE)，不过输入仍旧是字。

（2）语料：中文维基百科(BERT)->百科类+新闻资讯类+对话类(ERNIE)。

意义：

（1）个人觉得更加符合中文应用场景（分词的需求）。

（2）官方放出了代码+预训练模型+训练数据（估计民间PyTorch的wrapper，PyTorch的实现马上就会来的，不要着急）。

（3）对语义知识建模的手段相信可以继续深化，此处赞刘知远老师的回答。

总之，是良心的工作，赞。

```

### 九.后续方案讨论

![img](http://wx2.sinaimg.cn/mw690/aba7d18bgy1g14wp6x1eoj20n30a2q4n.jpg)

从上图可以基本看到，该比赛是头条主办的。同时上图给出了第一名和第三名的答辩题目。第一名和第三名仍旧是基于BERT的方案设计，第一名加了一些手工特征。从三者的分享方案可以看到，两个句子作为输入的分类问题，比如句子相似度匹配，比如自然语言推理等任务，对相似性传递的分析策略较多，也是一个比较有趣的点。同时，该任务也再次证明了BERT的强大。

### 十.相关补充(SemEval2019 Task 8 on Fact-Checking in Community Forums)

该[赛道](https://competitions.codalab.org/competitions/20022)分为两个子任务，分别是问题分类和答案分类。其中，答案分类赛道上，国内的汽车之家的团队拿到了冠军，冠军方案如下：

![img2](http://wx4.sinaimg.cn/mw690/aba7d18bgy1g15osw73maj20gz07mgou.jpg)

三个虚线框分别代表三种方案，简单的基于BERT做FineTuning时，只需要T\[CLS\]，但是融合T\[CLS\]到T\[SEP\]再到TN，也是一种思路，类似RNN的HiddenState的融合策略。该方案的线上结果是**82%**。

### 参考：

1.[虚假新闻检测数据集](https://blog.csdn.net/Totoro1745/article/details/84678858)

2.虚假新闻检测任务的综述文章，《Fake News Detection on Social Media: A Data Mining Perspective》

3.虚假新闻检测的一个专用平台，[Fake News Challenge](http://www.fakenewschallenge.org/)

4.[第二名方案分享-来自美团](references/WSDM2019_Fake_News_Classification/report2.pdf)

5.[第一名方案分享](references/WSDM2019_Fake_News_Classification/report2.pdf)

6.[第三名方案分享](references/WSDM2019_Fake_News_Classification/report2.pdf)

7.[SemEval2019-事实分类-汽车之家方案](https://tech.china.com/article/20190307/kejiyuan0129249545.html)

8.[Lessons Learned from Applying Deep Learning for NLP Without Big Data](https://towardsdatascience.com/lessons-learned-from-applying-deep-learning-for-nlp-without-big-data-d470db4f27bf)

小数据场景下的分类Trick总结，有些Trick很有意思。

