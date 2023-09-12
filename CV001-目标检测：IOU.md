IOU(Intersection over Union) ，即交并比，是目标检测中的一个重要概念，一般指”预测的边框“和"真实的边框"的交集和并集的比。

# 1.原理

通过下面的例子来了解IOU是如何计算的。

![Intersection Over Union IoU Bird detection by various models](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/1-bird-detection-by-different-models-1024x387.jpg)

有三个模型A、B、C，输入图像分别得到预测边框。其中，红框是真实边框Ground Truth，青色的是模型预测的边框。

根据上图，很明显可以得出以下几点：

- 相比于模型B，模型A的预测边框与真实边框有更多的重叠。
- 模型C与真实边框有更高的重叠，但是其与背景也有更多的重叠。
- 因此，从模型B和C来看，仅根据真是边框和预测边框的重叠度来衡量模型效果不够公平，还要考虑定位的精度。
- 所以，对于以下情况需要进行惩罚：
  - 预测边框溢出真实边框

根据上述几点，则可以设计IOU。

IOU表示真实边框和预测边框的重叠面积与其合并面积的比指。当预测边框无法预测真实边框的实际范围，分母将变大，从而IoU更低。

IOU的范围是[0,1]。0表示真实边框每有任何重叠，1表示两者完全重叠。

![Intersection Over Union IoU Illustration](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/2-iou-illustration-1024x400.png)

![img](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/intersection-over-union-iou-1024x400.jpg)

借助IoU阈值，我们可以判断预测结果是真阳(TP)、假阳(FP)还是假阴(FN)。下面的例子显示了阈值为0.5时的情况：

![Prediction quality based on Intersection Over Union IoU](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/4-birds-prediction-types-1.jpg)

将检测结果判断为TP还是FP完全取决于要求：

- 当IOU阈值为0.5时，第一个预测是TP。
- 如果阈值为0.97，第一个预测变成FP。
- 同样，第二个预测将变成TP如果阈值为0.20.
- 理论上，如果把阈值设置为0，则第三个预测也可以变为TP。

# 2.目标分割中的IOU

IOU在目标检测中是一个辅助指标，但在目标分割中却是评价模型准确率的重要指标。

在目标分割中，预测的区域不一定是矩形的，可以是任何规则的或者不规则的形状，这意味着预测的分割掩码(segment masks)，而不是边界框。因此，这里是逐像素分析。此外，TP、FP和FN的定义略有不同，因为他们不是基于预定义的阈值。

- TP：真实区域(Ground Truth,GT)和分割掩码(S)之间的交集，数学上就是与运算(AND)
  $$
  TP=GT \bullet S
  $$
  

  

- FP:真实区域之外的预测区域。
  $$
  FP=(GT+S)-GT
  $$
  

- FN:模型没有预测到的真实区域中的像素数量
  $$
  FN=(GT+S)-S
  $$
  

因此IOU计算如下：
$$
IOU=\frac {TP}{(TP+FP+FN)}
$$

# 3.numpy实现

```python
import numpy as np


def get_iou(ground_truth, pred):
	# coordinates of the area of intersection.
	ix1 = np.maximum(ground_truth[0], pred[0])
	iy1 = np.maximum(ground_truth[1], pred[1])
	ix2 = np.minimum(ground_truth[2], pred[2])
	iy2 = np.minimum(ground_truth[3], pred[3])

	# Intersection height and width.
	i_height = np.maximum(iy2 - iy1 + 1, np.array(0.))
	i_width = np.maximum(ix2 - ix1 + 1, np.array(0.))

	area_of_intersection = i_height * i_width

	# Ground Truth dimensions.
	gt_height = ground_truth[3] - ground_truth[1] + 1
	gt_width = ground_truth[2] - ground_truth[0] + 1

	# Prediction dimensions.
	pd_height = pred[3] - pred[1] + 1
	pd_width = pred[2] - pred[0] + 1

	area_of_union = gt_height * gt_width + pd_height * pd_width - area_of_intersection

	iou = area_of_intersection / area_of_union

	return iou
```

# 4.Pytorch实现

```python
	import torch
	from torchvision import ops

	# Bounding box coordinates.
	ground_truth_bbox = torch.tensor([[1202, 123, 1650, 868]], dtype=torch.float)
	prediction_bbox = torch.tensor([[1162.0001, 92.0021, 1619.9832, 694.0033]], dtype=torch.float)

	# Get iou.
	iou = ops.box_iou(ground_truth_bbox, prediction_bbox)
	print('IOU : ', iou.numpy()[0][0])
```

