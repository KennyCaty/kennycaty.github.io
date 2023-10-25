---
title: LangChain（二）：Memory
date: 2023-08-17 15:53:36
tags: Langchain
categories: Langchain
description: LangChain中的Memory，聊天机器人如何存储会话
---



reference：

[LangChain&DeepLearningAI](https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/)



# 为什么需要Memory

当与这些模型进行交互时，它们自然而然地不会记得您之前说过的话或之前的对话。这对手构建聊天机器人等应用程序拼希望与其对话时是个间题， 因此，在本节申，将介绍记忆也就是如何加载先前对话的部分并将其输入到语言模型中，以便在与其交互时能够保持对话的连贯性。



当你使用一个大型语言模型进行聊天对话时，大型语言模型本身实际上是无状态的。语言模型本身不会记佳你到目前为上所进行的对话。每次调用API都是独立的。LLM需要提供**完整的对话作为上下文**（Context）才能有”记忆“

做法是将之前的**会话存储在内存中**，每次调用都将内存保存的会话和新的输入一起输入给LLM

随着对话变得越来越长。所需的内存量也变得非常长。因此，发送大量的tokens到LLM的成本也变得更加昂贵，而语LLM通常是根据需要处理的tokens数量来计费的。（比如OpenAI的key）





因此，LangChain存储提供了凡种方便的内存类型来存储和积累对话。

最简单的一种内存类型是 `ConversationBufferMemory`

## ConversationBufferMemory

加载OpenAI_key

```python
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file

import warnings
warnings.filterwarnings('ignore')
```



```python
from langchain.chat_models import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
```

下面代码涉及到**ConversationChain（会话链）**的概念，之后的章节会介绍

```python
llm = ChatOpenAI(temperature=0.0)
memory = ConversationBufferMemory()
conversation = ConversationChain(
    llm=llm, 
    memory = memory,
    verbose=True   # True代表每一次都会显示之前所有的详细对话以及Bot的详细思考过程
)
```



连续对话两次

```python
conversation.predict(input="Hi, my name is KennyCaty")
```

```python
conversation.predict(input="What is 1+1?")
```

![image-20230817161745019](./Langchain(2)/image-20230817161745019.png)



可以使用 `save_context` 来手动添加记忆

```python
memory.save_context({"input": "Hi"}, 
                    {"output": "What's up"})
```







# 控制内存无限增长

## ConversationBufferWindowMemory

LangChain提供了一个 `ConversationBufferWindowMemory` ，它的特点是只保留最近的K个交互，以避免缓冲区过大

```python
from langchain.memory import ConversationBufferWindowMemory
memory = ConversationBufferWindowMemory(k=5)   #只保留最近五次对话
```

同样的，如果使用

```python
from langchain.chat_models import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferWindowMemory

llm = ChatOpenAI(temperature=0.0)
memory = ConversationBufferWindowMemory(k=1)
conversation = ConversationChain(
    llm=llm, 
    memory = memory,
    verbose=False
)

#然后运行超过五次predict
conversation.predict(input="XXXX")
# ...
# 超过五次的部分就会被丢弃，遗忘
```





## ConversationTokenBufferMemory

它的特点是只保留最近的 `max_token_limit` 个token数，以避免缓冲区过大

同样的如果记录的token数量超过限制，则会删除之前的tokens

> 需要pip install tiktoken

```python
from langchain.memory import ConversationTokenBufferMemory
from langchain.llms import OpenAI
llm = ChatOpenAI(temperature=0.0)
```

```python
memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=30)
memory.save_context({"input": "AI is what?!"},
                    {"output": "Amazing!"})
memory.save_context({"input": "Backpropagation is what?"},
                    {"output": "Beautiful!"})
memory.save_context({"input": "Chatbots are what?"}, 
                    {"output": "Charming!"})
```

```python
memory.load_memory_variables({})
```

> {'history': 'AI: Beautiful!\nHuman: Chatbots are what?\nAI: Charming!'}
>
> 30个token之前的内容不会存储









## ConversationSummaryMemory

不再将记忆限制为基于最近会话的某个数量，而是**使用语言模型编写到目前为止的对话摘要，并将其作为记忆**

```python
from langchain.memory import ConversationSummaryBufferMemory
memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=100)
```

如果token数小于100，则会记录所有会话内容

如果token数大于100，则会自动生成为Summary（总结）





# 其他存储

还有许多其他内存类型，较为强大的是**向量存储**，在文本嵌入，数据库检索方面功能非常强大

还可以多种内存类型混合使用

除了使用内存类型，还可以将整个会话存储在传统数据库中
