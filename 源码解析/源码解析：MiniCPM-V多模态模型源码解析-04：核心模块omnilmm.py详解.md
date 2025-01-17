---
title: 'MiniCPM-V多模态模型源码解析-04：核心模块omnilmm.py'
categories:
  - [人工智能,multi-modal]
tags:
  - 多模态
  - llm
  - MiniCPM
  - 源码解析
date: 2025-01-14 14:00:00
---



### **1. 导入模块**
```python
import gc  # 垃圾回收模块
import math  # 数学计算模块
import timm  # 视觉模型库
import torch  # PyTorch深度学习框架
from torch import Tensor  # PyTorch中的张量类型
import torch.nn as nn  # PyTorch中的神经网络模块
from torch.nn import CrossEntropyLoss  # 交叉熵损失函数
from typing import List, Optional, Tuple, Union  # 类型注解

from transformers import AutoConfig, AutoModelForCausalLM  # Hugging Face的自动配置和因果语言模型
from transformers import MistralForCausalLM, MistralModel, MistralConfig  # Mistral模型相关类
from transformers.modeling_outputs import BaseModelOutputWithPast, CausalLMOutputWithPast  # 模型输出类型

from omnilmm.model.utils import build_transform  # 图像变换构建工具
from omnilmm.model.resampler import Resampler  # 重采样器模块
```

---

### **2. 定义常量**
```python
DEFAULT_IMAGE_PATCH_TOKEN = "<im_patch>"  # 默认的图像patch token
DEFAULT_IM_START_TOKEN = "<im_start>"  # 默认的图像开始token
DEFAULT_IM_END_TOKEN = "<im_end>"  # 默认的图像结束token
```

---

### **3. OmniLMMConfig 类**
```python
class OmniLMMConfig(MistralConfig):
    model_type = "omnilmm"  # 模型类型为OmniLMM
```
- 继承自`MistralConfig`，用于配置OmniLMM模型。
- `model_type` 设置为 `"omnilmm"`，表示这是一个OmniLMM模型。

---

### **4. Identity 类**
```python
class Identity(torch.nn.Identity):
    def forward(self, input: Tensor, **kwargs) -> Tensor:
        return super().forward(input)  # 直接返回输入，不做任何处理
```
- 继承自`torch.nn.Identity`，用于占位或传递输入，不做任何变换。

---

### **5. create_vision_module 函数**
```python
def create_vision_module(config):
    # 使用timm库创建一个视觉模型，模型名为'eva02_enormous_patch14_clip_224.laion2b_plus'
    vision_tower = timm.create_model('eva02_enormous_patch14_clip_224.laion2b_plus',
                                     pretrained=False,  # 不使用预训练权重
                                     num_classes=0,  # 输出类别数为0
                                     dynamic_img_size=True,  # 动态图像尺寸
                                     dynamic_img_pad=True)  # 动态图像填充

    # 如果视觉模型是VisionTransformer类型，并且有注意力池化层，则将其替换为Identity
    if isinstance(vision_tower, timm.models.VisionTransformer):
        if vision_tower.attn_pool is not None:
            vision_tower.attn_pool = Identity()

    # 使用倒数第二层的输出，将最后一层替换为Identity
    vision_tower.blocks[-1] = Identity()

    # 获取嵌入维度
    embed_dim = config.hidden_size
    # 创建Resampler对象，用于对视觉特征进行重采样
    resampler = Resampler(
        grid_size=int(math.sqrt(config.num_query)),  # 网格大小
        embed_dim=embed_dim,  # 嵌入维度
        num_heads=embed_dim // 128,  # 注意力头数
        kv_dim=vision_tower.embed_dim,  # 键值维度
    )
    return vision_tower, resampler  # 返回视觉模型和重采样器
```
- 该函数用于创建视觉模块，包括视觉模型和重采样器。

---

