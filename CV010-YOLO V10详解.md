# 引言

### 背景介绍

实时物体检测一直是计算机视觉领域的研究热点，旨在低延迟下准确预测图像中物体的类别和位置。该技术广泛应用于自动驾驶、机器人导航和物体跟踪等实际应用中。近年来，基于卷积神经网络（CNN）的物体检测器因其高效的性能而受到广泛关注，其中YOLO系列因其出色的性能和效率平衡而脱颖而出。

### 研究内容

本文旨在解决YOLO系列在实际部署中依赖非极大值抑制（NMS）导致的推理延迟问题，并通过优化模型架构进一步提升其性能和效率。具体来说，本文提出了以下两个主要目标：

1. 提出一种无需NMS训练的一致双分配策略，以实现高效的端到端检测。
2. 提出一种全面的效率-准确性驱动的模型设计策略，从后处理和模型架构两方面提升YOLO的性能和效率。

### 研究难点

YOLO系列在实际应用中面临的主要挑战包括：

1. **NMS依赖性**：传统的YOLO训练过程中采用一对多标签分配策略，导致推理过程中需要依赖NMS进行后处理，这不仅增加了推理延迟，还使得模型对NMS超参数敏感，难以实现最优的端到端部署。
2. **模型架构设计**：尽管已有大量研究探索了不同的模型架构设计策略，但YOLO系列在各个组件的设计上仍存在计算冗余，限制了模型的性能和效率。

### 相关工作

本文回顾了现有的实时物体检测器和端到端物体检测器的相关研究：

1. **传统YOLO系列**：YOLOv1、YOLOv2和YOLOv3是典型的三部分检测架构，包括主干、颈部和头部。后续的YOLOv4和YOLOv5引入了CSPNet设计，并结合数据增强策略和多种模型尺度。YOLOv6、YOLOv7和YOLOv8分别提出了BiC、E-ELAN和C2f等新的组件设计。
2. **端到端物体检测器**：DETR系列通过引入Transformer架构和匈牙利损失实现了一对一匹配预测，消除了手工设计的组件和后处理。其他研究如Learnable NMS和关系网络也尝试通过不同的方法实现端到端检测。

### 研究方法

本文提出了一种无需NMS训练的一致双分配策略和全面的效率-准确性驱动的模型设计策略：

1. **一致双分配策略**：通过引入双标签分配和一致的匹配度量，结合一对多和一对一标签分配的优势，实现高效的端到端检测。
2. **效率-准确性驱动的模型设计策略**：从模型架构的各个组件入手，提出了轻量级分类头、空间-通道解耦下采样和排名引导的块设计等优化方法，减少计算冗余并提升模型性能。

### 实验设计

本文在COCO数据集上对提出的YOLOv10模型进行了广泛的实验验证，具体包括：

1. **数据集**：使用COCO数据集进行训练和评估，采用标准的训练-验证-测试划分。
2. **实验设置**：所有模型在8块NVIDIA 3090 GPU上进行训练，采用SGD优化器，并结合Mosaic、Mixup和复制粘贴等数据增强策略。
3. **评估指标**：使用标准平均精度（AP）和不同IoU阈值下的AP值评估模型性能，并测量推理延迟以评估效率。

### 结果与分析

实验结果表明，YOLOv10在多个模型尺度上均取得了显著的性能和效率提升：

1. **性能提升**：YOLOv10-S在相似的AP下比RT-DETR-R18快1.8倍，YOLOv10-B在相同性能下比YOLOv9-C减少了46%的延迟。
2. **参数和计算量减少**：YOLOv10-S和YOLOv10-B分别比RT-DETR-R18和YOLOv9-C减少了2.8倍和25%的参数数量和FLOPs。
3. **全面优势**：YOLOv10在多个模型尺度上均优于现有先进模型，展示了其在计算-准确性权衡上的优越性。

### 总体结论

本文通过提出一致双分配策略和全面的效率-准确性驱动的模型设计策略，成功提升了YOLO系列的性能和效率。实验结果验证了YOLOv10在多个模型尺度上的优越性，展示了其在实时物体检测领域的潜力。未来的工作将进一步探索减少小模型中一对多训练和无需NMS训练之间的性能差距的方法。

