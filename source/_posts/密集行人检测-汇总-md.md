---
title: 密集行人检测-汇总.md
date: 2020-01-05 20:08:16
tags: [行人检测]
---

### 密集场景下的人群检测

行人检测论文汇总：https://www.starlg.cn/2018/08/17/Pedestrian-Detection-Sources/

#### 1. Repulsion Loss: Detecting Pedestrians in a Crowd

<img src="/images/Repulsion loss.png" style="zoom:35%">

* face++（2017.11）CVPR2018

* 行人检测：类内遮挡（crowd遮挡）

* 问题：行人T与B重叠时，检测器检测到的行人特征相似，原本应该检测T的bbox偏移到B上。

* 解决：提出Repulsion损失

  * 要求：pred_box不仅接近目标T，并且远离周围的GT_box
  * 即：目标的吸引、目标的排斥
  * Figure1：pred_box偏移到B时会产生惩罚
  * <img src="/images/Repulsion loss2.png" style="zoom:55%">

* ##### Repulsion Loss：https://blog.csdn.net/gbyy42299/article/details/83956648

  - Repulsion loss定义：$L=L_{Attr}+\alpha*L_{RepGT}+\beta*L_{RepBox}$

  - $L_{Attr}：$即smooth l1损失，让pred和gt更接近

  - $L_{RepGT}：$使得pred_box $P$和周围的非目标框$G$尽可能远离（$G$是除匹配上的目标框意外的IOU最大的目标框，即和它overlap第二大的gt_box）

    - **使用IOG而不是smooth l1 loss：**若使用smooth l1 loss来进行pred_box和非目标框的排斥，只会让距离线性的增加，确保越远远好；但是IOG这类用交集区域来衡量loss的，却是检测框衡量的实质，只关注于最小化与非目标框的重叠区域。

    - **IOG 而不是IOU：**使用Intersection over GT（IOG）而不是IOU，因为如果使用IOU，网络可以学习到<u>放大pred_box</u>来包含gt_box就能够使得距离最小（**因为当$B^P$完全包含$G_{Rep}^P$时，IOU的分子部分交集时是固定的，而可以通过分母部分，扩大pre_box的面积，则并集的面积会增大，达到减少IOU的目的，但是实际上overlap区域并没有减小，没有达到实际效果**），这并不是我们想要的，而我们使用$IoG(B,G)=\frac{area(B\cap G)}{area(G)}$，确保分母面积不改变，就可以规避这样的问题，只会要求分子overlap的区域减少

    - 计算RepGT，当IoG越大，RepGT的loss就越大：
      $$
      L_{RepGT}=\frac{\sum_{P \in P_+}Smooth_{ln}(IoG(B^P,G_{Rep}^{P}))}{|P_+|}
      $$

      $$
      Smooth_{ln}=
      \begin{cases}
        -ln(1-x), &  x \leq \sigma \\
        \frac{x-\sigma}{1-\sigma}-ln(1-\sigma), & x > \sigma
      \end{cases}
      $$

      **Note:**$P_+$为所有positive的proposals，$B^P$为预测的positive的box，G为gt_box。$\sigma$是一个调整$L_{RepGT}$敏感程度的超参数，$\sum$和$|P_+|$相当于取平均

      

  - $L_{RepBox}：$使得pred_box $P_i$和周围的其他pred_box $P_j$尽可能远离，分别匹配上不同的gt_box，采用IOU来衡量距离
    $$
    L_{RepBox}=\frac{\sum_{i\neq j}Smooth_{ln}(IoU(B^{P_i}, B^{P_j}))}{\sum_{i\neq j}1[IoU(B^{P_i},B^{P_j})>0]+\epsilon}
    $$

#### 2. Occlusion-aware R-CNN: Detecting pedestrians in a Crowd

* 中科院（2018.7）

* 目标：解决人群遮挡

* 主要：优化目标+网络结构

* 优化目标：

  * $L_{reg}$：预测框逼近gt框

  * $L_{com}$：属于同一gt框的多个预测框尽量集中

    <img src="/images/aggregation loss.png" style="zoom:55%">

* 网络结构：

  * 修改ROI Pooling：人体具有特殊的结构，利用这个先验，将ROI区域切分出5个子区域，蓝色所示。对每个蓝色框经过ROI Pooling操作后变成$m\times m$的特征图，再将特征进行element-wise sum，将5个局部的特征合并与1个整体的特征合并，去做最后的分类和回归。

  * my_opinion：仅仅做element-wise sum，缺乏严谨性；可能需要更合理的特征融合方式。

    <img src="/images/POROI.png" style="zoom:55%">

#### [4. Bi-box regression for pedestrian detection and occlusion estimation 2018](http://openaccess.thecvf.com/content_ECCV_2018/papers/CHUNLUAN_ZHOU_Bi-box_Regression_for_ECCV_2018_paper.pdf)