### **6. OmniLMMModel 类**
```python
class OmniLMMModel(MistralModel):
    config_class = OmniLMMConfig  # 配置类为OmniLMMConfig

    def __init__(self, config: OmniLMMConfig, mm_vision_tower=None, mm_hidden_size=None, tune_clip=True):
        super(OmniLMMModel, self).__init__(config)  # 调用父类构造函数

        # 如果配置中有视觉模型，则创建视觉模块
        if hasattr(config, "mm_vision_tower"):
            vision_tower, resampler = create_vision_module(config)

            # HACK: 为了FSDP（完全分片数据并行），将视觉模型包装在列表中
            self.vision_tower = [vision_tower]
            self.resampler = resampler
            if tune_clip:
                self.vision_tower = self.vision_tower[0]  # 如果调优CLIP，则直接使用视觉模型

        self.vision_config = lambda x: None  # 视觉配置，初始化为空函数
```
- 继承自`MistralModel`，是OmniLMM的核心模型类。
- 初始化时根据配置创建视觉模块。

---

#### **6.1  initialize_vision_modules 方法**
```python
def initialize_vision_modules(self, vision_tower, no_randaug, num_query, image_size, tune_clip=False):
    self.config.mm_vision_tower = vision_tower  # 设置视觉模型
    self.config.use_mm_proj = True  # 使用多模态投影
    self.config.num_query = num_query  # 设置查询数量
    self.config.image_size = image_size  # 设置图像尺寸

    # 如果没有视觉模型，则创建并加载预训练权重
    if not hasattr(self, 'vision_tower'):
        vision_tower, resampler = create_vision_module(self.config)
        state_dict = torch.load(
            '/tt/data/public/multimodal/multimodal_model_ckpts/timm/eva02_enormous_patch14_clip_224.laion2b_plus.pt')
        vision_tower.load_state_dict(state_dict, strict=False)
        del state_dict
        gc.collect()
    else:
        # 如果视觉模型是列表，则取第一个元素
        if isinstance(self.vision_tower, list):
            vision_tower = self.vision_tower[0]
        else:
            vision_tower = self.vision_tower
        resampler = self.resampler
    self.vision_tower = vision_tower if tune_clip else [vision_tower]  # 根据是否调优CLIP设置视觉模型
    self.resampler = resampler  # 设置重采样器

    # 构建训练和评估时的图像变换
    train_img_transform = build_transform(
        is_train=True, randaug=not no_randaug, input_size=self.config.image_size, std_mode='OPENAI_CLIP')
    eval_img_transform = build_transform(
        is_train=False, input_size=self.config.image_size, std_mode='OPENAI_CLIP')

    return dict(
        image_processor=(train_img_transform, eval_img_transform),  # 返回图像处理器
        image_token_len=num_query,  # 返回图像token长度
        vision_config=self.vision_config  # 返回视觉配置
    )
```
- 该方法用于初始化视觉模块，包括加载预训练权重、设置图像变换等。

---

#### **6.2 get_vision_embedding 方法**
```python
def get_vision_embedding(self, pixel_values):
    if isinstance(self.vision_tower, list):
        vision_tower = self.vision_tower[0]  # HACK: 为了FSDP，取列表中的第一个元素
    else:
        vision_tower = self.vision_tower

    dtype = vision_tower.pos_embed.data.dtype  # 获取视觉模型的位置嵌入数据类型
    vision_embedding = vision_tower.forward_features(
        pixel_values.type(dtype))  # 获取视觉特征
    if hasattr(vision_tower, 'num_prefix_tokens') and vision_tower.num_prefix_tokens > 0:
        vision_embedding = vision_embedding[:,
                                            vision_tower.num_prefix_tokens:]  # 去掉前缀token
    res = self.resampler(vision_embedding)  # 对视觉特征进行重采样
    return res  # 返回重采样后的视觉特征
```
- 该方法用于从输入图像中提取视觉特征，并进行重采样。

#### **6.3 get_vllm_embedding 方法**

