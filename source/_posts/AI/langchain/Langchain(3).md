---
title: Langchain（三）：Chain
date: 2023-08-17 16:50:33
tags:
description: LangChain中最重要的关键构建块 Chain
---



reference：

[LangChain&DeepLearningAI](https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/)



# Chain

Chain通常将LLM与Prompt一起结合使用

有了Chain，可以将一堆这些构建块放在一起操作

Chain的一大优势在于可以同时对多个输入一起运行

示例：

首先需要向之前一样加载环境变量：

```python
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file
# 在.env 里面添加OPENAI_API_KEY
```





我们读入一个csv文件

```python
import pandas as pd
df = pd.read_csv('Data.csv')
df.head()
```

![image-20230817165713751](./Langchain(3)/image-20230817165713751.png)







# LLMChain

这是一个**简单但是非常强大的Chain**，为后面介绍的许多Chain打下了基础

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import LLMChain
```

初始化要使用的LLM

```python
llm = ChatOpenAI(temperature=0.9)
```

定义Prompt

```python
prompt = ChatPromptTemplate.from_template(
    "What is the best name to describe \
    a company that makes {product}?"
)
```



组合为Chain

```python
chain = LLMChain(llm=llm, prompt=prompt)
```

运行Chain

```python
product = "Queen Size Sheet Set"
chain.run(product)
```





# Sequential Chains

顺序链按顺序运行一系列链



## SimpleSequentialChain

当我们的预期只**有一个输入并返回只有一个输出**的子链是，这种方法很有效

```python
from langchain.chains import SimpleSequentialChain

llm = ChatOpenAI(temperature=0.9)

# prompt template 1
first_prompt = ChatPromptTemplate.from_template(
    "What is the best name to describe \
    a company that makes {product}?"
)

# Chain 1
chain_one = LLMChain(llm=llm, prompt=first_prompt)

# prompt template 2
second_prompt = ChatPromptTemplate.from_template(
    "Write a 20 words description for the following \
    company:{company_name}"
)
# chain 2
chain_two = LLMChain(llm=llm, prompt=second_prompt)

overall_simple_chain = SimpleSequentialChain(chains=[chain_one, chain_two],
                                             verbose=True)
overall_simple_chain.run(product)
```

这将第一个Chain的输出作为第二个Chain的输入



## SequentialChain

当存在多个输入和多个输出时，使用SequentialChain

![image-20230817183448287](./Langchain(3)/image-20230817183448287.png)

```python
from langchain.chains import SequentialChain

llm = ChatOpenAI(temperature=0.9)

# prompt template 1: translate to english
first_prompt = ChatPromptTemplate.from_template(
    "Translate the following review to english:"
    "\n\n{Review}"
)
# chain 1: input= Review and output= English_Review
chain_one = LLMChain(llm=llm, prompt=first_prompt, 
                     output_key="English_Review"
                    )
second_prompt = ChatPromptTemplate.from_template(
    "Can you summarize the following review in 1 sentence:"
    "\n\n{English_Review}"
)
# chain 2: input= English_Review and output= summary
chain_two = LLMChain(llm=llm, prompt=second_prompt, 
                     output_key="summary"
                    )

# prompt template 3: translate to english
third_prompt = ChatPromptTemplate.from_template(
    "What language is the following review:\n\n{Review}"
)
# chain 3: input= Review and output= language
chain_three = LLMChain(llm=llm, prompt=third_prompt,
                       output_key="language"
                      )

# prompt template 4: follow up message
fourth_prompt = ChatPromptTemplate.from_template(
    "Write a follow up response to the following "
    "summary in the specified language:"
    "\n\nSummary: {summary}\n\nLanguage: {language}"
)
# chain 4: input= summary, language and output= followup_message
chain_four = LLMChain(llm=llm, prompt=fourth_prompt,
                      output_key="followup_message"
                     )

# overall_chain: input= Review 
# and output= English_Review,summary, followup_message
overall_chain = SequentialChain(
    chains=[chain_one, chain_two, chain_three, chain_four],
    input_variables=["Review"],
    output_variables=["English_Review", "summary","followup_message"],
    verbose=True
)

review = df.Review[5]
overall_chain(review)
```





## RouterChain

路由链是一种Chain，可以根据输入的内容，动态的选择下一个要调用的Chain

一个很好的想象方式是，如果你有多个子链。每个子链专门处理特定类型的输入，你可似有一个路由链。首先决定将其传递给哪个子链然后将其传递给该链



![image-20230817183725858](./Langchain(3)/image-20230817183725858.png)



定义子Chain的模板

```python
physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise\
and easy to understand manner. \
When you don't know the answer to a question you admit\
that you don't know.

Here is a question:
{input}"""


math_template = """You are a very good mathematician. \
You are great at answering math questions. \
You are so good because you are able to break down \
hard problems into their component parts, 
answer the component parts, and then put them together\
to answer the broader question.

