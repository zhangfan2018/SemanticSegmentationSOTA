# SemanticSegmentationSOTA/边界分割
*******Boundary-aware Context NN for medical image segmentation.***********
1.金字塔边界提取模块，原特征图减去池化后的特征图，得到边界特征图；
2.多任务学习模块，对UNet每个尺度的特征图进行卷积和上采样，分别得到边界和目标分割任务头，
通过Gate机制实现interactive attention，融合边界分割和目标分割任务，以促进学习；
3.通过Gate机制实现不同尺度特征的融合；

******Contour loss boundary-aware learning for salient object detection****
1.目标分割任务中，靠近目标中心的像素比较一致，分割容易；靠近边界的像素差异较大，比较模糊，
分割困难。采用带权重的交叉熵损失，生成距离图，越靠近边界的权重较大，靠近目标中心的权重较小，
以增强边界的学习。

*************GatedFullyFusionforSemanticSegmentation***********************
1.PSPNet作为backbone,最底层采用PSP模块；
2.采用Gate机制的GFF实现不同尺度特征的融合；
3.采用密集特征融合，DFP对不同尺度分割结果进行融合，对于小目标具有更加的分割效果。