```python
def get_vllm_embedding(self, data):
    # 如果数据中没有视觉隐藏状态，则从像素值中提取视觉特征
    if 'vision_hidden_states' not in data:
        pixel_values_list = data['pixel_values']
        vision_hidden_states = []
        for pixel_values in pixel_values_list:
            if len(pixel_values) > 0:
                # 提取视觉特征
                vision_hidden_states.append(self.get_vision_embedding(pixel_values.unsqueeze(0))[0])
            else:
                vision_hidden_states.append([])
    else:
        vision_hidden_states = data['vision_hidden_states']

    # 获取输入token的嵌入
    inputs_embeds = self.embed_tokens(data['input_ids'])
    # 将视觉特征的数据类型转换为与输入嵌入一致
    vision_hidden_states = [i.type(inputs_embeds.dtype) 
        if isinstance(i, torch.Tensor) else i for i in vision_hidden_states
    ]

    # HACK: 替换回原始嵌入以支持LLaVA预训练
    orig_embeds_params = getattr(self, 'orig_embeds_params', None)

    new_input_embeds = []
    cur_image_idx = 0
    for cur_input_ids, cur_input_embeds in zip(data['input_ids'], inputs_embeds):
        # 如果当前输入中没有图像patch token，则直接使用原始嵌入
        if (cur_input_ids == self.vision_config.im_patch_token).sum() == 0:
            cur_input_embeds = cur_input_embeds + (0. * dummy_image_features).sum()
            new_input_embeds.append(cur_input_embeds)
            continue

        # 如果使用图像开始和结束token
        if self.vision_config.use_im_start_end:
            cur_image_features = vision_hidden_states[cur_image_idx]
            num_patches = cur_image_features.shape[0]
            # 检查图像开始和结束token的数量是否一致
            if (cur_input_ids == self.vision_config.im_start_token).sum() != (cur_input_ids == self.vision_config.im_end_token).sum():
                raise ValueError(
                    "The number of image start tokens and image end tokens should be the same.")
            # 找到图像开始token的位置
            image_start_tokens = torch.where(
                cur_input_ids == self.vision_config.im_start_token)[0]
            for image_start_token_pos in image_start_tokens:
                cur_image_features = vision_hidden_states[cur_image_idx].to(
                    device=cur_input_embeds.device)
                num_patches = cur_image_features.shape[0]
                # 检查图像结束token是否紧随图像开始token
                if cur_input_ids[image_start_token_pos + num_patches + 1] != self.vision_config.im_end_token:
                    raise ValueError(
                        "The image end token should follow the image start token.")
                # 如果存在原始嵌入参数，则拼接嵌入
                if orig_embeds_params is not None:
                    cur_new_input_embeds = torch.cat((cur_input_embeds[:image_start_token_pos].detach(), cur_input_embeds[image_start_token_pos:image_start_token_pos+1], cur_image_features,
                                                     cur_input_embeds[image_start_token_pos + num_patches + 1:image_start_token_pos + num_patches + 2], cur_input_embeds[image_start_token_pos + num_patches + 2:].detach()), dim=0)
                else:
                    cur_new_input_embeds = torch.cat(
                        (cur_input_embeds[:image_start_token_pos+1], cur_image_features, cur_input_embeds[image_start_token_pos + num_patches + 1:]), dim=0)
                cur_image_idx += 1
            new_input_embeds.append(cur_new_input_embeds)
        else:
            raise NotImplementedError
    inputs_embeds = torch.stack(new_input_embeds, dim=0)

    return inputs_embeds, vision_hidden_states
```

- 该方法用于从输入数据中提取视觉特征，并将其与文本嵌入结合，生成多模态输入嵌入。

------

#### **6.4 forward 方法**

