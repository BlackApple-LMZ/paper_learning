摘要 https://www.cnblogs.com/fourmi/p/9785377.html 
segnet是用于进行像素级别图像分割的全卷积网络，分割的核心组件是一个encoder 网络，及其相对应的decoder网络，后接一个象素级别的分类网络。encoder网络：其结构与VGG16网络的前13层卷积层的结构相似。decoder网络：作用是将由encoder的到的低分辨率的feature maps 进行映射得到与输入图像featuremap相同的分辨率进而进行像素级别的分类。Segnet的亮点：decoder进行上采样的方式，直接利用与之对应的encoder阶段中进行max-pooling时的polling index 进行非线性上采样，这样做的好处是上采样阶段就不需要进行学习。 上采样后得到的feature maps 是非常稀疏的，因此，需要进一步选择合适的卷积核进行卷积得到dense featuremaps 。作者与FCN，DeepLab-LargeFOV, DenconvNet结构进行比较，统筹内存与准确率，Segnet实现良好的分割效果。SegNet主要用于场景理解应用，需要在进行inference时考虑内存的占用及分割的准确率。同时，Segnet的训练参数较少（将前面提到的VGG16的全连接层剔除），可以用SGD进行end-to-end训练。
总结一下就是：上采样采用了max pooling过程的indices，上采样阶段就不需要学习参数，对应下面网络结构红色部分；因为用indices补0得到的结果是稀疏的，所以后面还需要再接一些卷积核得到dense featuremaps，对应右侧decoder部分的蓝色卷积层；
max pooling index的三点好处：（i）提高边界划分 (ii)减少训练的参数 (iii)这种形式可以广泛的应用在其他encoder-decoder结构。
结构
      SegNet由编码网络，解码网络后接一个分类层组成。编码网络由13个卷积层组成，与VGG16的前13层卷积相同，将VGG16在大型数据集上训练得到的权重值作为编码网络的权重初始值，为了保留encoder 最深层输出的到高分辨率的feature maps，删掉VGG16中的全连接层，这么做的一个好处是可以大幅度减少encoder层中训练参数的数量（from 134M to 14.7M），每一层encoder都对应着一层decoder，因此decoder网络也是13层，在decoder网络输出后接一个多分类的soft-max分类器对每个像素生成类别概率。
 
      encod网络中的每一个编码器通过一组卷积核来产生一系列的feature maps，后接一层BN+RELU+Max-pooling(2x2,stride=2)，Max-pooling用于实现小空间移动上的空间不变性,同时，可以在feature map上有较大的感受野，但由于使用Max-pooling 导致分辨率上的损失。这种损失对边界界定产生不利的影响，因此encoder网络要在进行下采样前着重捕捉和保存边界信息。由于实际内存的限制，所以无法保存feature map 的全部信息。本文提出只保存encode Max-pooling中feature map value值最大的位置。对于pooling 2*2的窗口可以用2比特位实现，相比存储feature map的float精度，效率较高，但对准确率会有轻微的损失，但仍适用实际应用。
     decode网络中的decoder 利用对应encoder feature map中保存的max-index对输入的feature map进行上采样，产生的稀疏feature maps后接一系列可训练的卷积核，输出密集的feature maps,后接BN用于规范化处理正则化减弱过拟合，与输入对应的decoder产生多通道feature map，虽然输入只有（RGB）三通道。其他的encoder,decoder的通道数，尺寸大小都是一一对应。decoder输出的高维度的特征表示被送入一个可训练的soft-max多分类器，对每个像素进行单独分类。soft-max输出的是一个K(类别数)同的的概率图，预测后的分割图中的每个类别是每个像素中概率值最大所对应的类别。
    DeconvNet与U-Net有与Se gNet相似的结构，但DeconvNet有更多的训练参数，需要更多的计算资源而且不利于进行end-to-end的训练。U-Net不利用max-pooling index而是将整个feature map 送到decoder 拼接到上采样后的feature map，十分占用内存资源，因此U-Net中无法使用到conv5和max-pool5模块，而SegNet却可以利用VGG16全部预训练过的卷积层的权重值。

Encoder部分大同小异，但是decoder部分差别挺大的；
然后论文介绍了一些decoder部分的变体，然后进行比较哪种decoder的设计方式更好；
SegNet-Basic 只有4组encoder和decoder，上采样还是采用的index，每个卷积核后面都有BN，都不用bias，decoder的卷积核没有ReLU。

将SegNet与FCN进行比较，SegNet是上采样之后通过一堆的filter来增加density，FCN是将encoder期间的feature map经过1*1的卷积核进行扩维，然后与decoder上采样得到的对应大小的map进行fuse，然后再进行上采样。

训练
    数据集：CamVid （训练：367张，测试：233，图片尺寸：360X480,11个类别）
    优化方法：SGD
    learning_rate:0.1
    momentum: 0.9
    mini-batch: 12
loss: cross-entropy 也用到了解决样本不均衡问题的方法，基于真实的class来确定该class的loss权重；median frequency balancing

分析
使用三种评价指标：global accuracy 数据集中被正确分类的像素所占的比例。class average accuracy 每种类别准确率和取平均。mIoU 惩罚FalsePositivate 的预测,在每个类别上计算IOU然后取平均值。
   实现性能最好的是当encoder 的feature map全部被保存，对于一个给定的encoder 网络，更大的decode网络可以提升性能，当inference 内存被限制时，encoder层压缩后的feature maps(降维,max-pooling indices) 可以进行保存并用适当的解码器来提升性能。

