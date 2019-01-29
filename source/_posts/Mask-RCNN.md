---
title: Mask RCNN
date: 2018-05-05 09:23:56
categories: 目标检测
tags: [计算机视觉，目标检测]

---

这篇论文是2017年4月份的KaiMing大神的论文，又是一个里程碑的论文，综合了目标检测和
图像分割领域。现在COCO排行榜上基本都是以Mask RCNN为baseline改进的detector
<!--more-->

# 我的工作
2017年12月份，我开始看这份工作，当初FB的[Detetron平台](https://github.com/facebookresearch/Detectron)还没开放出来，我选择了图森开放的mxnet版本的[mx-maskrcnn](https://github.com/TuSimple/mx-maskrcnn)，从此也开启了mxnet的炼丹之路。

# 论文解读
## 文章的一些point                                                              
1、Mask RCNN使用了**RoiAlign**代替了Faster rcnn的roipooling(对于特征的粗糙的空间
量化)，roiAlign能够解决像素点上的非对齐特质，尤其减弱了量化的粗糙性。
2、**decouple** mask和class的预测。我们独立地预测了一个二值的mask对于每个类，因>此对于类之间没有了竞争，然后依靠roi的分类分支去预测类别。相比FCN，FCN使用了每个>像素多个类别的分类，couple分割和分类，但是在我们这个实验上面，效果不是很好。
3、不像之前对于mask的预测是用fc层的，我们使用**全卷积层**的表示不仅**减少了参数>量而且更精确**。
4、对于cityscape数据集，带标签的训练数据只有千张，所以对于小的数据集，用101-layer和50layer的差距不大
5、类别内重叠问题：我们的方法能够有效的提升**重叠物体**的分割精度
6、为了解决cityscape某些类物体训练样本少的问题，我们先用COCO做预训练，
7、我们专门使用了cls分支来去做class的预测，用来去选择**mask分支**的mask。
8、我们在mask分支使用**FCN**来预测m\*m的mask，这样能够保持m\*m的物体的空间结构而
且不用把他降维到缺少空间信息的向量表示。而且FCN相比于FC层需要更少的参数。

## 实验对比
1、FCN进行的语义分析，使用的是对每个像素softmax和多项交叉熵损失。而maskrcnn，使>用的是对**每个像素的softmax和二值损失**。
2、类无关的mask：maskrcnn使用的一个单个的m\*m的mask；而每个类一个m\*m的mask的AP>更低
3、使用FCN的全卷积代替MLP的全连接。
4、我们把Maskrcnn去掉mask 分支，就相当于可以表示成faster rcnn+roialign，他只比带
mask分支的maskrcnn低了0.9个AP点；所以这个0.9的提升在于多任务的训练**(即检测和分>割的合并训练)**

## 论文的总结
　　Mask R-CNN是在faster R-CNN上的扩展，在其已有的用于bbx分支上添加了一个并行的>用于预测目标掩码的分支。Mask R-CNN的训练很简单，只是在R-CNN的基础增加了少量的计>算量，大约为5fps。另外，R-CNN掩码能够更好地适用于其他任务，例如估计同一图片中人>物的姿态，本文在COCO挑战中的3种任务（包括实例分割、边界框目标探测、任务关键点检>测）中都获得了最好的成绩。

# 细节
　　MaskRCNN是在Faster RCNN基础上做的改进。

**Faster RCNN**
　　faster RCNN由两个阶段组成，第一阶段为RPN，提出候选目标的bbox，第二阶段为fast RCNN的本质，使用RoiPool从每一个候选区域使用分类和bbox回归来提取特征。
**Mask RCNN**
　　Mask RCNN采用了相同的两部分，第一部分为RPN，第二部分为与预测class和box offset并行的对于每个Roi输出binary mask。
　　我们的方法跟随fast RCNN的灵感，应用bbox分类和回归并行的。(大大简化RCNN的多阶
段pipeline)
　　在训练的时候，我们定义了一个多任务的loss对于每个采样的Roi的损失函数Lclass+Lbox+Lmask。对每个RoI的mask分支，其输出维度为K\*m\*m。其中K表示对m\*m的图像编码K个
二分类mask，每一个mask有K个类别。所以需要应用单像素的sigmoid进行二分类，并定义Lmask为平均二分类cross-entropy loss。对于类别为k的RoI，Lmask定义在第k个掩膜（其他>掩膜输出对loss没贡献） 对Lmask的定义允许网络为每个类别生成mask且在类别之间没有竞
争。用专门的分类分支来预测类别标签（标签也用来选择输出mask），这样就解耦了类别与
mask预测。而FCN用的是单像素的softmax和multinomialcross-entropy loss，在这种情况>下，mask和classes之间存在竞争，mask rcnn使用per-pixel sigmoid和binary loss就不会
。通过实验表时这是提高instance segmentation的关键点。

**Mask Representation:**
　　一个mask对目标的空间布局进行编码。因此不像class标签或者box offset必将通过全>连接层进入一个短的output向量，提取空间结构的mask能够在像素级别即pixel-to-pixel下
完成(由对应的convolutions提供)，本质还是卷积代替全连接，维持空间信息。
　　对每个RoI，使用FCN预测m*m个mask，这就允许每个mask分支保持明确的m*m的目标空间
布局，而不用将其陷入向量描述（缺少空间维度）。不同于先前的模型使用fc layers进行mask预测，Mask RCNN的全连接层需要更少的参数，且精度更高
　　这种pixel-to-pixel的行为需要RoI特征，这些特征是小的特征图，为了一致的较好的>保持明确的单像素空间对应关系，Mask RCNN提出了RoIAlign，这是对mask预测的关键。

**RoiAlign:**
　　RoIPool是对每个RoI提取小的特征图（比如7\*7）的标准操作，RoiPool首先将一个floating number的RoI量化为discrete granularity（离散粒度）的特征图，然后被量化后的RoI被分成小的空间bin（本身已被量化），最后将每个bin聚合成特征值（通常使用maxpooling）。量化执行的方式为，比如[x/16]，其中16为特征图的stride，[]表示取整。同样的，
当划分bin（比如7\*7）的时候，也要执行量化。 这些量化操作在RoI和提取的特征中引入>了misaligments，这也许对分类没有影响，但对预测pixel-accurate的mask有较大的负面作
用。
　　移除RoIPool 的粗糙量化，从而调整特征的提取与输入。Mask rcnn提出的改变很简单>：避免任何的RoI边界（boundaries）或者bin的量化（比如使用x/16而不是[x/16]）。我们
在每个RoI bin中的4个常规的采样位置使用双线性插值计算输入特征的准确值，并将结果融
合（使用max或average）。RoIAlign对算法有了很大的提高。

**网络结构**
　　Mask RCNN的结构分为3个部分，第一个是主干网络用来进行特征提取，第二个是头结构
用来做边界框识别（分类和回归），第三个就是mask预测用来对每一个ROI进行区分。
　　**主干网络**，使用了50层的*ResNet和FPN*(feature pyramid network)，这两种的结
合能够给Mask RCNN提供良好的精度和速度上的提升。
　　**头结构**，在网络的头部，我们近似使用faster rcnn之前的结构，添加一个全卷积mask预测分支。特别地，我们从ResNet和FPN来扩展Faster R-CNN box heads。 图3右部分是
该网络的头结构，其中14X14是空间分辨率spatial resolution，256是频道数，3X3卷积核>进行卷积4次得到14X14X256，再2×2 且步长 stride为 2进行deconvs，并且在隐层使用ReLU，得到28X28X256再进行全连接得到输出，作为网络主干的输入。

# 我的复现
在4块TITAN X上跑了20多小时，把在cityscape上的数据结果跑出来了。
mAP=0.316
<img src="img/maskrcnn.png">
