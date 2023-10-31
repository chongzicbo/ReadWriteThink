使用DETR模型在CPPE-5数据集上Finetune，主要包括三个步骤：

- 数据集处理
- 模型训练和验证
- 模型推理

确保已经安装以下依赖库：

```python
pip install -q datasets transformers evaluate timm albumentations
```

# 数据处理

使用datasets加载数据集，使用albumentations增强数据集，使用transformers训练模型。

## 数据集下载

```
import datasets
from datasets import load_dataset

config = datasets.DownloadConfig(resume_download=True, max_retries=100)
cppe5 = load_dataset(
    path="/data/huggingface/data/cppe-5",
    download_config=config,
)
print(cppe5)
print(cppe5["train"][0])
```

我是提前把数据集下载好放到别的目录下。

CPPE-5数据集包含带有标注的图像，用于识别COVID-19大流行背景下的医疗个人防护装备(PPE)。

​       该数据集包含1000个图像的训练集和一个包含29个图像的测试集。

```
{'image_id': 15,
 'image': <PIL.JpegImagePlugin.JpegImageFile image mode=RGB size=943x663 at 0x7F9EC9E77C10>,
 'width': 943,
 'height': 663,
 'objects': {'id': [114, 115, 116, 117],
  'area': [3796, 1596, 152768, 81002],
  'bbox': [[302.0, 109.0, 73.0, 52.0],
   [810.0, 100.0, 57.0, 28.0],
   [160.0, 31.0, 248.0, 616.0],
   [741.0, 68.0, 202.0, 401.0]],
  'category': [4, 4, 0, 0]}
  }
```

每个样本包含以下字段：

* `image_id`: 图片id

* `image`:  PIL.Image对象

* `width`: 图片宽度

* `height`: 图片高度