```python
def forward(
    self,
    input_ids: torch.LongTensor = None,
    attention_mask: Optional[torch.Tensor] = None,
    past_key_values: Optional[List[torch.FloatTensor]] = None,
    inputs_embeds: Optional[torch.FloatTensor] = None,
    use_cache: Optional[bool] = None,
    output_attentions: Optional[bool] = None,
    output_hidden_states: Optional[bool] = None,
    images: Optional[torch.FloatTensor] = None,
    return_dict: Optional[bool] = None,
    **kwargs
) -> Union[Tuple, BaseModelOutputWithPast]:
    # HACK: 替换回原始嵌入以支持LLaVA预训练
    orig_embeds_params = getattr(self, 'orig_embeds_params', None)

    if inputs_embeds is None and past_key_values is None:
        inputs_embeds = self.embed_tokens(input_ids)  # 获取输入token的嵌入

        vision_tower = getattr(self, 'vision_tower', None)
        # 如果存在视觉模型且输入不是单token或处于训练模式，并且有图像输入
        if vision_tower is not None and (input_ids.shape[1] != 1 or self.training) and images is not None:
            if type(images) is list:
                image_features = []
                for image in images:
                    image_forward_out = self.get_vision_embedding(image.unsqueeze(0))[0]
                    image_features.append(image_forward_out)
            else:
                image_features = self.get_vision_embedding(images)

            # 创建虚拟图像特征
            dummy_image_features = torch.zeros(
                self.config.num_query,
                self.config.hidden_size,
                device=inputs_embeds.device,
                dtype=inputs_embeds.dtype)

            new_input_embeds = []
            cur_image_idx = 0
            for cur_input_ids, cur_input_embeds in zip(input_ids, inputs_embeds):
                # 如果当前输入中没有图像patch token，则直接使用原始嵌入
                if (cur_input_ids == self.vision_config.im_patch_token).sum() == 0:
                    cur_input_embeds = cur_input_embeds + (0. * dummy_image_features).sum()
                    new_input_embeds.append(cur_input_embeds)
                    continue

                # 如果使用图像开始和结束token
                if self.vision_config.use_im_start_end:
                    cur_image_features = image_features[cur_image_idx]
                    num_patches = cur_image_features.shape[0]
                    # 检查图像开始和结束token的数量是否一致
                    if (cur_input_ids == self.vision_config.im_start_token).sum() != (cur_input_ids == self.vision_config.im_end_token).sum():
                        raise ValueError(
                            "The number of image start tokens and image end tokens should be the same.")
                    # 找到图像开始token的位置
                    image_start_tokens = torch.where(
                        cur_input_ids == self.vision_config.im_start_token)[0]
                    for image_start_token_pos in image_start_tokens:
                        cur_image_features = image_features[cur_image_idx].to(
                            device=cur_input_embeds.device)
                        num_patches = cur_image_features.shape[0]
                        # 检查图像结束token是否紧随图像开始token
                        if cur_input_ids[image_start_token_pos + num_patches + 1] != self.vision_config.im_end_token:
                            raise ValueError(
                                "The image end token should follow the image start token.")
                        # 如果存在原始嵌入参数，则拼接嵌入
                        if orig_embeds_params is not None:
                            cur_new_input_embeds = torch.cat((cur_input_embeds[:image_start_token_pos].detach(), cur_input_embeds[image_start_token_pos:image_start_token_pos+1], cur_image_features,
                                                             cur_input_embeds[image_start_token_pos + num_patches + 1:image_start_token_pos + num_patches + 2], cur_input_embeds[image_start_token_pos + num_patches + 2:].detach()), dim=0)
                        else:
                            cur_new_input_embeds = torch.cat(
                                (cur_input_embeds[:image_start_token_pos+1], cur_image_features, cur_input_embeds[image_start_token_pos + num_patches + 1:]), dim=0)
                        cur_image_idx += 1
                    new_input_embeds.append(cur_new_input_embeds)
                else:
                    raise NotImplementedError
            inputs_embeds = torch.stack(new_input_embeds, dim=0)
            input_ids = None

    # 调用父类的forward方法
    return super(OmniLMMModel, self).forward(
        input_ids=input_ids, attention_mask=attention_mask, past_key_values=past_key_values,
        inputs_embeds=inputs_embeds, use_cache=use_cache,
        output_attentions=output_attentions, output_hidden_states=output_hidden_states,
        return_dict=return_dict,
        **kwargs
    )
```

- 该方法用于模型的前向传播，处理多模态输入（文本和图像），并调用父类的 `forward` 方法。

---

### **7. OmniLMMForCausalLM 类**
```python
class OmniLMMForCausalLM(MistralForCausalLM):
    config_class = OmniLMMConfig  # 配置类为OmniLMMConfig

    def __init__(self, config, mm_vision_tower=None, tune_clip=True):
        super(MistralForCausalLM, self).__init__(config)  # 调用父类构造函数
        self.model = OmniLMMModel(
            config, mm_vision_tower=mm_vision_tower, tune_clip=tune_clip)  # 初始化OmniLMM模型

        self.lm_head = nn.Linear(
            config.hidden_size, config.vocab_size, bias=False)  # 定义语言模型头

        # 初始化权重并应用最终处理
        self.post_init()
```
- 继承自`MistralForCausalLM`，用于实现因果语言模型。
- 初始化时创建OmniLMM模型，并定义语言模型头。

