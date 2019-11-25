## Symmetry-constrained Rectification Network for Scene Text Recognition

[CVPR2019 Paper](http://arxiv.org/abs/1908.01957)

旷视的一篇做不规则文本识别的文章，主要的思路是对STN做了改进，然而其本质依然是先进行文本的矫正，然后再输入识别网络进行识别。

（非文章内容）首先，为什么要这么做，一直都说文本识别有两条路，一条路就是先进行矫正，矫正到一个比较清晰可辨的程度，再进行识别；另一条路就是直接搞一个可识别形变的网络。为什么后者的方法几乎没有，我觉得也是从常理上来分析吧，首先高形变这样的数据集就比较少，所以网络去学正常文本的能力很强，而且大部分实际的文本依然是以一种比较正常的形式出现。其次就是看了效果，发现这篇文章做的一些矫正的效果真的还比较不错。

进入正题。普通的STN使用定位网络去寻找文本的上下各K各key points然后去做TPS插值，文章指出这样的做法忽略了文本的一些先验信息，比如对称性，所以这篇文章的做法是从检测文本的中心线开始，依据检测到的中心线，再去上次上下各对应的关键点，最后一步是相同的TPS

<img src="E:\ocr-files\paper-reading\_assets\scrn-pipeline0.png" style="zoom:80%;" />

至于中心线的生成方法，主要是通过先进行单个字符的检测然后再进行逻辑上的推算。

<_assets\scrn-center-line.png" style="zoom:80%;" />

矫正的整体流程如下：

<img src="E:\ocr-files\paper-reading\_assets\scrnrectify.png" style="zoom:80%;" />



