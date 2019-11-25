

## Chinese Street View Text: Large-scale Chinese Text Reading with Partially Supervised Learning

[CVPR2019](http://arxiv.org/abs/1909.07808)

文章主要两个关键贡献：
- 作者发布了一个公开的C-SVT数据集，其中包含3w张带有全标注的图片以及40w张带有弱标签的图片
- 为了利用这样的弱监督信息，作者提出了一套弱监督的训练框架Online Proposal Matching module (OPM)，使用数据集中的弱标记方式，加强了模型的性能。

<img src="_assets\svt-result.png" style="zoom:80%;" />

可以看出加入了弱监督协助训练的效果要明显优于只用全监督信息的效果。

### 中英文的区别
- 中文字符的种类要远远多于拉丁字符
- 但是现有的中文数据集的量是十分有限的

### C-SVT
- full annotation：
  -	29966张图片，其中文本包括水平垂直或curve的
  -	划分了train，valid，test，4:1:1
  -	以4边形或多边形标注
  -	29966张图片，243537个文本线，1509256个字符
<img src="E:\ocr-files\paper-reading\_assets\svt-tab2.png" style="zoom:80%;" />

- weak annotations：
  - 弱监督的图片仅仅包含文本的labels，以及他大概出现的范围
  - 包含500w的中文字符
  - 主要用于优化识别性能

### Text Reading

<img src="E:\ocr-files\paper-reading\_assets\svt-frame.png" style="zoom:80%;" />

#### Text Detection Branch

典型的FPN网络，使用quadrangles作为box的表征方式

#### Perspective RoI Transform

将检测的得到的proposal区域对应的特征进行提取以及转换

#### Text Recognition Branch

CRNN+Attention进行文本区域的识别

#### Partially Supervised Learning Previous

文章的核心内容是进行弱监督学习的这一个步骤，主要提出了Online Proposal Matching，使用弱标记数据进行辅助训练，主要的思路如下：
- 通过检测模块得到一张图片上的许多proposal
- 经由CNN-RNN-decoder得到对应的特征$F_s^W$
- 由弱监督数据中的label得到其character embedding特征
- 由attention decoder将两个特征转换到相同的域中
- 对比所有的proposal的特征与label特征的欧氏距离，设定一个阈值，划分正负样本

<img src="E:\ocr-files\paper-reading\_assets\svt-opmloss.png" style="zoom:80%;" />

