这是德国波恩大学Photogrammetry and Robotics Lab新开源了用于激光雷达SLAM中闭环检测的代码：OverlapNet（https://github.com/PRBonn/OverlapNet）作者是SuMa++的作者。

### 摘要：
用3D激光数据解决SLAM的闭环问题，采用深度神经网络，基于 LiDAR 数据的不同线索搜寻回路闭合，估计两个范围图像的重叠率，以及两次扫描之间的相对偏航角。实验表明该方法具有很不错的效果。
### 特点：
网络输入不同类型的数据信息，包括深度，法向量，强度和语义，输出是重叠率和相对的偏航角，对每个激光束产生一个概率分布；
### 贡献：
1. 无需两个雷达扫描的相对位姿，使用深度神经网络直接学习它们之间的重叠率，以及相对偏航角。大量训练数据可以通过里程计数据集自动获得。
2. 根据预测的重叠率以及里程计信息检测闭环
3. 提升 SLAM 系统 SuMa 的制图表现。
4. 为ICP提供好的初始化来修正扫描匹配的结果

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0722-OverlapNet%20Loop%20Closing%20for%20LiDAR-based%20SLAM/11.png)

考虑range image的重叠率（第一幅图像中，可以无遮挡的投影到第二幅图像上的像素百分比，是单向的），
激光雷达之间的重叠率：首先定义三维到二维的映射：

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0722-OverlapNet%20Loop%20Closing%20for%20LiDAR-based%20SLAM/22.png)

然后对这个二维的map计算重叠率（valid是指map中有效的点数，因为有的区域不存在对应的投影点；分子的意思是如果对应的像素差在一定范围内，有效数量加1）：

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0722-OverlapNet%20Loop%20Closing%20for%20LiDAR-based%20SLAM/33.png)

作者想用这个等式建立训练集，因为直接用SLAM估计的位姿然后计算重叠率误差很大。对于旋转，作者选择旋转多个角度，用最大重叠值代表最终重叠值。

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0722-OverlapNet%20Loop%20Closing%20for%20LiDAR-based%20SLAM/44.png)

作者把深度图，向量图，强度图，还用RangeNet++做了个语义图一起作为输入。一个输出是角度特征向量，一个输出是两次扫描之间的重叠率。

### 网络结构：
- Legs：共享参数，用的是全卷积层，输出是feature volumes 1*360*128
- Delta head：用来估计重叠率，Delta Layer 由一系列简单的复制、转置和绝对差操作组成。
- Correlation Head：是一个分类全连接层，输入是两个特征图，输出是360个分类，分别对应360度角。
### 损失函数：
- 重叠率是回归问题
- 偏航角是分类问题（要求最低重叠率为30%，实验部分有说明这个参数）
### SLAM系统：
采用的SuMa，这篇论文以后也要读一下；SuMa的回环检测将最近的帧作为回环候选帧，在大场景下容易失败；

读了接近2个小时，每天早晨读一篇论文的想法有点不太现实啊。。。


