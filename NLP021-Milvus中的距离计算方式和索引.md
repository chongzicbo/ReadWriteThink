# 1.距离计算

|    浮点型向量    | 二进制向量 |
| :--------------: | ---------- |
| 欧几里得距离(L2) | Jaccard    |
|     点积(IP)     | Hamming    |
|                  | Tanimoto   |

## 1.1 欧几里得距离(L2)

公式如下：
$$
d(a,b)
=d(b,a)
=\sqrt{\sum_{i=1}^n (b_i-a_i)^2}
$$
其中，$a=(a_1,a_2,\ldots,a_n)$ 和$b=(b_1,b_2,\dots,b_n)$是欧几里得空间中的两个点。

当数据是连续时，L2是最常用的距离计算公式。

## 1.2 内积(IP)

公式如下：
$$
p(A,B)=A \cdot B =\sum_{i=1}^n {a_i \times b_i }
$$
其中$A$ 和$B$是向量，$||A||$和$||B||$是$A$和$B$的范数。在根据方向来计算相似度时，IP更有用。

如果使用内积来计算向量相似度，必须对向量进行归一化，归一化后的内积相当于余弦相似度。

假设$X'$是向量$X$归一化后的:
$$
X'=(x_1',x_2',\dots,x_n ')
$$

$$
x_i ' = \frac {x_i}{||X||}=\frac {x_i} {\sum _{i=1} ^n {(x_i)^2}}
$$

## 1.3 Jaccard 距离

Jaccard相似度定义为两个集合的交集除以并集，公式如下：
$$
J(A,B)=\frac{|A  \bigcap B|}{|A| +|B| - |A\bigcap B|}
$$


Jaccard距离定义如下：
$$
d_j{(A,B)}=1-J{(A,B)}
$$


## 1.4 Tanimoto 距离



Tanimoto距离计算如下：
$$
d_t=\frac{A \cdot B }{ |A| ^2 +|B|^2 -A \cdot B}
$$


## 1.5 Hamming 距离

汉明距离用于衡量二进制数据字符串。两个长度相等的字符串之间的距离是两个字符串位不同的位的位数。

例如，假设有两个字符串，karolin 和kathrin.
$$
karolin  \bigoplus kathrin =3
$$


# 2.索引

Milvus支持的多数向量索引类型都是使用近似最近邻搜索算法(ANNS)。通常，精确搜索非常耗时，与其相比，ANNS不再局限于返回最精确的结果，而是搜索目标的邻居。ANNS通过牺牲可接受的精度来提升搜索效率。

根据实现方法，ANNS向量索引可以分为以下四种：

- Tree-based index
- Graph-based index
- Hash-based index
- Quantization-based index

根据数据类型，Milvus支持的索引分为两类：

- 用于浮点型向量的索引
  - 对于一个128维的浮点型向量，需要的存储空间是128* 4bytes=512bytes
  - 索引类型包括：FLAT, IVF_FLAT, IVF_PQ, IVF_SQ8, ANNOY, 和HNSW.
- 用于二进制向量的索引
  - 对于一个128维的二进制向量，需要的存储空间是128/8=16bytes
  - 索引类型包括：BIN_FLAT 和BIN_IVF_FLAT.

|          | 浮点型向量               |                                                        |              | 二进制向量               |                                  |
| -------- | ------------------------ | ------------------------------------------------------ | ------------ | ------------------------ | -------------------------------- |
| 支持索引 | 分类                     | 使用场景                                               | 支持索引     | 分类                     | 使用场景                         |
| Flat     | N/A                      | 1.检索小数据集<br /> 2.100%召回率                      | BIN_FLAT     | N/A                      | 1.小数据集<br />2.100%召回率     |
| IVF_FLAT | Quantization-based index | 1.高速查询<br />2.尽可能高的召回率                     | BIN_IVF_FLAT | Quantization-based index | 1.高速查询<br />尽可能高的召回率 |
| IVF_SQ8  | Quantization-based index | 1.高速查询<br />2.有限的内存资源<br />3.召回率妥协     |              |                          |                                  |
| IVF_PQ   | Quantization-based index | 1.非常高效的查询<br />2.有限内存资源<br />3.召回率妥协 |              |                          |                                  |
| HNSW     | Graph-based index        | 1.高速查询<br />2.尽可能高的召回率<br />3.大内存资源   |              |                          |                                  |
| ANNOY    | Tree-based index         | 1.低维向量                                             |              |                          |                                  |