* ```
  objects
  ```

   一个字典，表示图片中包含的内容:

  * `id`: 标注id
  * `area`:bounding box面积
  * `bbox`:  [COCO 格式](https://albumentations.ai/docs/getting_started/bounding_boxes_augmentation/#coco) 的bbox
  * `category`: 目标类别 `Coverall (0)`, `Face_Shield (1)`, `Gloves (2)`, `Goggles (3)` and `Mask (4)`

## 数据预处理

还需要对数据进行转换成DETR模型需要的格式。

先可视化看看数据。

```python
import numpy as np
import os
from PIL import Image, ImageDraw

image = cppe5["train"][0]["image"]
annotations = cppe5["train"][0]["objects"]  # 标注
draw = ImageDraw.Draw(image)
categories = cppe5["train"].features["objects"].feature["category"].names
print(categories)
id2label = {index: x for index, x in enumerate(categories, start=0)}
label2id = {v: k for k, v in id2label.items()}

for i in range(len(annotations["id"])):
    box = annotations["bbox"][i]
    class_idx = annotations["category"][i]
    x, y, w, h = tuple(box)
    draw.rectangle((x, y, x + w, y + w), outline="red", width=1)
    draw.text((x, y), id2label[class_idx], fill="white")


image.save("./res.jpg")
```

![CPPE-5 Image Example](F:\Learning\ReadWriteThink\images\CV006-使用DETR进行目标检测\TdaqPJO.png)

用于对象检测的数据集的一个常见问题是边界框“拉伸”到图像边缘之外。这种“失控”的边界框可能会在训练过程中引发错误，应在现阶段加以解决。在这个数据集中有几个关于这个问题的数据样本，需要加以剔除。

```python
remove_idx = [590, 821, 822, 875, 876, 878, 879]
keep = [i for i in range(len(cppe5["train"])) if i not in remove_idx]
cppe5["train"] = cppe5["train"].select(keep)
```

要微调模型，必须对数据进行预处理，以精确匹配预训练模型所使用的方法。AutoImageProcessor负责处理图像数据，以创建DETR模型可以训练的pixel_value、pixel_mask和labels。图像处理器具有一些属性：

* `image_mean = [0.485, 0.456, 0.406 ]`
* `image_std = [0.229, 0.224, 0.225]`

这些是在模型预训练期间用于归一化图像的平均值和标准偏差。在对预先训练的图像模型进行推断或微调时至关重要。

## 图像增强

* 加载图像处理器

```python
from transformers import AutoImageProcessor
checkpoint = "facebook/detr-resnet-50"
image_processor = AutoImageProcessor.from_pretrained(
    checkpoint, cache_dir="/data/bocheng/huggingface/model/"
)
```

在使用图片处理器进行处理之前先对图像进行增强处理。

```python
import albumentations
import numpy as np
import torch

import albumentations
import numpy as np
import torch

# 图像增强操作
transform = albumentations.Compose(
    [
        albumentations.Resize(480, 480),
        albumentations.HorizontalFlip(p=1.0),
        albumentations.RandomBrightnessContrast(p=1.0),
    ],
    bbox_params=albumentations.BboxParams(
        format="coco", label_fields=["category"]
    ),  # bbox的类型 coco [x_min, y_min, width, height]
)
```

并把标注转换成期望的格式。

```python
def formatted_anns(image_id, category, area, bbox):
    annotations = []
    for i in range(0, len(category)):
        new_ann = {
            "image_id": image_id,
            "category_id": category[i],
            "isCrowd": 0,
            "area": area[i],
            "bbox": list(bbox[i]),
        }
        annotations.append(new_ann)

    return annotations


# transforming a batch
def transform_aug_ann(examples):
    image_ids = examples["image_id"]
    images, bboxes, area, categories = [], [], [], []
    # 处理一个batch的图片
    for image, objects in zip(examples["image"], examples["objects"]):
        image = np.array(image.convert("RGB"))[:, :, ::-1]  # 转换为RGB图像格式，再转换为RGB通道顺序
        # 图像增强
        out = transform(
            image=image, bboxes=objects["bbox"], category=objects["category"]
        )

        area.append(objects["area"])
        images.append(out["image"])
        bboxes.append(out["bboxes"])
        categories.append(out["category"])

    targets = [
        {"image_id": id_, "annotations": formatted_anns(id_, cat_, ar_, box_)}
        for id_, cat_, ar_, box_ in zip(image_ids, categories, area, bboxes)
    ]

    return image_processor(images=images, annotations=targets, return_tensors="pt")


cppe5["train"] = cppe5["train"].with_transform(transform_aug_ann)
print(cppe5["train"][15])


```

定义一个collate_fn

```python
def collate_fn(batch):
    pixel_values = [item["pixel_values"] for item in batch]
    encoding = image_processor.pad(pixel_values, return_tensors="pt")
    labels = [item["labels"] for item in batch]
    batch = {}
    batch["pixel_values"] = encoding["pixel_values"]
    batch["pixel_mask"] = encoding["pixel_mask"]
    batch["labels"] = labels
    return batch
```



# 模型训练

```python
from transformers import AutoModelForObjectDetection, DetrForObjectDetection, DetrConfig


# checkpoint = "facebook/detr-resnet-50"
model = DetrForObjectDetection.from_pretrained(
    pretrained_model_name_or_path=checkpoint,  # os.path.join(checkpoint, "pytorch_model.bin"),
    id2label=id2label,
    label2id=label2id,
    ignore_mismatched_sizes=True,
    cache_dir="/data/bocheng/huggingface/model/",
)
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="detr-resnet-50_finetuned_cppe5",
    per_device_train_batch_size=8,
    num_train_epochs=10,
    fp16=True,
    save_steps=200,
    logging_steps=50,
    learning_rate=1e-5,
    weight_decay=1e-4,
    save_total_limit=2,
    remove_unused_columns=False,
)
from transformers import Trainer

trainer = Trainer(
    model=model,
    args=training_args,
    data_collator=collate_fn,
    train_dataset=cppe5["train"],
    tokenizer=image_processor,
)
trainer.train()
```



# 模型验证

```python
def val_formatted_anns(image_id, objects):
    annotations = []
    for i in range(0, len(objects["id"])):
        new_ann = {
            "id": objects["id"][i],
            "category_id": objects["category"][i],
            "iscrowd": 0,
            "image_id": image_id,
            "area": objects["area"][i],
            "bbox": objects["bbox"][i],
        }
        annotations.append(new_ann)

    return annotations


# Save images and annotations into the files torchvision.datasets.CocoDetection expects
def save_cppe5_annotation_file_images(cppe5):
    output_json = {}
    path_output_cppe5 = f"{os.getcwd()}/cppe5/"

    if not os.path.exists(path_output_cppe5):
        os.makedirs(path_output_cppe5)

    path_anno = os.path.join(path_output_cppe5, "cppe5_ann.json")
    categories_json = [
        {"supercategory": "none", "id": id, "name": id2label[id]} for id in id2label
    ]
    output_json["images"] = []
    output_json["annotations"] = []
    for example in cppe5:
        ann = val_formatted_anns(example["image_id"], example["objects"])
        output_json["images"].append(
            {
                "id": example["image_id"],
                "width": example["image"].width,
                "height": example["image"].height,
                "file_name": f"{example['image_id']}.png",
            }
        )
        output_json["annotations"].extend(ann)
    output_json["categories"] = categories_json

    with open(path_anno, "w") as file:
        json.dump(output_json, file, ensure_ascii=False, indent=4)

    for im, img_id in zip(cppe5["image"], cppe5["image_id"]):
        path_img = os.path.join(path_output_cppe5, f"{img_id}.png")
        im.save(path_img)

    return path_output_cppe5, path_anno


import torchvision


class CocoDetection(torchvision.datasets.CocoDetection):
    def __init__(self, img_folder, image_processor, ann_file):
        super().__init__(img_folder, ann_file)
        self.image_processor = image_processor

    def __getitem__(self, idx):
        # read in PIL image and target in COCO format
        img, target = super(CocoDetection, self).__getitem__(idx)

        # preprocess image and target: converting target to DETR format,
        # resizing + normalization of both image and target)
        image_id = self.ids[idx]
        target = {"image_id": image_id, "annotations": target}
        encoding = self.image_processor(
            images=img, annotations=target, return_tensors="pt"
        )
        pixel_values = encoding["pixel_values"].squeeze()  # remove batch dimension
        target = encoding["labels"][0]  # remove batch dimension

        return {"pixel_values": pixel_values, "labels": target}


# im_processor = AutoImageProcessor.from_pretrained(
#     "./detr-resnet-50_finetuned_cppe5/checkpoint-1200"
# )

path_output_cppe5, path_anno = save_cppe5_annotation_file_images(cppe5["test"])
test_ds_coco_format = CocoDetection(path_output_cppe5, image_processor, path_anno)

import evaluate
from tqdm import tqdm

model = AutoModelForObjectDetection.from_pretrained(
    "./detr-resnet-50_finetuned_cppe5/checkpoint-1200"
)
module = evaluate.load("ybelkada/cocoevaluate", coco=test_ds_coco_format.coco)

val_dataloader = torch.utils.data.DataLoader(
    test_ds_coco_format,
    batch_size=8,
    shuffle=False,
    num_workers=4,
    collate_fn=collate_fn,
)

with torch.no_grad():
    for idx, batch in enumerate(tqdm(val_dataloader)):
        pixel_values = batch["pixel_values"]
        pixel_mask = batch["pixel_mask"]

        labels = [
            {k: v for k, v in t.items()} for t in batch["labels"]
        ]  # these are in DETR format, resized + normalized

        # forward pass
        outputs = model(pixel_values=pixel_values, pixel_mask=pixel_mask)

        orig_target_sizes = torch.stack(
            [target["orig_size"] for target in labels], dim=0
        )
        results = image_processor.post_process(
            outputs, orig_target_sizes
        )  # convert outputs of model to COCO api

        module.add(prediction=results, reference=labels)
        del batch

results = module.compute()
print(results)

```



# 模型推理

```python
from transformers import pipeline, AutoModelForObjectDetection, AutoImageProcessor
import requests
from PIL import Image, ImageDraw
import torch

url = "/home/bocheng/dev/mylearn/CV-Learning/ObjectDetection/cppe5/1002.png"
model_path = "./detr-resnet-50_finetuned_cppe5/checkpoint-1200"
image = Image.open(url)

# 1.使用pipeline接口进行推理
# image_processor = AutoImageProcessor.from_pretrained(
#     "./detr-resnet-50_finetuned_cppe5/checkpoint-1200"
# )
# model = AutoModelForObjectDetection.from_pretrained(
#     "./detr-resnet-50_finetuned_cppe5/checkpoint-1200"
# )
# obj_detector = pipeline(
#     "object-detection", model=model, image_processor=image_processor
# )
# out = obj_detector(image)
# for x in out:
#     print(x)
# out.save("./out.jpg")


image_processor = AutoImageProcessor.from_pretrained(model_path)
model = AutoModelForObjectDetection.from_pretrained(model_path)

with torch.no_grad():
    inputs = image_processor(images=image, return_tensors="pt")
    outputs = model(**inputs)
    target_sizes = torch.tensor([image.size[::-1]])
    results = image_processor.post_process_object_detection(
        outputs, threshold=0.2, target_sizes=target_sizes
    )[0]

for score, label, box in zip(results["scores"], results["labels"], results["boxes"]):
    box = [round(i, 2) for i in box.tolist()]
    print(
        f"Detected {model.config.id2label[label.item()]} with confidence "
        f"{round(score.item(), 3)} at location {box}"
    )
draw = ImageDraw.Draw(image)

for score, label, box in zip(results["scores"], results["labels"], results["boxes"]):
    box = [round(i, 2) for i in box.tolist()]
    x, y, x2, y2 = tuple(box)
    draw.rectangle((x, y, x2, y2), outline="red", width=1)
    draw.text((x, y), model.config.id2label[label.item()], fill="white")

image.save("./out.jpg")

```



原文：[Object detection (huggingface.co)](https://huggingface.co/docs/transformers/tasks/object_detection)

文章合集：