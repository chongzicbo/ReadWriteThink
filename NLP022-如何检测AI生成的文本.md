目前文本生成是人工智能领域发展快速且最令人兴奋的领域之一。人工智能模型如GPT-3、Bloom、BERT‘AlexalTM以及其它的模型，可以生成非常像人类的文本，这让人兴奋又担忧。这样的技术进步使我们能够以前所未有的方式发挥创造力。尽管如此，它们也产生了欺骗。这些模型越好，区分人类书写的文本和模型生成的文本就会更加困难。

自ChatGPT发布以来，全球各地的人们一直在测试这种人工智能模型的局限性，并使用它们来获得知识，但对于一些学生来说，还可以解决家庭作业和考试问题，这对这种技术的伦理影响提出了挑战。特别是当这些模型已经变得足够复杂，可以模仿人类的写作风格，并在多个段落中保持上下文时，它们仍然需要修复，即使它们的错误很小。

这提出了一个重要的问题，我的朋友和家人经常问我这个问题（自从ChatGPT发布以来，我被问了很多次这个问题…），

> 我们怎么能知道一个文本是人工编写的还是人工智能生成的？

这个问题对研究界来说并不新鲜；检测人工智能生成的文本，我们称之为“深度伪文本检测”。如今，有不同的工具可以用来检测文本是人工编写的还是人工智能生成，例如OpenAI的GPT-2。但是这些工具是如何工作的呢？

目前使用不同的方法来检测人工智能生成的文本；随着用于生成这些文本的模型越来越先进，人们正在研究和实施新的技术来检测这些文本。

本文将探讨5种不同的统计方法，可用于检测人工智能生成的文本。

# 1.N-gram 分析

N-gram是来自给定文本样本的N个单词或标记的序列。N-gram中的“N”是N-gram中有多少单词。例如：

- New York (2-gram).
- The Three Musketeers (3-gram).
- The group met regularly (4-gram).

通过分析文本中不同N-gram的频率，可以确定模式。例如，在我们刚刚介绍的三个N-gram示例中，第一个是最常见的，第三个是最不常见的。通过跟踪不同的N-gram，我们可以确定它们在人工智能生成的文本中或多或少比在人类书写的文本中更常见。例如，人工智能可能比人类作家更频繁地使用特定的短语或单词组合。通过在人类和人工智能生成的数据上训练我们的模型，我们可以找到人工智能与人类使用的N-gram频率之间的关系。

# 2.**Perplexity (困惑度)**

如果你在英语词典中查找“困惑”一词，它会被定义为惊讶或震惊，但特别是在人工智能和NLP的背景下，“困惑”衡量的是语言模型预测文本的自信程度。通过量化模型需要对新文本做出回应的时间，或者换句话说，模型对新文本的“惊讶”程度，来估计模型的困惑程度。例如，人工智能生成的文本可能会降低模型的困惑；模型对文本的预测就越好。困惑的计算速度很快，这使它比其他方法具有优势。

# 3.Burstiness(突发度)

在NLP中，Slava Katz将突发定义为某些单词出现在文档或一组文档中的“突发”现象。这个想法是，当一个单词在文档中使用一次时，它很可能在同一文档中再次使用。人工智能生成的文本表现出与人类书写的文本不同的突发模式，因为它们没有选择其他同义词所需的认知过程。

# 4.Stylometry(风格)

风格学是对语言风格的研究，它可以用来识别作者，或者在这种情况下，识别文本的来源（人类与人工智能）。每个人都使用语言。不同的是，有些人喜欢短句，有些人更喜欢连词的长句。不同的人使用分号和长破折号（以及其他独特的标点符号）的方式不同。此外，有些人使用被动语态多于主动语态，或者使用更复杂的词汇。人工智能生成的文本可能表现出不同的风格特征，甚至不止一次地就同一主题写作。由于人工智能没有风格，这些不同的风格可以用来检测人工智能是否写了文本。

# 5.Consistency (一致性)和Coherence(连贯性)分析

继风格之后，由于人工智能模型没有自己的风格，它们生成的文本有时需要更大的一致性和长期连贯性。例如，人工智能可能会自相矛盾，或者在文本中间突然改变主题和风格，从而导致更难以理解的思想流。



```python
import nltk
from collections import Counter
from sklearn.feature_extraction.text import CountVectorizer
from textblob import TextBlob
import numpy as np

text = "This is some sample text to detect. This is a text written by a human? or is it?!"

# 1. N-gram Analysis
def ngram_analysis(text, n=2):
    n_grams = nltk.ngrams(text.split(), n)
    freq_dist = nltk.FreqDist(n_grams)
    print(freq_dist)

ngram_analysis(text)

# 2. Perplexity
# Perplexity calculation typically requires a language model
# This is just an example of how perplexity might be calculated given a probability distribution
def perplexity(text, model=None):
    # This is a placeholder. In practice, we would use the model to get the probability of each word
    prob_dist = nltk.FreqDist(text.split())
    entropy = -1 * sum([p * np.log(p) for p in prob_dist.values()])
    return np.exp(entropy)

print(perplexity(text))

# 3. Burstiness
def burstiness(text):
    word_counts = Counter(text.split())
    burstiness = len(word_counts) / np.std(list(word_counts.values()))
    return burstiness

print(burstiness(text))

# 4. Stylometry
def stylometry(text):
    blob = TextBlob(text)
    avg_sentence_length = sum(len(sentence.words) for sentence in blob.sentences) / len(blob.sentences)
    passive_voice = text.lower().count('was') + text.lower().count('were')
    vocabulary_richness = len(set(text.split())) / len(text.split())
    return avg_sentence_length, passive_voice, vocabulary_richness

print(stylometry(text))

# 5. Consistency and Coherence Analysis
# Again, this is a simple way to explain the idea of calculating the consistency of a text. In reality, more complex algorithms are used.
def consistency(text):
    sentences = text.split(".")
    topics = [sentence.split()[0] for sentence in sentences if sentence]
    topic_changes = len(set(topics))
    return topic_changes

print(consistency(text))
```

## 

>  原文  [How Do We Know if a Text Is AI-generated? | by Sara A. Metwalli | May, 2023 | Towards Data Science (medium.com)](https://medium.com/towards-data-science/how-do-we-know-if-a-text-is-ai-generated-82e710ea7b51)