## 2.1 FLAT

对于数据集比较小（百万级），要求高准确率的相似度检索应用，可以选择FLAT索引。FLAT不会对向量进行压缩，是唯一能够保证精确搜索结果的索引。FLAT产生的结果可以与其它召回率低于100%的索引产生的结果进行比较。

FLAT会采取暴力的方法对输入与数据集中的每个向量进行比较，因此结果比较准确。当然，FLAT也是最慢的索引，不适合大数据集。

在Milvus中关于FLAT索引也没有额外的参数设置，不要求数据训练或者额外的存储空间。

![image-20230526104646750](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526104646750.png)

## 2.2 IVF_FLAT

IVF_FLAT会将向量集合划分为$nlist$个簇，然后比较你的输入向量和各个簇的中心距离。根据你设置的搜索参数$nprobe$,从最近的$nprobe$个簇中进行搜索，返回最终的结果。通过调整参数$nprobe$可以让查询准确率和速度之间达到平衡。当$nprobe$增大时，查询时间也会增大。

![image-20230526105703598](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526105703598.png)

## 2.3 IVF_SQ8

IVF_FLAT没有对向量进行压缩，所以索引数据和原始向量数据的大小一样。举例，如果原始的1B SIFT数据集是476GB,IVF_FLAT索引文件也差不多470多GB。加载所有的索引文件到内存会消耗约470GB存储。

当磁盘、CPU和GPU内存资源有限时，IVF_SQ8索引更合适。IVF_SQ8通过执行标量量化把每个FLOAT(4 bytes)转换为UINT8(1字节)，因此，磁盘、CPU、GPU内存会减少70-75%.对于1B的SIFT数据集，IVF_SQ8索引仅需要140GB的存储。

![image-20230526111105149](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526111105149.png)

## 2.4 IVF_PQ

PQ(Product Quantization)将原来的高维向量空间均匀分解为m个低维向量空间，然后对分解后的低维向量空间进行量化。PQ不是计算目标向量与所有单元的中心之间的距离，而是计算每个低维空间的聚类中心与目标向量之间的距离，这大大降低了算法的时间复杂度和空间复杂度。IVF_PQ在量化向量的乘积(product)之前执行IVF索引聚类。它的索引文件比IVF_SQ8更小，但在搜索时也会损失精度。

![image-20230526154814187](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526154814187.png)

## 2.5 HNSW

HNSW(Hierarchical Navigable Small World Graph)是基于图的算法。它根据一定的规则为图像构建一个多层导航结构。在这种结构中，上层更稀疏，节点间的距离更远；下层更密集，节点之间的距离更近。搜索从最上层开始，找到该层最接近目标的节点，然后进入下一层开始另一次搜索。经过多次迭代，它可以快速接近目标位置。为了提高性能，HNSW将图的每层的节点的最大度(degree)限制为M。此外，可以指定efConstruction（构建索引时）或ef(搜索目标时)来指定搜索范围.

![image-20230526155534257](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526155534257.png)

## 2.6 ANNOY

ANNOY(Approximate Nearest Neighbors Oh Yeah)是一种使用超平面将高维空间划分为多个子空间，然后将其存储在树结构中的索引。

调整ANNOY只需要两个主要参数：$ntrees$:书的数量 $search_k$:搜索期间检查节点的数量。

- $n_trees$ 在构建期间提供，并且影响构建时间和索引大小。值越大，结果越准确，但索引越大。
- $search\_k$ 运行期间提供，影响搜索性能。值越大，结果越准确，返回结果的时间越长。

如果没有提供$search_k$，它将默认为$n * n_trees$，其中n是近似最近邻居的数量。否则，$search_k$和$n_trees$大致独立，即如果$search_k$保持不变，$n_trees$的值将不会影响搜索时间，反之亦然。基本上，考虑到您能负担得起的内存量，建议将$n_trees$设置得尽可能大，考虑到查询的时间限制，建议将$search_k$设置得尽尽可能大。

![image-20230526160416727](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230526160416727.png)

# 参考

- [In-memory Index Milvus documentation](https://milvus.io/docs/index.md#floating)