# 数据集

现有工作主要使用MS-COCO (Lin et al.， 2014)、Visual Genome (Krishna et al.， 2017)和YFCC100M (Thomee et al.， 2016)三个数据集。虽然MS-COCO和Visual Genome是高质量的人工标记数据集，但它们的规模很小，每个数据集大约有10万张训练图片。相比之下，其他计算机视觉系统在多达35亿张照片上进行了训练(Mahajan et al.， 2018)。YFCC100M有1亿张照片，是一个可能的替代方案，但每张图片的元数据稀疏且质量参差不齐。许多图像使用自动生成的文件名，如20160716 113957.jpg格式的“标题”或包含相机曝光设置的“描述”。经过过滤，只保留带有自然语言标题和/或英文描述的图像后，数据集缩小了6倍，只有1500万张照片。这与ImageNet的大小大致相同。

自然语言监督的一个主要动机是这种形式的大量数据在互联网上公开可用。由于现有的数据集不能充分反映这种可能性，因此只考虑这些数据集的结果会低估这一研究方向的潜力。为了解决这个问题，我们构建了一个新的数据集，其中包含4亿对(图像，文本)对，这些数据来自互联网上各种公开可用的资源。为了尝试覆盖尽可能广泛的视觉概念集，我们搜索(图像，文本)对作为构建过程的一部分，其文本包含500,000个查询集中的一个.我们通过在每个查询中包含多达20000对（图像、文本）来大致平衡类别结果。生成的数据集具有与用于训练GPT-2的WebText数据集相似的总字数。我们将此数据集称为WebImageText的WIT。

# 选择高效预训练方法

![image-20230920164353607](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230920164353607.png)

先前的研究表明**对比目标函数(contrastive objectives)**的表现可以超越等价的预测目标函数(predictive objective)。

论文利用对比学习的思路提出了一种更简单的预测任务：**将一段text作为整体预测其与哪个image匹配.**

即给定$N$个(image, text)对，CLIP的目标是**从**$N×N$ **个可能的(image, text)对中找到实际出现的**.

为了完成这个任务，CLIP**同时训练一个image encoder和一个text encoder**，对于$N$个真实(image, text)对，CLIP需要**最大化其对应的image embedding和text embedding之间的余弦相似度**，对于$N2−N$个错误(image, text)对，则最小化。

如下图伪码所示，矩阵$np.dot(I_e,T_e.T)$是$N^2$个对的余弦相似度矩阵，它被分别按行、列视为一个batch的N分类概率输出，通过优化其与 $labels=[1,2,...,N]$ 的交叉熵，间接实现最大化矩阵对角元素

![image-20230920164243479](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230920164243479.png)

# 模型选择

对于图像编码器，使用了两种模型。

- Resnet-50。做了一些改进，如ResNetD、antialiased rect-2 blur pooling、用注意力池化机制取代了全局平均池化层
- VIT。只做了一些小改动，embedding输入Transformer前加了一个layer normalization。

对于文本编码器，论文采用了Transformer，输入文本上使用了BPE，文本序列前后分别添加了[SOS]和[EOS] token，文本representation通过[EOS]对应的输出获得

对于image encoder，缩放采用EfficientNet的方法，对于text encoder，则只缩放宽度，论文称CLIP的表现对text encoder并不敏感

# 模型训练

