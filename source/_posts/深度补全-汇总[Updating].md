## 深度补全

#### 1 2017-《Sparse Invariant CNNs》

Method:

- 提出一个能显式考虑缺失数据location的卷积操作——`Sparse RCNN`
- 提出了针对稀疏输入的数据的特殊CNN结构（同时处理`mask`和`input`）
- 提出了堆叠的sparse CNN的网络结构，kernel size从11x11逐渐递减到3x3
- 提出sparse数据的skip connection结构

<img src="./markdown图片/completion1.png" alt="屏幕快照 1" style="zoom:40%;" >

#### 2 2019-FuseNet

《Learning Joint 2D-3D Representations for Depth Completion》

- 提出了2D-3D Block的模块：该模块同时获得2D的特征，以及**3D的特征**（在3D点云中寻找该点附近的点云，通过**MLP编码**，计算邻近点云的特征来表示对该点云的**权重**），将2D的特征和3D的特征融合，输出这个block的最终特征
  - 第一条branch为**多尺度的2D conv**（由绿色的上下两个scale组成，第一个scale是stride=1的，第二个scale是stride=2的）
  - 第二条branch是**2个continuous convolution**（对点云进行操作的），最后把点云进行投影到2D平面空间。
  - 把两条branch**融合**（仅使用element-wise sum），得到这个2D-3D block的输出

<img src="./markdown图片/completion2.png" alt="屏幕快照 1" style="zoom:35%;" >

#### 3 2019-《Sparse and noisy LiDAR completion with RGB guidance and uncertainty》

- 提出global和local信息的概念（采用双分支结构）
  - global：LiDAR数据缺少或不正确的区域（物体边界）【使用RGB来指导预测depth】
  - local：LIDAR数据充足的地方
- 提出特征融合策略（采用early fusion和late fusion）
  - early fusion：global的guidance map来指导LIDAR
  - late fusion：confidence map作为权重来指导两个branch的融合
- 提出`confidence mask`（指引不确定性区域）

<img src="./markdown图片/completion3.png" alt="屏幕快照 1" style="zoom:40%;" >

#### 4 2019-《Self-Supervised Sparse-to-Dense: Self-Supervised Depth Completion from LiDAR and Monocular Camera》

- 设计网络：学习一个映射，从稀疏深度到密集深度

- 提出了一个self-supervised网络（不需要密集的depth标签）
  - 不用密集的gt_depth作为标签（所以是自监督的）
  - `Photometric loss`:对前后帧用PnP解决(对前后帧的**特征点先匹配**，代码中这是对数据的预处理中做的，采用的是`SIFT算法`，描述前后帧的特征匹配)，计算相机位姿变换，用根据位姿变换生成的rgb图和标准的rgb图计算**误差**，评估相机位姿的准确性(即间接评估了预测深度的准确性)。
  - `Depth loss`：计算存在depth区域的depth预测准确度

**补充：**PnP算法：通过一组对应的**图像2D点**和其在**世界坐标系下的3D点**，求解出**相机坐标系下的3D点**，将相机坐标系下的3D点和世界坐标系下的3D点计算得到**相机位姿**。【同理把世界坐标系下的3D点看成是上一帧相机坐标下的3D点，就能求出两帧之间相机的变换矩阵】

<img src="./markdown图片/completion4.png" alt="屏幕快照 1" style="zoom:40%;" >

白色方框：变量

红色：depth网络

蓝色：计算模块（无可学习的参数）

绿色：损失函数

**training**：需要当前帧$RGBd_1$和相邻帧$RGB_2$提供监督信号（$RGBd=RGB+depth$）

**inference**：只需要当前帧$RGBd_1$作为输入，输出预测深度$pred_1$

#### 5 DeepLiDAR

Method

- 提出从sparse LiDAR depth+surface和color image（分别预测dense depth）【surface normal是预先计算得到】
- 提出encoder-decoder结构（**DCU模块**）：融合`sparse depth`和`color image`
- 学习一个**confidence mask来解决遮挡**；生成**attention map来融合depth**(来自两个pathway的color image和surface normals中的深度预测)【confidence mask(降低在遮挡区域的权重)来解决混合的雷达信号】

<img src="./markdown图片/DeepLiDAR.png" alt="屏幕快照 1" style="zoom:40%;" >

