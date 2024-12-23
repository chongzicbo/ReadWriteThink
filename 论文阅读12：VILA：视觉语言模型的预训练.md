# 摘要

视觉语言模型（VLMs）随着大型语言模型的成功迅速发展。在视觉指导调整方面，人们已经做出了越来越多的努力来扩展LLMs以获取视觉输入，但缺乏对视觉语言预训练过程的深入研究，在该过程中，模型学会了在两种模态上进行联合建模。在这项工作中，我们通过逐步可控的比较来检查VLM预训练的设计选项。我们介绍了三个主要发现：(1) 在预训练期间冻结LLMs可以实现不错的零样本性能，但缺乏上下文学习能力，这需要解冻LLM；(2) 并行预训练数据是有益的，而仅使用图像-文本对则不是最优的；(3) 在指令微调期间，将纯文本指令数据重新混合为图像-文本数据不仅可以解决纯文本任务的退化，还可以提高VLM任务的准确性。通过改进的预训练配方，我们构建了VILA，这是一个视觉语言模型家族，它在主要基准测试中一致性地优于现有最先进模型，例如LLLaVA-1.5，无需任何附加功能。多模态预训练也有助于揭示VILA的吸引人的属性，包括多图像推理、增强上下文学习和更好的世界知识。VILA还可以在Jetson Orin上进行部署，用于设备上的VLM。

#  引言

大型语言模型（LLMs）已经展示了在自然语言任务中的卓越能力[4,8,10,15,16,19,31,46,51,59-61]。通过增强LLMs以支持视觉输入，可以让最终模型继承一些吸引人的属性，如指令遵循、零样本泛化和小样本上下文学习（ICL），从而增强各种视觉语言任务的能力[1,2,6,9,14,20,35,39,73]。统一视觉和语言来用于协作推理的中心挑战在于连接LLM和视觉基础模型（例如，CLIP编码器）：这两个基础模型通常分别进行预训练，然后通过视觉语言联合训练进行对齐。该领域的大部分努力都集中在改进视觉语言指令微调过程，即监督微调（SFT）或从人类反馈中学习强化学习（RLHF）[38, 39, 57]。然而，缺乏对预训练过程的深入研究，其中模型是在大规模的图像-文本数据集/语料库上进行训练的[11,54,74]。这个过程成本高昂，但对模态对齐至关重要

在这项工作中，我们旨在探索增强视觉语言模型预训练的不同设计选项。特别是，我们旨在回答“视觉语言模型预训练中的各种设计选项如何影响下游性能？”我们遵循了预训练+ SFT流程，并消除了不同设计选项对预训练的过度关注数据集属性和训练协议。我们发有几个发现：

![image-20241204144844318](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241204144844318-1733923519445-1-1733923522511-3.png)

（1）在预训练期间冻结LLM训练可以取得不错的零样本性能，但不具备上下文学习（ICL）能力，而更新LLMs则能鼓励深度嵌入对齐，我们认为这对ICL很重要；

（2）交错视觉语言数据对于预训练至关重要，它提供了准确的梯度更新并保持了纯文本的能力；

（3）在SFT期间添加仅文本指令数据可以进一步改善纯文本的退化并提高视觉语言任务准确性。

![image-20241204163514960](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20241204163514960.png)

我们介绍了设计视觉语言模型的实用指导，称为VILA。VILA在广泛的视觉语言任务中（见图1），由于改进的预训练的帮助，明显优于最先进的模型[38]。此外，我们观察到预训练过程解锁了模型的几个有趣能力，例如：（i）多图像推理（尽管在SFT期间模型只看到单个图像-文本对）；（ii）更强的上下文学习能力；以及（iii）增强的世界知识。我们希望我们的发现能为未来的视觉语言模型提供一个良好的预训练配方。

# 背景

模型架构。多模态大型语言模型（LLMs）通常可以分为两种设置：基于跨注意力的[6, 35]和基于自回归的[2, 20, 39]。后者VLM家族将图像标记成视觉标记，这些标记与文本标记相关联，并作为输入提供给LLMs（即将视觉输入视为外语）。这是通过用视觉嵌入增强输入的自然扩展，可以处理任意交织的图像-文本输入。在这项研究中，我们专注于自回归VLM的预训练，因为它具有灵活性和流行性。如图2所示，自回归VLM由三个组件组成：一个视觉编码器、一个LLM和一个投影器（projector），该投影器连接了两种模态的嵌入。投影器可以是简单的线性层[39]或更强大的Transformer块[7, 18]——我们将在我们的实验中比较它们的有效性。该模型接收视觉和文本输入并生成文本输出。

训练阶段。遵循常见做法[7,20,39]，我们研究如何使用视觉输入支持来增强预训练的纯文本LLM。训练可以分为三个阶段：

1. 投影器初始化。LLM和ViT分别进行预训练，而投影器通常从随机权重初始化。因此，我们首先在冻结ViT和LLMs的同时，对投影器进行预训练，以保持图像-标题对，这符合现有文献[18, 35, 39]。
2. 视觉语言预训练。然后我们在视觉语言语料库上预训练模型（LLM和投影器）。我们考虑两种类型的语料库：交错图像-文本语料库（例如。我们专注于预训练过程的研究，这是视觉语言对齐中最昂贵和最重要的部分。
3. 视觉指令微调。最后，我们在视觉语言指导数据集上进一步对预训练模型进行指令微调。我们将现有的视觉语言数据集转换为FLAN[64]风格（即，使用数据集特定的提示），遵循[18]。请在补充材料中找到视觉指令数据的混合数据。