Here is a question:
{input}"""

history_template = """You are a very good historian. \
You have an excellent knowledge of and understanding of people,\
events and contexts from a range of historical periods. \
You have the ability to think, reflect, debate, discuss and \
evaluate the past. You have a respect for historical evidence\
and the ability to make use of it to support your explanations \
and judgements.

Here is a question:
{input}"""


computerscience_template = """ You are a successful computer scientist.\
You have a passion for creativity, collaboration,\
forward-thinking, confidence, strong problem-solving capabilities,\
understanding of theories and algorithms, and excellent communication \
skills. You are great at answering coding questions. \
You are so good because you know how to solve a problem by \
describing the solution in imperative steps \
that a machine can easily interpret and you know how to \
choose a solution that has a good balance between \
time complexity and space complexity. 

Here is a question:
{input}"""
```

封装成模板列表

```python
prompt_infos = [
    {
        "name": "physics", 
        "description": "Good for answering questions about physics", 
        "prompt_template": physics_template
    },
    {
        "name": "math", 
        "description": "Good for answering math questions", 
        "prompt_template": math_template
    },
    {
        "name": "History", 
        "description": "Good for answering history questions", 
        "prompt_template": history_template
    },
    {
        "name": "computer science", 
        "description": "Good for answering computer science questions", 
        "prompt_template": computerscience_template
    }
]
```



导入包

```python
from langchain.chains.router import MultiPromptChain # 用于在不同提示模板之间进行路由
from langchain.chains.router.llm_router import LLMRouterChain,RouterOutputParser
from langchain.prompts import PromptTemplate
```

这几个import的类的作用是：

- `MultiPromptChain` 是一种路由链，它使用一个语言模型链来选择不同的提示模板。提示模板是一种预定义的生成语言模型输入的方法，适用于某个任务。`MultiPromptChain` 可以根据语言模型的输出，解析出目标链的名称和输入，然后调用相应的链来完成任务。
- `LLMRouterChain` 是一种路由链，它使用一个语言模型链来执行路由。它可以根据语言模型的输出，解析出目标链的名称和输入，然后调用相应的链来完成任务。
- `RouterOutputParser` 是一个输出解析器，它用于解析路由链的输出，将其转换为一个字典，包含目标链的名称和输入。它可以指定默认的目标链、下一个输入的类型和键名。
- `PromptTemplate` 是一个类，它用于创建和处理提示模板。它可以指定模板字符串、输入变量、输出解析器等参数。它可以使用不同的模板语法，如 `str.format` 或 `jinja2`。它还提供了一些方法，如 `format`、`from_template`、`to_json` 等。



```python
llm = ChatOpenAI(temperature=0)
# 定义目标Chain
destination_chains = {}
for p_info in prompt_infos:
    name = p_info["name"]
    prompt_template = p_info["prompt_template"]
    prompt = ChatPromptTemplate.from_template(template=prompt_template)
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[name] = chain  
    
destinations = [f"{p['name']}: {p['description']}" for p in prompt_infos]
destinations_str = "\n".join(destinations)
```



默认Chain，当路由无法决定使用哪个子链时，会调用该Chain

```python
default_prompt = ChatPromptTemplate.from_template("{input}")
default_chain = LLMChain(llm=llm, prompt=default_prompt)
```



定义用于LLM在不同Chain之间路由的模板

这个模板包含了要完成的任务的说明，以及输出应该具有的特定格式

~~~python
MULTI_PROMPT_ROUTER_TEMPLATE = """Given a raw text input to a \
language model select the model prompt best suited for the input. \
You will be given the names of the available prompts and a \
description of what the prompt is best suited for. \
You may also revise the original input if you think that revising\
it will ultimately lead to a better response from the language model.

<< FORMATTING >>
Return a markdown code snippet with a JSON object formatted to look like:
```json
{{{{
    "destination": string \ name of the prompt to use or "DEFAULT"
    "next_inputs": string \ a potentially modified version of the original input
}}}}
```

REMEMBER: "destination" MUST be one of the candidate prompt \
names specified below OR it can be "DEFAULT" if the input is not\
well suited for any of the candidate prompts.
REMEMBER: "next_inputs" can just be the original input \
if you don't think any modifications are needed.

<< CANDIDATE PROMPTS >>
{destinations}

<< INPUT >>
{{input}}

<< OUTPUT (remember to include the ```json)>>"""
~~~



```python
router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(
    destinations=destinations_str
)
router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),# 它将帮助这个Chain决定在哪些子Chain中路由
)

router_chain = LLMRouterChain.from_llm(llm, router_prompt)
```



定义总链

```python
chain = MultiPromptChain(router_chain=router_chain, 
                         destination_chains=destination_chains, 
                         default_chain=default_chain, verbose=True
                        )
```

```python
chain.run("What is black body radiation?")

chain.run("what is 2 + 2")

chain.run("Why does every cell in our body contain DNA?")
```

