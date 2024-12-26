---
title: 'llama3源码解析-02：tokenizer模块解析'
categories:
  - [人工智能,nlp,llm]
tags:
  - nlp
  - llm
  - llama
---

![llama](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/Llama3_Repo.jpeg)

## Tokenizer类

以下是 `tokenizer.py` 中 `Tokenizer` 类的逐行详细解释：

```python
class Tokenizer:
    """
    Tokenizing and encoding/decoding text using the Tiktoken tokenizer.
    """
    # 特殊 token 的字典，用于存储特殊 token 及其对应的 token ID
    special_tokens: Dict[str, int]

    # 保留的特殊 token 数量，默认为 256
    num_reserved_special_tokens = 256

    # 正则表达式模式，用于匹配文本中的子词和特殊字符
    pat_str = r"(?i:'s|'t|'re|'ve|'m|'ll|'d)|[^\r\n\p{L}\p{N}]?\p{L}+|\p{N}{1,3}| ?[^\s\p{L}\p{N}]+[\r\n]*|\s*[\r\n]+|\s+(?!\S)|\s+"  # noqa: E501

    def __init__(self, model_path: str):
        """
        Initializes the Tokenizer with a Tiktoken model.

        Args:
            model_path (str): The path to the Tiktoken model file.
        """
        # 检查模型文件是否存在
        assert os.path.isfile(model_path), model_path

        # 加载 Tiktoken 模型的 BPE（Byte Pair Encoding）合并表
        mergeable_ranks = load_tiktoken_bpe(model_path)
        # 获取基础 token 的数量
        num_base_tokens = len(mergeable_ranks)
        # 定义特殊 token 列表
        special_tokens = [
            "<|begin_of_text|>",
            "<|end_of_text|>",
            "<|reserved_special_token_0|>",
            "<|reserved_special_token_1|>",
            "<|reserved_special_token_2|>",
            "<|reserved_special_token_3|>",
            "<|start_header_id|>",
            "<|end_header_id|>",
            "<|reserved_special_token_4|>",
            "<|eot_id|>",  # end of turn
        ] + [
            # 生成剩余的保留特殊 token
            f"<|reserved_special_token_{i}|>"
            for i in range(5, self.num_reserved_special_tokens - 5)
        ]
        # 将特殊 token 映射到 token ID，ID 从基础 token 数量开始递增
        self.special_tokens = {
            token: num_base_tokens + i for i, token in enumerate(special_tokens)
        }
        # 初始化 Tiktoken 的 Encoding 对象
        self.model = tiktoken.Encoding(
            name=Path(model_path).name,
            pat_str=self.pat_str,
            mergeable_ranks=mergeable_ranks,
            special_tokens=self.special_tokens,
        )
        # 记录日志，表示模型已加载
        logger.info(f"Reloaded tiktoken model from {model_path}")

        # 获取词汇表大小
        self.n_words: int = self.model.n_vocab
        # 获取 BOS（Begin of Sequence）token 的 ID
        self.bos_id: int = self.special_tokens["<|begin_of_text|>"]
        # 获取 EOS（End of Sequence）token 的 ID
        self.eos_id: int = self.special_tokens["<|end_of_text|>"]
        # 设置 pad token 的 ID，默认为 -1
        self.pad_id: int = -1
        # 定义停止 token 集合，包含 EOS 和 EOT（End of Turn）token
        self.stop_tokens = {
            self.special_tokens["<|end_of_text|>"],
            self.special_tokens["<|eot_id|>"],
        }
        # 记录词汇表大小、BOS ID 和 EOS ID
        logger.info(
            f"#words: {self.n_words} - BOS ID: {self.bos_id} - EOS ID: {self.eos_id}"
        )

    def encode(
        self,
        s: str,
        *,
        bos: bool,
        eos: bool,
        allowed_special: Union[Literal["all"], AbstractSet[str]] = set(),
        disallowed_special: Union[Literal["all"], Collection[str]] = (),
    ) -> List[int]:
        """
        Encodes a string into a list of token IDs.

        Args:
            s (str): The input string to be encoded.
            bos (bool): Whether to prepend the beginning-of-sequence token.
            eos (bool): Whether to append the end-of-sequence token.
            allowed_tokens ("all"|set[str]): allowed special tokens in string
            disallowed_tokens ("all"|set[str]): special tokens that raise an error when in string

        Returns:
            list[int]: A list of token IDs.
        """
        # 确保输入是字符串
        assert type(s) is str

        # Tiktoken 分词器单次编码的最大字符数
        TIKTOKEN_MAX_ENCODE_CHARS = 400_000

        # 最大连续非空格或空格字符数
        MAX_NO_WHITESPACES_CHARS = 25_000

        # 将输入字符串分割为子串，确保每个子串不超过最大字符数
        substrs = (
            substr
            for i in range(0, len(s), TIKTOKEN_MAX_ENCODE_CHARS)
            for substr in self._split_whitespaces_or_nonwhitespaces(
                s[i : i + TIKTOKEN_MAX_ENCODE_CHARS], MAX_NO_WHITESPACES_CHARS
            )
        )
        # 初始化 token ID 列表
        t: List[int] = []
        # 对每个子串进行编码
        for substr in substrs:
            t.extend(
                self.model.encode(
                    substr,
                    allowed_special=allowed_special,
                    disallowed_special=disallowed_special,
                )
            )
        # 如果需要，在开头添加 BOS token
        if bos:
            t.insert(0, self.bos_id)
        # 如果需要，在结尾添加 EOS token
        if eos:
            t.append(self.eos_id)
        # 返回编码后的 token ID 列表
        return t

    def decode(self, t: Sequence[int]) -> str:
        """
        Decodes a list of token IDs into a string.

        Args:
            t (List[int]): The list of token IDs to be decoded.

        Returns:
            str: The decoded string.
        """
        # 将 token ID 序列解码为字符串
        return self.model.decode(cast(List[int], t))

    @staticmethod
    def _split_whitespaces_or_nonwhitespaces(
        s: str, max_consecutive_slice_len: int
    ) -> Iterator[str]:
        """
        Splits the string `s` so that each substring contains no more than `max_consecutive_slice_len`
        consecutive whitespaces or consecutive non-whitespaces.
        """
        # 初始化当前子串的长度和类型
        current_slice_len = 0
        current_slice_is_space = s[0].isspace() if len(s) > 0 else False
        slice_start = 0

        # 遍历字符串，按空格或非空格进行分割
        for i in range(len(s)):
            is_now_space = s[i].isspace()

            if current_slice_is_space ^ is_now_space:
                current_slice_len = 1
                current_slice_is_space = is_now_space
            else:
                current_slice_len += 1
                if current_slice_len > max_consecutive_slice_len:
                    yield s[slice_start:i]
                    slice_start = i
                    current_slice_len = 1
        yield s[slice_start:]
```

