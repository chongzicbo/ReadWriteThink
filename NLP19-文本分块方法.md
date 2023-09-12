# 定长分块

这是最常见、最直接的分块方法：我们只需决定一个文本块中的分词数量，以及选择性地决定文本块之间可否进行交叠(overlap)。一般来说，我们希望在文本块块之间保留一些交叠，这样可以防止上下文的语义不会丢失。大多数常见情况下，定长分块是最佳路径。与其他形式的分块相比，定长分块计算成本低且易于使用，因为不需要使用任何NLP库。

以下是使用[LangChain](https://link.juejin.cn/?target=https%3A%2F%2Fapi.python.langchain.com%2Fen%2Flatest%2Fapi_reference.html)执行定长分块的示例：

```
text = "..." # your text
from langchain.text_splitter import CharacterTextSplitter
text_splitter = CharacterTextSplitter(
    separator = "\n\n",
    chunk_size = 256,
    chunk_overlap  = 20
)
docs = text_splitter.create_documents([text])
```



# 按句分块

之前提到过，很多嵌入模型针对句子的嵌入进行了优化。自然地，就按照一个句子一个分场进行切分，有多种方法和工具可用于执行此操作，包括：

- **简单切分：** 最简单的方法是按句号（“。”）和换行符切分成多个句子。虽然快速简单，但这种方法不会考虑所有可能的边界情况。一个非常简单的例子：

  ```
  text = "..." # your text
  docs = text.split("。")
  ```

- [**NLTK**](https://link.juejin.cn/?target=https%3A%2F%2Fwww.nltk.org%2F)：自然语言工具包 (NLTK) 是一个流行的Python库，用于处理人类的语言数据。它提供了一个句子分词器(tokenizer)，可以将文本分割成句子，帮助创建更有意义的文本块。如果想结合LangChain使用NLTK可以执行以下操作：

```
text = "..." # your text
from langchain.text_splitter import NLTKTextSplitter
text_splitter = NLTKTextSplitter()
docs = text_splitter.split_text(text)
```

- [**spaCy**](https://link.juejin.cn/?target=https%3A%2F%2Fspacy.io%2F)：spaCy是另一个执行NLP任务的强大Python库。它提供了复杂的句子切分功能，可有效地将文本分割成单独的句子，从而在生成的文本块中更好地保留上下文。如果想结合LangChain使用spaCy可以执行以下操作：

```
text = "..." # your text
from langchain.text_splitter import SpacyTextSplitter
text_splitter = SpaCyTextSplitter()
docs = text_splitter.split_text(text)
```

# 递归分块

递归分块使用一组分隔符以层级和迭代的方式将输入文本切分为更小的文本块。如果切分文本的初始尝试没有生成所需大小或结构的块，则该方法会使用不同的分隔符或标准在样新生成的文本块上反复使用当前的方法，直到达到所需文本块的大小或结构。这意味着虽然文本块的大小不会完全相同，但依然“渴望”形成相似的大小。

以下是结合LangChain使用递归分块的例子：

```
text = "..." # your text
from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
    # Set a really small chunk size, just to show.
    chunk_size = 256,
    chunk_overlap  = 20
)

docs = text_splitter.create_documents([text])
```

# 特定分块

Markdown和LaTeX是可能遇到的结构化和格式化内容的两个示例。在这些情况下，您可以使用特定的方法在分块过程中保留内容的原始结构。

- [**Markdown**](https://link.juejin.cn/?target=https%3A%2F%2Fwww.markdownguide.org%2F)：Markdown是一种轻量级标记语言，通常用于格式化文本。通过识别Markdown语法（例如标题、列表和代码块），可以根据内容结构和层级体系智能地切分内容，从而产生语义上更连贯的文本块。例如：

```
from langchain.text_splitter import MarkdownTextSplitter
markdown_text = "..."

markdown_splitter = MarkdownTextSplitter(chunk_size=100, chunk_overlap=0)
docs = markdown_splitter.create_documents([markdown_text])
```

- [**LaTex**](https://link.juejin.cn/?target=https%3A%2F%2Fwww.latex-project.org%2F)：LaTeX是一种文档系统和标记语言，通常用于学术论文和技术文档。通过解析LaTeX命令和环境，可以创建尊重内容逻辑组织的文本块（例如，章节、小节和方程），从而获得更准确、与上下文相关的结果。例如：

```
from langchain.text_splitter import LatexTextSplitter
latex_text = "..."
latex_splitter = LatexTextSplitter(chunk_size=100, chunk_overlap=0)
docs = latex_splitter.create_documents([latex_text])
```

# 基于聚类的分块

## **KMeans聚类**

KMeans聚类是一种基于语义相似度对句子进行分组的技术。通过使用句子嵌入和K-means等聚类算法，我们可以实现句子聚类。

```
from sentence_transformers import SentenceTransformer
from sklearn.cluster import KMeans

# Load the Sentence Transformer model
model = SentenceTransformer('all-MiniLM-L6-v2')

# Define a list of sentences (your text data)
sentences = ["This is an example sentence.", "Another sentence goes here.", "..."]

# Generate embeddings for the sentences
embeddings = model.encode(sentences)

# Choose an appropriate number of clusters (here we choose 5 as an example)
num_clusters = 3

# Perform K-means clustering
kmeans = KMeans(n_clusters=num_clusters)
clusters = kmeans.fit_predict(embeddings)
```

你可以在这里看到聚类对句子列表的步骤是：

1. 加载句子转换模型。在本例中，我们将使用HuggingFace的sentence-transformers/all-MiniLM-L6-v2中的all-MiniLM-L6-v2模型。

2. 定义句子并使用模型中的encode()方法生成它们的嵌入。

3. 定义聚类技术和聚类数量(这里我们使用3个聚类的KMeans)，最后将其拟合到数据集中。

KMeans聚类虽然有益，但也有一些明显的缺点。主要的限制包括：

1. **句子顺序丢失：**KMeans聚类不保留句子的原始顺序，这可能会扭曲叙事的自然流程。这很重要。

2. **计算效率：**KMeans可能是计算密集型的，并且速度很慢，特别是在使用大型文本语料库或使用大量集群时。对于实时应用程序或处理大数据来说，这可能是一个重大缺陷。

## **Adjacent Sentences聚类**

为了克服KMeans聚类的一些限制，特别是句子顺序的丢失，一种替代方法是基于语义相似度对Adjacent Sentences进行聚类。这种方法的基本前提是，在文本中连续出现的两个句子比相隔较远的两个句子更有可能在语义上相关。

```
import numpy as np
import spacy

# Load the Spacy model
nlp = spacy.load('en_core_web_sm')

def process(text):
    doc = nlp(text)
    sents = list(doc.sents)
    vecs = np.stack([sent.vector / sent.vector_norm for sent in sents])

    return sents, vecs

def cluster_text(sents, vecs, threshold):
    clusters = [[0]]
    for i in range(1, len(sents)):
        if np.dot(vecs[i], vecs[i-1]) < threshold:
        clusters.append([])
        clusters[-1].append(i)
 
    return clusters

def clean_text(text):
    # Add your text cleaning process here
    return text

# Initialize the clusters lengths list and final texts list
clusters_lens = []
final_texts = []

# Process the chunk
threshold = 0.3
sents, vecs = process(text)

# Cluster the sentences
clusters = cluster_text(sents, vecs, threshold)

for cluster in clusters:
    cluster_txt = clean_text(' '.join([sents[i].text for i in cluster]))
    cluster_len = len(cluster_txt)
 
    # Check if the cluster is too short
    if cluster_len < 60:
        continue
 
    # Check if the cluster is too long
    elif cluster_len > 3000:
        threshold = 0.6
        sents_div, vecs_div = process(cluster_txt)
        reclusters = cluster_text(sents_div, vecs_div, threshold)
 
        for subcluster in reclusters:
            div_txt = clean_text(' '.join([sents_div[i].text for i in subcluster]))
            div_len = len(div_txt)
 
            if div_len < 60 or div_len > 3000:
                continue
 
            clusters_lens.append(div_len)
            final_texts.append(div_txt)
 
 else:
     clusters_lens.append(cluster_len)
     final_texts.append(cluster_txt)
```

这段代码的关键要点：

1. **文本处理：**将每个文本块传递给处理函数。该函数使用SpaCy库创建句子嵌入，句子嵌入用于表示文本块中每个句子的语义。

2. **聚类创建：**cluster_text函数根据其嵌入的余弦相似性来形成句子的聚类。如果余弦相似度小于指定的阈值，则开始一个新的群集。

3. **长度检查：**代码检查每个簇的长度。如果簇太短(少于60个字符)或太长(超过3000个字符)，则调整阈值，并针对特定集群重复该过程，直到达到可接受的长度。

Adjacent Sentences聚类方法具有独特的优点：

1. **语境连贯：**通过考虑语义和语境的连贯，生成主题一致的语块。

2. **灵活性：**平衡上下文保存和计算效率，提供可调整的块大小。

3. **阈值调整：**允许用户根据自己的需要，通过调整相似度阈值来微调分块过程。

4. **序列保存：**保留文本中句子的原始顺序，这对于顺序语言模型和文本顺序重要的任务至关重要。