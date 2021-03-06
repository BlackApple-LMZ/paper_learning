发表在2018的IROS上，代码也开源了，用的人挺多，据说效果比loam要好；参考 https://zhuanlan.zhihu.com/p/115986186
### 摘要：
一套轻量级的雷达里程计和建图系统，可以在嵌入式设备上实时的估计6自由度的姿态；地面优化，对采集的点云先分割，然后提取地面和边缘特征，用来对连续两帧的scan数据估计位姿，估计的过程用到了两步的LM算法。实验表明效果和LOAM有的一比，但是更快。
### 介绍：
阐述了LOAM对计算资源要求较高，要计算每个点的roughness值，因此在小的嵌入式上就比较吃力。
### 主要贡献：
- 1本文提出的使用两步优化的算法分别计算z，横滚角，俯仰角以及x，y，偏航角。
- 2集成了一个回环检测的算法以减小漂移。
- 3提出了一个点云分割的方案，以减少噪声的影响以及计算量。
### 系统：
![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0728-0729-Lego-loam%20Lightweight%20and%20ground-optimization%20lidar%20odomotry%20and%20mapping%20on%20variable%20terrain/11.png)

系统由5个部分组成：首先是点云分割，将点云投影到一个范围图像中去分割。接下来是特征提取，然后雷达里程计估计位姿，特征会被处理后送到建图模块中，将它们配准到全局的点云地图中。最后，位姿集成模块使用位姿估计的结果和建图的结果获得最后的输出。

#### 1点云分割：
使用了《Fast Segmentation of 3D Point Clouds for Ground Vehicles》中的方法进行地面点的区分：将点云投影到range image，分辨率为1800\*16，1800是360除以0.2，0.2是激光雷达扫描点的间隔是0.2，16就是雷达扫描的16线。range图的每个像素存储对应三维点到sensor的真实距离，这一步分割出的地面不用于下一步的聚类；然后使用《Fast Range Image-based Segmentation of Sparse 3D Laser Scans for Online Operation》的方法进行分割出组并打上标签，若组内点过少就会把这部分点视为不稳定的点给剔除掉，此外将将低于30个数据点的类别都当作噪点处理，整个距离图像就可以被分成若干个比较大的类别。

每个点保存三个信息：分割标签（地面点或分割点）、在距离图像中的行和列的索引、传感器的距离值
#### 2特征提取
特征提取方法与LOAM中的方法类似，但是，相比与直接从原始点云数据中提取特征点。我们从已经分组的数据中提取特征，另S为从同一行中提取出的点的集合，使用从分割的过程中计算出的距离信息，我们计算S中点的粗糙程度：

 ![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0728-0729-Lego-loam%20Lightweight%20and%20ground-optimization%20lidar%20odomotry%20and%20mapping%20on%20variable%20terrain/22.png)
 
将距离图像水平均分为若干个子图像，对每一个点，都能找到一个S集合，用上面的公式计算每个点在集合S中的平滑度，然后排序，进行特征点的选取，根据阈值，区分边缘点和平面点。从每一行中选取不属于地面点，且具有最大c值的nFme个边缘点，组成集合Fme，从每一行中选取最小c值的nFmp个平面点，属于地面点或分割点都行，组成集合Fmp。然后再缩小选取点的数量，构成了新的两个特征点集合Fe和Fp。
#### 3、雷达里程计
雷达里程计模块估计连续两次扫描的位姿信息，使用从点-线和点-面匹配获得。从另一个角度说，即要找到与上一帧的对应点。这部分的内容可以从LOAM中找到。就是对当前时刻的Fe和Fp，分别寻找与上一时刻Fme和Fmp的对应点关系。构建点线和点面关系。

一些调整可以提升匹配的准确性与快速性：
- 标签匹配：只对具有相同的标签的点云进行匹配（这一步的处理也不是没有问题的，在实际的实验中，如果地面不是特别的平坦，也就是说相邻帧间的地面有了一定的变动，这个时候，LeGO-LOAM就不能很好的运行）
- 两步的优化：将平面与边缘特征分开估计运动参数，最终发现计算时间减少了35%。LOAM是一步优化，就是对平面特征或者边缘特征同时进行优化，本文是分开优化，对平面特征优化z、roll和pitch，基于上一步的结果对边缘特征优化剩余全部的变量。
- 分析：由于地面在相邻帧间基本保持不变，所以，使用点到面的约束关系，可以计算出竖直维度的变动z、roll和pitch。当算出竖直维度的变动时，可以以此为初值输入到第二步的优化中，减少迭代次数，算出水平维度的变动x、y和yaw，提升计算效率。有点东西
#### 4、雷达建图
将当前的特征与周围点云地图进行匹配修正位姿估计的结果。这里再次使用了LM方法。还是可以从LOAM中查看细节。

LeGO-LOAM最大的不同就是最后的点云地图怎么存储，相比于存点云地图，我们将每一帧的特征存储。然后从特征去生成周围的点云就有了两种方法：
- 第一种方法，可以从可以将当前场景下可以看到的特征选取，简单点可以选择当前位置周围100m位置看到的特征。就相当于local map；
- 第二种方法就是pose-graph的方法，用局部共视的图去添加新的约束。这部分理解不是很清晰，大概就是像视觉那样，引入了后端优化和回环检测。