评估。在我们的消融研究中，我们在4个视觉语言任务上评估了微调后的模型：OKVQA[45]和TextVQA[55]的准确性，以及CIDEr分数对于COCO[37]和Flickr[67]。我们评估了0次和4次性能，这反映了模型的上下文学习能力。

# 研究方法

这篇论文提出了一种新的视觉语言模型预训练方法，用于解决视觉语言任务中的对齐问题。具体来说，

1. **冻结LLM的预训练**：首先，研究了在预训练过程中冻结LLM的效果。通过比较冻结和不冻结LLM的情况，发现冻结LLM可以实现不错的零样本性能，但缺乏上下文学习能力。更新LLM则有助于深度嵌入对齐，从而提高上下文学习能力。

   ![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/0b342b9ad00b53007fcc40ae305f2427-image.png)

   

2. **交错预训练数据**：其次，研究了交错预训练数据的效果。与单独使用图像-文本对相比，交错预训练数据提供了更准确的梯度更新，并且能够保持纯文本能力。

   ![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/47eb7023774716556be91ef808c951b2-image.png)

   

3. **联合微调**：最后，研究了在微调过程中重新混合纯文本指令数据的效果。通过在微调过程中加入纯文本指令数据，不仅能够恢复纯文本任务的性能下降，还能提高视觉语言任务的准确性。

# 实验设计

1. **数据集选择**：使用了MMC4和COYO两种图像-文本数据集进行预训练。MMC4是一个交错的图像-文本数据集，而COYO是一个图像-文本对数据集。为了增加多样性，还将MMC4和COYO数据集进行了混合。
2. **预训练阶段**：首先对投影仪进行预训练，同时冻结LLM和ViT。然后，在视觉语言数据集上进行预训练，考虑了交错图像-文本数据集和图像-文本对数据集。
3. **微调阶段**：在预训练模型的基础上，使用视觉语言指令数据集进行微调。将现有的视觉语言数据集转换为FLAN风格的数据，并加入了1M的纯文本指令微调数据。
4. **评估指标**：在四个视觉语言任务上评估微调后的模型性能，包括OKVQA和TextVQA的准确率，以及COCO和Flickr的CIDEr得分。评估了0-shot和4-shot性能，以反映模型的上下文学习能力。

# 结果与分析

1. **冻结LLM的影响**：冻结LLM在预训练过程中不会影响0-shot性能，但会显著降低上下文学习能力（4-shot性能下降）。更新LLM则有助于提高深度嵌入对齐，从而提高上下文学习能力。

2. **交错预训练数据的效果**：使用交错图像-文本数据集进行预训练比单独使用图像-文本对数据集更有效。交错数据集提供了更准确的梯度更新，并且能够保持纯文本能力。

3. 联合微调的效果：在微调过程中加入纯文本指令数据不仅能够恢复纯文本任务的性能下降，还能提高视觉语言任务的准确性。联合微调使得模型在预训练时使用短文本时不会丧失纯文本能力，从而解锁了更好的视觉多样性。

   ![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/aef7863993e6fa2d03574be64d0606f0-image.png)

# 总体结论

这篇论文通过探索不同的预训练设计选项，提出了一种新的视觉语言模型预训练方法。该方法通过更新LLM、使用交错预训练数据和联合微调，显著提高了视觉语言模型在多个基准测试中的性能。研究表明，多模态预训练不仅提高了视觉语言任务的准确性，还揭示了VILA模型的多图像推理、增强的上下文学习和更好的世界知识等有趣特性。希望这些发现能够为未来视觉语言模型的研究提供有价值的指导。

# 论文评价

## 优点与创新

1. **零样本性能**：在预训练过程中冻结LLMs可以实现不错的零样本性能，但缺乏上下文学习能力，需要解冻LLMs。
2. **交错预训练数据**：交错预训练数据比单独的图像-文本对更有效。
3. **文本指令数据的重新混合**：在指令微调期间重新混合纯文本指令数据不仅可以弥补纯文本任务的退化，还可以提高VLM任务的准确性。
4. **多模态预训练的优势**：通过增强的预训练方法，VILA模型在主要基准测试中一致优于最先进的模型（如LLaVA-1.5），并且展现出多图像推理、增强的上下文学习和更好的世界知识等吸引人的特性。
5. **部署灵活性**：VILA模型可以在Jetson Orin上进行设备端的VLM部署。

## 不足与反思

1. **计算预算限制**：由于计算预算有限，未能将预训练语料库扩展到十亿规模，这留作未来的工作。
2. **高分辨率图像的影响**：尽管使用高分辨率图像（336x336）可以显著提高TextVQA任务的准确性，但也会增加计算成本，并限制了上下文学习的演示数量。未来工作可以通过在预训练阶段使用较低分辨率的图像，并在后期训练阶段提高分辨率来减少训练时间。
3. **数据压缩方法的探索**：尽管观察到使用“下采样”投影器可以减少图像令牌的数量，但为了获得最佳准确性，论文中没有使用任何令牌压缩方法，这留作未来的工作。



