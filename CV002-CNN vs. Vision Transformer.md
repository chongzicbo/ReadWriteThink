这篇博客来自 Vision Transformers 论文 [AN IMAGE IS WORTH 16X16 WORDS: TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE](https://arxiv.org/pdf/2010.11929.pdf)。本文提出使用纯Transformer直接应用于图像patches 进行图像分类任务。在对大量数据进行预训练后， Vision Transformers （ViT）在多个基准测试中优于最先进的卷积网络，同时需要更少的计算资源进行训练。

Transformers由于其计算效率和可扩展性而成为NLP中的首选模型。在计算机视觉中，卷积神经网络（CNN）架构仍然占主导地位，但一些研究人员已经尝试将自注意相结合。作者尝试将标准Transformer直接应用于图像，并发现当在中等大小的数据集上训练时，与类似ResNet的架构相比，模型的准确性适中。然而，当在更大的数据集上训练时，ViT取得了更优异的结果，并在多个图像识别基准上接近或超过了现有技术。

![img](https://miro.medium.com/v2/resize:fit:700/0*brmcPLvJpiQWjZpY)

图1（取自原始论文）描述了一个模型，该模型通过将2D图像转换为2D patches序列来处理2D图像。然后利用可训练的线性投影将它们映射到恒定的隐向量。可学习embedding被预先添加到patches序列中，其在Transformer编码器输出端的状态作为图像的表示。图像表示然后通过分类头(classification head)进行预训练或微调。还需要添加位置嵌入以保留位置信息，嵌入向量的序列用作Transformer编码器的输入，Transformer编码器由多头自注意模块和MLP和MLP模块交替组成。

在过去，长期以来，CNN一直是图像处理任务的首选。它们擅长通过卷积层捕捉局部空间模式，从而实现分层特征提取。CNN擅长从大量图像数据中学习，并在图像分类、目标检测和分割等任务中取得了显著成功。

虽然CNN在各种计算机视觉任务中有着良好的记录，并能有效地处理大规模数据集，但VIT在全局依赖性和上下文理解至关重要的场景中具有优势。然而，VIT通常需要大量的训练数据才能实现与CNN相当的性能。此外，由于其可并行性，CNN在计算上是高效的，使其在实时和资源受限的应用中更加实用。

# Example: CNN vs. Vision Transformer

在本节中，我们将使用CNN和VIT方法，在Kaggle中提供的猫和狗数据集上训练VIT。首先，我们将从Kaggle下载带有25000个RGB图像的猫和狗数据集。如果你还没有，你可以阅读[这里](https://www.kaggle.com/docs/api)的说明，了解如何设置你的Kaggle API凭据。下面的Python代码将把数据集下载到您当前的工作目录中。

```python
from kaggle.api.kaggle_api_extended import KaggleApi

api = KaggleApi()
api.authenticate()

# we write to the current directory with './'
api.dataset_download_files('karakaggle/kaggle-cat-vs-dog-dataset', path='./')
```

下载文件后，可以使用以下命令解压缩文件。

```
!unzip -qq kaggle-cat-vs-dog-dataset.zip
!rm -r kaggle-cat-vs-dog-dataset.zip
```

使用以下命令克隆VIT GitHub存储库。这个存储库在vision_tr目录下拥有VIT所需的所有代码。

```
!git clone https://github.com/RustamyF/vision-transformer.git
!mv vision-transformer/vision_tr .
```

下载的数据需要进行清理和准备，以便训练我们的图像分类器。创建以下实用程序函数以清理和加载Pytorch的DataLoader格式的数据。

```
import torch.nn as nn
import torch
import torch.optim as optim

from torchvision import datasets, models, transforms
from torch.utils.data import DataLoader, Dataset
from PIL import Image
from sklearn.model_selection import train_test_split

import os


class LoadData:
    def __init__(self):
        self.cat_path = 'kagglecatsanddogs_3367a/PetImages/Cat'
        self.dog_path = 'kagglecatsanddogs_3367a/PetImages/Dog'

    def delete_non_jpeg_files(self, directory):
        for filename in os.listdir(directory):
            if not filename.endswith('.jpg') and not filename.endswith('.jpeg'):
                file_path = os.path.join(directory, filename)
                try:
                    if os.path.isfile(file_path) or os.path.islink(file_path):
                        os.unlink(file_path)
                    elif os.path.isdir(file_path):
                        shutil.rmtree(file_path)
                    print('deleted', file_path)
                except Exception as e:
                    print('Failed to delete %s. Reason: %s' % (file_path, e))

    def data(self):
        self.delete_non_jpeg_files(self.dog_path)
        self.delete_non_jpeg_files(self.cat_path)

        dog_list = os.listdir(self.dog_path)
        dog_list = [(os.path.join(self.dog_path, i), 1) for i in dog_list]

        cat_list = os.listdir(self.cat_path)
        cat_list = [(os.path.join(self.cat_path, i), 0) for i in cat_list]

        total_list = cat_list + dog_list

        train_list, test_list = train_test_split(total_list, test_size=0.2)
        train_list, val_list = train_test_split(train_list, test_size=0.2)
        print('train list', len(train_list))
        print('test list', len(test_list))
        print('val list', len(val_list))
        return train_list, test_list, val_list


# data Augumentation
transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
])


class dataset(torch.utils.data.Dataset):

    def __init__(self, file_list, transform=None):
        self.file_list = file_list
        self.transform = transform

    # dataset length
    def __len__(self):
        self.filelength = len(self.file_list)
        return self.filelength

    # load an one of images
    def __getitem__(self, idx):
        img_path, label = self.file_list[idx]
        img = Image.open(img_path).convert('RGB')
        img_transformed = self.transform(img)
        return img_transformed, label
```

## **CNN 方法 **

该图像分类器的CNN模型由三层2D卷积组成，核大小为3，步长为2，最大池化层为2。在卷积层之后，有两个全连接的层，每个层由10个节点组成。下面是一个代码片段，说明了这种结构：

```
class Cnn(nn.Module):
    def __init__(self):
        super(Cnn, self).__init__()

        self.layer1 = nn.Sequential(
            nn.Conv2d(3, 16, kernel_size=3, padding=0, stride=2),
            nn.BatchNorm2d(16),
            nn.ReLU(),
            nn.MaxPool2d(2)
        )

        self.layer2 = nn.Sequential(
            nn.Conv2d(16, 32, kernel_size=3, padding=0, stride=2),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2)
        )

        self.layer3 = nn.Sequential(
            nn.Conv2d(32, 64, kernel_size=3, padding=0, stride=2),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2)
        )

        self.fc1 = nn.Linear(3 * 3 * 64, 10)
        self.dropout = nn.Dropout(0.5)
        self.fc2 = nn.Linear(10, 2)
        self.relu = nn.ReLU()

    def forward(self, x):
        out = self.layer1(x)
        out = self.layer2(out)
        out = self.layer3(out)
        out = out.view(out.size(0), -1)
        out = self.relu(self.fc1(out))
        out = self.fc2(out)
        return out
```

使用Tesla T4（g4dn-xlarge）GPU机器进行10个epoch的训练。[Jupyter Notebook](https://github.com/RustamyF/vision-transformer/blob/master/vit_example.ipynb)可在项目的[GitHub存储库](https://github.com/RustamyF/vision-transformer)中获得，其中包含训练循环的代码。以下是每个历元的训练循环的结果。



## VIT方法

VIT架构采用可定制的尺寸设计，可根据特定要求进行调整。对于这种大小的图像数据集，这种架构仍然很大。

```
from vision_tr.simple_vit import ViT
model = ViT(
    image_size=224,
    patch_size=32,
    num_classes=2,
    dim=128,
    depth=12,
    heads=8,
    mlp_dim=1024,
    dropout=0.1,
    emb_dropout=0.1,
).to(device)
```

VIT中的每个参数都起着关键作用，如下所述：

* image_size=224：此参数指定模型的输入图像的所需大小（宽度和高度）。在这种情况下，期望图像的大小为224x224像素。
* patch_size=32：图像被划分为较小的patch，此参数定义每个patch的大小（宽度和高度）。在这种情况下，每个patch都是32x32像素。
* num_classes=2：该参数表示分类任务中的类数。在这个例子中，模型被设计为将输入分为两类（猫和狗）。
* dim=128：它指定了模型中嵌入向量的维度。嵌入捕获每个图像patch的表示。
* depth=12：此参数定义VIT（编码器模型）中的深度或层数。更高的深度允许更复杂的特征提取。
* heads=8：该参数表示模型的自注意机制中的注意头的数量。
* mlp_dim=1024：指定模型中多层感知器（mlp）隐藏层的维度。MLP负责在自注意层之后的token表示。
* dropout=0.1：此参数控制dropout率，这是一种用于防止过拟合的正则化技术。它在训练过程中随机将一小部分输入单位设置为0。
* emb_dropout=0.1：它定义了专门应用于token嵌入的丢弃率。这种dropout有助于防止在训练过程中过度依赖特定token。



使用Tesla T4（g4dn-xlarge）GPU机器对用于分类任务的VIT进行20个epoch的训练。训练进行了20个epoch（而不是CNN使用的10个时期），因为训练损失的收敛缓慢。

CNN方法在10个epoch中达到了75%的准确率，而VIT模型达到了69%的准确率，并且花费了更长的时间来训练。



# 结论

总之，当比较CNN和Vision Transformer模型时，在模型大小、内存要求、准确性和性能方面存在显著差异。传统上，CNN模型以其紧凑的尺寸和高效的内存利用率而闻名，这使得它们适合于资源受限的环境。它们已被证明在图像处理任务中非常有效，并在各种计算机视觉应用中表现出优异的准确性。另一方面，VIT提供了一种强大的方法来捕捉图像中的全局相关性和上下文理解，从而提高了某些任务的性能。然而，与CNN相比，VIT往往具有更大的模型尺寸和更高的内存要求。虽然它们可以实现令人印象深刻的准确性，特别是在处理较大的数据集时，但计算需求可能会限制它们在资源有限的情况下的实用性。最终，CNN和Vision Transformer模型之间的选择取决于手头任务的具体要求，同时考虑可用资源、数据集大小以及模型复杂性、准确性和性能之间的权衡等因素。随着计算机视觉领域的不断发展，这两种架构都有望取得进一步的进步，使研究人员和从业者能够根据自己的具体需求和限制做出更明智的选择。



> 原文：[Vision Transformers vs. Convolutional Neural Networks | by Fahim Rustamy, PhD | Medium](https://medium.com/@faheemrustamy/vision-transformers-vs-convolutional-neural-networks-5fe8f9e18efc)