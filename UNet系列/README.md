# SemanticSegmentationSOTA/UNet系列/
一.论文综述
图像分割的U-Net系列方法
https://mp.weixin.qq.com/s?__biz=MzI5MDUyMDIxNA==&mid=2247493160&idx=1&sn=93c06b65e04d8c7034fd50d916c6e136&chksm=ec1c0bd1db6b82c72e18cbdf01e609127c4ed452ec1a41e13000c0f86945b34d84cd25736e76&mpshare=1&scene=24&srcid=0411rQnM9nDaB8nR884MwCcl&sharer_sharetime=1586566488017&sharer_shareid=1457b5f9adcd4bad2bd4c8a878b3fa6f&key=042d77279f472613c451ae20809978132c9b887a431a130847400df30a73c9333b65913e454a414e227312e67423e0e75722443d66070bc2db8953d290fdbb653dcc446b915165029d20df3128e314b2&ascene=1&uin=MjQ4OTg4MDUxMQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A0bmQV5Ju34Dt%2FXaY%2BDMsqI%3D&pass_ticket=z07bwmrtJGVo64voTtPlUhFO4HVcWg0bO1w5f7WgW6vlddv9G8KfX20XCfztXKfr
Unet神经网络为什么会在医学图像分割表现好？ 
https://mp.weixin.qq.com/s?__biz=MzI5MDUyMDIxNA==&mid=2247491119&idx=1&sn=a2ad7c18b132f6df0e3cf3875fcfd642&chksm=ec1ff3d6db687ac018bf5660678eb7d8587fc5a249961500950315641f84769e8f0c70d1116c&mpshare=1&scene=24&srcid=0411WAjG8FCNLfiIM8E023SS&sharer_sharetime=1586565634456&sharer_shareid=1457b5f9adcd4bad2bd4c8a878b3fa6f&key=6ed4a988d1572e44df0c18b9d991b30edca482b4629e551d26578b1e1dbae911b16063d2cda10dcf7acfcd94f90a75911f94187b36f55f53ff0a2d65bd9a2c122f600095f410cc2f8897e82dc189923e&ascene=1&uin=MjQ4OTg4MDUxMQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A9X7FUjmfxwYlUh3fQGE8sM%3D&pass_ticket=z07bwmrtJGVo64voTtPlUhFO4HVcWg0bO1w5f7WgW6vlddv9G8KfX20XCfztXKfr
当UNet遇见ResNet会发生什么？
https://mp.weixin.qq.com/s?__biz=MzI5MDUyMDIxNA==&mid=2247494378&idx=3&sn=ae010f07660ccf4ea60dd00e97bdecf8&chksm=ec1c0713db6b8e05584adc2bfa8a97f53b67ee5eb697351809284952132c473b6473f3530953&scene=0&xtrack=1&key=f4f05596ca2dcc74043ddc67d7050254e10cfa19fd3163cacf9f8c421746972c605d9438303b18d26b65c39b4eea319c6f305e7662201d648add8e3ed4659b565af3c53196d9de54fb77394f374c21d1&ascene=1&uin=MjQ4OTg4MDUxMQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A6UFdyBrC9MHyK%2BMqXM2hpQ%3D&pass_ticket=z07bwmrtJGVo64voTtPlUhFO4HVcWg0bO1w5f7WgW6vlddv9G8KfX20XCfztXKfr

二.论文解读
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
MultiRes block通过串联3*3卷积、跳跃连接融合不同尺度特征图；采用Res path消除低尺度和高尺度特征融合的差异；

***********************AttentionU-Net: LearningWheretoLookforthePancreas***********************
1.采用 Attention Gate机制实现不同尺度特征的融合，以抑制不相关区域，提升显著性特，从而目标不同尺度和大小的变化。
2.分别对低尺度和高尺度特征进行卷积，然后相加，Relu激活，卷积，sigmoid激活，重采样；






