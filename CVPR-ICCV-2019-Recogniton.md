## Symmetry-constrained Rectification Network for Scene Text Recognition

[CVPR2019 Paper](http://arxiv.org/abs/1908.01957)

旷视的一篇做不规则文本识别的文章，主要的思路是对STN做了改进，然而其本质依然是先进行文本的矫正，然后再输入识别网络进行识别。

（非文章内容）首先，为什么要这么做，一直都说文本识别有两条路，一条路就是先进行矫正，矫正到一个比较清晰可辨的程度，再进行识别；另一条路就是直接搞一个可识别形变的网络。为什么后者的方法几乎没有，我觉得也是从常理上来分析吧，首先高形变这样的数据集就比较少，所以网络去学正常文本的能力很强，而且大部分实际的文本依然是以一种比较正常的形式出现。其次就是看了效果，发现这篇文章做的一些矫正的效果真的还比较不错。

进入正题。普通的STN使用定位网络去寻找文本的上下各K各key points然后去做TPS插值，文章指出这样的做法忽略了文本的一些先验信息，比如对称性，所以这篇文章的做法是从检测文本的中心线开始，依据检测到的中心线，再去上次上下各对应的关键点，最后一步是相同的TPS

<img src="_assets\scrn-pipeline0.png" style="zoom:80%;" />

### TCL genertae

至于中心线的生成方法，主要是通过先进行单个字符的检测然后再进行逻辑上的推算。

<img src="_assets\scrn-center-line.png" style="zoom:80%;" />

文章将一个文本实例定义为由多个字符组成的序列$A=\{A_1, A_2,...,A_m\}$，并且每一个字符$A_i$拥有一个BOX$B_i$围绕该字符。并且对于每一个instance，提出了其需要的geometric attributes：

- $C=\{c_{head},c_1,c_2,…,c_i,c_{tail}\}$，中心店的集合，其中head和tail代表最左和最右的box的两条边的中心点
- 对于每一个$B_i$，都有一个$geo_i=\{c_i;s_i,\phi_i;\theta_i\}$
- $c_i$代表中心店坐标
- $s_i$代表该box的尺寸，数学上数值为高度的一半
- $\phi_i$代表该box的旋转偏移量，数学上为中线相对于水平的角度
- $\theta_i$代表相对于下一个点的旋转角度

作者设计了一个轻量级的CNN网络来对这些属性进行预测，$F=\{f_1,f_2,...,f_6\}$，其中$f_1$为probability，$f_2$为$s$，3-6代表$cos,sin(\theta, \phi)$，角度值会经过归一化来保证其满足数学原理。

矫正的整体流程如下：

<img src="_assets\scrnrectify.png" style="zoom:80%;" />



- 经过一个属性提取网络后，会首先进行均匀采样，获得等距离的CP
- 随后根据公式进行FP的计算：

<img src="_assets\scrn-rectify-2.png" style="zoom:80%;" />

本文的一个新式的创意在于调整了原先的TPS关键点的生成方式，通过分析text的对称性，找到整个instance的中心线，然后通过计算一定的偏转角度来找到更加准确的FPs。

同时通过计算character size以及偏转角来得到关键点也是相当关键的，因为按照一般的方向性来计算的话，其实是无法适应一些扭曲比较严重的文本的，效果对比如下：

<img src="_assets\scrn-compare.png" style="zoom:67%;" />

## ESIR: End-to-end Scene Text Recognition via Iterative Image Rectificatio

[Paper](https://arxiv.org/abs/1812.05824.pdf)

设计了一种基于识别效果来驱使的，end-to-end trainable scene text recognition system (ESIR)能够迭代的去除透视变换而导致的失真问题，从而最终将一张图片矫正为“正面平行视图”

<img src="_assets\esir-total.png" style="zoom:80%;" />

### Line-Fitting Transformation

该文同样也是通过寻找中心线来进行关键点的选取，对于文本而言，其中心线一般来说方向是在水平方向上的一条平滑的直线或曲线，故作者使用了多项式方程来拟合这么一条直线，同时定义了许多小线段往中心线的两侧衍生来代表整个instance的范围。

- polynomial

<img src="_assets\esir-poly.png" style="zoom:80%;" />

- line segments

<img src="_assets\esir-seg.png" style="zoom:80%;" />

文中定义K=4，L=20，那么由于可以认为文本的两侧的seg长度是一样长的，所以整个localization网络需要学习的参数一共是3L+K+1个。

<img src="_assets\esir-ln.png" style="zoom:67%;" />

整个localization网络的结构也很简单，一旦得到所有的参数后，将由TPS进行矫正

### Iterative Rectification

提出了一个重复进行矫正的方案，由流程图里面可以看出这个的亮点在于每次矫正不是依托于输入的矫正图片进行矫正，而是从原图上进行矫正，这么做的原因在于

- 每一个进行矫正后会损失边缘的一些像素，并且由于插值算法，整个图片会失真，所以不断在矫正后的图片上进行矫正会导致整体的信息丢失
- 这里的iter轮数是预设的
- 同时该训练是不需要进行任何多余的标注的，所有的效果都是由最终的识别loss回传驱使

```tex
It should be noted that the training of the localization network does not require any extra annotation of fitting lines but is completely driven by the gradients that are back-propagated from the recognition network. The underlying principle is that higher recognition performance is usually achieved when scene text distortions are better estimated and corrected.
```

<img src="_assets\esir-arch.png" style="zoom:67%;" />

### L和N的参数设置对比

<img src="_assets\esir-L-N.png" style="zoom:50%;" />

