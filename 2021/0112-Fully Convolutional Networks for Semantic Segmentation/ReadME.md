### CNN的缺点：
CNN在进行convolution和pooling过程中丢失了图像细节，即feature map size逐渐变小，所以不能很好地指出物体的具体轮廓、指出每个像素具体属于哪个物体，无法做到精确的分割。

翻译：https://blog.csdn.net/jieshaoxiansen/article/details/82285484 

### 摘要：
卷积网络在特征分层领域是非常强大的视觉模型。我们证明了经过端到端、像素到像素训练的卷积网络超过语义分割中最先进的技术。我们的核心观点是建立“全卷积”网络，输入任意尺寸，经过有效的推理和学习产生相应尺寸的输出。我们定义并指定全卷积网络的空间，解释它们在空间范围内dense prediction任务(预测每个像素所属的类别)和获取与先验模型联系的应用。我们改编当前的分类网络(AlexNet [22] ,the VGG net [34] , and GoogLeNet [35] )到完全卷积网络和通过微调 [5] 传递它们的学习表现到分割任务中。然后我们定义了一个跳跃式的架构，结合来自深、粗层的语义信息和来自浅、细层的表征信息来产生准确和精细的分割。我们的完全卷积网络成为了在PASCAL VOC最出色的分割方式（在2012年相对62.2%的平均IU提高了20%），NYUDv2，和SIFT Flow,对一个典型图像推理只需要花费不到0.2秒的时间。

重点就是采用全卷积，进行像素级别的预测，backbone可以用经典的那些网络结构，并且还搞了一个跳跃式的架构，in-network upsampling and multilayer、combinations效果就是在当时分割比较好。

### 全连接网络：
将一个分类的网络改成全连接的样子；
![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0112-Fully%20Convolutional%20Networks%20for%20Semantic%20Segmentation/11.png)

第三节介绍的内容有点没整明白是在讲啥；

### 语义分割结构：
https://www.cnblogs.com/gujianhan/p/6030639.html

上面这篇文章介绍了作者对FCN的理解。

FCN对图像进行像素级的分类，从而解决了语义级别的图像分割（semantic segmentation）问题。与经典的CNN在卷积层之后使用全连接层得到固定长度的特征向量进行分类（全联接层＋softmax输出）不同，FCN可以接受任意尺寸的输入图像，采用反卷积层对最后一个卷积层的feature map进行上采样, 使它恢复到输入图像相同的尺寸，从而可以对每个像素都产生了一个预测, 同时保留了原始输入图像中的空间信息, 最后在上采样的特征图上进行逐像素分类。

最后逐个像素计算softmax分类的损失, 相当于每一个像素对应一个训练样本。

FCN还是相当经典的结构；

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2021/0112-Fully%20Convolutional%20Networks%20for%20Semantic%20Segmentation/22.png)
