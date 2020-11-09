# VisLocOdomMapCVPR2020_Champion
Champion solution of VisLocOdomMapCVPR2020, notes of read papers and principle
## 关于视觉定位挑战赛
视觉定位是一个估计6自由度（DoF）相机姿态的问题，从中获取一个给定的图像相对于一个参考场景表示。视觉定位是增强、混合和虚拟现实等应用以及机器人技术（如自动驾驶汽车）的关键技术。

为了评估较长时间内的视觉定位，官方提供了基准数据集，旨在评估由季节（夏季、冬季、春季等）和照明（黎明、白天、日落、夜晚）条件变化引起的较大外观变化的6自由度姿态估计精度。每个数据集由一组参考图像及其相应的地面真实姿态和一组查询图像组成。官方为每个数据集提供一个三角化的三维模型，并可用于基于结构的定位方法。

主页地址：
>https://www.visuallocalization.net/
>

## 冠军方案
方案名称：Hierarchical Localization - SuperPoint + SuperGlue(简称，Hloc+SP+SG)

粗定位：NetVLAD3 4 retrieval (trained on Pitts-30k, top 50)
细定位：SP+SG+RANSAC PnP

方案采用了分级定位(Hierarchical Localization)的方案，即先粗定位再细定位。
该方案的主要特点在于特征匹配阶段使用了最近比较火的SuperPoint+SuperGlue(后续简称SP+SG)，
这两个网络在之前有过介绍，当时只是提到了二者在大视角特征匹配时效果极佳。
本方案的成功应用，可见这两个网络在该定位任务中也能发光发热。

## NetVLAD
我们解决了大规模视觉位置识别的问题，该任务是快速准确地识别给定查询照片的位置。
我们提出以下三个主要贡献。首先，我们开发了一种卷积神经网络（CNN）架构，
该架构可以以端到端的方式直接进行位置识别任务的训练。
该体系结构的主要组件NetVLAD是一个新的广义VLAD层，
其灵感来自图像检索中常用的“局部聚合描述符向量”图像表示。
该层很容易插入任何CNN体系结构中，并可以通过反向传播进行训练。
其次，我们根据新的弱监督排名损失，制定了培训程序，从Google Street View Time Machine
下载的描述随着时间推移描述相同地点的图像，从而以端到端的方式学习体系结构的参数。
最后，我们表明，在两个具有挑战性的位置识别基准上，
所提出的体系结构明显优于非学习图像表示和现成的CNN描述符，
并且在标准图像检索基准上优于当前最新的紧凑型图像表示。
#### Github
[netvlad_tf_open](https://github.com/uzh-rpg/netvlad_tf_open)
#### Paper
[NetVLAD: CNN architecture for weakly supervised place recognition](https://arxiv.org/abs/1511.07247)

## Hierarchical-Localization
用于最先进的6自由度视觉定位的模块化工具箱。它实现了分层本地化，利用图像检索和特征匹配，
并且快速，准确和可扩展。该代码库与我们的用于特征匹配的图形神经网络SuperGlue相结合，
在CVPR 2020上赢得了室内/室外本地化挑战。
#### Github
[Hierarchical-Localization github address](https://github.com/cvg/Hierarchical-Localization)
#### Paper
[From Coarse to Fine: Robust Hierarchical Localization at Large Scale](https://arxiv.org/abs/1812.03506)

## COLMAP: Structure-from-Motion Revisited
是J. L. Sconberger等人于2016年发表在CVPR的文章。本文针对增量式SFM中三角化/BA等步骤进行了改进，
能够比较明显地提升SFM的精确率/鲁棒性以及重建完整性。
#### Github
[colmap](https://github.com/colmap/colmap)

[官方教程](https://colmap.github.io)
#### Paper
[CVPR2017](https://demuc.de/tutorials/cvpr2017/)

## 一种深度学习特征SuperPoint
本文出自近几年备受瞩目的创业公司MagicLeap，发表在CVPR 2018。
这篇文章设计了一种自监督网络框架，能够同时提取特征点的位置以及描述子。
相比于patch-based方法，本文提出的算法能够在原始图像提取到像素级精度的
特征点的位置及其描述子。本文提出了一种单映性适应（Homographic Adaptation）
的策略以增强特征点的复检率以及跨域的实用性（这里跨域指的是synthetic-to-real的能力，
网络模型在虚拟数据集上训练完成，同样也可以在真实场景下表现优异的能力）。

#### Paper
[一种深度学习特征SuperPoint](https://mp.weixin.qq.com/s?__biz=MzI3NDIyMjcyNg==&mid=2652161378&idx=1&sn=250c89d7a19fd2f4bfe2ca81f29030b5&chksm=f0f73e8bc780b79dc236ae58b0a879a345dacc7e912de6e3176dc9e1df0d1c7a43a4f4ade1c2&scene=21#wechat_redirect)


## 一种基于注意力机制特征匹配网络SuperGlue
论文全名《SuperGlue:Learning Feature Matching with Graph Neural Networks》，
ETHZ ASL与Magicleap联名之作，CVPR 2020 Oral，一作是来自ETHZ的实习生，
二作是当年CVPR2018 SuperPoint的作者Daniel DeTone。
本文提出了一种能够同时进行特征匹配以及滤除外点的网络。
其中特征匹配是通过求解可微分最优化转移问题（ optimal transport problem）来解决，
损失函数由GNN来构建。基于注意力机制提出了一种灵活的内容聚合机制，
这使得SuperGlue能够同时感知潜在的3D场景以及进行特征匹配。该算法与传统的，
手工设计的特征相比，能够在室内外环境中位姿估计任务中取得最好的结果，
该网络能够在GPU上达到实时，预期能够集成到sfm以及slam算法中。
SuperGlue是一种特征匹配网络，它的输入是两张图像中特征点以及描述子
（手工特征或者深度学习特征均可），输出是图像特征之间的匹配关系。

作者认为学习特征匹配可以被视为找到两簇点的局部分配关系。

作者受到了 Transformer的启发，同时将self-和cross-attention利用特征点位置
以及其视觉外观进行匹配。

#### Paper
[一种基于注意力机制特征匹配网络SuperGlue：端到端深度学习SLAM的重要里程碑](https://mp.weixin.qq.com/s?__biz=MzI3NDIyMjcyNg==&mid=2652161392&idx=1&sn=dbe9e449f5107dbc1bc6824b8fd85832&chksm=f0f73e99c780b78f2adf803a5752d834638595a1d36d4d6919d5e548d325d3925024302acc05&scene=21#wechat_redirect)


## Reference
- [**Localization**][Hierarchical-Localization](https://github.com/cvg/Hierarchical-Localization), **PDF**, **[[From Coarse to Fine: Robust Hierarchical Localization at Large Scale,CVPR 2019](https://arxiv.org/abs/1812.03506)]**, **[[SuperGlue: Learning Feature Matching with Graph Neural Networks, CVPR 2020](https://arxiv.org/abs/1911.11763)]**, 目前视觉定位挑战赛[visuallocalization.net/benchmark](https://www.visuallocalization.net/benchmark/) TOP 1的算法（使用了Hierarchical Localization - SuperPoint + SuperGlue）。

