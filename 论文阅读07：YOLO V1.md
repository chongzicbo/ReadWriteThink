# 前置

之前的目标检测算法如R-CNN都包含两个步骤：

- 检测目标位置（生成检测矩形框）
- 使用分类器对矩形框进行分类

分类完之后还需要进行一些后处理，比如对重复的矩形框进行消除，整个pipeline相对比较复杂，难以优化。



# YOLO V1核心思想

![image-20230914122434417](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230914122434417.png)

将整张图片作为网络的输入，直接在输出层对BBox的位置和类别进行回归。

- Resize image：将输入图片resize到448x448。
- Run ConvNet：使用CNN提取特征，FC层输出分类和回归结果。
- Non-max Suppression：非极大值抑制筛选出最终的结果。

YOLO V1将对象检测重新定义为一个单一的回归问题，直接从图像像素到边界框坐标和类概率。只需在图像上看一次（YOLO）即可预测存在哪些对象以及它们在哪里。单个卷积网络同时预测多个边界框和这些框的类概率。YOLO对全图像进行训练，并直接优化检测性能。

与传统的目标检测方法相比，这种统一的模型有几个优点。

- 速度非常快。将检测框定为回归问题，因此我们不需要复杂的pipeline。
- YOLO对图像进行全局推理。与基于滑动窗口和区域建议(region proposal-based)的技术不同，YOLO在训练和测试期间看到整个图像，因此它隐式地编码关于类及其外观的上下文信息。
- YOLO的泛化性能更好。



# YOLO V1算法原理

![image-20230914135753449](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230914135753449.png)

首先将输入图片分成$S × S$个网格（grid cell），如果某个object的中心落在这个网格中，则这个网格就负责预测这个object。

每个网格需要预测B个BBox的位置信息和confidence（置信度），一个BBox对应着**四个位置信息**(x,y,w,h)**和一个confidence信息**。其中$(x,y)$是BBox中心点的位置，$(w,h)$是BBox的宽和高，这是相对于整张图片的。$(x,y,w,h)$相对于单元格归一化到0-1之间。例如图片的宽是width，高为height,BBox中心所在的网格坐标为$x_0,y_0$,则BBox的实际坐标为
$$
x/(width/S)-x_0,y/(height/S)-y_0
$$

$$
w/width, h/height
$$

置信度confidence分数反映了模型对BBox包含目标对象的可能性，以及它认为BBox预测的准确性。前者记为$Pr(object)$，后者用$IOU^{truth}_{pred}$。$Confidence=Pr(object) * IOU^{truth}_{pred}$。

每个网格除了预测B个BBox外，还要预测$C$个类别概率:$Pr(Class_i | Object)$,表示由该单元格负责预测的BBox中的目标属于各类的概率，不管该单元格预测多少个BBox，该单元格值预测一组类别概率值。

对于每个bounding box的class-specific confidence score:
$$
\begin{array}{c}{{P r(c l a s s_{i}|o b j e c t)*c o n f i d e n c e=}}\\ {{P r(o b j e c t)*I O{U}_{p r e d}^{t r u t h}=P r(c l a s s_{i})*I O{U}_{p r e d}^{t r u t h}}}\end{array}
$$
每张图片会输出一个$S \times S \times (B *5+C)$的向量。



## 网络结构

![image-20230914143302537](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/image-20230914143302537.png)