---

#### **7.1 forward 方法**
```python
def forward(
    self,
    input_ids: torch.LongTensor = None,
    attention_mask: Optional[torch.Tensor] = None,
    past_key_values: Optional[List[torch.FloatTensor]] = None,
    inputs_embeds: Optional[torch.FloatTensor] = None,
    labels: Optional[torch.LongTensor] = None,
    use_cache: Optional[bool] = None,
    output_attentions: Optional[bool] = None,
    output_hidden_states: Optional[bool] = None,
    images: Optional[torch.FloatTensor] = None,
    return_dict: Optional[bool] = None,
    **kwargs
) -> Union[Tuple, CausalLMOutputWithPast]:
    output_attentions = output_attentions if output_attentions is not None else self.config.output_attentions
    output_hidden_states = (
        output_hidden_states if output_hidden_states is not None else self.config.output_hidden_states
    )
    return_dict = return_dict if return_dict is not None else self.config.use_return_dict

    # 调用模型进行前向传播
    outputs = self.model(
        input_ids=input_ids,
        attention_mask=attention_mask,
        past_key_values=past_key_values,
        inputs_embeds=inputs_embeds,
        use_cache=use_cache,
        output_attentions=output_attentions,
        output_hidden_states=output_hidden_states,
        return_dict=return_dict,
        images=images,
        **kwargs
    )

    hidden_states = outputs[0]  # 获取隐藏状态
    logits = self.lm_head(hidden_states)  # 计算logits

    loss = None
    if labels is not None:
        # 计算损失
        shift_logits = logits[..., :-1, :].contiguous()
        shift_labels = labels[..., 1:].contiguous()
        loss_fct = CrossEntropyLoss()
        shift_logits = shift_logits.view(-1, self.config.vocab_size)
        shift_labels = shift_labels.view(-1)
        shift_labels = shift_labels.to(shift_logits.device)
        loss = loss_fct(shift_logits, shift_labels)

    if not return_dict:
        output = (logits,) + outputs[1:]
        return (loss,) + output if loss is not None else output

    return CausalLMOutputWithPast(
        loss=loss,
        logits=logits,
        past_key_values=outputs.past_key_values,
        hidden_states=outputs.hidden_states,
        attentions=outputs.attentions,
    )
```
- 该方法用于模型的前向传播，计算logits和损失。

---

#### **7.2 prepare_inputs_for_generation 方法**

```python
def prepare_inputs_for_generation(
    self, input_ids, past_key_values=None, attention_mask=None, inputs_embeds=None, **kwargs
):
    if past_key_values:
        input_ids = input_ids[:, -1:]  # 如果存在过去的键值对，则只使用最后一个token

    # 如果传递了inputs_embeds，则只在第一步生成时使用
    if inputs_embeds is not None and past_key_values is None:
        model_inputs = {"inputs_embeds": inputs_embeds}
    else:
        model_inputs = {"input_ids": input_ids}

    # 更新模型输入
    model_inputs.update(
        {
            "past_key_values": past_key_values,
            "use_cache": kwargs.get("use_cache"),
            "attention_mask": attention_mask,
            "images": kwargs.get("images", None),
        }
    )
    return model_inputs
```

- 该方法用于为生成任务准备输入数据。

------

#### **7.3 generate_vllm 方法**

```python
def generate_vllm(
    self,
    input_ids: torch.LongTensor = None,
    images: Optional[torch.FloatTensor] = None,
    vision_hidden_states=None,
    return_vision_hidden_states=False,
    **kwargs
):
    model_inputs = {'input_ids': input_ids}
    if vision_hidden_states is None:
        model_inputs['pixel_values'] = images  # 如果没有视觉隐藏状态，则传递像素值
    else:
        model_inputs['vision_hidden_states'] = vision_hidden_states  # 否则传递视觉隐藏状态

    with torch.inference_mode():
        # 获取输入嵌入和视觉隐藏状态
        inputs_embeds, vision_hidden_states = self.model.get_vllm_embedding(model_inputs)

        # 生成文本
        result = self.generate(
            inputs_embeds=inputs_embeds,
            **kwargs
        )

    if return_vision_hidden_states:
        return result, vision_hidden_states  # 如果需要返回视觉隐藏状态，则一起返回

    return result  # 否则只返回生成结果
```

