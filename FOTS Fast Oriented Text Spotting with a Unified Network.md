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



<img src="_assets\FOTS-affine.png" style="zoom:50%;" />

### Text Recognition



