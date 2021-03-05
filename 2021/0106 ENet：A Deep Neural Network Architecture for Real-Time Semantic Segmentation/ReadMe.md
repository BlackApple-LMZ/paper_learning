## 摘要：
许多移动应用需要实时语义分割模型。深度神经网络需要大量的浮点运算，导致运行时间长，从而降低了时效性。ENet（efficient neural network），相比于 SegNet，在速度块，浮点计算量少，参数少，且有相似的精度。
## Related work：
目前语义分割多采用encoder加decoder的方式，采用fcn的方式，把vgg16的全连接层全部cut掉；

参考的这篇文章：

https://blog.csdn.net/qq_21167623/article/details/102061211
## 网络结构：
![image](https://github.com/BlackApple-LMZ/paper_learning/2021/0106%20ENet%EF%BC%9AA%20Deep%20Neural%20Network%20Architecture%20for%20Real-Time%20Semantic%20Segmentation/11.png)

分成了几个阶段， 
- initial：初始化模块，可大大减少输入的尺寸；
- Stage 1：encoder 阶段。包括 5 个 bottleneck，第一个 bottleneck 做下采样，后面 4 个重复的 bottleneck；
- Stage 2-3：encoder 阶段。stage2 的 bottleneck2.0 做了下采样，后面有时加空洞卷积，或分解卷积。stage3 没有下采样，其他都一样；
- Stage 4~5：属于 decoder 阶段；

### 初始化阶段：
![image](https://github.com/BlackApple-LMZ/paper_learning/2021/0106%20ENet%EF%BC%9AA%20Deep%20Neural%20Network%20Architecture%20for%20Real-Time%20Semantic%20Segmentation/22.png)
- Conv：3x3，stride 2，num 13
- Maxpooling：2x2，stride 2，num 3
- 将两边结果 concat 一起，合并成 16 通道，这样可以上来显著减少存储空间。
### Bottleneck模块：基于resnet的思想
![image](https://github.com/BlackApple-LMZ/paper_learning/2021/0106%20ENet%EF%BC%9AA%20Deep%20Neural%20Network%20Architecture%20for%20Real-Time%20Semantic%20Segmentation/33.png)

- 基本的bottlenet：第一个 1x1 卷积实现降维；第二个 1x1 卷积实现升维；使用 PReLU 激活函数，ReLU 降低了模型精度（网络层数较少）；drop 防止过拟合；最后元素级加法融合；
- 下采样的bottlenet：主分支加入maxpooling，1x1变成了2x2的卷积，并且用了zero pad，卷积可以是regular, dilated or full convolution这三种形式，有时候也会是asymmetric convolution，正则化是Spatial Dropout；
- 整个网络中没有使用 bias，只有 weights。这样可减少内核调用和内存操作，因为 cuDNN 会使用单独的内核进行卷积和 bias 相加。这种方式对准确性没有任何影响；在decoder阶段，max pooling替换成了max unpooling，padding替换成了空间卷积，同样也没有bias；在最后一个上采样模块没有使用pooling indices，仅在网络的最后搞一个全连接层，就这一个就占了decoder的大部分时间。

https://zhuanlan.zhihu.com/p/31379024 trick解读
## 设计方面的一些选择：
1. 特征图的分辨率：语义分割中下采样图片有很多问题：损失大部分空间信息，先下采样后上采样会增加模型复杂，并且带来计算消耗；这里upsample 采用的是 SegNet 的方法进行上采样，结合下采样中的 pooling indices；下采样有一个好处就是像素的感受野很大，包含的信息更丰富，更容易进行区分；并且用了空洞卷积。
2. 网络的初始层不应该直接面向分类做贡献，而且尽可能的提取输入的特征。
3. Encoder 和 Decoder 不是镜像对称的，网络的大部分实现 Encoder，较少部分实现 Decoder。Encoder 主要进行信息处理和过滤，而 Decoder 上采样编码器的输出，并进行细节微调；
4. Encoder 阶段 Feature map 8 倍下采样
