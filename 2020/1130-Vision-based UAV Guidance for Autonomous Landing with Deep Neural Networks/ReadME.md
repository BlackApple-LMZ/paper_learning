### 中间：
用卷积神经网络做飞机的姿态估计；

1. 飞机模型：F-16战斗机模型
2. 自动路径规划和着陆：

降落分为三个阶段：approach, glide, and flare

- Approach 进近阶段是计算初始位置到下滑的初始位置的直线；
- Glide 就是沿着下滑道滑行；
- Flare 就是在距地面40英尺的高度开始将飞机抬起一点，减小飞机的垂直速度，防止着陆对飞机造成结构性的损坏；

这三个过程的规划参考的这篇文章：Automatic path planning and control design for autonomous landing of UAVs using dynamic inversion

3. 初始数据集的采集
高级的着陆指引和低级的控制器配合控制；仿真环境用的airsim，这是数据采集框架：

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/1130-Vision-based%20UAV%20Guidance%20for%20Autonomous%20Landing%20with%20Deep%20Neural%20Networks/11.png)

### 数据预处理部分：
图片与heading的label绑定，总共10次飞行27724张图片，数据集按照721的比例分为训练，验证，测试数据集；图像的分辨率是512*288*3，然后水平翻转图像加倍数据量；为了避免不相关的特征，整了一个512\*160的ROI区间，像素值归一化到了[0,1]，为了消除时间序列的影响，shuffle一下；
### CNN网络结构：
- 基于NVIDIA的一个网络，通过输入汽车视角的图片预测方向盘的转角；
- 网络采用的是一个steer angle预测的网络：五个卷积层后面跟了flatten和5个全连接层；
- 激活函数采用的是ReLU函数。

![image](https://github.com/BlackApple-LMZ/paper_learning/edit/main/2020/1130-Vision-based%20UAV%20Guidance%20for%20Autonomous%20Landing%20with%20Deep%20Neural%20Networks/22.png)
### 训练的初始化策略：
学习率10e-5，动量0.99，采用Nadam作为优化方法，当验证的损失函数5个epoch不下降的时候，就将学习率减半；角度是以弧度表示的，batch size是32；loss函数采用的是MSE。

