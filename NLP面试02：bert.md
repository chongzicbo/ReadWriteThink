# 问题1：bert的具体网络结构，以及训练过程，bert为什么火，它在什么的基础上改进了些什么？

bert是用了transformer的encoder侧的网络，作为一个文本编码器，使用大规模数据进行预训练，预训练使用两个loss，一个是mask LM，遮蔽掉源端的一些字（可能会被问到mask的具体做法，15%概率mask词，这其中80%用[mask]替换，10%随机替换一个其他字，10%不替换，至于为什么这么做，那就得问问BERT的作者了\{捂脸}），然后根据上下文去预测这些字，一个是next sentence，判断两个句子是否在文章中互为上下句，然后使用了大规模的语料去预训练。在它之前是GPT，GPT是一个单向语言模型的预训练过程（它和gpt的区别就是bert为啥叫双向 bi-directional），更适用于文本生成，通过前文去预测当前的字。下图为transformer的结构，bert的网络结构则用了左边的encoder。

![img](https://pic3.zhimg.com/v2-47a5a577603efb220d80167c4c291806_r.jpg)

# 问题2，讲讲multi-head attention的具体结构

BERT由12层transformer layer（encoder端）构成，首先word emb , pos emb（可能会被问到有哪几种position embedding的方式，bert是使用的哪种）, sent emb做加和作为网络输入，每层由一个multi-head attention, 一个feed forward 以及两层layerNorm构成，一般会被问到multi-head attention的结构，具体可以描述为，

step1

一个768的hidden向量，被映射成query， key， value。 然后三个向量分别切分成12个小的64维的向量，每一组小向量之间做attention。不妨假设batch_size为32，seqlen为512，隐层维度为768，12个head

hidden(32 x 512 x 768) -> query(32 x 512 x 768) -> 32 x 12 x 512 x 64

hidden(32 x 512 x 768) -> key(32 x 512 x 768) -> 32 x 12 x 512 x 64

hidden(32 x 512 x 768) -> val(32 x 512 x 768) -> 32 x 12 x 512 x 64

step2

然后query和key之间做attention，得到一个32 x 12 x 512 x 512的权重矩阵，然后根据这个权重矩阵加权value中切分好的向量，得到一个32 x 12 x 512 x 64 的向量，拉平输出为768向量。

32 x 12 x 512 x 64(query_hidden) * 32 x 12 x 64 x 512(key_hidden) -> 32 x 12 x 512 x 512

32 x 12 x 64 x 512(value_hidden) * 32 x 12 x 512 x 512 (权重矩阵) -> 32 x 12 x 512 x 64

然后再还原成 -> 32 x 512 x 768

简言之是12个头，每个头都是一个64维度分别去与其他的所有位置的hidden embedding做attention然后再合并还原。





# 问题2.5: Bert 采用哪种Normalization结构，LayerNorm和BatchNorm区别，LayerNorm结构有参数吗，参数的作用？

采用LayerNorm结构，和BatchNorm的区别主要是做规范化的维度不同，BatchNorm针对一个batch里面的数据进行规范化，针对单个神经元进行，比如batch里面有64个样本，那么规范化输入的这64个样本各自经过这个神经元后的值（64维），LayerNorm则是针对单个样本，不依赖于其他数据，常被用于小mini-batch场景、动态网络场景和 RNN，特别是自然语言处理领域，就bert来说就是对每层输出的隐层向量（768维）做规范化，图像领域用BN比较多的原因是因为每一个卷积核的参数在不同位置的神经元当中是共享的，因此也应该被一起规范化。

```python
class BertLayerNorm(nn.Module):
    def __init__(self, hidden_size, eps=1e-5):
        super(BertLayerNorm, self).__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.bias = nn.Parameter(torch.zeros(hidden_size))
        self.variance_epsilon = eps

    def forward(self, x):
        u = x.mean(-1, keepdim=True)
        s = (x - u).pow(2).mean(-1, keepdim=True)
        x = (x - u) / torch.sqrt(s + self.variance_epsilon)
        return self.weight * x + self.bias
```

帖一个LayerNorm的实现，可以看到module中有weight和bias参数，以Sigmoid激活函数为例，批量归一化之后数据整体处于函数的非饱和区域， 只包含线性变换，破坏了之前学习到的特征分布。为了恢复原始数据分布，具体实现中引入了变换重构以及可学习参数w和b ，也就是上面的weight和bias，简而言之，规范化后的隐层表示将输入数据限制到了一个全局统一的确定范围，为了保证模型的表达能力不因为规范化而下降，引入了b是再平移参数，w是再缩放参数。（过激活函数前规范化，之后还原）



# 问题3：transformer attention的时候为啥要除以根号D

至于attention后的权重为啥要除以$\sqrt {d_k}$  ，作者在论文中的解释是点积后的结果大小是跟维度成正比的，所以经过softmax以后，梯度就会变很小，除以 $\sqrt {d_k}$ 后可以让attention的权重分布方差为1，而不是$ d_k$  。

具体细节（简而言之就是softmax如果某个输入太大的话就会使得权重太接近于1，梯度很小）

[transformer中的attention为什么scaled](https://www.zhihu.com/question/339723385/answer/782509914)



# 问题4：wordpiece的作用

wordpiece其核心思想是将单词打散为字符，然后根据片段的组合频率，最后单词切分成片段处理。和原有的分词相比，能够极大的降低OOV的情况，例如cosplayer, 使用分词的话如果出现频率较低则是UNK，但bpe可以把它切分吃cos play er, 模型可以词根以及前缀等信息，学习到这个词的大致信息，而不是一个OOV。

[](https://zhuanlan.zhihu.com/p/198964217)

wordpiece与BPE(Byte Pair Encoding)算法类似，也是每次从词表中选出两个子词合并成新的子词。与BPE的最大区别在于，如何选择两个子词进行合并：BPE选择频数最高的相邻子词合并，而WordPiece选择能够提升语言模型概率最大的相邻子词加入词表。

# 问题5： 如何优化BERT效果：

1 感觉最有效的方式还是数据。

2 把现有的大模型ERNIE_2.0_large, Roberta，roberta_wwm_ext_large、roberta-pair-large等进行ensemble，然后蒸馏原始的bert模型，这是能有效提高的，只是操作代价比较大。

3 BERT上面加一些网络结构，比如attention，rcnn等，个人得到的结果感觉和直接在上面加一层transformer layer的效果差不多，模型更加复杂，效果略好，计算时间略增加。

4 改进预训练，在特定的大规模数据上预训练，相比于开源的用百科，知道等数据训练的更适合你的任务（经过多方验证是一种比较有效的提升方案）。以及在预训练的时候去mask低频词或者实体词（听说过有人这么做有收益，但没具体验证）。

5 文本对抗，作者了解的不多。感兴趣可以看看⬇️



# 问题6 如何优化BERT性能

1 压缩层数，然后蒸馏，直接复用12层bert的前4层或者前6层，效果能和12层基本持平，如果不蒸馏会差一些。

2 双塔模型（短文本匹配任务），将bert作为一个encoder，输入query编码成向量，输入title编码成向量，最后加一个DNN网络计算打分即可。离线缓存编码后的向量，在线计算只需要计算DNN网络。

3 int8预估，在保证模型精度的前提下，将Float32的模型转换成Int8的模型。

4 提前结束，大致思想是简单的case前面几层就可以输出分类结果，比较难区分的case走完12层，但这个在batch里面计算应该怎么优化还没看明白，有的提前结束有的最后结束，如果在一个batch里面的话就不太好弄。感兴趣的可以看看⬇️。

5 ALBERT 做了一些改进优化，主要是不同层之间共享参数，以及用矩阵分解降低embedding的参数，可以看看⬇️。

# 问题7 self-attention相比lstm优点是什么？

bert通过使用self-attention + position embedding对序列进行编码，lstm的计算过程是从左到右从上到下（如果是多层lstm的话），后一个时间节点的emb需要等前面的算完，而bert这种方式相当于并行计算，虽然模型复杂了很多，速度其实差不多。

# 问题8 BERT的一些改进。

* ERNIE_1.0(baidu) ： 模型结构不变，预训练的时候第一阶段基于切词mask，第二阶段基于实体mask，让模型在预训练的过程中根据上下文去学到一些词级别，实体级别，短语级别的信息。

![image-20201212113938997](https://gitee.com/chengbo123/images/raw/master/image-20201212113938997.png)

* ERNIE_2.0：模型结构加入task embedding（作者将预训练任务分为：Word-aware， Structure-aware，Semantic-aware），不同类型的任务选取不同的task embedding，然后加了很多预训练任务，很多数据。

![img](https://pic3.zhimg.com/v2-bad961513fedd5f07e4da48224cb7a46_r.jpg)

* ALBERT
  Factorized Embedding Parameterization，矩阵分解降低embedding参数量。
  Cross-layer Parameter Sharing，不同层之间共享参数。
  Sentence Order Prediction（SOP），next sentence任务的负样本增强。
  个人感觉降低参数量相对于优化效果以及计算速度来说，并不是特别重要。

# 1、不考虑多头的原因，self-attention中词向量不乘QKV参数矩阵，会有什么问题？

Self-Attention的核心是用文本中的其它词来增强目标词的语义表示，从而更好的利用上下文的信息。
self-attention中，sequence中的每个词都会和sequence中的每个词做点积去计算相似度，也包括这个词本身。
如果不乘QKV参数矩阵，那这个词对应的q,k,v就是完全一样的。
在相同量级的情况下，qi与ki点积的值会是最大的（可以从“两数和相同的情况下，两数相等对应的积最大”类比过来）。
那在softmax后的加权平均中，该词本身所占的比重将会是最大的，使得其他词的比重很少，无法有效利用上下文信息来增强当前词的语义表示。
而乘以QKV参数矩阵，会使得每个词的q,k,v都不一样，能很大程度上减轻上述的影响。
当然，QKV参数矩阵也使得多头，类似于CNN中的多核，去捕捉更丰富的特征/信息成为可能。

# 2、为什么BERT选择mask掉15%这个比例的词，可以是其他的比例吗？

BERT采用的Masked LM，会选取语料中所有词的15%进行随机mask，论文中表示是受到完形填空任务的启发，但其实与CBOW也有异曲同工之妙。
从CBOW的角度，这里 p=15% 有一个比较好的解释是：在一个大小为1/p=100/15==7  的窗口中随机选一个词，类似CBOW中滑动窗口的中心词，区别是这里的滑动窗口是非重叠的。
那从CBOW的滑动窗口角度，10%~20%都是还ok的比例。
上述非官方解释，是来自我的一位朋友提供的一个理解切入的角度，供参考。

# 3、使用BERT预训练模型为什么最多只能输入512个词，最多只能两个句子合成一句？

这是Google BERT预训练模型初始设置的原因，前者对应Position Embeddings，后者对应Segment Embeddings

![img](https://pic3.zhimg.com/v2-ef0fa015ddfbae62bf2f0fc85a8fa3f2_r.jpg)

在BERT中，Token，Position，Segment Embeddings 都是通过学习来得到的，pytorch代码中它们是这样的

```python
self.word_embeddings = Embedding(config.vocab_size, config.hidden_size) self.position_embeddings = Embedding(config.max_position_embeddings, config.hidden_size) self.token_type_embeddings = Embedding(config.type_vocab_size, config.hidden_size)
```

上述BERT pytorch代码来自:https://github.com/xieyufei1993/Bert-Pytorch-Chinese-TextClassification，结构层次非常清晰。
而在BERT config中
"max_position_embeddings": 512 "type_vocab_size": 2
因此，在直接使用Google 的BERT预训练模型时，输入最多512个词（还要除掉[CLS]和[SEP]），最多两个句子合成一句。这之外的词和句子会没有对应的embedding。
当然，如果有足够的硬件资源自己重新训练BERT，可以更改 BERT config，设置更大max_position_embeddings 和 type_vocab_size值去满足自己的需求。

# 4、为什么BERT在第一句前会加一个[CLS]标志?

BERT在第一句前会加一个[CLS]标志，最后一层该位对应向量可以作为整句话的语义表示，从而用于下游的分类任务等。
为什么选它呢，因为与文本中已有的其它词相比，这个无明显语义信息的符号会更“公平”地融合文本中各个词的语义信息，从而更好的表示整句话的语义。
这里补充一下bert的输出，有两种：
一种是get_pooled_out()，就是上述[CLS]的表示，输出shape是[batch size,hidden size]。
一种是get_sequence_out()，获取的是整个句子每一个token的向量表示，输出shape是[batch_size, seq_length, hidden_size]，这里也包括[CLS]，因此在做token级别的任务时要注意它。

# 5、Self-Attention 的时间复杂度是怎么计算的？

Self-Attention时间复杂度： $O(n^2 \cdot d)$ ，这里，n是序列的长度，d是embedding的维度。
Self-Attention包括三个步骤：相似度计算，softmax和加权平均，它们分别的时间复杂度是：
相似度计算可以看作大小为(n,d)和(d,n)的两个矩阵相乘：$(n,d)* (d,n)=O(n^2 \cdot d)$  ，得到一个(n,n)的矩阵
softmax就是直接计算了，时间复杂度为 $O(n^2)$
加权平均可以看作大小为(n,n)和(n,d)的两个矩阵相乘：$(n,n)*(n,d)=O(n^2 \cdot 2)$  ，得到一个(n,d)的矩阵
因此，Self-Attention的时间复杂度是$O(n^2 \cdot d)$  。
这里再分析一下Multi-Head Attention，它的作用类似于CNN中的多核。
多头的实现不是循环的计算每个头，而是通过 transposes and reshapes，用矩阵乘法来完成的。

```
In practice, the multi-headed attention are done with transposes and reshapes rather than actual separate tensors. —— 来自 google BERT 源码
```

Transformer/BERT中把 d ，也就是hidden_size/embedding_size这个维度做了reshape拆分，可以去看Google的TF源码或者上面的pytorch源码：
hidden_size (d) = num_attention_heads (m) * attention_head_size (a)，也即 d=m*a
并将 num_attention_heads 维度transpose到前面，使得Q和K的维度都是(m,n,a)，这里不考虑batch维度。
这样点积可以看作大小为(m,n,a)和(m,a,n)的两个张量相乘，得到一个(m,n,n)的矩阵，其实就相当于m个头，时间复杂度是 $O(n^2 \cdot m^2 \cdot a)=O(n^2 \cdot d \cdot m)$ 。
张量乘法时间复杂度分析参见：[矩阵、张量乘法的时间复杂度分析](https://liwt31.github.io/2018/10/12/mul-complexity/)
因此Multi-Head Attention时间复杂度就是$O(n^2 \cdot d \cdot m)$  ，而实际上，张量乘法可以加速，因此实际复杂度会更低一些。
不过，对于做 transposes and reshapes 的逻辑，个人没有理的很明白，希望大佬看到能留言解答一下，感谢。

Transformer/BERT中把 d ，也就是hidden_size/embedding_size这个维度做了reshape拆分，可以去看Google的TF源码或者上面的pytorch源码：
hidden_size (d) = num_attention_heads (m) * attention_head_size (a)，也即 d=m*a
并将 num_attention_heads 维度transpose到前面，使得Q和K的维度都是(m,n,a)，这里不考虑batch维度。
这样点积可以看作大小为(m,n,a)和(m,a,n)的两个张量相乘，得到一个(m,n,n)的矩阵，其实就相当于m个头，时间复杂度是 $O(n^2 \cdot m^2 \cdot a)=O(n^2 \cdot d \cdot m)$ 。
张量乘法时间复杂度分析参见：[矩阵、张量乘法的时间复杂度分析](https://liwt31.github.io/2018/10/12/mul-complexity/)
因此Multi-Head Attention时间复杂度就是$O(n^2 \cdot d \cdot m)$  ，而实际上，张量乘法可以加速，因此实际复杂度会更低一些。
不过，对于做 transposes and reshapes 的逻辑，个人没有理的很明白，希望大佬看到能留言解答一下，感谢。

# 6、Transformer在哪里做了权重共享，为什么可以做权重共享？

Transformer在两个地方进行了权重共享：
（1）Encoder和Decoder间的Embedding层权重共享；
（2）Decoder中Embedding层和FC层权重共享。
对于（1），《Attention is all you need》中Transformer被应用在机器翻译任务中，源语言和目标语言是不一样的，但它们可以共用一张大词表，对于两种语言中共同出现的词（比如：数字，标点等等）可以得到更好的表示，而且对于Encoder和Decoder，嵌入时都只有对应语言的embedding会被激活，因此是可以共用一张词表做权重共享的。
论文中，Transformer词表用了bpe来处理，所以最小的单元是subword。英语和德语同属日耳曼语族，有很多相同的subword，可以共享类似的语义。而像中英这样相差较大的语系，语义共享作用可能不会很大。
但是，共用词表会使得词表数量增大，增加softmax的计算时间，因此实际使用中是否共享可能要根据情况权衡。
该点参考：https://www.zhihu.com/question/333419099/answer/743341017
对于（2），Embedding层可以说是通过onehot去取到对应的embedding向量，FC层可以说是相反的，通过向量（定义为 x）去得到它可能是某个词的softmax概率，取概率最大（贪婪情况下）的作为预测值。
那哪一个会是概率最大的呢？在FC层的每一行量级相同的前提下，理论上和 x 相同的那一行对应的点积和softmax概率会是最大的（可类比本文问题1）。
因此，Embedding层和FC层权重共享，Embedding层中和向量 x 最接近的那一行对应的词，会获得更大的预测概率。实际上，Decoder中的Embedding层和FC层有点像互为逆过程。
通过这样的权重共享可以减少参数的数量，加快收敛。
但开始我有一个困惑是：Embedding层参数维度是：(v,d)，FC层参数维度是：(d,v)，可以直接共享嘛，还是要转置？其中v是词表大小，d是embedding维度。
查看 pytorch 源码发现真的可以直接共享：

```python
fc = nn.Linear(d, v, bias=False) # Decoder FC层定义  weight = Parameter(torch.Tensor(out_features, in_features)) # Linear层权重定义
```

Linear 层的权重定义中，是按照 (out_features, in_features) 顺序来的，实际计算会先将 weight 转置在乘以输入矩阵。所以 FC层 对应的 Linear 权重维度也是 (v,d)，可以直接共享。

# 7、BERT非线性的来源在哪里？

前馈层的gelu激活函数和self-attention，self-attention是非线性的，感谢评论区指出。



# 8、BERT的三个Embedding直接相加会对语义有影响吗？

这是一个非常有意思的问题，苏剑林老师也给出了回答，真的很妙啊：

Embedding的数学本质，就是以one hot为输入的单层全连接。
也就是说，世界上本没什么Embedding，有的只是one hot。
在这里想用一个例子再尝试解释一下：

假设 token Embedding 矩阵维度是 [4,768]；position Embedding 矩阵维度是 [3,768]；segment Embedding 矩阵维度是 [2,768]。

对于一个字，假设它的 token one-hot 是[1,0,0,0]；它的 position one-hot 是[1,0,0]；它的 segment one-hot 是[1,0]。

那这个字最后的 word Embedding，就是上面三种 Embedding 的加和。

如此得到的 word Embedding，和concat后的特征：[1,0,0,0,1,0,0,1,0]，再过维度为 [4+3+2,768] = [9, 768] 的全连接层，得到的向量其实就是一样的。

再换一个角度理解：

直接将三个one-hot 特征 concat 起来得到的 [1,0,0,0,1,0,0,1,0] 不再是one-hot了，但可以把它映射到三个one-hot 组成的特征空间，空间维度是 4*3*2=24 ，那在新的特征空间，这个字的one-hot就是[1,0,0,0,0...] (23个0)。

此时，Embedding 矩阵维度就是 [24,768]，最后得到的 word Embedding 依然是和上面的等效，但是三个小Embedding 矩阵的大小会远小于新特征空间对应的Embedding 矩阵大小。

当然，在相同初始化方法前提下，两种方式得到的 word Embedding 可能方差会有差别，但是，BERT还有Layer Norm，会把 Embedding 结果统一到相同的分布。



BERT的三个Embedding相加，本质可以看作一个特征的融合，强大如 BERT 应该可以学到融合后特征的语义信息的。



参考：https://www.zhihu.com/question/374835153

下面两个问题也非常好，值得重点关注，但网上已经有很好的解答了，如下：



# 9、Transformer的点积模型做缩放的原因是什么？

参考：https://www.zhihu.com/question/339723385



# 10、在BERT应用中，如何解决长文本问题？

参考：https://www.zhihu.com/question/3274



## 1.BERT 的基本原理是什么？

BERT 来自 Google 的论文Pre-training of Deep Bidirectional Transformers for Language Understanding

[1]

，BERT 是“Bidirectional Encoder Representations from Transformers”的首字母缩写，整体是一个自编码语言模型（Autoencoder LM），并且其设计了两个任务来预训练该模型。

- 第一个任务是采用 MaskLM 的方式来训练语言模型，通俗地说就是在输入一句话的时候，随机地选一些要预测的词，然后用一个特殊的符号[MASK]来代替它们，之后让模型根据所给的标签去学习这些地方该填的词。
- 第二个任务在双向语言模型的基础上额外增加了一个句子级别的连续性预测任务，即预测输入 BERT 的两段文本是否为连续的文本，引入这个任务可以更好地让模型学到连续的文本片段之间的关系。

最后的实验表明 BERT 模型的有效性，并在 11 项 NLP 任务中夺得 SOTA 结果。

BERT 相较于原来的 RNN、LSTM 可以做到并发执行，同时提取词在句子中的关系特征，并且能在多个不同层次提取关系特征，进而更全面反映句子语义。相较于 word2vec，其又能根据句子上下文获取词义，从而避免歧义出现。同时缺点也是显而易见的，模型参数太多，而且模型太大，少量数据训练时，容易过拟合。

## 2.BERT 是怎么用 Transformer 的？

BERT 只使用了 Transformer 的 Encoder 模块，原论文中，作者分别用 12 层和 24 层 Transformer Encoder 组装了两套 BERT 模型，分别其中层的数量(即，Transformer Encoder 块的数量)为

，隐藏层的维度为 ，自注意头的个数为 。在所有例子中，我们将前馈/过滤器(Transformer Encoder 端的 feed-forward 层)的维度设置为 ，即当 时是 ；当 是 。

图示如下：

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/1e636c52b97565ebe971eedaec12ab31.jpeg?x-oss-process=image/resize,w_670)

**「需要注意的是，与 Transformer 本身的 Encoder 端相比，BERT 的 Transformer Encoder 端输入的向量表示，多了 Segment Embeddings。」**

## 3.BERT 的训练过程是怎么样的？

在论文原文中，作者提出了两个预训练任务：Masked LM 和 Next Sentence Prediction。

#### 3.1 **「Masked LM」**

Masked LM 的任务描述为：给定一句话，随机抹去这句话中的一个或几个词，要求根据剩余词汇预测被抹去的几个词分别是什么，如下图所示。

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/1631777ed608e26bf04a397161893e27.jpeg?x-oss-process=image/resize,w_670)

BERT 模型的这个预训练过程其实就是在模仿我们学语言的过程，思想来源于 **「完形填空」** 的任务。具体来说，文章作者在一句话中随机选择 15% 的词汇用于预测。对于在原句中被抹去的词汇， 80% 情况下采用一个特殊符号 [MASK] 替换， 10% 情况下采用一个任意词替换，剩余 10% 情况下保持原词汇不变。

这么做的主要原因是：在后续微调任务中语句中并不会出现 [MASK] 标记，而且这么做的另一个好处是：预测一个词汇时，模型并不知道输入对应位置的词汇是否为正确的词汇（ 10% 概率），这就迫使模型更多地依赖于上下文信息去预测词汇，并且赋予了模型一定的纠错能力。上述提到了这样做的一个缺点，其实这样做还有另外一个缺点，就是每批次数据中只有 15% 的标记被预测，这意味着模型可能需要更多的预训练步骤来收敛。

#### 3.2 Next Sentence Prediction

Next Sentence Prediction 的任务描述为：给定一篇文章中的两句话，判断第二句话在文本中是否紧跟在第一句话之后，如下图所示。

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/d01f8035c53860023a474f8fb75a850f.jpeg?x-oss-process=image/resize,w_670)

这个类似于 **「段落重排序」** 的任务，即：将一篇文章的各段打乱，让我们通过重新排序把原文还原出来，这其实需要我们对全文大意有充分、准确的理解。

Next Sentence Prediction 任务实际上就是段落重排序的简化版：只考虑两句话，判断是否是一篇文章中的前后句。在实际预训练过程中，文章作者从文本语料库中随机选择 50% 正确语句对和 50% 错误语句对进行训练，与 Masked LM 任务相结合，让模型能够更准确地刻画语句乃至篇章层面的语义信息。

BERT 模型通过对 Masked LM 任务和 Next Sentence Prediction 任务进行联合训练，使模型输出的每个字 / 词的向量表示都能尽可能全面、准确地刻画输入文本（单句或语句对）的整体信息，为后续的微调任务提供更好的模型参数初始值。

## 4.为什么 BERT 比 ELMo 效果好？ELMo 和 BERT 的区别是什么？

#### 4.1 为什么 BERT 比 ELMo 效果好？

从网络结构以及最后的实验效果来看，BERT 比 ELMo 效果好主要集中在以下几点原因：

1. LSTM 抽取特征的能力远弱于 Transformer
2. 拼接方式双向融合的特征融合能力偏弱(没有具体实验验证，只是推测)
3. 其实还有一点，BERT 的训练数据以及模型参数均多余 ELMo，这也是比较重要的一点

#### 4.2 ELMo 和 BERT 的区别是什么？

ELMo 模型是通过语言模型任务得到句子中单词的 embedding 表示，以此作为补充的新特征给下游任务使用。因为 ELMO 给下游提供的是每个单词的特征形式，所以这一类预训练的方法被称为“Feature-based Pre-Training”。而 BERT 模型是“基于 Fine-tuning 的模式”，这种做法和图像领域基于 Fine-tuning 的方式基本一致，下游任务需要将模型改造成 BERT 模型，才可利用 BERT 模型预训练好的参数。

## 5.BERT 有什么局限性？

从 XLNet 论文中，提到了 BERT 的两个缺点，分别如下：

- BERT 在第一个预训练阶段，假设句子中多个单词被 Mask 掉，这些被 Mask 掉的单词之间没有任何关系，是条件独立的，然而有时候这些单词之间是有关系的，比如”New York is a city”，假设我们 Mask 住”New”和”York”两个词，那么给定”is a city”的条件下”New”和”York”并不独立，因为”New York”是一个实体，看到”New”则后面出现”York”的概率要比看到”Old”后面出现”York”概率要大得多。

- - 但是需要注意的是，这个问题并不是什么大问题，甚至可以说对最后的结果并没有多大的影响，因为本身 BERT 预训练的语料就是海量的(动辄几十个 G)，所以如果训练数据足够大，其实不靠当前这个例子，靠其它例子，也能弥补被 Mask 单词直接的相互关系问题，因为总有其它例子能够学会这些单词的相互依赖关系。

- BERT 的在预训练时会出现特殊的[MASK]，但是它在下游的 fine-tune 中不会出现，这就出现了预训练阶段和 fine-tune 阶段不一致的问题。其实这个问题对最后结果产生多大的影响也是不够明确的，因为后续有许多 BERT 相关的预训练模型仍然保持了[MASK]标记，也取得了很大的结果，而且很多数据集上的结果也比 BERT 要好。但是确确实实引入[MASK]标记，也是为了构造自编码语言模型而采用的一种折中方式。

另外还有一个缺点，是 BERT 在分词后做[MASK]会产生的一个问题，为了解决 OOV 的问题，我们通常会把一个词切分成更细粒度的 WordPiece。BERT 在 Pretraining 的时候是随机 Mask 这些 WordPiece 的，这就可能出现只 Mask 一个词的一部分的情况，例如：

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/e10bca1791d6e251f95164621cde5e61.png?x-oss-process=image/resize,w_670)

probability 这个词被切分成”pro”、”#babi”和”#lity”3 个 WordPiece。有可能出现的一种随机 Mask 是把”#babi” Mask 住，但是”pro”和”#lity”没有被 Mask。这样的预测任务就变得容易了，因为在”pro”和”#lity”之间基本上只能是”#babi”了。这样它只需要记住一些词(WordPiece 的序列)就可以完成这个任务，而不是根据上下文的语义关系来预测出来的。类似的中文的词”模型”也可能被 Mask 部分(其实用”琵琶”的例子可能更好，因为这两个字只能一起出现而不能单独出现)，这也会让预测变得容易。

为了解决这个问题，很自然的想法就是词作为一个整体要么都 Mask 要么都不 Mask，这就是所谓的 Whole Word Masking。这是一个很简单的想法，对于 BERT 的代码修改也非常少，只是修改一些 Mask 的那段代码。

**「TODO：另外还有别的缺点及其改进，看到相关论文再补充。」**

## 6.BERT 的输入和输出分别是什么？

BERT 模型的主要输入是文本中各个字/词(或者称为 token)的原始词向量，该向量既可以随机初始化，也可以利用 Word2Vector 等算法进行预训练以作为初始值；输出是文本中各个字/词融合了全文语义信息后的向量表示，如下图所示（为方便描述且与 BERT 模型的当前中文版本保持一致，统一以 **「字向量」** 作为输入）：

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/2763d893bee68d13fd9b9017a3e77d44.jpeg?x-oss-process=image/resize,w_670)

从上图中可以看出，**BERT 模型通过查询字向量表将文本中的每个字转换为一维向量，作为模型输入；模型输出则是输入各字对应的融合全文语义信息后的向量表示。**此外，模型输入除了字向量(英文中对应的是 Token Embeddings)，还包含另外两个部分：

1. 文本向量(英文中对应的是 Segment Embeddings)：该向量的取值在模型训练过程中自动学习，用于刻画文本的全局语义信息，并与单字/词的语义信息相融合
2. 位置向量(英文中对应的是 Position Embeddings)：由于出现在文本不同位置的字/词所携带的语义信息存在差异（比如：“我爱你”和“你爱我”），因此，BERT 模型对不同位置的字/词分别附加一个不同的向量以作区分

最后，BERT 模型将字向量、文本向量和位置向量的加和作为模型输入。特别地，在目前的 BERT 模型中，文章作者还将英文词汇作进一步切割，划分为更细粒度的语义单位（WordPiece），例如：将 playing 分割为 play 和##ing；此外，对于中文，目前作者未对输入文本进行分词，而是直接将单字作为构成文本的基本单位。

需要注意的是，上图中只是简单介绍了单个句子输入 BERT 模型中的表示，实际上，在做 Next Sentence Prediction 任务时，在第一个句子的首部会加上一个[CLS] token，在两个句子中间以及最后一个句子的尾部会加上一个[SEP] token。

## 7.针对句子语义相似度/多标签分类/机器翻译翻译/文本生成的任务，利用 BERT 结构怎么做 fine-tuning？

#### 7.1 针对句子语义相似度的任务

![新知达人, 【面试经验】关于BERT，面试官们都怎么问](https://img.shangyexinzhi.com/xztest-image/article/ad720936d2804b5783e9d16f224ab5c5.png?x-oss-process=image/resize,w_670)

bert fine tuning classification

实际操作时，上述最后一句话之后还会加一个[SEP] token，语义相似度任务将两个句子按照上述方式输入即可，之后与论文中的分类任务一样，将[CLS] token 位置对应的输出，接上 softmax 做分类即可(实际上 GLUE 任务中就有很多语义相似度的数据集)。

#### 7.2 针对多标签分类的任务

多标签分类任务，即 MultiLabel，指的是一个样本可能同时属于多个类，即有多个标签。以商品为例，一件 L 尺寸的棉服，则该样本就有至少两个标签——型号：L，类型：冬装。

对于多标签分类任务，显而易见的朴素做法就是不管样本属于几个类，就给它训练几个分类模型即可，然后再一一判断在该类别中，其属于那个子类别，但是这样做未免太暴力了，而多标签分类任务，其实是可以 **「只用一个模型」** 来解决的。

利用 BERT 模型解决多标签分类问题时，其输入与普通单标签分类问题一致，得到其 embedding 表示之后(也就是 BERT 输出层的 embedding)，有几个 label 就连接到几个全连接层(也可以称为 projection layer)，然后再分别接上 softmax 分类层，这样的话会得到

，最后再将所有的 loss 相加起来即可。这种做法就相当于将 n 个分类模型的特征提取层参数共享，得到一个共享的表示(其维度可以视任务而定，由于是多标签分类任务，因此其维度可以适当增大一些)，最后再做多标签分类任务。

#### 7.3 针对翻译的任务

针对翻译的任务，我自己想到一种做法，因为 BERT 本身会产生 embedding 这样的“副产品”，因此可以直接利用 BERT 输出层得到的 embedding，然后在做机器翻译任务时，将其作为输入/输出的 embedding 表示，这样做的话，可能会遇到 UNK 的问题，为了解决 UNK 的问题，可以将得到的词向量 embedding 拼接字向量的 embedding 得到输入/输出的表示(对应到英文就是 token embedding 拼接经过 charcnn 的 embedding 的表示)。

#### 7.4 针对文本生成的任务

关于生成任务，搜到以下几篇论文：

[2]BERT has a Mouth, and It Must Speak: BERT as a Markov Random Field Language Model

[3]MASS: Masked Sequence to Sequence Pre-training for Language Generation

[4]Unified Language Model Pre-training for Natural Language Understanding and Generation



## 8.BERT 应用于有空格丢失或者单词拼写错误等数据是否还是有效？有什么改进的方法？

#### 8.1 BERT 应用于有空格丢失的数据是否还是有效？

按照常理推断可能会无效了，因为空格都没有的话，那么便成为了一长段文本，但是具体还是有待验证。而对于有空格丢失的数据要如何处理呢？一种方式是利用 Bi-LSTM + CRF 做分词处理，待其处理成正常文本之后，再将其输入 BERT 做下游任务。

#### 8.2 BERT 应用于单词拼写错误的数据是否还是有效？

如果有少量的单词拼写错误，那么造成的影响应该不会太大，因为 BERT 预训练的语料非常丰富，而且很多语料也不够干净，其中肯定也还是会含有不少单词拼写错误这样的情况。但是如果单词拼写错误的比例比较大，比如达到了 30%、50%这种比例，那么需要通过人工特征工程的方式，以中文中的同义词替换为例，将不同的错字/别字都替换成同样的词语，这样减少错别字带来的影响。例如花被、花珼、花呗、花呗、花钡均替换成花呗。

## 9.BERT 的 embedding 向量如何得来的？

以中文为例， **「BERT 模型通过查询字向量表将文本中的每个字转换为一维向量，作为模型输入(还有 position embedding 和 segment embedding)；模型输出则是输入各字对应的融合全文语义信息后的向量表示。」**

而对于输入的 token embedding、segment embedding、position embedding 都是随机生成的，需要注意的是在 Transformer 论文中的 position embedding 由 sin/cos 函数生成的固定的值，而在这里代码实现中是跟普通 word embedding 一样随机生成的，可以训练的。作者这里这样选择的原因可能是 BERT 训练的数据比 Transformer 那篇大很多，完全可以让模型自己去学习。

## 10.BERT 模型为什么要用 mask？它是如何做 mask 的？其 mask 相对于 CBOW 有什么异同点？

#### 10.1 BERT 模型为什么要用 mask?

BERT 通过在输入 X 中随机 Mask 掉一部分单词，然后预训练过程的主要任务之一是根据上下文单词来预测这些被 Mask 掉的单词。其实这个就是典型的 Denosing Autoencoder 的思路，那些被 Mask 掉的单词就是**在输入侧加入的所谓噪音。**类似 BERT 这种预训练模式，被称为 DAE LM。因此总结来说 BERT 模型 [Mask] 标记就是引入噪音的手段。

关于 DAE LM 预训练模式，优点是它能比较自然地融入双向语言模型，同时看到被预测单词的上文和下文，然而缺点也很明显，主要在输入侧引入[Mask]标记，导致预训练阶段和 Fine-tuning 阶段不一致的问题。

#### 10.2 它是如何做 mask 的？

给定一个句子，会随机 Mask 15%的词，然后让 BERT 来预测这些 Mask 的词，如同上述 10.1 所述，在输入侧引入[Mask]标记，会导致预训练阶段和 Fine-tuning 阶段不一致的问题，因此在论文中为了缓解这一问题，采取了如下措施：

如果某个 Token 在被选中的 15%个 Token 里，则按照下面的方式随机的执行：

- 80%的概率替换成[MASK]，比如 my dog is hairy → my dog is [MASK]
- 10%的概率替换成随机的一个词，比如 my dog is hairy → my dog is apple
- 10%的概率替换成它本身，比如 my dog is hairy → my dog is hairy

这样做的好处是，BERT 并不知道[MASK]替换的是这 15%个 Token 中的哪一个词( **「注意：这里意思是输入的时候不知道[MASK]替换的是哪一个词，但是输出还是知道要预测哪个词的」** )，而且任何一个词都有可能是被替换掉的，比如它看到的 apple 可能是被替换的词。这样强迫模型在编码当前时刻的时候不能太依赖于当前的词，而要考虑它的上下文，甚至对其上下文进行”纠错”。比如上面的例子模型在编码 apple 是根据上下文 my dog is 应该把 apple(部分)编码成 hairy 的语义而不是 apple 的语义。

#### 10.3 其 mask 相对于 CBOW 有什么异同点？

**「相同点」** ：CBOW 的核心思想是：给定上下文，根据它的上文 Context-Before 和下文 Context-after 去预测 input word。而 BERT 本质上也是这么做的，但是 BERT 的做法是给定一个句子，会随机 Mask 15%的词，然后让 BERT 来预测这些 Mask 的词。

**「不同点」** ：首先，在 CBOW 中，每个单词都会成为 input word，而 BERT 不是这么做的，原因是这样做的话，训练数据就太大了，而且训练时间也会非常长。

其次，对于输入数据部分，CBOW 中的输入数据只有待预测单词的上下文，而 BERT 的输入是带有[MASK] token 的“完整”句子，也就是说 BERT 在输入端将待预测的 input word 用[MASK] token 代替了。

另外，通过 CBOW 模型训练后，每个单词的 word embedding 是唯一的，因此并不能很好的处理一词多义的问题，而 BERT 模型得到的 word embedding(token embedding)融合了上下文的信息，就算是同一个单词，在不同的上下文环境下，得到的 word embedding 是不一样的。

其实自己在整理这个问题时，萌生了新的问题，具体描述如下：

**「为什么 BERT 中输入数据的[mask]标记为什么不能直接留空或者直接输入原始数据，在 self-attention 的 Q K V 计算中，不与待预测的单词做 Q K V 交互计算？」**

这个问题还要补充一点细节，就是数据可以像 CBOW 那样，每一条数据只留一个“空”，这样的话，之后在预测的时候，就可以将待预测单词之外的所有单词的表示融合起来(均值融合或者最大值融合等方式)，然后再接上 softmax 做分类。

乍一看，感觉这个 idea 确实有可能可行，而且也没有看到什么不合理之处，但是需要注意的是，这样做的话，需要每预测一个单词，就要计算一套 Q、K、V。就算不每次都计算，那么保存每次得到的 Q、K、V 也需要耗费大量的空间。总而言之，这种做法确实可能也是可行，但是实际操作难度却很大，从计算量来说，就是预训练 BERT 模型的好几倍(至少)，而且要保存中间状态也并非易事。其实还有挺重要的一点，如果像 CBOW 那样做，那么文章的“创新”在哪呢~

## 11.BERT 的两个预训练任务对应的损失函数是什么(用公式形式展示)？

BERT 的损失函数由两部分组成，第一部分是来自 Mask-LM 的 **「单词级别分类任务」** ，另一部分是 **「句子级别的分类任务」** 。通过这两个任务的联合学习，可以使得 BERT 学习到的表征既有 token 级别信息，同时也包含了句子级别的语义信息。具体损失函数如下：

其中

是 BERT 中 Encoder 部分的参数， 是 Mask-LM 任务中在 Encoder 上所接的输出层中的参数， 则是句子预测任务中在 Encoder 接上的分类器参数。因此，在第一部分的损失函数中，如果被 mask 的词集合为 M，因为它是一个词典大小 |V| 上的多分类问题，那么具体说来有：

在句子预测任务中，也是一个分类问题的损失函数：

因此，两个任务联合学习的损失函数是：

具体的预训练工程实现细节方面，BERT 还利用了一系列策略，使得模型更易于训练，比如对于学习率的 warm-up 策略，使用的激活函数不再是普通的 ReLu，而是 GeLu，也使用了 dropout 等常见的训练技巧。

## 12.词袋模型到 word2vec 改进了什么？word2vec 到 BERT 又改进了什么？

#### 12.1 词袋模型到 word2vec 改进了什么？

词袋模型(Bag-of-words model)是将一段文本（比如一个句子或是一个文档）用一个“装着这些词的袋子”来表示，这种表示方式不考虑文法以及词的顺序。 **「而在用词袋模型时，文档的向量表示直接将各词的词频向量表示加和」** 。通过上述描述，可以得出词袋模型的两个缺点：

- 词向量化后，词与词之间是有权重大小关系的，不一定词出现的越多，权重越大。
- 词与词之间是没有顺序关系的。

而 word2vec 是考虑词语位置关系的一种模型。通过大量语料的训练，将每一个词语映射成一个低维稠密向量，通过求余弦的方式，可以判断两个词语之间的关系，word2vec 其底层主要采用基于 CBOW 和 Skip-Gram 算法的神经网络模型。

因此，综上所述，词袋模型到 word2vec 的改进主要集中于以下两点：

- 考虑了词与词之间的顺序，引入了上下文的信息
- 得到了词更加准确的表示，其表达的信息更为丰富

#### 12.2 word2vec 到 BERT 又改进了什么？

word2vec 到 BERT 的改进之处其实没有很明确的答案，如同上面的问题所述，BERT 的思想其实很大程度上来源于 CBOW 模型，如果从准确率上说改进的话，BERT 利用更深的模型，以及海量的语料，得到的 embedding 表示，来做下游任务时的准确率是要比 word2vec 高不少的。实际上，这也离不开模型的“加码”以及数据的“巨大加码”。再从方法的意义角度来说，BERT 的重要意义在于给大量的 NLP 任务提供了一个泛化能力很强的预训练模型，而仅仅使用 word2vec 产生的词向量表示，不仅能够完成的任务比 BERT 少了很多，而且很多时候直接利用 word2vec 产生的词向量表示给下游任务提供信息，下游任务的表现不一定会很好，甚至会比较差。

# Bert的双向体现在什么地方？

mask+attention，mask的word结合全部其他encoder word的信息

 

# Bert的是怎样实现mask构造的？

MLM：将完整句子中的部分字mask，预测该mask词
NSP：为每个训练前的例子选择句子 A 和 B 时，50% 的情况下 B 是真的在 A 后面的下一个句子， 50% 的情况下是来自语料库的随机句子，进行二分预测是否为真实下一句

# 在数据中随机选择 15% 的标记，其中80%被换位[mask]，10%不变、10%随机替换其他单词，这样做的原因是什么？

mask只会出现在构造句子中，当真实场景下是不会出现mask的，全mask不match句型了
随机替换也帮助训练修正了[unused]和[UNK]
强迫文本记忆上下文信息

# 为什么BERT有3个嵌入层，它们都是如何实现的？

input_id是语义表达，和传统的w2v一样，方法也一样的lookup
segment_id是辅助BERT区别句子对中的两个句子的向量表示，从[1,embedding_size]里面lookup
position_id是为了获取文本天生的有序信息，否则就和传统词袋模型一样了，从[511,embedding_size]里面lookup

# bert的损失函数？

MLM:在 encoder 的输出上添加一个分类层,用嵌入矩阵乘以输出向量，将其转换为词汇的维度,用 softmax 计算mask中每个单词的概率
NSP:用一个简单的分类层将 [CLS] 标记的输出变换为 2×1 形状的向量,用 softmax 计算 IsNextSequence 的概率
MLM+NSP即为最后的损失

# 手写一个multi-head attention？

tf.multal(tf.nn.softmax(tf.multiply(tf.multal(q,k,transpose_b=True),1/math.sqrt(float(size_per_head)))),v)

 

# 长文本预测如何构造Tokens？

head-only：保存前 510 个 token （留两个位置给 [CLS] 和 [SEP] ）
tail-only：保存最后 510 个token
head + tail ：选择前128个 token 和最后382个 token（文本在800以内）或者前256个token+后254个token（文本大于800tokens）

# 你用过什么模块？bert流程是怎么样的？

modeling.py
首先定义处理好输入的tokens的对应的id作为input_id,因为不是训练所以input_mask和segment_id都是采取默认的1即可
在通过embedding_lookup把input_id向量化，如果存在句子之间的位置差异则需要对segment_id进行处理，否则无操作；再进行position_embedding操作
进入Transform模块，后循环调用transformer的前向过程，次数为隐藏层个数，每次前向过程都包含self_attention_layer、add_and_norm、feed_forward和add_and_norm四个步骤
输出结果为句向量则取[cls]对应的向量（需要处理成embedding_size），否则也可以取最后一层的输出作为每个词的向量组合all_encoder_layers[-1]

# 知道分词模块：FullTokenizer做了哪些事情么？

BasicTokenizer：根据空格等进行普通的分词
包括了一些预处理的方法：去除无意义词，跳过'\t'这些词，unicode变换，中文字符筛选等等
WordpieceTokenizer：前者的结果再细粒度的切分为WordPiece
中文不处理，因为有词缀一说：解决OOV

# Bert中如何获得词意和句意？

get_pooled_out代表了涵盖了整条语句的信息
get_sentence_out代表了这个获取每个token的output 输出，用的是cls向量

# 源码中Attention后实际的流程是如何的？

Transform模块中：在残差连接之前，对output_layer进行了dense+dropout后再合并input_layer进行的layer_norm得到的attention_output
所有attention_output得到并合并后，也是先进行了全连接，而后再进行了dense+dropout再合并的attention_output之后才进行layer_norm得到最终的layer_output

# 为什么要在Attention后使用残差结构？

残差结构能够很好的消除层数加深所带来的信息损失问题

 

# 平时用官方Bert包么？耗时怎么样？

第三方：bert_serving
官方：bert_base
耗时：64GTesla，64max_seq_length，80-90doc/s
在线预测只能一条一条的入参，实际上在可承受的计算量内batch越大整体的计算性能性价比越高

# 你觉得BERT比普通LM的新颖点？

mask机制
next_sentence_predict机制

# elmo、GPT、bert三者之间有什么区别？

特征提取器：elmo采用LSTM进行提取，GPT和bert则采用Transformer进行提取。很多任务表明Transformer特征提取能力强于LSTM，elmo采用1层静态向量+2层LSTM，多层提取能力有限，而GPT和bert中的Transformer可采用多层，并行计算能力强。
单/双向语言模型：GPT采用单向语言模型，elmo和bert采用双向语言模型。但是elmo实际上是两个单向语言模型（方向相反）的拼接，这种融合特征的能力比bert一体化融合特征方式弱。
GPT和bert都采用Transformer，Transformer是encoder-decoder结构，GPT的单向语言模型采用decoder部分，decoder的部分见到的都是不完整的句子；bert的双向语言模型则采用encoder部分，采用了完整句子



# 其它bert面试题

https://vodkazy.cn/2020/10/10/%E6%88%91%E6%83%B3%E5%8E%BB%E9%9D%A2%E8%AF%95%E7%B3%BB%E5%88%97%E2%80%94%E2%80%94BERT/

https://blog.csdn.net/qq_34832393/article/details/104356462

https://www.shangmayuan.com/a/cc156d3a10a44284ad6ead1c.html

Linear 层的权重定义中，是按照 (out_features, in_features) 顺序来的，实际计算会先将 weight 转置在乘以输入矩阵。所以 FC层 对应的 Linear 权重维度也是 (v,d)，可以直接共享。

上述BERT pytorch代码来自:https://github.com/xieyufei1993/Bert-Pytorch-Chinese-TextClassification，结构层次非常清晰。
而在BERT config中
"max_position_embeddings": 512 "type_vocab_size": 2
因此，在直接使用Google 的BERT预训练模型时，输入最多512个词（还要除掉[CLS]和[SEP]），最多两个句子合成一句。这之外的词和句子会没有对应的embedding。
当然，如果有足够的硬件资源自己重新训练BERT，可以更改 BERT config，设置更大max_position_embeddings 和 type_vocab_size值去满足自己的需求。

# 参考

[1]https://zhuanlan.zhihu.com/p/151412524

[2]https://zhuanlan.zhihu.com/p/132554155

[3]https://blog.csdn.net/lijiaqi0612/article/details/104735006

[4]https://www.shangyexinzhi.com/article/554884.html

[5]https://aijishu.com/a/1060000000092072