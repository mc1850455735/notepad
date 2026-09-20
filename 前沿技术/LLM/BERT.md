
BERT ( Bidirectional Encoder Representations from Transformers，基于 Transformer 的双向编码表示模型 ) 是 Google 提出的一种基于 Transformer Encoder 的预训练语言模型。它是 NLP 里非常经典的预训练语言模型，也是后来很多大模型和下游任务方法的重要基础。它的核心特点是**双向建模**，也就是说，在理解一个词时，BERT 会同时结合它左边和右边的上下文信息。BERT 先通过大规模无监督语料做预训练，主要任务包括 Masked Language Model 和 Next Sentence Prediction，然后再针对具体下游任务做微调。它在文本分类、命名实体识别、问答和句子匹配等任务上表现很好，是 NLP 预训练模型发展中的一个里程碑。

## BERT 为什么重要

在 BERT 之前，很多语言模型是**单向的**，比如：
- 从左到右看上下文
- 或从右到左看上下文

这样的问题是，对一个词的理解不够完整。而 BERT 的特点是它是双向的，可以同时利用左右上下文来表示当前词，结合前后的词，更准确理解词的含义。

## BERT 的模型结构

BERT 基于 Transformer Encoder，不是 Decoder。

这一点很常被问，你可以记住：

- BERT = Encoder-only  
- GPT = Decoder-only  

所以 BERT 更擅长：

- 理解类任务
- 表征学习
- 分类、抽取、匹配、阅读理解

而不像 GPT 那样更偏生成。

## BERT 的训练分两步

### 1. 预训练

#### MLM：Masked Language Model

随机把句子中的一些词遮住，让模型去预测被遮住的词。

例如：
```
I love [MASK] learning.
```

模型要预测 `[MASK] = deep`

这个任务让 BERT 学会利用双向上下文。

#### NSP：Next Sentence Prediction

给模型两句话，让它判断第二句话是否是第一句话的下一句。

这个任务原本是为了学习句子间关系。  
不过后来很多改进模型对 NSP 是否必要提出了不同看法。

### 2. 微调

预训练完成后，再把 BERT 应用到具体任务上进行微调，比如：

- 文本分类
- 情感分析
- 命名实体识别
- 句子匹配
- 阅读理解

BERT 先学通用语言知识，再通过少量任务数据适配具体应用。

## BERT 的输入形式

BERT 的输入一般由三部分 embedding 相加得到：

- Token Embedding
- Segment Embedding
- Position Embedding

其中常见特殊 token 有：

- `[CLS]`：放在句首，常用于分类任务的整体表示
- `[SEP]`：分隔两个句子
- `[MASK]`：预训练时用于遮盖词

## BERT 的优点

### 1. 双向上下文建模强

比传统单向语言模型更适合理解任务。

### 2. 预训练 + 微调范式很成功

推动了 NLP 从“任务专用模型”走向“通用预训练模型”。

### 3. 对很多下游任务效果好

特别是在分类、抽取、 匹配、问答。

## BERT 的局限

### 不擅长文本生成

因为它是 Encoder-only，不像 GPT 那样天然适合自回归生成。

### 2. 预训练目标和生成任务不完全一致

MLM 是“挖空填词”，而不是一步一步生成下一个词。

### 3. 长文本处理能力有限

原始 BERT 的输入长度通常有限，比如 512 token 左右。

### 4. 推理成本较高

尤其是大规模部署时，速度和资源消耗会是问题。

## BERT vs GPT

- BERT 是 Encoder-only，主要面向语言理解任务，采用双向建模
- GPT 是 Decoder-only，主要面向文本生成任务，采用自回归方式逐步预测下一个 token

BERT 更强在理解，GPT 更强在生成。

## 为什么 BERT 是双向的

你可以答：

因为 BERT 使用 MLM 预训练任务，不是按传统从左到右预测下一个词，而是通过预测被 mask 的词来学习上下文表示，因此可以同时利用左右信息。


我也可以接着帮你整理 **BERT、GPT、Transformer 三者区别** 的面试回答。