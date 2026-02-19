# Transformer 架构



## **三种神经网络的核心架构**

### 前馈神经网络（Feedforward Neural Network，FNN）

> 别名:**全连接前馈神经网络**
>
> 多层感知机 (Multilayer Perceptron, MLP)

每一层的神经元都和上下两层的每一个神经元完全连接

![图片描述](https://raw.githubusercontent.com/datawhalechina/happy-llm/main/docs/images/2-figures/1-0.png)

### 卷积神经网络（Convolutional Neural Network，CNN）

即训练参数量远小于全连接神经网络的卷积层来进行特征提取和学习

![图片描述](https://raw.githubusercontent.com/datawhalechina/happy-llm/main/docs/images/2-figures/1-1.png)

### 循环神经网络（Recurrent Neural Network，RNN）

能够使用历史信息作为输入、包含环和自重复的网络

![图片描述](https://raw.githubusercontent.com/datawhalechina/happy-llm/main/docs/images/2-figures/1-2.png)



******



## 架构演化

1. 曾经的霸主：RNN 和 LSTM

   在以前，处理文字（序列数据）的首选是 RNN（循环神经网络）。

   - **原理**：像人读书一样，一个字一个字地读，读了前面的才能理解后面的。
   - **代表作**：早期的预训练模型 **ELMo**。

2. 为什么 RNN 被淘汰了？（两个致命伤）

   尽管 RNN 模拟了人类的阅读顺序，但它有两个搞不定的问题：

   - **慢（无法并行计算）**：因为必须“按顺序读”，GPU 的强大算力使不上劲。就像排队打饭，前面的人没打完，后面的人只能等着。
   - **健忘（长距离遗忘）**：处理长文章时，读到结尾就忘了开头。虽然 LSTM（长短期记忆网络）加了“门控机制”来缓解，但效果依然有限。

3. 新王登基：Transformer 的崛起

为了解决上述问题，学者们发明了 **Transformer** 架构。

- **核心绝招**：**注意力机制（Attention）**。它不再强求顺序，而是让模型一眼望去就能看到全局，直接计算词与词之间的关联（比如代词“它”到底指代前文哪个词）。
- **结果**：它既能让 GPU **大规模并行计算**（速度飞快），又能精准捕捉**极远距离**的逻辑关系。



******



## 注意力机制

注意力机制有三个核心变量：**Query**（查询值）、**Key**（键值）和 **Value**（真值）

注意力机制就是一个**“根据需求（Query），在海量信息（Key-Value）中进行模糊匹配并提取重点”**的**加权求和**过程

**本质是相似度计算**

例如，当我们的 Query 为“fruit”，我们可以分别给三个 Key 赋予如下的权重：

```
{
    "apple":0.6,
    "banana":0.4,
    "chair":0
}
```

那么，我们最终查询到的值应该是：

​												value=0.6∗10+0.4∗5+0∗2=8    （key * 注意力分数 = value）

得出结论：对与不同的key分配不同的注意力分数，当key与Query相关性越高，赋予的注意力权重应当越大。关键在于如何找到合理计算注意力分数的方法



**词向量**： 通过合理的训练拟合，词向量能够表征语义信息，从而让语义相近的词在向量空间中距离更近，语义较远的词在向量空间中距离更远

> 语义相似的两个词对应的词向量的点积应该大于0，而语义不相似的词向量点积应该小于0



1. 那么，我们就可以用点积来计算词之间的相似度。假设我们的 Query 为“fruit”，对应的词向量为 $q$ ；我们的 Key 对应的词向量为 $k = [v_{apple} v_{banana} v_{chair}]$ ,则我们可以计算 Query 和每一个键的相似程度：

$$
x = qK^T
$$

2. 此处的 $K$ 即为将所有 Key 对应的词向量堆叠形成的矩阵。基于矩阵乘法的定义，$x$ 即为 $q$ 与每一个 $k$ 值的点积。现在我们得到的 $x$ 即反映了 Query 和每一个 Key 的相似程度，我们再通过一个 Softmax 层将其转化为和为 1 的权重：( Softmax即归一化函数)

$$
\text{softmax}(x)_i = \frac{e^{x_i}}{\sum_j e^{x_j}}
$$

3. 这样，得到的向量就能够反映 Query 和每一个 Key 的相似程度，同时又相加权重为 1，也就是我们的注意力分数了。最后，我们再将得到的注意力分数和值向量做对应乘积即可。根据上述过程，我们就可以得到注意力机制计算的基本公式：

$$
\text{attention}(Q, K, V) = \text{softmax}(qK^T)v
$$

4. 不过，此时的值还是一个标量，同时，我们此次只查询了一个 Query。我们可以将值转化为维度为 $d_v$ 的向量，同时一次性查询多个 Query，同样将多个 Query 对应的词向量堆叠在一起形成矩阵 Q，得到公式：

$$
\text{attention}(Q, K, V) = \text{softmax}(QK^T)V
$$

5. 目前，我们离标准的注意力机制公式还差最后一步。在上一个公式中，如果 Q 和 K 对应的维度 $d_k$ 比较大，softmax 放缩时就非常容易受影响，使不同值之间的差异较大，从而影响梯度的稳定性。因此，我们要将 Q 和 K 乘积的结果做一个放缩：

$$
\text{attention}(Q, K, V) = \text{softmax}(\frac{QK^T}{\sqrt{d_k}})V
$$

利用Pytorch可简单实现注意力机制代码

```python
'''注意力计算函数'''
def attention(query, key, value, dropout=None):
    '''
    args:
    query: 查询值矩阵
    key: 键值矩阵
    value: 真值矩阵
    '''
    # 获取键向量的维度，键向量的维度和值向量的维度相同
    d_k = query.size(-1) 
    # 计算Q与K的内积并除以根号dk
    # transpose——相当于转置
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    # Softmax
    p_attn = scores.softmax(dim=-1)
    if dropout is not None:
        p_attn = dropout(p_attn)
        # 采样
     # 根据计算结果对value进行加权求和
    return torch.matmul(p_attn, value), p_attn
```



### 自注意力

在编写代码实现时，**自注意力**与**普通注意力**在算法逻辑上是完全一样的，唯一的区别在于**输入源**

- **参数对应**：`attention(Q, K, V)` 函数通常接收三个参数。
- **输入相同**：在自注意力中，由于我们要计算的是“序列内部”的关系，所以 `Q`、`K`、`V` 全部由同一个原始输入 `x` 演化而来。
- **逻辑简化**：代码 `attention(x, x, x)` 形象地表达了：**“我用我自己（Q）去匹配我自己（K），最后从我自己（V）里提取特征。”**

虽然代码传参看似都是 `x`，但在 `attention` 函数内部，通常会通过三组**不同的线性层（Linear Layers）**对 `x` 进行变换：

1. $Q = x \cdot W^Q$
2. $K = x \cdot W^K$
3. $V = x \cdot W^V$

**总结：** 这一行代码强调的是**信息的来源是同一份数据**，而不是说 ![img](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==) 三个矩阵在数值上完全相等





### 掩码自注意力

给自注意力加了一个“眼罩”，让模型在处理序列时，只能看到当前时刻之前的词，而不能“偷看"未来的词"



**为什么要“掩码”？**

在训练 **Transformer 的解码器（Decoder）** 时，我们的目标是预测下一个词。

- **如果不加掩码**：模型在预测第 $i$ 个词时，由于自注意力的全局特性，它能直接“看到”后面已经给出的正确答案（第 $i+1,i+2,...$ 个词)，这会导致模型直接抄答案，学不到推理能力。
- **加入掩码后**：迫使模型只能根据已经生成的“历史信息”来预测未来，符合逻辑上的**因果律**



**为什么会需要掩码？**

Transformer 相对于 RNN 的核心优势之一即在于其可以并行计算

例如，如果待学习的文本序列是 【BOS】I like you【EOS】，那么，模型会按如下顺序进行预测和学习：

```
Step 1：输入 【BOS】，输出 I
Step 2：输入 【BOS】I，输出 like
Step 3：输入 【BOS】I like，输出 you
Step 4：输入 【BOS】I like you，输出 【EOS】
```

理论上来说，只要学习的语料足够多，通过上述的过程，模型可以学会任意一种文本序列的建模方式，也就是可以对任意的文本进行补全

我们可以发现，上述过程是一个串行的过程，也就是需要先完成 Step 1，才能做 Step 2

如果对于每一个训练语料，模型都需要串行完成上述过程才能完成学习，那么很明显没有做到并行计算，计算效率很低

因此：Transformer 就提出了掩码自注意力的方法。掩码自注意力会生成一串掩码，来遮蔽未来信息

待学习的文本序列仍然是 【BOS】I like you【EOS】，我们使用的注意力掩码是【MASK】，那么模型的输入为：

```
<BOS> 【MASK】【MASK】【MASK】【MASK】
<BOS>    I   【MASK】 【MASK】【MASK】
<BOS>    I     like  【MASK】【MASK】
<BOS>    I     like    you  【MASK】
<BOS>    I     like    you   </EOS>
```



观察上述的掩码，我们可以发现其实则是一个和文本序列等长的上三角矩阵。我们可以简单地通过创建一个和输入同等长度的上三角矩阵作为注意力掩码，再使用掩码来遮蔽掉输入即可

在具体实现中，我们通过以下代码生成 Mask 矩阵：

```
# 创建一个上三角矩阵，用于遮蔽未来信息。
# 先通过 full 函数创建一个 1 * seq_len * seq_len 的矩阵
mask = torch.full((1, args.max_seq_len, args.max_seq_len), float("-inf"))
# triu 函数的功能是创建一个上三角矩阵
mask = torch.triu(mask, diagonal=1)
```

生成的 Mask 矩阵会是一个上三角矩阵，上三角位置的元素均为 -inf，其他位置的元素置为0。

在注意力计算时，我们会将计算得到的注意力分数与这个掩码做和，再进行 Softmax 操作：

```
# 此处的 scores 为计算得到的注意力分数，mask 为上文生成的掩码矩阵
scores = scores + mask[:, :seqlen, :seqlen]
scores = F.softmax(scores.float(), dim=-1).type_as(xq)
```

通过做求和，上三角区域（也就是应该被遮蔽的 token 对应的位置）的注意力分数结果都变成了 `-inf`，而下三角区域的分数不变。再做 Softmax 操作，`-inf` 的值在经过 Softmax 之后会被置为 0，从而忽略了上三角区域计算的注意力分数，从而实现了注意力遮蔽。





### 多头注意力

就是让模型拥有“多重人格”或“多个视角”，同时从不同的维度去观察和理解同一个序列



单头注意力就像只用一个人的眼睛看世界，难免会有偏见或遗漏。

- **单一视角局限性**：一个注意力矩阵往往只能捕捉一种关系（比如只关注了“语法”）。
- **并行多样性**：多头允许模型在同一时间：
  - **头 A** 关注：**指代关系**（这个“它”指的是哪个名词？）。
  - **头 B** 关注：**动宾关系**（这个动作的主语是谁？）。
  - **头 C** 关注：**距离关系**（相邻的词是否有特殊含义？）。



所谓的多头注意力机制其实就是将原始的输入序列进行多组的自注意力处理；然后再将每一组得到的自注意力结果拼接起来，再通过一个线性层进行处理，得到最终的输出。

1. **线性映射（Linear）**：将输入的 $Q,K,V$  通过不同的参数矩阵 $W$ 投影到多个低维的“子空间”。
2. **并行计算（Attention）**：在每个子空间内，独立地计算自注意力。
3. **拼接（Concat）**：把所有头计算出来的结果横向拼接在一起。
4. **最后整合（Final Linear）**：通过一个最终的线性层，将拼接后的高维向量还原回原始维度。

$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O
$$

其中：
$$
\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$


******



## **序列建模架构与实现**

- **1：Seq2Seq (任务范式/思维)**
  - *定义*：输入序列到输出序列的抽象映射（$N->M$）
  - *本质*：一种解决变长序列问题的通用方法论
  - *应用*：翻译、摘要、对话等
- **2：Encoder-Decoder (逻辑框架/分工)**
  - *定义*：将 Seq2Seq 任务拆解为“压缩”与“展开”两个阶段
  - *角色*：
    - **Encoder**：语义抽取与上下文压缩
    - **Decoder**：条件概率生成与自回归输出
  - *核心纽带*：上下文向量 (Context Vector)
- **3：基础组件 (底层神经网络工具)**
  - **3.1 FNN (前馈神经网络)**：负责位置维度的特征变换与非线性投射
  - **3.2 Layer Norm (层归一化)**：负责训练过程中的数值稳定与梯度平滑
  - **3.3 Residual Connection (残差连接)**：负责深层网络中的信息无损传递





### **Seq2Seq** 模型

Encoder-Decoder 架构在处理序列数据时的一种**具体实现**

Encoder-Decoder 是一种通用的**框架设计**（也可以用于图片转文字等）；而 Seq2Seq 更多是指代一种**模型类别或任务**，即“将一个序列转换成另一个序列”



Seq2Seq强调的是一种架构范式，是抽象关系的映射，

模型输入的是一个自然语言序列 $input=(x1,x2,x3...xn)$ ，输出的是一个可能不等长的自然语言序列 $output=(y1,y2,y3...ym)$

强调万物皆序列，这种思维极大地统一了 NLP 任务

- **翻译**：源语言序列 $->$ 目标语言序列
- **对话**：问题序列 $->$ 回答序列
- **代码生成**：自然语言序列 $->$ 代码字符序列
- **甚至图像描述**：像素序列（特征提取后）$->$ 文本描述序列

虽然这个“思维”是稳定的，但实现它的“零件”一直在进化：

- **早期（RNN 时期）**：用递归的方式，一个词一个词地“啃”序列。
- **中期（Attention 时期）**：给递归加了“视力”，能回头看重点。
- **现在（Transformer 时期）**：抛弃递归，用全并行的方式处理序列



### Encoder-Decoder

在 Transformer 中，使用注意力机制的是其两个核心组件——Encoder（编码器）和 Decoder（解码器）

核心工作原理

1. **编码器 (Encoder)**：负责“理解”输入。它接收长度可变的输入序列（如一串中文），并将其压缩成一个固定形状的**编码状态**（Context Vector，上下文向量），这个向量包含了输入的全部语义特征。
2. **解码器 (Decoder)**：负责“生成”输出。它从编码器提供的上下文向量出发，结合之前已经生成的输出，逐步预测出下一个元素（如对应的英文单词），直到生成完整的输出序列。



**词嵌入与编码器**

在实际的模型（如 Transformer 架构）中，流程是这样的：

1. **输入**：一串文字。
2. **词嵌入层 (Embedding Layer)**：把文字变成初始向量。
3. **编码器层 (Encoder Layers)**：对这些向量进行多层计算（如 Self-Attention），整合全句信息。
4. **输出**：最终生成的 **Context Vector**。

**总结：** 词嵌入是编码器的**原材料**，编码器产生的上下文向量是经过深加工后的**成品**



### **基础组件**



#### 前馈神经网络（FNN）

```python
class MLP(nn.Module):
    '''前馈神经网络'''
    def __init__(self, dim: int, hidden_dim: int, dropout: float):
        super().__init__()
        # 定义第一层线性变换，从输入维度到隐藏维度
        self.w1 = nn.Linear(dim, hidden_dim, bias=False)
        # 定义第二层线性变换，从隐藏维度到输入维度
        self.w2 = nn.Linear(hidden_dim, dim, bias=False)
        # 定义dropout层，用于防止过拟合
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        # 前向传播函数
        # 首先，输入x通过第一层线性变换和RELU激活函数
        # 最后，通过第二层线性变换和dropout层
        return self.dropout(self.w2(F.relu(self.w1(x))))
```

Transformer 的前馈神经网络是由两个线性层中间加一个 RELU 激活函数组成的，以及前馈神经网络还加入了一个 Dropout 层来防止过拟合



#### 层归一化(Layer Norm, LN)

神经网络主流的归一化一般有两种，批归一化（Batch Norm）和层归一化（Layer Norm）



归一化核心是为了让不同层输入的取值范围或者分布能够比较一致。由于深度神经网络中每一层的输入都是上一层的输出，因此多层传递下，对网络中较高的层，之前的所有神经层的参数变化会导致其输入的分布发生较大的改变。也就是说，随着神经网络参数的更新，各层的输出分布是不相同的，且差异会随着网络深度的增大而增大。但是，需要预测的条件分布始终是相同的，从而也就造成了预测的误差。因此，在深度神经网络中，往往需要归一化操作，将每一层的输入都归一化成标准正态分布



1. 它是怎么做的？（原理）

   它的操作非常直观：

   - **计算均值和方差**：把一个单词向量里所有的数字拿出来，算出它们的平均值和波动程度（方差）
   - **标准化处理**：每个数字减去均值，再除以标准差
   - **结果**：不管原始数值是多少，处理后都变成了**均值为 0、方差为 1** 的标准分布

2. 为什么要这么做？（痛点）
   - **稳住数值**：防止深层网络中数值忽大忽小导致的**梯度爆炸或消失**
   - **加速收敛**：大家都在同一个“量级”下对话，模型不需要费力去适应剧烈的数值波动，训练起来又快又稳



#### 残差连接



1. 它是怎么做的？（原理）

   它的公式极其简单：
   $$
   Output = F(x) + x
   $$

   - $x$：输入信号（原封不动）
   - $F(x)$：经过这一层神经网络（如 FNN 或 Attention）处理后的信号
   - **相加**：把处理过的信号和原始信号**直接加在一起**

2. 为什么要这么做？（痛点）
   - **防止遗忘（梯度消失）**：网络太深时，信号传着传着就“弄丢了”。有了残差连接，原始信息可以走“直达电梯”无损传递，保证深层网络也能看到最初的输入
   - **降低学习难度**：如果这一层神经网络学不到有用的东西，它只需要把参数变为 0，那么 ![img](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==)。也就是说，**模型至少不会比上一层变差**，这大大降低了深层模型的训练门槛



### 简单实现



**Encoder**

在实现上述组件之后，我们可以搭建起 Transformer 的 Encoder

Encoder 由 N 个 Encoder Layer 组成

每一个 Encoder Layer 包括一个注意力层和一个前馈神经网络

因此，我们可以首先实现一个 Encoder Layer

```python
class EncoderLayer(nn.Module):
  '''Encoder层'''
    def __init__(self, args):
        super().__init__()
        # 一个 Layer 中有两个 LayerNorm，分别在 Attention 之前和 MLP 之前
        self.attention_norm = LayerNorm(args.n_embd)
        # Encoder 不需要掩码，传入 is_causal=False
        self.attention = MultiHeadAttention(args, is_causal=False)
        self.fnn_norm = LayerNorm(args.n_embd)
        self.feed_forward = MLP(args.dim, args.dim, args.dropout)

    def forward(self, x):
        # Layer Norm
        norm_x = self.attention_norm(x)
        # 自注意力
        h = x + self.attention.forward(norm_x, norm_x, norm_x)
        # 经过前馈神经网络
        out = h + self.feed_forward.forward(self.fnn_norm(h))
        return out
```

然后我们搭建一个 Encoder，由 N 个 Encoder Layer 组成，在最后会加入一个 Layer Norm 实现规范化

```python
class Encoder(nn.Module):
    '''Encoder 块'''
    def __init__(self, args):
        super(Encoder, self).__init__() 
        # 一个 Encoder 由 N 个 Encoder Layer 组成
        self.layers = nn.ModuleList([EncoderLayer(args) for _ in range(args.n_layer)])
        self.norm = LayerNorm(args.n_embd)

    def forward(self, x):
        "分别通过 N 层 Encoder Layer"
        for layer in self.layers:
            x = layer(x)
        return self.norm(x)
```

通过 Encoder 的输出，就是输入编码之后的结果



**Decoder**

类似的，我们也可以先搭建 Decoder Layer，再将 N 个 Decoder Layer 组装为 Decoder

与 Encoder 不同的是，Decoder 由两个注意力层和一个前馈神经网络组成

第一个注意力层是一个掩码自注意力层，即使用 Mask 的注意力计算，保证每一个 token 只能使用该 token 之前的注意力分数

第二个注意力层是一个多头注意力层，该层将使用第一个注意力层的输出作为 query，使用 Encoder 的输出作为 key 和 value，来计算注意力分数

最后，再经过前馈神经网络

```python
class DecoderLayer(nn.Module):
  '''解码层'''
    def __init__(self, args):
        super().__init__()
        # 一个 Layer 中有三个 LayerNorm，分别在 Mask Attention 之前、Self Attention 之前和 MLP 之前
        self.attention_norm_1 = LayerNorm(args.n_embd)
        # Decoder 的第一个部分是 Mask Attention，传入 is_causal=True
        self.mask_attention = MultiHeadAttention(args, is_causal=True)
        self.attention_norm_2 = LayerNorm(args.n_embd)
        # Decoder 的第二个部分是 类似于 Encoder 的 Attention，传入 is_causal=False
        self.attention = MultiHeadAttention(args, is_causal=False)
        self.ffn_norm = LayerNorm(args.n_embd)
        # 第三个部分是 MLP
        self.feed_forward = MLP(args.dim, args.dim, args.dropout)

    def forward(self, x, enc_out):
        # Layer Norm
        norm_x = self.attention_norm_1(x)
        # 掩码自注意力
        x = x + self.mask_attention.forward(norm_x, norm_x, norm_x)
        # 多头注意力
        norm_x = self.attention_norm_2(x)
        h = x + self.attention.forward(norm_x, enc_out, enc_out)
        # 经过前馈神经网络
        out = h + self.feed_forward.forward(self.ffn_norm(h))
        return out
```

然后同样的，我们搭建一个 Decoder 块

```python
class Decoder(nn.Module):
    '''解码器'''
    def __init__(self, args):
        super(Decoder, self).__init__() 
        # 一个 Decoder 由 N 个 Decoder Layer 组成
        self.layers = nn.ModuleList([DecoderLayer(args) for _ in range(args.n_layer)])
        self.norm = LayerNorm(args.n_embd)

    def forward(self, x, enc_out):
        "Pass the input (and mask) through each layer in turn."
        for layer in self.layers:
            x = layer(x, enc_out)
        return self.norm(x)
```

完成上述 Encoder、Decoder 的搭建，就完成了 Transformer 的核心部分，接下来将 Encoder、Decoder 拼接起来再加入 Embedding 层就可以搭建出完整的 Transformer 模型



## 搭建一个 Transformer

前面剖析了 Attention 机制和 Transformer 的核心——Encoder、Decoder 结构

基于上面实现的组件，搭建起一个完整的 Transformer 模型



### 1. Embedding 层

在 NLP 任务中，我们往往需要将自然语言的输入转化为机器可以处理的向量

在深度学习中，承担这个任务的组件就是 Embedding 层

Embedding 层其实是一个存储固定大小的词典的嵌入向量查找表

在输入神经网络之前，我们往往会先让自然语言输入通过分词器 tokenizer

分词器的作用是把自然语言输入切分成 token 并转化成一个固定的 index

例如，输入“我喜欢你”，那么，分词器可以将输入转化成：

```
input: 我
output: 0

input: 喜欢
output: 1

input：你
output: 2
```

实际情况下，tokenizer 的工作会比这更复杂

分词有多种不同的方式，而词表大小则往往高达数万数十万

因此，Embedding 层的输入往往是一个形状为 $$(batch\_size, seq\_len, 1)$$的矩阵

第一个维度是一次批处理的数量

第二个维度是自然语言序列的长度

第三个维度则是 token 经过 tokenizer 转化成的 index 值

例如，对上述输入，Embedding 层的输入会是：`[[[0],[1],[2]]]`

其 `batch_size` 为1，`seq_len` 为3，转化出来的 `index` 如上

而 Embedding 内部其实是一个可训练的（Vocab_size，embedding_dim）的权重矩阵

词表里的每一个值，都对应一行维度为 embedding_dim 的向量

对于输入的值，会对应到这个词向量，然后拼接成（batch_size，seq_len，embedding_dim）的矩阵输出

上述实现并不复杂，我们可以直接使用 torch 中的 Embedding 层

```python
self.tok_embeddings = nn.Embedding(args.vocab_size, args.dim)
```



### 2. 位置编码

注意力机制可以实现良好的并行计算

但同时，其注意力计算的方式也导致序列中相对位置的丢失

在注意力机制的计算过程中，对于序列中的每一个 token

其他各个位置对其来说都是平等的，即“我喜欢你”和“你喜欢我”在注意力机制看来是完全相同的

但无疑这是注意力机制存在的一个巨大问题

因此，为使用序列顺序信息，保留序列中的相对位置信息

Transformer 采用了位置编码机制，该机制也在之后被多种模型沿用

具体实现方式复杂，不做具体解释

```python

class PositionalEncoding(nn.Module):
    '''位置编码模块'''

    def __init__(self, args):
        super(PositionalEncoding, self).__init__()
        # Dropout 层
        # self.dropout = nn.Dropout(p=args.dropout)

        # block size 是序列的最大长度
        pe = torch.zeros(args.block_size, args.n_embd)
        position = torch.arange(0, args.block_size).unsqueeze(1)
        # 计算 theta
        div_term = torch.exp(
            torch.arange(0, args.n_embd, 2) * -(math.log(10000.0) / args.n_embd)
        )
        # 分别计算 sin、cos 结果
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        self.register_buffer("pe", pe)

    def forward(self, x):
        # 将位置编码加到 Embedding 结果上
        x = x + self.pe[:, : x.size(1)].requires_grad_(False)
        return x
```





### 3. 一个完整的 Transformer

上述所有组件，再按照下图的 Tranfromer 结构拼接起来就是一个完整的 Transformer 模型了

![图片描述](https://raw.githubusercontent.com/datawhalechina/happy-llm/main/docs/images/2-figures/3-1.png)

基于之前所实现过的组件，我们实现完整的 Transformer 模型

```python
class Transformer(nn.Module):
   '''整体模型'''
    def __init__(self, args):
        super().__init__()
        # 必须输入词表大小和 block size
        assert args.vocab_size is not None
        assert args.block_size is not None
        self.args = args
        self.transformer = nn.ModuleDict(dict(
            wte = nn.Embedding(args.vocab_size, args.n_embd),# 词嵌入
            wpe = PositionalEncoding(args),# 位置编码
            drop = nn.Dropout(args.dropout),# 防止过拟合
            encoder = Encoder(args),# Encoder
            decoder = Decoder(args),# Decoder
        ))
        # 最后的线性层，输入是 n_embd，输出是词表大小
        self.lm_head = nn.Linear(args.n_embd, args.vocab_size, bias=False)

        # 初始化所有的权重
        self.apply(self._init_weights)

        # 查看所有参数的数量
        print("number of parameters: %.2fM" % (self.get_num_params()/1e6,))

    '''统计所有参数的数量'''
    def get_num_params(self, non_embedding=False):
        # non_embedding: 是否统计 embedding 的参数
        n_params = sum(p.numel() for p in self.parameters())
        # 如果不统计 embedding 的参数，就减去
        if non_embedding:
            n_params -= self.transformer.wte.weight.numel()
        return n_params

    '''初始化权重'''
    def _init_weights(self, module):
        # 线性层和 Embedding 层初始化为正则分布
        if isinstance(module, nn.Linear):
            torch.nn.init.normal_(module.weight, mean=0.0, std=0.02)
            if module.bias is not None:
                torch.nn.init.zeros_(module.bias)
        elif isinstance(module, nn.Embedding):
            torch.nn.init.normal_(module.weight, mean=0.0, std=0.02)
    
    '''前向计算函数'''
    def forward(self, idx, targets=None):
        # 输入为 idx，维度为 (batch size, sequence length, 1)；targets 为目标序列，用于计算 loss
        device = idx.device
        b, t = idx.size()
        assert t <= self.args.block_size, f"不能计算该序列，该序列长度为 {t}, 最大序列长度只有 {self.args.block_size}"

        # 通过 self.transformer
        # 首先将输入 idx 通过 Embedding 层，得到维度为 (batch size, sequence length, n_embd)
        print("idx",idx.size())
        # 通过 Embedding 层
        tok_emb = self.transformer.wte(idx)
        print("tok_emb",tok_emb.size())
        # 然后通过位置编码
        pos_emb = self.transformer.wpe(tok_emb) 
        # 再进行 Dropout
        x = self.transformer.drop(pos_emb)
        # 然后通过 Encoder
        print("x after wpe:",x.size())
        enc_out = self.transformer.encoder(x)
        print("enc_out:",enc_out.size())
        # 再通过 Decoder
        x = self.transformer.decoder(x, enc_out)
        print("x after decoder:",x.size())

        if targets is not None:
            # 训练阶段，如果我们给了 targets，就计算 loss
            # 先通过最后的 Linear 层，得到维度为 (batch size, sequence length, vocab size)
            logits = self.lm_head(x)
            # 再跟 targets 计算交叉熵
            loss = F.cross_entropy(logits.view(-1, logits.size(-1)), targets.view(-1), ignore_index=-1)
        else:
            # 推理阶段，我们只需要 logits，loss 为 None
            # 取 -1 是只取序列中的最后一个作为输出
            logits = self.lm_head(x[:, [-1], :]) # note: using list [-1] to preserve the time dim
            loss = None

        return logits, loss
```