# 研究方法

### 无NMS训练的一致双重分配

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/5f60bbd9bccdfacc4ce2a53bce9011dc-image.png)

- 双重标签分配。

  为了实现无需NMS的训练，论文提出了一致的双重分配策略。该策略结合一对多和多对一的标签分配策略的优点。具体来说，双重分配策略包括一个多对一的头和一个一对一的头。在训练过程中，两个头共同优化，使得主干和颈部能够享受多对一分配提供的丰富监督信号。在推理过程中，丢弃多对一头，只使用一对一头来做出预测，从而实现无需NMS的高效端到端部署。

- 一致的匹配度量。

为了确保两个头之间的和谐监督，论文提出了一致的匹配度量。该度量公式如下：
$$
m(\alpha,\beta)=s\cdot p^{\alpha}\cdot IoU(\hat{b},b)^{\beta}
$$
其中，*p* 是分类分数，$\hat{b}$ 和 *b* 分别表示预测和实例的边界框，*s* 表示空间先验，指示预测的锚点是否在实例内，*α* 和 *β* 是两个重要的超参数，平衡语义预测任务和位置回归任务的影响。

### 全局效率-准确性驱动的模型设计

除了后处理，YOLOs的模型架构也对效率-准确性权衡提出了巨大挑战[50, 8, 29]。尽管以前的工作探索了各种设计策略，但仍然缺乏对YOLOs中各个组件的全面检查。因此，模型架构展现出不可忽视的计算冗余和受限能力，这阻碍了其实现高效率和性能的潜力。在这里，我们旨在从效率和准确性角度全面执行YOLOs的模型设计。

效率驱动的模型设计。YOLO中的组件包括茎(stem)、下采样层、带有基本构建块的阶段以及头部。茎的计算成本较低，因此我们对其他三个部分采用效率驱动的模型设计。

（1）轻量级分类头部。在YOLO中，分类和回归头部通常共享相同的架构。然而，它们在计算开销上表现出显著的差异。例如，在YOLOv8-S中，分类头部的FLOPs和参数数量分别为回归头部的2.5倍和2.4倍。然而，在分析分类错误和回归错误的影响后（见表6），我们发现回归头部对YOLOs的性能承担了更大的重要性。因此，我们可以减少分类头部的开销，而不用担心会大幅损害性能。因此，我们简单地采用轻量级的架构用于分类头部，它由两个深度可分离的卷积[25, 9]组成，核大小为3x3，然后是一个1x1卷积。

（2）空间通道解耦的下采样。YOLO通常利用常规的3x3标准卷积，步长为2，同时实现空间下采样（从H x W到$\frac{H}{2} \times \frac{W}{2}$）和通道变换（从C到2C）。这引入了不可忽视的计算成本，即O(29*H**W**C*2)，以及参数数量，即O(18$C^2$)。相反，我们提出将空间缩减和通道增加操作解耦，以实现更高效的缩减。具体来说，我们首先利用逐点卷积来调节通道维度，然后利用深度卷积来进行空间缩减。这将计算成本降低到$O(2HWC^2+ \frac{9}{2}HWC)$，参数数量减少到$O(2C^2+18C)$。同时，在缩减过程中最大化信息保留，从而在延迟减少的同时具有竞争力。

