公式无法显示，请忽略或下载查看。
##  Naive Bayes

​        运用朴素贝叶斯在亚马逊食评价数据集上进行分类。

#### 一、假设选择

​        此次处理的数据为文本。文本数据分类标准不明显，不能依靠特征值即确定类别。因此选用文本中的单词为特征，对文本进行分类。先选定部分单词，设立一个单词表，固定单词表中单词的位置，以特征值 $x_\alpha=j$  表示字典中的第 $\alpha$ 个单词在该条文本 $x$ 中出现的次数，依次为特征对文本进行分类。相关公式如下，并取对数做方便计算的推导。
$$
\begin{align}
\operatorname*{argmax}_y P(y | \mathbf{x}) \ \
&\operatorname*\propto{argmax}_c \; \overset{}\\\hat\pi_c\prod_{\alpha=1}^{d} {\hat\theta}_{{\alpha}c}^{x_\alpha} && \\
&=\operatorname*{argmax}_c \;  \log{\hat\pi_c} + \sum_{\alpha=1}^{d}{x}_{{\alpha}}\log{\hat\theta}_{{\alpha}c} && 
\end{align}
$$

​        其中 $\hat\pi_c$ 为类先验概率，$\hat\theta_\alpha$ 为参数估计，描述特征单词的出现概率。

#### 二、文本向量化

​        文本中包含上万单词，不能以所有单词为特征，因而决定选择出现频率靠前的若干单词，形成单词集。需固定每个单词在列表中的位置，形成长度和单词集长度相同的向量，并统计单词集中的单词在语句中出现的次数，以出现次数为向量中该单词的值。

​        代码实现如下：

```
# 特征集
from collections import Counter
wordscount=Counter(allwords)#统计训练集中所有单词，按频数生成字典
vocabdict=wordscount.most_common(300)#以频数最高300个单词为特征集

# 将输入文本列表转换成向量形式
linevec=[0]*len(vocablist)#生成长度和单词集长度相同的向量
for k in line:
    if k in vocablist:#如果语句中单词在单词表中
        linevec[vocablist.index(k)]=line.count(k)#标记单词在该句子中出现次数
```

#### 三、数据集处理

**判别标准**

​        通过对数据集的了解，Summary和Text处信息密集，Score通过1至5数值给出评分，这几点是判别评论性质的重要依据。对评分做简单统计分析，可知70%左右的评分在4分及以上，均分略超过4，即以4及4以上评分指向的评论为正面评论，其余为负面。以此为依据，将评论分别标记为0，1。

```
for i in range(len(labels)):#正面评价label为0，其余为1
	    if labels[i]>=4:labels[i]=0
	    else:labels[i]=1
```

**重复数据**

​        接着对数据进行处理。通过 $$info()$$ 查看数据集情况。可知共568454位用户对568454件商品进行评论，选择去除同一用户对同一商品的相同评价。

```
>>> data=file.drop_duplicates(subset=['ProductId','UserId','Text'],keep='first',inplace=False)
```

**缺失数据**

​        简便起见，选择Summary部分的文本作为测试文本。再去除重复数据后重新查看数据集信息，可知Summary特征有所缺失，但数量较少，选择去除缺失值所在行。

**文本处理**

​        每条评论中重要的信息为单词，需要将不属于单词的部分去除，将含义不大的单词去除，并对单词本身的格式做一定处理。具体操作为去特殊字符、去空格、大小写转换、删除停用词、提取词干。在处理单词时，需下载调取相关词表。

#### 四、程序相关

**训练集、测试集生成**

​         在随机生成训练集和测试集时，固定随机数生成器的种子，便于调试检验。以全部Summary的70%为训练集，30%为测试集。

```
def tt_set(allset):
    import random
    np.random.seed(1000)
    np.random.shuffle(allset)
    testset=allset[0:int(len(allset)*0.3)]#测试集
    trainset=allset[int(len(allset)*0.3):]#训练集
    return[trainset,testset]
```

**数据操作格式**

​        经实践，$for$ 循环遍历列表耗时过长，选择改用 $DataFrame$ 格式对数据进行操作。主要将文本、标签、特征向量整合成 $DataFrame$ 格式数据，数值一一对应。通过对标签列$(labels)$值的判断，划分出正面评价及和负面评价集，再进行计算操作。

**参数训练**

​        主要根据相关公式对训练集进行操作。在正负评价情况下，计算单词表中每个单词出现次数在所有单词中的占比，并进行平滑处理、取对数。详见代码。

#### 四、运行结果和改进方向

**预测准确率**

​        在选定seed为1000和999的情况下生成训练集和测试集，并对不同的单词表长度进行测试，准确率的变化情况如图所示。

![result](https://github.com/1container/Naive_Bayes/blob/main/result.png)

​        可知在特征向量长度为300时，分类器准确率即可超80%；特征向量长达3000时，准确率可超85%，准确率随特征向量长度的增加而增加。但实际运行时，特长向量长达5000时，因算力不够程序报错。长度为300时，运行时间约为30秒，长度至3000前时，运行时间超10分钟，且随向量长度增加准确率替身率也在放缓。85%大致为程序的准确率上线。

**改进方向**

​        在向量长度达3000以后，改变单词表大小对模型精度改进作用不大。要想改进模型，可考虑选用包含信息更多，用语相对更规范的Text文本数据进行训练、判别。或在特征向量的生成中，不只计算词出现次数，也将词性、词与词之间的关系纳入考虑范围。

**参考**：

[NLTK最详细功能介绍 - 农夫三拳有點疼 - 博客园 (cnblogs.com)](https://www.cnblogs.com/chen8023miss/p/11458571.html)