* 利用一个网络：行人检测+遮挡估计（即并行两个分支，分别输出两个bbx，一个完整的行人框，另一个行人的可见部分框）

  <img src="/images/Bi-box.png" style="zoom:55%">

* 网络结构图

  <img src="/images/network.png" style="zoom:55%">

* **整体估计分支**处理流程与通用目标检测一致

* **可见部分估计分支**

  * 正样本条件：proposal不仅要和整个标注框的重叠程度大于一定阈值；还要和标注框内行人可见部分重叠程度大于一定阈值。（蓝色框满足前者条件，但是不满足后者条件，所以不作为正样本）

    <img src="/images/label.png" style="zoom:25%">

#### [5. Occluded Pedestrian Detection Through Guided Attention in CNNs 2018](http://openaccess.thecvf.com/content_cvpr_2018/papers/Zhang_Occluded_Pedestrian_Detection_CVPR_2018_paper.pdf)

* 正常的FasterRCNN结构+attention网络
* attetion network
  * Self Attention Net：channel-wise attention（SE block）
  * Visible-box Attention Net：对人体可见部分建模（估计**可见部分框**）
    * 提出一个occlusion pattern（类似内嵌小网络）：预测可见部分的4个类别（全部可见、上部可见、左侧可见、右侧可见）
  * Part Attention Net：
    * 认为visible-box有时可能出现的不规律，不好分类到4个类别
    * 对visible-box的训练代价大
    * 使用body part detection作为额外的监督（head、shoulder、arm，etc）
    * 在行人检测数据集上，没有body part的标注；则选择采用**预训练**的part detection网络（在MPII Pose数据集上）；该网络提供14个人体关键点，不用再在行人检测数据集上finetuning，就可以有好的效果。
    * 使用 人体关键点检测网络 中的特征（具有较好的对人体特征的响应）来作为有效的特征表示来监督attention network

<img src="/images/attention.png" style="zoom:75%">

<img src="/images/attention2.png" style="zoom:65%">

#### 6. What Can Help Pedestrian Detection? 2017

* 存在的问题：
  * 行人与背景的辨识度小
  * 密集场景：人与人之间的边界模糊
* 解决：用额外的特征提升行人检测器的性能
* 实验：
  * 采用梯度、分割、热力信息
  * 时间序列
  * 深度通道
* 最后采用像素分割作为额外的特征监督；提取conv1_2/conv2_2/con3_3/conv4_3的特征图进行channel cocncat，再通过全卷积的结构，生成通道特征。再将通道特征concat到原body network上，输入RPN和FRCNN。

<img src="/images/hyperlearner.png" style="zoom:65%">

<img src="/images/body.png" style="zoom:45%">

#### [7. Scale-aware Fast R-CNN for Pedestrian Detection 2015](https://arxiv.org/pdf/1510.08160.pdf)

* 问题：行人尺度，根据距离远近，行人具有相差极大的size，且特征表示差异也极大
* 解决：在一个网络中设计大、小网络，大网络针对大size的行人检测；小网络针对小size的行人检测；
  * 使用scale-aware加权层作为门函数；该层根据proposal的高度（宽度随着行人姿态的变化很大）来判断行人的size。根据proposal的size来判定给大小网络分别赋予不同的权重。
  * 最终将大小网络的预测结果进行reweight，输出最终结果

<img src="/images/SA-FRCNN.png" style="zoom:65%">

#### 8. JointNet 2019

使用head和body同时来检测在crowd scene下的人体

##### Algorithm1：

$IoH=\frac{Area\ of\ Overlap}{Area\ of\ Head-box}$

$H$为人头检测得到的集合（NMS之后），$B_1,B_2$为行人检测得到的集合（NMS之前后之后的）

* 寻找不能匹配的head，加入到$H_m$中
  * 对每个head，遍历所有body（这里的body遍历的是<u>NMS之后</u>的body），如果他们的$IoH> \lambda$，就把他们这个配对的score加入到集合中；最后从这个集合选取最大的score值如果$<\beta$，就说明是未被匹配的head

* 不能匹配的head有两种情况：（1）本应该与该head匹配的body由于occlusion被NMS抑制；（2）这个head本身就是一个false positive(虚警)
* 当按以上的方法得到$H_m$之后，再使用一遍后处理方法，根据score小于low_thresh或大于high_thresh，删除或增加这个head与其对应的body

* 得到不能匹配的head（$H_m$），根据其和body的overlap值，如果大于high_thresh，说明是和当前已有的body框有较大overlap，但是它又是未匹配的head，所以大概率是本应该与其匹配的body由于occlusion被NMS抑制，这种head应该add它的body（这里的body遍历的是<u>NMS之前</u>的body）；而如果overlap小于low_thresh，说明确实是一个false psitive，应该delete