（3）基于rank引导的模块设计：YOLOs通常在所有阶段使用相同的基本构建块，例如YOLOv8中的瓶颈块。为了彻底检查YOLOs的这种同质设计，我们利用内在秩来分析每个阶段的冗余。具体来说，我们计算每个阶段中最后一个基本块的最后一个卷积的数值秩，这计算了大于阈值的奇异值的数量。图3.(a)展示了YOLOv8的结果，表明深层阶段和大型模型更容易表现出更多的冗余。这一观察表明，简单地为所有阶段应用相同的块设计对于最佳的容量-效率权衡是次优的。为了解决这个问题，我们提出了一种基于秩的块设计方案，旨在通过紧凑的架构设计降低被证明是冗余的阶段复杂度。我们首先提出了一个紧凑的倒置块（CIB）结构，它采用廉价的深度可分离卷积进行空间混合，以及成本效益高的点对点卷积进行通道混合，如图3.(b)所示。它可以作为高效的基本构建块，例如嵌入在ELAN结构中（图3.(b)）。然后，我们提倡一种基于秩的块分配策略，以实现最佳效率，同时保持有竞争力的容量。具体来说，给定一个模型，我们根据其内在秩按升序对所有阶段进行排序。我们进一步检查用CIB替换领先阶段的基本块的性能变化。如果与给定模型相比没有性能下降，我们就继续替换下一个阶段，否则就停止该过程。因此，我们可以在不同阶段和模型规模上实现自适应的紧凑块设计，实现更高的效率而不损害性能。

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/0af9b20597598897c15c90a02d5c1093-1733232140019-3.png)

### 准确性驱动的模型设计。

我们进一步探索大核卷积和自注意力在准确性驱动设计中的应用，旨在以最小的成本提升性能。

（1）大核卷积。采用大核深度卷积是一种有效的方法来扩大感受野并增强模型的能力[10, 40, 39]。然而，简单地在所有阶段都利用它们可能会引入浅层特征中的污染，同时也会在高分辨率阶段引入显著的I/O开销和延迟[8]。因此，我们建议在深度阶段利用CIB中的大核深度卷积。具体来说，我们增加了CIB中第二个3x3深度卷积的核大小为7x7，随后[39]。此外，我们采用结构重参数化技术[11,10,59]来引入另一个3x3深度卷积分支，以缓解优化问题而不增加推理开销。此外，随着模型规模的增加，其感受野自然扩大，使用大核卷积的好处逐渐减弱。因此，我们只在小型模型规模上采用大核卷积。

(2) 部分自注意力(PSA)。自注意力[58]由于其显著的全球建模能力[38, 14, 76]而被广泛应用于各种视觉任务。然而，它表现出高计算复杂性和内存占用。为了解决这个问题，鉴于普遍存在的注意力头重用[69]，我们提出了一个高效的局部自注意力(PSA)模块设计，如图3.(c)所示。具体来说，在1x1卷积之后，我们将通道中的特征均匀划分为两部分。我们只将一部分输入到由多头自注意力模块(MHSA)和前馈网络(FFN)组成的NPSA块中。然后将两部分通过1x1卷积连接并融合。此外，我们遵循[22]的方法，将查询和键的维度分配给MHSA中的值的一半，并用BatchNorm[27]替换LayerNorm[1]以进行快速推理。此外，PSA仅在分辨率最低的第4阶段之后放置，避免了过多的开销。

# 实验

### 实现细节

我们选择YOLOv8[21]作为我们的基线模型，因为它在延迟准确性和各种模型规模的可用性方面表现良好。我们采用了一致的NMS-free训练双重分配，并基于此进行了全体的效率-准确性驱动的模型设计，从而带来了我们的YOLOv10模型。YOLOv10具有与YOLOv8相同的变体，即N/S/M/L/X。此外，我们通过简单增加YOLOv10-M的宽度尺度因子，推导出了一个新的变体YOLOv10-B。我们在相同的全局从零开始设置[21, 65, 62]下，在COCO[35]上验证了所提出的检测器。此外，所有模型的延迟都在T4 GPU和TensorRT FP16上进行测试，遵循[78]的方法。

### 与最先进技术的比较

如表1所示，我们的YOLOv10在各种模型规模上实现了最先进的性能和端到端的延迟。我们首先将YOLOv10与我们基线模型，即YOLOv8进行比较。在N/S/M/L/X五种变体中，我们的YOLOv10实现了1.2%/1.4%/0.5%/0.3%/0.5%的AP改进，参数减少了28%/ 36%/ 41%/ 44%/ 57%，计算减少了23%/ 24%/ 25%/ 27%/ 38%，延迟减少了70%/65%/50%/41%/37%。与其他YOLO相比，

![image-20241203212603338](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241203212603338.png)