### 逐行解释总结
1. **特殊 token 处理**:
   - `special_tokens` 字典存储特殊 token 及其对应的 token ID。
   - `num_reserved_special_tokens` 定义了保留的特殊 token 数量。
   - `pat_str` 是正则表达式模式，用于匹配文本中的子词和特殊字符。

2. **初始化方法 (`__init__`)**:
   - 加载 Tiktoken 模型的 BPE 合并表，并初始化特殊 token。
   - 设置 BOS、EOS 和 pad token 的 ID，并定义停止 token 集合。
   - 初始化 Tiktoken 的 `Encoding` 对象，用于实际的编码和解码操作。

3. **编码方法 (`encode`)**:
   - 将输入字符串分割为子串，确保每个子串不超过最大字符数。
   - 对每个子串进行编码，并根据参数决定是否添加 BOS 和 EOS token。
   - 返回编码后的 token ID 列表。

4. **解码方法 (`decode`)**:
   - 将 token ID 序列解码为字符串。

5. **子串分割方法 (`_split_whitespaces_or_nonwhitespaces`)**:
   - 将字符串按空格或非空格进行分割，确保每个子串不超过最大连续字符数。
   
   - ### **什么保留空格？**
   
     1. **语义完整性**:
        - 空格在文本中用于分隔单词、标点符号等，是文本结构的重要组成部分。
        - 如果丢弃空格，分词结果可能会将多个单词合并，导致语义错误。例如：
          - 输入: `"Hello, how are you?"`
          - 丢弃空格: `"Hello,howareyou?"`（分词结果错误）
          - 保留空格: `"Hello, how are you?"`（分词结果正确）
     2. **特殊 token 处理**:
        - 某些特殊 token（如 `<|end_of_text|>`）可能被空格包围，保留空格可以确保这些特殊 token 被正确识别和处理。
     3. **模型输入格式**:
        - 许多语言模型（如 GPT）的输入需要保留空格，以确保生成的文本格式正确。

### 对输入字符串进行tokenize的详细流程

