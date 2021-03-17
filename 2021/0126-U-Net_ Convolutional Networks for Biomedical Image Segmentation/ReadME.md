### 摘要：
成功训练一个深度网络需要大量已标注的训练样本。在这篇论文中，我们提出了一个新的网络和训练策略。为了更有效的利用标注数据，我们使用数据扩张的方法(data augmentation)。我们的网络由两部分组成：一个收缩路径(contracting path)来获取上下文信息以及一个对称的扩张路径(expanding path)用以精确定位。我们使用这个网络获得了赢得了ISBI cell tracking challenge 2015.不仅如此，这个网络非常的快，对一个512\*512的图像，使用一块GPU只需要不到一秒的时间。

总结一下就是：提出了新的网络和训练策略，数据增强，网络就是两个path，效果是非常的快。解决医学图像分割问题，提出了一种数据增强方法来有效利用标注数据；提出了一种U型的网络结构可以同时获取上下文信息和位置信息；该方法在2015年的ISBI cell tracking比赛中获得了多项第一。

参考知乎的博主：

https://zhuanlan.zhihu.com/p/90418337

### 简介：
医学领域图像分割标注数据相对不足，与fcn(fully convolutional natwork)不同的是，我们的网络在上采样部分依然有大量的特征通道(feature channels)，这使得网络可以将环境信息向更高的分辨率层(higher resolution layers)传播。形成了一个U形的结构，

采用了Overlap-tile strategy:

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0126-U-Net_%20Convolutional%20Networks%20for%20Biomedical%20Image%20Segmentation/11.png)

譬如要补全上图中黄框区域的上下文成蓝框区域，具体的做法是将黄框和蓝框之间右侧和下侧的像素通过镜像拷贝的方式拷贝到左侧和上侧，以补全蓝框。

### 数据增强策略：
通过对原始图像进行弹性形变以获得补充图像，这可以让网络学习形变不变性（这点主要是针对医学图像，因为医学图像组织啥的特别容易发生形变，所以这方面进行数据增强很有效果）；加权Loss：增大对粘连的同类物体之间的“background”像素的loss权重，使得每个物体的分割轮廓是清晰的（就是下面最后一张图，不同cell之间的background区域的loss很大，图里面显示是红色）。

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0126-U-Net_%20Convolutional%20Networks%20for%20Biomedical%20Image%20Segmentation/22.png)

Encoder：左半部分，由两个3x3的卷积层（ReLU）+2x2的max polling层（stride=2）反复组成，每经过一次下采样，通道数翻倍；

Decoder：右半部分，由一个2x2的上采样卷积层（ReLU）+Concatenation（crop[3]对应的Encoder层的输出feature map然后与Decoder层的上采样结果相加）+2个3x3的卷积层（ReLU）反复构成；

最后一层通过一个1x1卷积将通道数变成期望的类别数。

下面是U-Net的网络结构：

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0126-U-Net_%20Convolutional%20Networks%20for%20Biomedical%20Image%20Segmentation/33.png)

可以开到结构很对称，上采样的方式和FCN应该是一样的，没有看具体的代码。

https://blog.csdn.net/natsuka/article/details/78565229 

### 三、训练
我们采用随机梯度下降法训练。

为了最大限度的使用GPU显存，比起输入一个大的batch size，我们更倾向于大量输入tiles。我们使用了很高的momentum（0.99）。

最后一层使用交叉熵函数与softmax。（交叉熵函数如下所示）
 
为了使某些像素点更加重要，我们在公式中引入了w(x)。我们对每一张标注图像预计算了一个权重图，来补偿训练集中每类像素的不同频率，使网络更注重学习相互接触的细胞之间的小的分割边界。我们使用形态学操作计算分割边界。权重图计算公式如下：
 
wc是用于平衡类别频率的权重图，d1代表到最近细胞的边界的距离，d2代表到第二近的细胞的边界的距离。基于经验我们设定w0=10，σ≈5像素。

网络中权重的初始化：我们的网络的权重由高斯分布初始化，分布的标准差为(N/2)^0.5,N为每个神经元的输入节点数量。例如，对于一个上一层是64通道的3\*3卷积核来说，N=9\*64。

### 四、数据增加

在只有少量样本的情况况下，要想尽可能的让网络获得不变性和鲁棒性，数据增加是必不可少的。因为本论文需要处理显微镜图片，我们需要平移与旋转不变性，并且对形变和灰度变化鲁棒。将训练样本进行随机弹性形变是训练分割网络的关键。我们使用随机位移矢量在粗糙的3\*3网格上(random displacement vectors on a coarse 3 by 3 grid)产生平滑形变(smooth deformations)。 位移是从10像素标准偏差的高斯分布中采样的。然后使用双三次插值计算每个像素的位移。在contracting path的末尾采用drop-out 层更进一步增加数据。
