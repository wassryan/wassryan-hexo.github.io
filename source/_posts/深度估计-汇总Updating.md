---
title: 深度估计-汇总Updating...
date: 2019-12-05 20:21:10
tags: [深度估计]
---

## 深度估计

### 1. 利用双目立体视觉的空间约束

left/right image作为各自的监督信号

1.1. 《Unsupervised Monocular Depth Estimation with Left-Right Consistency》

Method：利用获得的左右两张图片，分别作为各自的监督信号，训练两试图相互转换的function。利用计算视差，通过视差计算得到深度。

<img src="/images/depth1.png">



1.2. 《Semi-Supervised Deep Learning for Monocular Depth Map Prediction》——无代码

*半监督：监督loss+无监督loss+smoothness loss*

- 监督：左右图各自预测的depth（和sparse depth去）计算loss(在**sparse depth有值的区域**）
- 无监督：预测的depth和单视图，通过相机内参转换到另一视图，计算warp视图的loss（间接监督了depth）

<img src="/images/depth2.png">



### 2. 利用视频相邻帧作为监督信号

预测相机位姿和depth

2.1.《Unsupervised Learning of Depth and Ego-Motion from Video》

Method：

- 前后帧和当前帧，通过`pose CNN`的方法来预测pose的参数，再把相邻帧warp到当前帧（过程中使用到`depth CNN`预测得到的depth作为输入），计算合成的视图的像素之间的loss。
- 提出使用`explainability mask`来识别场景中**运动的物体、遮挡或消失的物体**（*这些物体是和相机位姿变换规律违背的，无法预测的，所以让这些物体的像素产生的loss降低*）

<img src="/images/depth3.png">



2.2. 《Digging Into Self-Supervised Monocular Depth Estimation》2019

* 提出了`minimum reprojection loss`代替`average loss`代替对前后帧获得的loss取平均的策略【解决遮挡问题】
* 一个`full-resolution multi-scale`的采样方法（对每个level的depth map都upsample到输入分辨率的尺寸去做reproject）：【解决visual artifacts(视觉假象)】
* 一个`auto-masking loss`(对图像中相对静止或静止的像素mask取0) 【忽视和camera motion假设的相反的像素】

<img src="/images/depth9.png">



### 3. 结合表面法向量联合训练

3.1. 《GeoNet: Geometric Neural Network》

Method：

- 提出使用`depth`和`normal`联合训练，同时从**单张图片**预测**normal和depth**
- Depth-to-Normal：由于直接用`CNN`以图像为输入预测depth是有缺陷的（无法考虑到像素点之间的**几何约束**），所以提出使用**粗糙的normal来联合depth**去预测refined normal。
- Normal-to-Depth：使用**核函数**，去从normal和depth计算refined depth

<img src="/images/depth4.png">



3.2. 2019-《Enforcing geometric constraints of virtual normal for depth prediction》

Method:

- 改进surface normal，提出virtual normal【相⽐比 surface normal(选取的⽤用于计算normal的平⾯面的local patch太⼩小，并且由于3D点云的噪声点对其影响 很⼤大)，virtual normal由于特殊的设定，能够对local patch和3D点云噪声有鲁棒性】

<img src="/images/depth10.png">



### 4. 分解depth estimation任务

分解为：`view synthesis`+`stereo matching`

4.1. 《Single View Stereo Matching》2018

Method:

- depth estimation可以被分解为两个问题：`view synthesis`+`stereo matching`
- 结合view synthesis（从左视图预测右视），并计算视差图，进而计算得深度图
- `view synthesis`：采用视差概率图加权（预测每个视差值下的概率，加权得到最终的视差图，生成右视图，使得loss对其可导，能用神经网络训练）
- `stereo matching`：采用DispNetC（其中的1D correlation layer）和DispFullNet

<img src="/images/depth5.png">



### 5. 结合运动和边缘信息联合训练

5.1. 《LEGO: Learning Edge with Geometry all at Once by Watching Videos》2018

（论文有点难看懂，以后再看）

<img src="/images/depth6.png">



### 6. 利用分割作为attention联合depth来训练

6.1. 《Look Deeper into Depth: Monocular Depth Estimation with Semantic Booster and Attention-Driven Loss》2018

- 分析数据集`depth`的分布，提出更多关注正样本的loss
- 提出使用**语义分割**和depth预测的loss【更多中心在loss这一块】
- 设计网络结构：融合depth和semantic的特征

<img src="/images/depth7.png">



### 7. 改进Loss

7.1. 《Deep Ordinal Regression Network for Monocular Depth Estimation》2018

【使用Ordinal Loss代替MSE Loss】

- 当前的网络的问题：
  - 使用MSE作为loss：slow convergence/ unsatisfactory local solutions
  - 复杂的网络(为了获得high-resolution的depth map)：skip-connections/multi-layer deconv
- Method
  - spacing-increasing discretization(SID) strategy：离散depth并recast【depth prediction的**不确定性**随着depth的增加**而增加**(远距离，depth的预测范围更大)，所以在预测larger depth时需要**允许**相对**更大的error**，来避免larger depth差值的影响产生更大的影响。】
  - 采用multi-scale network(ASPP模块)：避免使用spatial pooling 或 并行地使用多尺度特征

<img src="/images/depth8.png">