结合一个具体的字符串示例，详细说明 `Tokenizer` 如何处理输入字符串。假设我们已经有一个 Tiktoken 模型文件，并且输入字符串为：

```
"Hello, how are you? <|end_of_text|>"
```

我们将逐步分析 `Tokenizer` 如何处理这个字符串。

---

#### **1. 输入字符串**
```python
s = "Hello, how are you? <|end_of_text|>"
```

---

#### **2. 参数设置**
假设调用 `encode` 方法时，参数如下：
```python
bos = True  # 添加 BOS token
eos = False  # 不添加 EOS token
allowed_special = {"<|end_of_text|>"}  # 允许的特殊 token
disallowed_special = ()  # 不允许的特殊 token（为空）
```

---

#### **3. 输入验证**
- **类型检查**:
  ```python
  assert type(s) is str
  ```
  输入是一个字符串，检查通过。

---

#### **4. 字符串分割**
由于输入字符串较短（远小于 `TIKTOKEN_MAX_ENCODE_CHARS = 400_000`），不需要按字符数分割。但为了演示，我们假设字符串较长，需要按空格和非空格进行分割。

##### **分割逻辑**
- 输入字符串: `"Hello, how are you? <|end_of_text|>"`
- 按空格和非空格分割：
  - `"Hello,"`（非空格）
  - `" how "`（空格）
  - `"are"`（非空格）
  - `" you? "`（空格）
  - `"<|end_of_text|>"`（非空格）

##### **分割结果**
```python
substrs = ["Hello,", " how ", "are", " you? ", "<|end_of_text|>"]
```

---

#### **5. 子串编码**
对每个子串进行编码，生成 token ID 序列。

##### **假设的 Tiktoken 模型**
假设 Tiktoken 模型的词汇表和编码规则如下：
- `"Hello"` → `[15496]`
- `","` → `[11]`
- `" how"` → `[703]`
- `"are"` → `[527]`
- `" you"` → `[366]`
- `"?"` → `[30]`
- `"<|end_of_text|>"` → `[50256]`（特殊 token）

##### **编码过程**
1. **子串 `"Hello,"`**:
   - 编码为 `[15496, 11]`。
2. **子串 `" how "`**:
   - 编码为 `[703]`。
3. **子串 `"are"`**:
   - 编码为 `[527]`。
4. **子串 `" you? "`**:
   - 编码为 `[366, 30]`。
5. **子串 `"<|end_of_text|>"`**:
   - 这是一个特殊 token，编码为 `[50256]`。

##### **编码结果**
```python
t = [15496, 11, 703, 527, 366, 30, 50256]
```

---

#### **6. 添加特殊 token**
根据参数设置，需要在开头添加 BOS token，不添加 EOS token。

##### **BOS token**
假设 BOS token 的 ID 为 `50257`：
```python
if bos:
    t.insert(0, 50257)
```

##### **更新后的 token 序列**
```python
t = [50257, 15496, 11, 703, 527, 366, 30, 50256]
```

---

#### **7. 返回编码结果**
最终的 token ID 序列为：
```python
[50257, 15496, 11, 703, 527, 366, 30, 50256]
```

---

#### **8. 解码过程**
如果需要将 token ID 序列解码回字符串，可以使用 `decode` 方法。

##### **解码逻辑**
- 将每个 token ID 映射回对应的子词或字符。
- 特殊 token（如 `<|end_of_text|>`）会被解码为原始字符串。

##### **解码结果**
```python
decoded_str = "<|begin_of_text|>Hello, how are you? <|end_of_text|>"
```

---

#### **9. 总结**
结合具体示例，`Tokenizer` 处理输入字符串的详细流程如下：
1. **输入字符串**: `"Hello, how are you? <|end_of_text|>"`
2. **分割字符串**: 按空格和非空格分割为 `["Hello,", " how ", "are", " you? ", "<|end_of_text|>"]`。
3. **编码子串**: 将每个子串编码为 token ID 序列 `[15496, 11, 703, 527, 366, 30, 50256]`。
4. **添加 BOS token**: 在开头插入 BOS token，得到 `[50257, 15496, 11, 703, 527, 366, 30, 50256]`。
5. **返回结果**: 返回最终的 token ID 序列。
6. **解码**: 将 token ID 序列解码回原始字符串 `"<|begin_of_text|>Hello, how are you? <|end_of_text|>"`。

通过这种分步骤的处理方式，`Tokenizer` 能够高效地将输入字符串转换为模型可处理的 token ID 序列，并支持特殊 token 的灵活处理。

### 总结

