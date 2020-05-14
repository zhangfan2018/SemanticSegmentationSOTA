# SemanticSegmentationSOTA/UNet系列/
**********************************nnUNet文章解读********************************************************
1.数据预处理
CT图像求前景的CT值，采用CT值的0.5%~0.95%对图像进行灰度值截断；
CT和MRI进行全局标准化，v' = (v - Mean)/std, mean和std是所有训练集中前景象素的均值和标准差；
像素空间归一化：对每个轴单独做归一化，并且采用所有训练集中spacing的中值进行归一化，采用三阶样条插值，对于各项异性比较严重的数据
（层内分辨率是层间分辨率的3倍以上）插值会出现人工结构，层间插值采用最近邻插值。对于图像标签，采用线性插值。

2.网络结构
采用 2D、3D、Cascade 3D UNet, 二阶Cascade Unet首先采用低分辨率的输入，接着采用高分辨率的输入。
Padding 卷积， instance normalization, Leaky ReLU;
自动设置patch size、batch size、pooling以最大化利用显存，优先最大化patch size, pooling的下限是4个体素；
第一层的卷积核个数为30，后面层逐渐递增。

3.训练参数
采用五折交叉验证进行训练，采用交叉熵损失和Dice损失的和作为损失函数，优化器为Adam,学习率为3e-4，weight decay为3e-5；
30个epoch内损失不降低，学习率降低0.2，如果学习率低于10-6或者1000epoch不下降，学习停止；
采用弹性形变、随机尺度缩放、随机旋转以及gamma增强，对于各项异性数据，增强作用于层内空间。

4.推理阶段
采用滑动窗，取patch size的一般进行预测；根据交叉验证结果，对2D、3D、以及Cascade模型进行集成，也对交叉验证的5个模型进行集成。

*********************************non-local UNet论文解读*************************************************
1.链接：https://mp.weixin.qq.com/s?__biz=MzUxNjcxMjQxNg==&mid=2247496999&idx=4&sn=f0d982e637a909736fdaa7beb6e84d97&chksm=f9a187a8ced60ebe9838710825652ca75a24fd856d1ae98a9ecf2db18665a83b666bede3f593&scene=0&xtrack=1&key=1f1e787ff7a3f90215d8e152336fabdc6135ac5bc4aae6737b0452ec9a166553ba14e610936e323cbc5b20e7d32eeba8a8973c6d703d4df6b2947fe9d98637dc071447cc28a66221c64aec3328aca7a3&ascene=1&uin=MjQ4OTg4MDUxMQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A1fFb%2FMT9pAoAIjXKnbVS94%3D&pass_ticket=uAqF69sdO2x6u8blAYWN1zHW%2BaV0LauVzve1wRTHD2XW4%2BoLkb2Sadp4EU86bsVs

*********************************MultiResUNet***********************************************************
1.代码地址：https://github.com/nibtehaz/MultiResUNet

