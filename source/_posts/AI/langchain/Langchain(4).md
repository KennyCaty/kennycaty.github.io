---
title: LangChain（四）： 问答检索
tags: Langchain
categories: Langchain
---









我们想要使用语言模型并将其与我们的许多文档结合起来，但是LLM一次只能检查几千个词。

那么，如何让LLM回答其中的所有问题呢？



这就是嵌入（Embeddings）和向量存储（Vector storage）发挥作用的地方



# Embeddings

![image-20230817195548859](./Langchain(4)/image-20230817195548859.png)

* 嵌入为文本片段创建**数值表示**，捕捉了它所应用的文本片段的语义含义
* 相似内容的文本片段将具有相似的向量，所以才能够在向量空间中比较文本



比如下面例子：前两个是宠物，后一个是汽车，向量化比较后，宠物的数值表示十分相似

![image-20230817195834645](./Langchain(4)/image-20230817195834645.png)

这让我们可以轻松找出哪些文本片段相似，在传递给LLM文本片段以回答问题时非常有用



# Vector Database

![image-20230817200334649](./Langchain(4)/image-20230817200334649.png)

向量数据库是**存储**在前一步中创建的这些**向量表示**的一种方式

创建它的方式是将其中的文本块填充为来自输入文档的块

* 当获得一个大的输入文档时，先将其分割成较小的块（chunks），这有助于创建比原始文档更小的文本片段，因为我们可能无法将整个文档传递给LLM
* 为每一个chunk创建一个embedding嵌入，并将其存储在向量数据库中





同样，我们将问题也embedding为向量表示

运行时使用Index在数据库中搜索最相关的文本片段

![image-20230817200843781](./Langchain(4)/image-20230817200843781.png)

选择最相关的前n个，将它们返回，可以将他们作为提示传递给LLM，并输出答案



# 示例

简单示例： 

```python
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file

#from langchain.chains import RetrievalQA # 对文档进行检索的Chain
from langchain.chat_models import ChatOpenAI
from langchain.document_loaders import CSVLoader
from langchain.vectorstores import DocArrayInMemorySearch # 向量存储之一
from IPython.display import display, Markdown

file = 'OutdoorClothingCatalog_1000.csv'
loader = CSVLoader(file_path=file, encoding='utf-8')

from langchain.indexes import VectorstoreIndexCreator 
#帮助我们创建向量存储

index = VectorstoreIndexCreator(
    vectorstore_cls=DocArrayInMemorySearch
).from_loaders([loader])

query ="Please list all your shirts with sun protection \
in a table in markdown and summarize each one."

response = index.query(query)

display(Markdown(response))
```





# 手动构建检索链的方法

最常用的构建检索链的方法是：stuff，将所有有关内容合并为一个文档

其他检索链方法

![image-20230817210856750](./Langchain(4)/image-20230817210856750.png)