`Tokenizer` 类负责文本的编码和解码，使用 Tiktoken 作为底层分词器。它支持特殊 token 的处理，并提供了灵活的编码和解码接口。通过 `encode` 方法，可以将文本转换为 token ID 序列；通过 `decode` 方法，可以将 token ID 序列转换回文本。此外，`Tokenizer` 还提供了对长文本的分割功能，确保编码过程不会因输入过长而失败。



## ChatFormat类

`ChatFormat` 类的主要功能是将对话消息（`Message`）编码为模型可以理解的 token 序列，特别适用于对话生成任务。

---

### **代码注释**

```python
class ChatFormat:
    def __init__(self, tokenizer: Tokenizer):
        """
        初始化 ChatFormat 类。

        Args:
            tokenizer (Tokenizer): 用于编码和解码的 Tokenizer 实例。
        """
        self.tokenizer = tokenizer
```

- **功能**: 初始化 `ChatFormat` 类，绑定一个 `Tokenizer` 实例，用于后续的编码操作。
- **参数**:
  - `tokenizer`: 一个 `Tokenizer` 对象，用于将文本转换为 token ID 序列。

---

```python
    def encode_header(self, message: Message) -> List[int]:
        """
        编码消息头，包括角色信息和分隔符。

        Args:
            message (Message): 包含角色和内容的消息字典。

        Returns:
            List[int]: 编码后的 token ID 序列。
        """
        tokens = []
        # 添加消息头开始标记
        tokens.append(self.tokenizer.special_tokens["<|start_header_id|>"])
        # 编码角色信息（如 "user" 或 "assistant"）
        tokens.extend(self.tokenizer.encode(message["role"], bos=False, eos=False))
        # 添加消息头结束标记
        tokens.append(self.tokenizer.special_tokens["<|end_header_id|>"])
        # 添加换行符（两个换行符，用于分隔消息头和内容）
        tokens.extend(self.tokenizer.encode("\n\n", bos=False, eos=False))
        return tokens
```

- **功能**: 编码消息头，包括角色信息和分隔符。
- **参数**:
  - `message`: 一个 `Message` 字典，包含 `role`（角色）和 `content`（内容）。
- **返回值**: 编码后的 token ID 序列。
- **详细步骤**:
  1. 添加消息头开始标记 `<|start_header_id|>`。
  2. 编码角色信息（如 `"user"` 或 `"assistant"`）。
  3. 添加消息头结束标记 `<|end_header_id|>`。
  4. 添加两个换行符 `\n\n`，用于分隔消息头和内容。

---

```python
    def encode_message(self, message: Message) -> List[int]:
        """
        编码整个消息，包括消息头和内容。

        Args:
            message (Message): 包含角色和内容的消息字典。

        Returns:
            List[int]: 编码后的 token ID 序列。
        """
        # 编码消息头
        tokens = self.encode_header(message)
        # 编码消息内容，并去除首尾空白字符
        tokens.extend(
            self.tokenizer.encode(message["content"].strip(), bos=False, eos=False)
        )
        # 添加消息结束标记 <|eot_id|>
        tokens.append(self.tokenizer.special_tokens["<|eot_id|>"])
        return tokens
```

- **功能**: 编码整个消息，包括消息头和内容。
- **参数**:
  - `message`: 一个 `Message` 字典，包含 `role`（角色）和 `content`（内容）。
- **返回值**: 编码后的 token ID 序列。
- **详细步骤**:
  1. 调用 `encode_header` 方法，编码消息头。
  2. 编码消息内容，并去除首尾空白字符。
  3. 添加消息结束标记 `<|eot_id|>`，表示当前消息的结束。

---

```python
    def encode_dialog_prompt(self, dialog: Dialog) -> List[int]:
        """
        编码整个对话，生成模型输入的 token 序列。

        Args:
            dialog (Dialog): 对话列表，包含多条消息。

        Returns:
            List[int]: 编码后的 token ID 序列。
        """
        tokens = []
        # 添加对话开始标记 <|begin_of_text|>
        tokens.append(self.tokenizer.special_tokens["<|begin_of_text|>"])
        # 编码每条消息
        for message in dialog:
            tokens.extend(self.encode_message(message))
        # 添加助手消息的开始标记，供模型生成回复
        tokens.extend(self.encode_header({"role": "assistant", "content": ""}))
        return tokens
```

- **功能**: 编码整个对话，生成模型输入的 token 序列。
- **参数**:
  - `dialog`: 一个 `Dialog` 列表，包含多条消息。
