---
title: 目标检测总结
date: 2020/2/18/22:41
tags: 
- ml
- dl 
- object-detection
categories: 
- ml-learn
---



###  改进主流网络

#### Two-stage

R-CNN：Selective Search

 $\longrightarrow$ Fast R-CNN： 将所有候选区域都送入CNN转为特征图的步骤改为直接将原图传入CNN后，在得到的特征图上寻找候选框对应的区域    

$~\longrightarrow$  Faster R-CNN:  增加了RPN，用来替代原先的候选框搜索   

 $~~\longrightarrow$  RFCN：Faster R-CNN中的ROI，用来从不同尺寸的特征图得到统一输出尺寸，由于ROI后的网络不具有平移不变性，且ROI后的层数会影响检测效率，改进的RFCN使用位置敏感得分图来代替感兴趣判断子网络（ROI和感兴趣判断子网络有区别吗？），并利用位置敏感的ROI池化层，直接对池化后结果进行判别。  

 $~~\longrightarrow$  Mask R-CNN： 在Faster R-CNN中池化的时候都进行了取整操作，会影响语义分割等像素级别任务的精度，Mask R-CNN使用双线性插值填补非整数位置像素。   

$~~\longrightarrow$ 后处理的改进： NMS/Soft-NMS/Softer-NMS

<!--more-->

#### One-stage

YOLO-v1

$~\longrightarrow$ SSD：在后几层卷积上对不同尺度上的特征图运用anchor方法进行候选框提取

$~~\longrightarrow$ YOLO-v2：增加了批归一化，高分辨率分类器，直接目标框位置检测，多尺度训练

$~\longrightarrow$ YOLO-v3：改进了backbone（v3中的池化基本由卷积实现，引入残差块），结合特征金字塔网络FPN

#### One-stage + Two-stage

$~\longrightarrow$ RON：将SSD与Faster R-CNN相结合，改进backbone使用了与RPN相似的策略来引导目标对象搜索

$~\longrightarrow$ RefineDet：融合了RPN，FPN，SSD，（比较复杂，看不懂）

### 小目标检测

#### backbone改进

$~\longrightarrow$ DetNet：基于RestNet-50改进

$~\longrightarrow$ DenseNet：使用了密集块进行层与层的连接

$~~\longrightarrow$ STDN：将DenseNet作为特征提取网络，引入尺寸转换层

#### 增加感受野

$\longrightarrow$ RFB：在Inception的基础上加入膨胀卷积/空洞卷积，增大感受野

$\longrightarrow$ TridenNet：以ResNet-101作为backbone，加入空洞卷积，通过多分支结构，权重共享和指定尺度过滤训练

#### 特征融合

FPN

$\longrightarrow$FPN+RPN

$\longrightarrow$DES：提高低层特征图语义信息与高层特征语义信息

$\longrightarrow$NAS-FPN：由于FPN是人为设计的，因此可使用强化学习在一个包含所有跨尺度连接的可拓展搜索空间中   选择最优的特征金字塔架构

#### 级联卷积神经网络

$\longrightarrow$Faster R-CNN：将两个RPN进行级联优化

$\longrightarrow$Intertive BBox：在检测阶段（应该就是推理阶段）使用级联的网络架构，将前一个检测模型回归得到的边界框初始化为下一个检测模型的边界框，迭代三次

$\longrightarrow$Integral loss：针对迭代式边界框回归（如Intertive BBox）单一IOU阈值的问题，将Intertive BBox的三次迭代改为三条支路，每条支路使用不同的IOU阈值

$\longrightarrow$ Cascade R-CNN：该模型基于前一阶段的输出进行训练，与Intertive BBox相比应该就是Intertive BBox在推理时的结构与其一致，Integral Loss使用的是不同重叠度阈值训练得到的。同时Cascade R-CNN也包含了不同IOU阈值的回归网络，用来提高IOU阈值下的正样本比例。

$\longrightarrow$HRNet：使用高分辨率的子网络作为第一阶段，然后逐渐并行添加较低分辨率的子网络，得到多个阶段的子网络的输出，之后输出又可以重新传到第一阶段的高分辨率特征图，与其进行融合。

#### 模型训练方式

$\longrightarrow$YOLO-v2：YOLO-v2的输入是416×416，而backbone在ImageNet的预训练模型使用的是224×224,v2就先在416×416的ImageNet上预训练了网络。

$\longrightarrow$SNIP：图像金子塔的尺寸归一化，使用图像金字塔来处理不同尺寸的数据，bp时只将与预训练模型所基于的训练数据尺寸相对应的ROI的梯度进行回传。

$\longrightarrow$perceptual GAN：将所有的物体分为大物体和小物体两个子集,通过挖掘不同尺度物体间的结构关联，perceptual GAN 包含两个子网络: 生成网络( 小物体) 和判别网络 ( 大物体) 。生成网络使用深度残差特征生成模型,通过引入低层细粒度的特征将原始较差的特征转换为高分辨的特征; 判别网络包含两个分支:对抗分支和感知分支,对抗分支用来分辨小物体生成的高分辨率特征与真实的大物体特征,感知分支则通过感知损失提升检测率 。

### 多类别物体检测

#### 训练方式优化

$\longrightarrow$LSDA：LSDA 首先利用分类数据集初始化神经网络,然后再利用检测数据集进行微调,最后将检测数据集训练的参数迁移到没有检测标签的类别上来实现提高目标检测类别数目

$\longrightarrow$YOLO9000：使用联合训练策略通过使用标记为检测的图像来学习边界框的坐标预测和目标类别的特定信息 ,将检
测和分类数据集混合用于模型训练。 当网络看到标记为检测的图像时,能够基于完整的 YOLOv2 损失函数进行反向传播;当看到一个分类的图像时,只能够从该架构的分类特定部分反向传播损失 。还使用了WordNet（看不懂）。

$\longrightarrow$软采样：将ROI 的梯度重设为与正实例重叠的函数,确保不确定的背景区域相对于难样本 ( hard negative) 的权重更小

#### 网络结构改进

$\longrightarrow$R-FCN-3000：R-FCN中的位置敏感得分对于多类别情况下计算量会急剧增加，因此提出R-FCN-3000，该网络分为两条支路，一条与R-FCN类似，进行大类别检测，另一条用来进行细粒度分类（所有类别），最终将两条支路得到的类别分数相乘，得到最终的分数

$\longrightarrow$HSJT：HSJT 首先利用目标类别之间的关系,建立新的层次网络模型,进一步提高识别性能;其次将边界框级标记图像和图像级标记图像结合起来进行联合训练 ,提出一种联合训练生成大规模半监督目标检测的方法 。

### 轻量化模型

$\longrightarrow$MobileNet-v1：将传统卷积分为深度可分离卷积（即一个卷积核只负责对一个特征图进行卷积）+逐点卷积（1×1卷积）

$~\longrightarrow$MobileNet-v2：增加了Linear bottlenecks，去除小维度输出层后的非线性激活层，增加了inverted residual block，对维度先扩增再缩减。

$~~\longrightarrow$MobileNet-v3：加入互补搜索技术组合（通过NAS进行架构设计），进行网络结构改进

$\longrightarrow$ShuffleNetv1：

$~\longrightarrow$ShuffleNetc2:



ILSVRC = ImageNet Large Scale Visual Recognition Challenge