YOLOv10在准确性和计算成本之间也展现了卓越的权衡。具体来说，对于轻量级和小型的模型，YOLOv10-N/S的性能超过了YOLOv6-3.0-N/S，分别提高了1.5 AP和2.0 AP，参数减少了51%，计算量减少了41%。对于中等规模的模型，与YOLOv9-C/ YOLO-MS相比，YOLOv10-B/M在相同或更好的性能下，延迟降低了46%/62%。对于大型模型，与Gold-YOLO-L相比，我们的YOLOv10-L在参数减少了68%，延迟降低了32%，并且AP显著提高了1.4%。此外，与RT-DETR相比，YOLOv10获得了显著的性能和延迟提升。值得注意的是，在相似的性能下，YOLOv10-S/X的推理速度比RT-DETR-R18/R101快了1.8倍和1.3倍。这些结果充分展示了YOLOv10作为实时端到端检测器的优越性。

我们还使用原始的一对多训练方法将YOLOv10与其他YOLO进行了比较。在这种情况下，我们考虑了模型前向过程（Latencyf）的性能和延迟[62, 21, 60]。如表1所示，YOLOv10在不同模型规模上也展现了最先进的表现和效率，这表明了我们架构设计的有效性。

###  模型分析

![image-20241203212826090](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241203212826090.png)

![image-20241203213021438](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241203213021438.png)

![image-20241203213123173](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241203213123173.png)

![image-20241203213200887](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241203213200887.png)

1. **消融研究**：基于YOLOv10-S和YOLOv10-M的消融研究表明，无NMS训练结合一致的双标签分配显著降低了YOLOv10-S的端到端延迟，同时保持了44.3%的AP竞争力。此外，效率驱动的设计减少了参数数量和计算量，并显著降低了延迟。

2. **双标签分配**：双标签分配为无NMS的YOLO提供了丰富的监督信息，并在推理时实现了高效性。一致匹配度量的引入进一步缩小了两个分支之间的监督差距，提高了性能。

3. **效率驱动模型设计**：效率驱动模型设计通过轻量级分类头、空间-通道解耦下采样和紧凑倒置块（CIB）等组件，有效减少了参数数量、FLOPs和延迟，同时保持了竞争性的性能。

4. **准确性驱动模型设计**：准确性驱动模型设计通过大核卷积和部分自注意力（PSA）模块，在不显著增加延迟的情况下提高了性能。

5. **大核卷积**：大核卷积的使用扩大了感受野并增强了模型能力，但在小模型中效果更佳。

6. **部分自注意力模块**：PSA模块通过减少自注意力头中的冗余来缓解优化问题，从而在不牺牲高效率的情况下提升了模型性能。

# YOLOv10代码

### C2fUIB介绍

**C2fUIB只是用CIB结构替换了YOLOv8中 C2f的Bottleneck结构**

**实现代码ultralytics/nn/modules/block.py**

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/8ed70c479c7530fe3d36f4f44fbbf2d8.png)

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/a3198a08d3b755ec2e1e85cb8979c2ee.png)

```python
class CIB(nn.Module):
    """Standard bottleneck."""

    def __init__(self, c1, c2, shortcut=True, e=0.5, lk=False):
        """Initializes a bottleneck module with given input/output channels, shortcut option, group, kernels, and
        expansion.
        """
        super().__init__()
        c_ = int(c2 * e)  # hidden channels
        self.cv1 = nn.Sequential(
            Conv(c1, c1, 3, g=c1),
            Conv(c1, 2 * c_, 1),
            Conv(2 * c_, 2 * c_, 3, g=2 * c_) if not lk else RepVGGDW(2 * c_),
            Conv(2 * c_, c2, 1),
            Conv(c2, c2, 3, g=c2),
        )

        self.add = shortcut and c1 == c2

    def forward(self, x):
        """'forward()' applies the YOLO FPN to input data."""
        return x + self.cv1(x) if self.add else self.cv1(x)

class C2fCIB(C2f):
    """Faster Implementation of CSP Bottleneck with 2 convolutions."""

    def __init__(self, c1, c2, n=1, shortcut=False, lk=False, g=1, e=0.5):
        """Initialize CSP bottleneck layer with two convolutions with arguments ch_in, ch_out, number, shortcut, groups,
        expansion.
        """
        super().__init__(c1, c2, n, shortcut, g, e)
        self.m = nn.ModuleList(CIB(self.c, self.c, shortcut, e=1.0, lk=lk) for _ in range(n))
```