- **返回值**: 编码后的 token ID 序列。
- **详细步骤**:
  1. 添加对话开始标记 `<|begin_of_text|>`。
  2. 遍历对话中的每条消息，调用 `encode_message` 方法进行编码。
  3. 添加助手消息的开始标记（包括角色信息和分隔符），供模型生成回复。

---

### **总结**

#### **功能**
`ChatFormat` 类的主要功能是将对话消息（`Message`）编码为模型可以理解的 token 序列。它特别适用于对话生成任务，能够处理多轮对话，并生成符合模型输入格式的 token 序列。

#### **核心方法**
1. **`encode_header`**:
   - 编码消息头，包括角色信息和分隔符。
   - 用于标识每条消息的角色（如 `"user"` 或 `"assistant"`）。

2. **`encode_message`**:
   - 编码整个消息，包括消息头和内容。
   - 在消息末尾添加结束标记 `<|eot_id|>`，表示当前消息的结束。

3. **`encode_dialog_prompt`**:
   - 编码整个对话，生成模型输入的 token 序列。
   - 在对话末尾添加助手消息的开始标记，供模型生成回复。

#### **特殊 token**
- `<|begin_of_text|>`: 对话开始标记。
- `<|start_header_id|>`: 消息头开始标记。
- `<|end_header_id|>`: 消息头结束标记。
- `<|eot_id|>`: 消息结束标记。

#### **适用场景**
- **对话生成**: 将多轮对话编码为模型输入，生成助手的回复。
- **消息格式化**: 确保每条消息的格式符合模型的要求，包括角色信息和内容分隔符。

#### **示例**
假设有以下对话：
```python
dialog = [
    {"role": "user", "content": "Hello, how are you?"},
    {"role": "assistant", "content": "I'm fine, thank you!"},
]
```
调用 `encode_dialog_prompt(dialog)` 后，生成的 token 序列可能如下：
```python
[
    <|begin_of_text|>,
    <|start_header_id|>, "user", <|end_header_id|>, "\n\n", "Hello, how are you?", <|eot_id|>,
    <|start_header_id|>, "assistant", <|end_header_id|>, "\n\n", "I'm fine, thank you!", <|eot_id|>,
    <|start_header_id|>, "assistant", <|end_header_id|>, "\n\n"
]
```

#### **总结**
`ChatFormat` 类通过定义清晰的对话格式和特殊 token，确保对话消息能够被模型正确理解和处理。它是对话生成任务中不可或缺的一部分，能够有效提升模型生成回复的准确性和连贯性。

## **Message** 实例解析

结合实际的 `Message` 输入，详细说明 `Tokenizer` 和 `ChatFormat` 两个类如何协同工作，从 `Message` 输入到输出的完整流程。我们将重点关注 `encode_message`、`decode_message` 和 `encode_dialog_prompt` 方法，并明确区分 `encode_message` 和 `encode_dialog_prompt` 的输入和输出。

---

### **1. 输入数据**
假设我们有以下 `Message` 和 `Dialog` 输入：

#### **单个消息（Message）**
```python
message = {
    "role": "user",
    "content": "Hello, how are you?"
}
```

#### **多轮对话（Dialog）**
```python
dialog = [
    {"role": "user", "content": "Hello, how are you?"},
    {"role": "assistant", "content": "I'm fine, thank you!"},
]
```

---

### **2. Tokenizer 和 ChatFormat 初始化**
首先，我们需要初始化 `Tokenizer` 和 `ChatFormat` 类。

```python
# 假设模型文件路径为 "tiktoken_model.model"
tokenizer = Tokenizer(model_path="tiktoken_model.model")
chat_format = ChatFormat(tokenizer)
```

---

### **3. encode_message 的流程**
`encode_message` 方法用于编码单个消息，包括消息头和内容。

#### **输入**
```python
message = {
    "role": "user",
    "content": "Hello, how are you?"
}
```

#### **步骤**
1. **编码消息头**:
   - 调用 `encode_header` 方法，生成消息头的 token 序列。
   - 假设 `encode_header` 返回的 token 序列为：
     ```python
     [50258, 366, 50259, 11, 11]  # <|start_header_id|>, "user", <|end_header_id|>, "\n\n"
     ```

2. **编码消息内容**:
   - 调用 `Tokenizer.encode` 方法，编码消息内容 `"Hello, how are you?"`。
   - 假设返回的 token 序列为：
     ```python
     [15496, 11, 703, 527, 366, 30]  # "Hello, how are you?"
     ```

