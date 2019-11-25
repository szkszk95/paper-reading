## Robust Scene Text Recognition with Automatic Rectification

白翔老师的**RARE (Robust text recognizer with Automatic REctification)**

[Paper]( https://arxiv.org/pdf/1603.03915.pdf )

之前在ICCV2019里也看过总结的RARE和CRNN文章，但是没太搞懂里面的TPS的locolization网络是怎么工作的，今天重新看了一下RARE和ASTER，理解了一下

TPS里面的localization网络为什么不需要标注，这么理解，因为TPS网络+SRN网络最后会回传一个识别的loss值，这个loss会一直传到TPS的L-net里面，而识别的准不准一定程度上也是取决于TPS抠的好不好，所以这里很重要的一个点在于TPS的2K个关键点的初始化选择上

![](_assets\RARE_init.png)

RARE中介绍了初始化方式，作者说a的方式最容易使得网络收敛，而b和c会取得比较差的效果，而随机初始化则根本没办法训练，可以看出来TPS的训练其实还是比较规整的，也符合一般的逻辑，因为现有场景中的字虽然说比如会形成一个环，但还是比较工程的情况。

