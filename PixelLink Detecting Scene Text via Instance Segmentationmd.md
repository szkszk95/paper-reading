

## PixelLink: Detecting Scene Text via Instance Segmentation

[Paper]( https://arxiv.org/abs/1801.01315 )

[Github]( https://github.com/ZJULearning/pixel_link )

文章首先分析了现有的方法的一些问题

- 现有的文本检测的方法基本是基于预测+回归这样传统的目标检测的两个部分组成
- 但是回归不是必须的，因为box这个步骤可以被语义分割来替代
- 所以提出了一个基于语义实例分割的方法，来替换掉现有方法里面的回归

OK，个人觉得，思路是好的，但是解释起来有点僵硬，我觉得这个回归其实还是一个比较不错的思路吧。

### 整体架构

- 网络由backbone部分提取图片的特征信息
- 经由FPN得到缩小后的特征图后，对每一个像素位置进行2通道的text/non-text预测分类以及8邻域的link预测
- 经由卡阈值去掉negative的pixel以及link
- 将pixel根据link相连，形成一个个实例区域
-  基于Opencv的minAreaRect，用最小外接矩形，把每一个连通分量转变成boundingbox，并且可以包含倾角
- 去除噪声区域

![](_assets\pixel-framework.png)

由上面的8个方向的pixel score map可以看出来，有一些不好分割的实例样本通过像素级别的语义分割，可以进行比较好的区分。

#### backbone

backbone基本与EAST一致。

![](_assets\pixel-backbone.png)

### Data augment

- 随机旋转，概率0.2，旋转0，90，180， 270度
- 随机crop
- 拉伸0.5-2
- 全部resize成512x512
- 短边少于10个像素的文本去掉，少于20%被保留的去掉
- 被去掉的文本loss反传时为0

### Loss

![](_assets\pixel-loss.png)

Pixel分类的损失如上面式子所示，L表示softmax_cross_entrop，W表示预测的pixel分类的权值矩阵，r表示负正样本比例，论文中r=3，S表示每个文本实例的面积。最终，当文本实例的面积比较大，占的loss损失就会被相对拉小，文本实例的面积比较小，占的loss损失就会被相对拉大。这样做的好处就是使得小的文本实例不会被大的文本实例的loss掩盖掉，也可以有loss回传。

OHEM

同时pixel分类任务还使用了OHEM策略，每次回传S（正样本实例像素和）+r*S（负样本像素和）的loss，更加有利于分类任务的学习。这点改进比EAST中所有像素的loss都回传可以得到更好的分类结果，而不是像EAST那样，一个实例中间的像素分类的好，边缘的像素分类的差的情况 

### Discussion

- pixellink使用更少的训练时据以及训练世界可以取得更加优秀得效果，原因：
  - 对感受野的需求不同，对比seglink以及EAST
  - 任务难度，其实取找box的四边或者四点，要比判断自己周围的是不是一个instance要难很多

- 训练图片的尺寸很重要512>384

- 去噪很重要
  - 这里主要由于使用了基于连通域的方法进行文本像素汇聚，而该方法对噪声比较敏感，最终就会产生一些比较小的false positive。论文基于长度，宽度，面积，长宽比等信息对false positive连通域进行了去除。主要方法就是，短边小于10个像素，或者面积小于300就会当做错误的连通域进行去除。


- instance balance很重要

  