3. **添加消息结束标记**:
   - 添加 `<|eot_id|>` 标记，假设其 ID 为 `50256`。
   - 最终的 token 序列为：
     ```python
     [50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256]
     ```

#### **输出**
```python
encoded_message = chat_format.encode_message(message)
# encoded_message = [50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256]
```

---

### **4. decode_message 的流程**
`decode_message` 方法用于将 token 序列解码回原始消息。

#### **输入**
```python
encoded_message = [50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256]
```

#### **步骤**
1. **解码 token 序列**:
   - 调用 `Tokenizer.decode` 方法，将 token 序列解码为字符串。
   - 假设解码结果为：
     ```python
     "<|start_header_id|>user<|end_header_id|>\n\nHello, how are you?<|eot_id|>"
     ```

2. **提取消息内容**:
   - 从解码结果中提取消息内容 `"Hello, how are you?"`。

#### **输出**
```python
decoded_message = tokenizer.decode(encoded_message)
# decoded_message = "<|start_header_id|>user<|end_header_id|>\n\nHello, how are you?<|eot_id|>"
```

---

### **5. encode_dialog_prompt 的流程**
`encode_dialog_prompt` 方法用于编码整个对话，生成模型输入的 token 序列。

#### **输入**
```python
dialog = [
    {"role": "user", "content": "Hello, how are you?"},
    {"role": "assistant", "content": "I'm fine, thank you!"},
]
```

#### **步骤**
1. **添加对话开始标记**:
   - 添加 `<|begin_of_text|>` 标记，假设其 ID 为 `50257`。
   - 当前的 token 序列为：
     ```python
     [50257]
     ```

2. **编码每条消息**:
   - 对每条消息调用 `encode_message` 方法，生成 token 序列。
   - 假设第一条消息的 token 序列为：
     ```python
     [50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256]
     ```
   - 假设第二条消息的 token 序列为：
     ```python
     [50258, 527, 50259, 11, 11, 366, 30, 703, 527, 366, 30, 50256]
     ```
   - 将两条消息的 token 序列合并：
     ```python
     [50257, 50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256, 50258, 527, 50259, 11, 11, 366, 30, 703, 527, 366, 30, 50256]
     ```

3. **添加助手消息的开始标记**:
   - 添加助手消息的开始标记（包括角色信息和分隔符），供模型生成回复。
   - 假设生成的 token 序列为：
     ```python
     [50258, 527, 50259, 11, 11]
     ```
   - 最终的 token 序列为：
     ```python
     [50257, 50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256, 50258, 527, 50259, 11, 11, 366, 30, 703, 527, 366, 30, 50256, 50258, 527, 50259, 11, 11]
     ```

#### **输出**
```python
encoded_dialog = chat_format.encode_dialog_prompt(dialog)
# encoded_dialog = [50257, 50258, 366, 50259, 11, 11, 15496, 11, 703, 527, 366, 30, 50256, 50258, 527, 50259, 11, 11, 366, 30, 703, 527, 366, 30, 50256, 50258, 527, 50259, 11, 11]
```

---

### **6. 区分 encode_message 和 encode_dialog_prompt**
#### **encode_message**
- **输入**: 单个 `Message` 字典。
- **输出**: 编码后的 token 序列，包含消息头、内容和结束标记。
- **用途**: 用于编码单条消息。

#### **encode_dialog_prompt**
- **输入**: 一个 `Dialog` 列表，包含多条消息。
- **输出**: 编码后的 token 序列，包含对话开始标记、所有消息的编码以及助手消息的开始标记。
- **用途**: 用于编码整个对话，生成模型输入的 token 序列。

---

### **7. 总结**
通过 `Tokenizer` 和 `ChatFormat` 两个类的协同工作，我们可以将 `Message` 和 `Dialog` 编码为模型可以理解的 token 序列，并能够将 token 序列解码回原始文本。以下是完整的流程总结：

1. **单个消息编码**:
   - 使用 `encode_message` 方法，将 `Message` 编码为 token 序列。
   - 输出包含消息头、内容和结束标记。

2. **对话编码**:
   - 使用 `encode_dialog_prompt` 方法，将 `Dialog` 编码为 token 序列。
   - 输出包含对话开始标记、所有消息的编码以及助手消息的开始标记。

3. **解码**:
   - 使用 `Tokenizer.decode` 方法，将 token 序列解码回原始文本。

通过这种方式，`Tokenizer` 和 `ChatFormat` 能够高效地处理对话数据，为语言模型提供格式化的输入。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)