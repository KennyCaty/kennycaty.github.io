---
title: transformer
date: 2023-10-23 15:40:32
tags:transformers
---





# 介绍

来源于一篇论文：[Attention is all you need](https://arxiv.org/abs/1706.03762)， 最开始用于机器翻译

TRM在做一个什么事情呢？举个例子

```mermaid
graph LR
	in(input:我爱你)-->T[TRM]-->out(output: I love you)
```

将TRM细化为两部分

```mermaid
graph LR
	in(input:我爱你)-->E[Encoders]-->D[Decoders]-->out(output: I love you)
```

再细化：
![image-20231023155037365](./transformer/image-20231023155037365.png)

注意，这n个Encoder结构相同，n个Decoder结构相同，但是Encoder与Decoder结构不同

这个结构相同，不代表参数相同，参数是独立训练的



下图是原论文的结构，左侧是Encoders（n个），右侧是Decoders（n个）

![image-20231023155616679](./transformer/image-20231023155616679.png)



# Encoder

![image-20231023155705351](./transformer/image-20231023155705351.png)

我们可以把单个Encoder分为三个部分

* 输入
* 注意力机制 self-attention
* 前馈神经网络 



## 输入部分

* Embedding（输入NLP的知识）
* 位置嵌入



Embedding就是把word转化为vector

> 我们为什么需要记录位置信息？（Position Encoding）

在RNN中，所有的u，w，v是共享的一套参数

![image-20231023160125230](./transformer/image-20231023160125230.png) 

在计算梯度时，远距离的梯度会被近距离的梯度主导（梯度消失）导致RNN不能记住太远的内容，且RNN不能并行计算所有输入（只能按序列关系一个一个计算，这是Transformer没有的）

因为Transformer本身没有位置信息，可以并行计算所有输入，所以我们需要位置编码来记住每个序列的位置



论文中的位置编码是使用sin和cos计算的
$$
PE_{(pos,2i)} = sin(pos/1000^{2i/d_{model}}) \\
PE_{(pos,2i+1)} = cos(pos/1000^{2i/d_{model}})
$$
![image-20231023160708376](./transformer/image-20231023160708376.png)

这么理解这个公式？在偶数的位置用sin，奇数的位置用cos，比如这个“爱”，用512维度的位置编码向量编码，奇偶位置分别用cos和sin编码

在得到位置编码向量后，与词向量相加，作为整个Transformer输入

![image-20231023160930761](./transformer/image-20231023160930761.png)



为什么位置嵌入有用？这个就是古圣先贤帮我们测试好的了，信他们就好了

![image-20231023161252466](./transformer/image-20231023161252466.png)

**但是这种相对位置信息会在注意力机制那里消失**







## Self-attention

什么是注意力机制

用一张图片举例，看到下面一张图片，可能每个人有不同的关注点，但是带有一个问题：**婴儿在干嘛**，带着这个问题再去看，就会去思考图中**哪些东西**与这句话**相关**

![image-20231023161539435](./transformer/image-20231023161539435.png)



原论文公式：
$$
Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
三个重点：**Q，K，V三个矩阵** 

![image-20231023162055847](./transformer/image-20231023162055847.png)

计算过程，F一般是点乘

![image-20231023162250947](./transformer/image-20231023162250947.png)



Q，K，V怎么来的呢？

在只有单词向量的情况下，x1分别与三个矩阵相乘得到$q_1, k_1, v_1$

![image-20231023162536760](./transformer/image-20231023162536760.png)

然后计算QK相似度

![image-20231023162801384](./transformer/image-20231023162801384.png)

实际情况中使用矩阵并行计算

![image-20231023162841024](./transformer/image-20231023162841024.png)



> 多头注意力机制

上面的例子只用了一套$w^q, w^k, w^v$参数，实际中可能是多套

![image-20231023163002410](./transformer/image-20231023163002410.png)

多个头就会有多个输出，需要合在一起输出

![image-20231023163122599](./transformer/image-20231023163122599.png)







## 残差

![image-20231023163213434](./transformer/image-20231023163213434.png)



> 什么是残差

![image-20231023163342292](./transformer/image-20231023163342292.png)

上图中x经过两层网络后，有与最开始的x进行计算然后作为真正的输出，这样有什么作用呢？

![image-20231023163512630](./transformer/image-20231023163512630.png)

A的输出，就是那个x，针对这个网络，做一个链式求导（对x求导）

![image-20231023163601713](./transformer/image-20231023163601713.png)

这里右边的括号里第一项因为有一个 1 ，确保了梯度不会为 0 （环节出现梯度消失的问题）





## LayNorm

为什么在这里使用Layer Normalization 而不是使用传统的BN（batch normalization）

因为BN效果差，所以不用（doge）



BN在干嘛，BN是对整个batch中的样本在同一纬度特征做处理（比如说我知道所有样本的第n个特征都是：身高）

![image-20231023164358486](./transformer/image-20231023164358486.png)

BN优点：

* 可以解决内部协变量偏移问题
* 缓解了梯度饱和问题（如果使用sigmoid激活函数的话），加快收敛

缺点：

* batch比较小的时候，效果差（因为是用样本的均值和方差来模拟全体的样本和方差）

* 在RNN中效果比较差







# Decoder

![image-20231023170306622](./transformer/image-20231023170306622.png)

## Mask多头注意力机制

为什么需要mask？

因为训练时候是并行计算所有输入，而实际预测时并不知道未来的word信息，所以这种训练方式并不好。

所以在训练时，mask掉未来的信息

![image-20231023170534843](./transformer/image-20231023170534843.png)



## 交互层

![image-20231023170731358](./transformer/image-20231023170731358.png)

所有生成的Encoder输出和每一个Decoder做交互

具体交互如下

![image-20231023170837818](./transformer/image-20231023170837818.png)

K，V来自Encoder，Q来自Decoder，做Attention

![image-20231023170924022](./transformer/image-20231023170924022.png)





# CODE

Transformer的三类应用

* 机器翻译类 - 使用Encoder+Decoder
* 文本分类BERT和图片分类VIT - 只使用Encoder
* 生成类模型 - 只使用Decoder



