没有开源、其他人解读的不多
### 摘要：
创建高精地图的传统方法涉及繁琐的手动标记物体，将二维图像语义分割与从一个相对便宜的16线激光雷达传感器采集的预构建点云地图相结合，在鸟瞰图中构建局部概率语义地图，对驾驶环境中的道路、人行道和车道等静态路标进行编码。
### 主要贡献：
利用16线激光雷达构建的稠密点云地图以及深度神经网络的最新语义标记的图像，在城市环境下生成密集的概率语义地图。

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/0805-Probabilistic%20Semantic%20Mapping%20for%20Urban%20Autonomous%20Driving%20Applications/11.png)

通过geometric transformations融合local point cloud map和semantic images，从而构建probabilistic map。整个系统分为semantic segmentation,point cloud semantic association, semantic mapping, 和map transformation.

1. 语义分割：使用DeepLabV3Plus网络结构从二维图像中提取语义信息。
2. 点云语义关联：通过多视图几何恢复深度需要显著的特征，但是条件比较严苛。本文提取密集点云地图的小区域，并将其投影到语义分割的图像中以检索深度信息，雷达和相机的相对位置关系都是事先标定过的。
3. 构建概率语义地图：就和概率栅格地图的思路很像，因为单次构建地图容易受噪声影响，所以就用概率融合更新的方式，构建语义地图。（这个概率语义地图是鸟瞰图）
4. 概率地图的转换：就是每来新的一帧，转换到local map坐标系下，然后进行融合更新。
实验：就是说明这种方法的建图可以辅助高精地图定位。