### PSA介绍

具体来说，我们在1×1卷积后将特征均匀地分为两部分。我们只将一部分输入到由多头自注意力模块（MHSA）和前馈网络（FFN）组成的NPSA块中。然后，两部分通过1×1卷积连接并融合。此外，遵循将查询和键的维度分配为值的一半，并用BatchNorm替换LayerNorm以实现快速推理。

**实现代码ultralytics/nn/modules/block.py**

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/82d0ed0834ac712354683efcb0108bc6.png)

```python
class Attention(nn.Module):
    def __init__(self, dim, num_heads=8,
                 attn_ratio=0.5):
        super().__init__()
        self.num_heads = num_heads
        self.head_dim = dim // num_heads
        self.key_dim = int(self.head_dim * attn_ratio)
        self.scale = self.key_dim ** -0.5
        nh_kd = nh_kd = self.key_dim * num_heads
        h = dim + nh_kd * 2
        self.qkv = Conv(dim, h, 1, act=False)
        self.proj = Conv(dim, dim, 1, act=False)
        self.pe = Conv(dim, dim, 3, 1, g=dim, act=False)

    def forward(self, x):
        B, _, H, W = x.shape
        N = H * W
        qkv = self.qkv(x)
        q, k, v = qkv.view(B, self.num_heads, -1, N).split([self.key_dim, self.key_dim, self.head_dim], dim=2)

        attn = (
            (q.transpose(-2, -1) @ k) * self.scale
        )
        attn = attn.softmax(dim=-1)
        x = (v @ attn.transpose(-2, -1)).view(B, -1, H, W) + self.pe(v.reshape(B, -1, H, W))
        x = self.proj(x)
        return x

class PSA(nn.Module):

    def __init__(self, c1, c2, e=0.5):
        super().__init__()
        assert(c1 == c2)
        self.c = int(c1 * e)
        self.cv1 = Conv(c1, 2 * self.c, 1, 1)
        self.cv2 = Conv(2 * self.c, c1, 1)
        
        self.attn = Attention(self.c, attn_ratio=0.5, num_heads=self.c // 64)
        self.ffn = nn.Sequential(
            Conv(self.c, self.c*2, 1),
            Conv(self.c*2, self.c, 1, act=False)
        )
        
    def forward(self, x):
        a, b = self.cv1(x).split((self.c, self.c), dim=1)
        b = b + self.attn(b)
        b = b + self.ffn(b)
        return self.cv2(torch.cat((a, b), 1))
```

### SCDown

OLOs通常利用常规的3×3标准卷积，步长为2，同时实现空间下采样（从H×W到H/2×W/2）和通道变换（从C到2C）。这引入了不可忽视的计算成本$O(9HWC^2)$和参数数量O$(18C^2)$。相反，我们提议将空间缩减和通道增加操作解耦，以实现更高效的下采样。具体来说，我们首先利用点对点卷积来调整通道维度，然后利用深度可分离卷积进行空间下采样。这将计算成本降低到O(2HWC^2 + 9HWC)，并将参数数量减少到O(2C^2 + 18C)。同时，它最大限度地保留了下采样过程中的信息，从而在减少延迟的同时保持了有竞争力的性能。

**实现代码ultralytics/nn/modules/block.py**

```
class SCDown(nn.Module):
    def __init__(self, c1, c2, k, s):
        super().__init__()
        self.cv1 = Conv(c1, c2, 1, 1)
        self.cv2 = Conv(c2, c2, k=k, s=s, g=c2, act=False)

    def forward(self, x):
        return self.cv2(self.cv1(x))
```



参考：[YOLOv10真正实时端到端目标检测（原理介绍+代码详见+结构框图）-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2426044)

文章合集：https://github.com/chongzicbo/ReadWriteThink

