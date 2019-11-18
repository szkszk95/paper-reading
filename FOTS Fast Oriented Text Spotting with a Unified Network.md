## FOTS: Fast Oriented Text Spotting with a Unified Network

[Paper]( https://arxiv.org/pdf/1801.01671.pdf )

[Github](https://github.com/jiangxiluning/FOTS.PyTorch)

FOTS是一个联合了检测以及识别end-to-end的联合文字识别方案，其中最为核心的部分是ROIRotate

ROIRotate通过仿射变换，将不对其的检测区域特征变换为对齐的固定尺寸的特征，从而

- 使得检测部分和识别部分能够共用特征提取层，大大缩短了整体pipeline的时间消耗
- 同时也能使得网络提供给识别模块的特征不仅仅只局限于抠出来的文字部分，能够有整幅图的信息在里面。

### Pipeline

网络整体的设计如下：

- 通过特征提取层提取得到CNN特征
- 通过检测分支得到预测的文本框
- 通过RoIRotate以及文本框从CNN特征中抽取对应文字部分的对齐特征
- 将对齐特征输入文字识别模块得到识别结果

![](_assets\FOTS-pipeline.png)

### Text Detection

检测模块的设计与EAST基本一致，采用了FPN，得到$1/4$原图大小的分割级别的特征图。文中上采样采用的是双线性插值。

![](_assets\FOTS-detection.png)

### RoIRotate

首先，这一步是让不同尺寸的文本区域的特征图能够转变称为同一个尺寸，并且是直接从检测的特征图中得到结果。

公式如下：

- $M$对应的是仿射变换矩阵，能够操作旋转缩放平移
- $x,y$对应在原特征图中的坐标值，
- $t,b,l,r$分别代表该点到达检测（或真实）文本框上下左右边的距离，$\theta$则代表文本框的偏转角度
- $h_t,w_t$是指定的希望得到的aligned-feature map的尺寸，文中设为8x8



<img src="_assets\FOTS-affine.png" style="zoom:50%;" />

在计算得到了新特征图的position以及旧特征图的转换关系之后，通过双线性插值进行映射，即可得到新的文本特征图。

### Text Recognition

由于Detect部分的特征仅仅只shrink了两次，还不足以提取出很好的特征，所以在recognition部分会先进行一系列的Conv以及1x2的pooling，随后将特征输入LSTM以及CTC模块进行loss的计算。

### Details

- 在imagenet上与训练的

- 训练时先用的Synth800k的数据，训了10轮，然后使用真实数据训练到收敛

- 不同的场景下使用不同的训练集

- Data augment：

  ```reStructuredText
  Data augmentation is important for robustness of deep neural networks
  ```

  - First, longer sides of images are resized from 640 pixels to 2560 pixels.
  - 所有的图片随机旋转正负10度
  - 保持宽度不变，高度随机拉伸0.8-1.2倍
  - 从大图中随机裁剪640x640的作为输入图像
  - OHEM，p:n=1:3，
  - 测试的时候经过NMS再进入recognition