- 该方法用于生成文本，支持视觉输入，并可以选择返回视觉隐藏状态。

------

#### **7.4 initialize_vision_tokenizer 方法**

```python
def initialize_vision_tokenizer(self, mm_use_im_start_end, tokenizer, device,
                                tune_mm_mlp_adapter=False):
    self.model.vision_config.use_im_start_end = mm_use_im_start_end  # 设置是否使用图像开始和结束token
    tokenizer.add_tokens([DEFAULT_IMAGE_PATCH_TOKEN], special_tokens=True)  # 添加图像patch token
    self.resize_token_embeddings(len(tokenizer))  # 调整token嵌入的大小

    if mm_use_im_start_end:
        # 添加图像开始和结束token
        num_new_tokens = tokenizer.add_tokens(
            [DEFAULT_IM_START_TOKEN, DEFAULT_IM_END_TOKEN], special_tokens=True)
        self.resize_token_embeddings(len(tokenizer))
        self.model.vision_config.im_start_token, self.model.vision_config.im_end_token = tokenizer.convert_tokens_to_ids(
            [DEFAULT_IM_START_TOKEN, DEFAULT_IM_END_TOKEN])

        if num_new_tokens > 0:
            # 对新token的嵌入进行平均初始化
            input_embeddings = self.get_input_embeddings().weight.data
            output_embeddings = self.get_output_embeddings().weight.data

            input_embeddings_avg = input_embeddings[:-num_new_tokens].mean(
                dim=0, keepdim=True)
            output_embeddings_avg = output_embeddings[:-num_new_tokens].mean(
                dim=0, keepdim=True)

            input_embeddings[-num_new_tokens:] = input_embeddings_avg
            output_embeddings[-num_new_tokens:] = output_embeddings_avg

        # 添加其他特殊token
        num_new_tokens = tokenizer.add_tokens(
            ['<box>', '</box>', '<ref>', '</ref>', '<quad>', '</quad>'], special_tokens=True)
        self.resize_token_embeddings(len(tokenizer))

        if num_new_tokens > 0:
            # 对新token的嵌入进行平均初始化
            input_embeddings = self.get_input_embeddings().weight.data
            output_embeddings = self.get_output_embeddings().weight.data

            input_embeddings_avg = input_embeddings[:-num_new_tokens].mean(
                dim=0, keepdim=True)
            output_embeddings_avg = output_embeddings[:-num_new_tokens].mean(
                dim=0, keepdim=True)

            input_embeddings[-num_new_tokens:] = input_embeddings_avg
            output_embeddings[-num_new_tokens:] = output_embeddings_avg

        if tune_mm_mlp_adapter:
            # 如果调优MLP适配器，则设置原始嵌入参数
            self.model.orig_embeds_params = [
                self.get_input_embeddings().weight.data.clone().to(device=device)]
            for p in self.get_input_embeddings().parameters():
                p.requires_grad = True
            for p in self.get_output_embeddings().parameters():
                p.requires_grad = False

    # 设置图像patch token的ID
    self.model.vision_config.im_patch_token = tokenizer.convert_tokens_to_ids(
        [DEFAULT_IMAGE_PATCH_TOKEN])[0]
    print(f'Tokenizer: {tokenizer}\n patch_token_id: {self.model.vision_config.im_patch_token}, visoin_config: {self.model.vision_config}', flush=True)
```

- 该方法用于初始化视觉tokenizer，添加特殊token并调整嵌入。

---

### **8. 注册模型**
```python
AutoConfig.register("omnilmm", OmniLMMConfig)  # 注册OmniLMMConfig
AutoModelForCausalLM.register(OmniLMMConfig, OmniLMMForCausalLM)  # 注册OmniLMMForCausalLM
```
- 将OmniLMM模型注册到Hugging Face的自动配置和模型工厂中。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)