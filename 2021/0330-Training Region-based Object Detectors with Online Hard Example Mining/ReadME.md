参考：https://zhuanlan.zhihu.com/p/58162337

https://zhuanlan.zhihu.com/p/77975552 

### 摘要：
- (1) 训练过程需要进行参数的空间搜索
- (2) 简单样本与难分辨样本之间的类别不平衡是亟需解决的问题
- (3) 自动地选择难分辨样本来进行训练不仅效率高而且性能好
- (4) 提出了OHEM算法，不仅效率高而且性能好，在各种数据集上表现优越

### 介绍：
- (1) 分类器

由于目标检测套用图像分类的分类思想，但图像分类的数据集和目标检测的数据集存在天然的差距，目标检测的目标框和背景框之间存在严重的不平衡

在滑动窗口检测器尤为严重，在DPM中甚至达到1：100,000，虽然在其他 检测器中有所减缓，但依然高达1：70

- (2) hard negative mining

当然，这个类别不平衡问题并不是新问题，之前有个hard nagetive mining的 算法是解决这个类别不平衡问题的，它的关键思想是逐渐地增加分辨错误的样本。这个算法需要迭代地交替训练，用样本集更新模型，然后再固定模型 来选择分辨错的目标框并加入到样本集中在传统目标检测中，用SVM做分 类器也用到hard negative mining这个算法来训练；在一些浅层的神经网络和 提升决策树中也用hard negative mining来进行训练。除此之外，使用深度学习的目标检测算法也用到了hard negative mining

- (3) why current state-of-the-art object detectors do not use hard negative mining?

那为什么不用hard negative mining，这主要是技术上的难度，hard negative mining需要交替地训练，而这对于使用线上优化的算法来说是不可能的，例如SGD(随机梯度下降算法)。使用SGD来训练网络需要上万次更新网络，如果每迭代几次就固定模型一次，这样的速度会慢得不可想象

- (4) online hard example mining(OHEM)

那OHEM是怎样解决类别不平衡的呢，OHEM是选择损失较大的候选ROI， 具体为什么选择损失较大的候选ROI，这个后面再仔细说

作者总结了一下，使用了OHEM之后，不仅避免了启发式搜索超参数，而且 提高了mAP。作者发现，训练集越大越困难，OHEM的效果就越好

### 系统框架：
![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0330-Training%20Region-based%20Object%20Detectors%20with%20Online%20Hard%20Example%20Mining/11.png)

#### (1) framework

图片和候选框做为Fast R-CNN的输入，Fast R-CNN分为两部分，一部分是卷积网络，包括卷积和池化层，另一部分是RoI网路，包括RoI池化层、全连接层和两个损失层(一个是分类，一个检测框回归)

#### (2) inference

在测试的时候，图片输入到卷积网络得到特征层，选择性搜索算法得到RoIs， 对于每个RoI，得到其对应的特征向量，然后每个特征向量输入到全连接层 并得到两个输出，一个是概率，一个检测框的坐标

#### (3) How it trains

Fast R-CNN是使用SGD来优化模型的，每个RoI的损失包括分类损失和回归损失，其中不断降低分类损失使得模型分类更准确，不断降低回归损失使得预测标注框更准确。

SGD是以mini-batch为单位来更新模型的。对于每个mini-batch，先从数据集中取N张，然后每张图片采样B/N个RoIs。

- a. Foreground RoIs

一个RoIs怎样才算作一个目标RoI(也就是含有目标的RoI)呢，在R-CNN, SPPnet, and MR-CNN等把RoI与真实框的交叉比(IOU)大于等于0.5即判定 为目标RoI，在本文中也是这样的设置

- b. Background RoIs

而如果要被判定为背景RoI，则要求该RoI与真实框的交叉比大于等于 bg_lo这个阈值并且小于0.5。虽然这样的设置能加快收敛和检测准确度， 但这样的设置会忽略不怎么出现但又十分重要的比较难分辨的背景。因此，在本文的OHTM方法中，作者去掉了这样的设置。

- c. Balancing fg-bg RoIs

为了解决目标框和背景框之间的不平衡，Fast R-CNN设置在一个 mini-batch中，它们之间的比例是1：3。作者发现，这样的一个比例对 于Fast R-CNN的性能是十分重要的，增大或者减小这个比例，都会使模型的性能有所下降，但使用OHEM便可以把这个比例值去掉。

### OHTM：
作者认为Fast R-CNN之前选择RoI的方法不仅效率低而且也不是最优的，于是作者提出了OHEM，OHEM不仅效率高而且性能也更优

- (1) Online hard example mining

我们知道，基于SVM的检测器，在训练时，使用hard example mining来选择样本需要交替训练，先固定模型，选择样本，然后再用样本集更新模型，这样反复交替训练直到模型收敛。

作者认为可以把交替训练的步骤和SGD结合起来。之所以可以这样，作者认为虽然SGD每迭代一次只用到少量的图片，但每张图片都包含上千个RoI，可以从中选择hard examples，这样的策略可以只在一个mini-batch中固定模型，因此模型参数是一直在更新的。

就是说对于ROI的loss进行一下筛选；

更具体的，在第t次迭代时，输入图片到卷积网络中得到特征图，然后 把特征图和所有的RoIs输入到RoI网络中并计算所有RoIs的损失，把损失从高到低排序，然后选择B/N个RoIs。这里有个小问题，位置上相邻的RoIs通过RoI网络后会输出相近的损失，这样损失就翻倍。作者为了解决这个问题，使用了NMS(非最大值抑制)算法，先把损失按高到低排 序，然后选择最高的损失，并计算其他RoI这个RoI的IoU(交叉比)，移除IoU大于一定阈值的RoI，然后反复上述流程直到选择了B/N个RoIs。

- (2) Implementation details

主要有两种方法

#### a. An obvious way
直接修改损失层，然后直接进行hard example selection。损失层计算所有的RoIs，然后按损失从大到小排序，当然这里有个NMS(非最大值抑制) 操作，选择hard RoIs并non-hard RoIs的损失置0。虽然这方法很直接，但效率是低下的，不仅要为所有RoI分配内存，还要对所有RoI进行反向传播，即使有些RoI损失为0。

#### b. A better way
为了解决这个问题，作者提出了上面这样的架构。这个架构有两个相同的RoI网络，不同的是其中一个只可读，另一个可读可写。我们看到(a)是只可读的，只对所有RoI做前向计算，所以只需分配内存给前向计算 操作，(b)既可读也可写，对被选择的hard RoIs不仅做前向计算也做反向 传播计算。

对于一次SGD迭代，计算过程如下:先计算出特征图，可读RoI网络对所有RoI执行前向计算并计算每个RoI的损失，然后选择hard RoIs。把这些hard RoIs输入到可读可写的RoI网络中执行前向前向计算和反向传播更新网络，并把可读可写的RoI网络的参数赋值给只可读的网络，一次迭代就完成了。

这个方式和第一种方式在内存空间是差不多的，但第二种方式的速度快了两倍。
