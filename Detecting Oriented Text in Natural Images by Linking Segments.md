## Detecting Oriented Text in Natural Images by Linking Segments

[Seglink](https://arxiv.org/pdf/1703.06520.pdf)

Seglink的我认为是基于ctpn的一个有力的改进，其思路大致上来说跟ctpn很接近，都是通过检测text的一部分，然后再将其拼接起来，达到对整个文本区域的检测。

seglink提出，将一个文本检测的目标拆成两个部分：

- segment：文本的一小块区域，覆盖检测目标的某一块区域
- link：用于将属于同一个文本的不同的segment相连接

与ctpn不同之处在于

- 没有采用fix宽度的box，而是让每一个anchor自行去预测每一个segment
- 设计了link来对segment进行各个方向上的连接，来达到对于不同旋转方向的文字的检测
- 通过link是与segment同时训练得到的，从而略去了后续连接的处理部门。

### 整体架构

seglink是基于SSD的检测框架来实现的，其主题结构也与ssd保持一致

<img src="_assets\seglink-arch.png" style="zoom:80%;" />

首先对于不同尺度的feature map，都会出一个预测的segment结果，然后对于每一个feature map上的结果，再进行同层、不同层的link。

同一层的link很好理解，就是将同一张feature map上的属于同一个文本的segment相连接。

设计不同层的目的也是为了让整个结果更加鲁棒，因为一个文本框也有可能被不同层的feature map捕捉到，仅仅使用单层的结果会丢失信息，通过不同层之间的link。可以将所有feature map上的segment作为一个整理来处理

#### Segment Detection

seglink中对于一个box的标记定义为$s=(x_{s},y_{s},w_{s},h_{s},\theta_{s})$。

对于特征图$F_{l}$，令其尺寸未$(W_{l},H_{l})$，原图尺寸大小为$(W_{I},H_{I})$，对于特征图中的每一个点，会生成一个defalut box，那么对于特征图中位置为$(x, y)$的点，其对应的defalut box的坐标为：
$$
x_{a} = \frac{W_{I}}{W_{l}}(x+0.5) \\
y_{a} = \frac{H_{I}}{H_{l}}(y+0.5)
$$
同时default box的高和宽都会被设置成一个常量$a_{l}=\gamma \frac{W_{I}}{w_{l}}$，文中$\gamma=1.5$。就是说每一层feature map都会选择不同长度box宽度。

那么针对Segment的预测器输出7通道的信息，分别是两通道的score map，和5通道的位置偏移量$(\Delta x_s,\Delta y_s,\Delta w_s,\Delta h_s,\Delta \theta_s)$。根据这些值即可计算segment的实际位置：
$$
x_s=a_l \Delta x_s+x_a \\
y_s=a_l \Delta y_s+y_a \\
w_s = a_l exp(\Delta w_s) \\
h_s = a_l exp(\Delta h_s) \\
\theta_s = \Delta \theta_s
$$

#### Within-Layer &  Cross-Layer Link

<img src="_assets\seglink-link.png" style="zoom: 80%;" />

在得到各个layer的segment之后，需要分析他们之间的连通关系，

- Within-Layer：层内的连接由16个通道表示，分别表示一个像素点的8邻域的softmax二分类结果
- Cross-Layer：层间的连接由8通道表示，上层feature map的尺寸是下层的两倍，所以上层中一个location的像素则代表了下层中4个点，所以层间的连接就是判断上层是否与这四个点中的某一个相连通

那么最终网络会输出一个31通道的结果，来代表segment以及link

<img src="_assets\seglink-output.png"  />

#### Loss

对于score map以及link采用softmax loss

对于offset采用smoothL1 loss

#### Combine Segments

在得到了所有的segment以及link之后，需要将这些segments通过link相融合得到最终的检测结果。首先对于segment和link要设置不同的阈值来确定其是否保留，那么在过滤结束后，剩下的segment和link可以看作是一张图里面的**点集**和**边集**。

通过DFS找到一个连通分量的所有边和点，记为$\mathcal{B}$.

<img src="_assets\seglink-combine.png"  />

- 首先计算所有的segment的倾斜角的平均值$\theta_b$
- 然后找到一条斜率为$\theta_b$的一条直线，使得所有的segment的中心点到这条直线的距离之和虽小
- 然后将所有的segment的中心点在这条直线上进行投影，并且找到所有投影点两头的两个点记为$(x_p.y_p),(x_q,y_q)$
- 最后基于此计算最终文本框的位置

至此，整个训练以及预测的流程就结束了。

### 数据生成

在生成了不同feature map 的所有default box后：

首先是score map的标注，满足以下两条要求的点被标为True

- defalut box的中心点落在一个真实文本框的范围内
- defalut box的边长以及文本框的边长满足：

$$
max(\frac{a_l}{h},\frac{h}{a_l}) \leqslant 1.5
$$



随后需要标注偏移量

<img src="_assets\seglink-gt.png" style="zoom:80%;" />

- 首先将word bound box以default box的中点为中心，顺时针旋转$\theta$，使其水平
- 随后裁剪其宽度使其与defalut box保持一致，再以default box的中点为中心，逆时针旋转$\theta$，得到此处default box所对应的标记框，并且计算所需要的5通道结果。