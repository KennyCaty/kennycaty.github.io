---
title: Langchain（一）：第一个LangChain程序，提示模板和解析器
date: 2023-08-17 12:58:38
tags: Langchain
categories: Langchain
description: LangChain的Model， Prompts和Parsers
---





reference：

[LangChain&DeepLearningAI](https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/)





# 概述

LangChain是一个用于构建LLM应用的开源开发框架，目前有Python和JavaScript两种包，它们**专注与组合和模块化**

特点：

* 有许多可与彼此或单独使用的个体组件

* 用例众多，极易入门



重要部分

* Models
  * LLMs：20+integrations
  * Chat Models
  * Text Embedding Models：10+integrations

* Prompts
  * Prompt Templates
  * Output Parsers：5+implementations
    * Retry/fixing logic
  * Example Selectors：5+implementations

* Indexes 将数据注入系统以与模型结合使用的方式
  * Document Loaders：50+implementations
  * Text Splitters：10+implementations
  * Vector stores：10+integrations
  * Retrievers：5+integrations/implementations

* Chains
  * Prompt + LLM + Output parsing
  * Can be used as building blocks for longer chains
  * More application specific chains：20+types

* Agents
  * Agent Types
  * Agent Toolkits





# Models, Prompts and Parsers

>  以下代码在**jupyter**中运行

## 引出模板

由普通调用GPT 引出Langchain调用GPT

```python
import openai
import os
#pip install python-dotenv
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())  # read local .env file
openai.api_key = os.environ['OPENAI_API_KEY']
```



```python
# 调用ChatGPT的辅助函数
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role":"user", "content":prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0,
    )
    return response.choices[0].message["content"]
```



```python
get_completion("What is 1+1?")
```

> '1+1 equals 2.'





为了推动模型，提示和解析器的Langchain抽象化，假设我们收到一封用户用其他语言写的电子邮件

```python
customer_email = """
Arrr,I be fuming that me blender lid \
flew off and splattered me kitchen walls \
with smoothie!And to make matters worse,\
the warranty don't cover the cost of \
cleaning up me kitchen.I need yer help \
right now,matey!
"""
```

要求LLM以平静和尊重的口吻将文本翻译为美式英语

```python
style = """American English \
in a calm and respectful tone.
"""
```

```python
prompt = f"""Tanslate the text \
that is delimited by triple backticks
into a style that is {style}.
text: ```{customer_email}```
"""
print(prompt)
```

> Tanslate the text that is delimited by triple backticks
> into a style that is American English in a calm and respectful tone.
> text: ```
> Arrr,I be fuming that me blender lid flew off and splattered me kitchen walls with smoothie!And to make matters worse,the warranty don't cover the cost of cleaning up me kitchen.I need yer help right now,matey!

```python
response = get_completion(prompt)  # 调用GPT
response
```

> 'I am quite frustrated that my blender lid unexpectedly flew off and made a mess of my kitchen walls with smoothie! Additionally, the warranty does not cover the expenses for cleaning up my kitchen. I kindly request your assistance at this moment, my friend.'



想象一下，如果有不同的客户用不同语言写邮件，英语法语德语等，需要生成一系列的提示来生成这样的翻译

**Langchain如何更优雅的做到这一点？**

使用Model将一些常用LLM API封装了起来

```python
from langchain.chat_models import ChatOpenAI
# 这是langchain GPT API 的抽象
chat = ChatOpenAI()
chat
```

> ChatOpenAI(cache=None, verbose=False, callbacks=None, callback_manager=None, tags=None, metadata=None, client=<class 'openai.api_resources.chat_completion.ChatCompletion'>, model_name='gpt-3.5-turbo', temperature=0.7, model_kwargs={}, openai_api_key='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', openai_api_base='', openai_organization='', openai_proxy='', request_timeout=None, max_retries=6, streaming=False, n=1, max_tokens=None, tiktoken_model_name=None)
>
> 以上都是可以在ChatOpenAI()中设置的参数，比如ChatOpenAI( temperature=0.0 )



定义模板

```python
template_string = """Translate the text \
that is delimited by triple backticks \
into a style that is {style}. \
text: ```{text}```
"""
```

为了重复使用这个模板，我们导入Langchain的提示模板

```python
from langchain.prompts import ChatPromptTemplate
prompt_template = ChatPromptTemplate.from_template(template_string)
```

可以使用 `prompt_template.messages[0].prompt` 查看prompt对象

可以使用 `prompt_template.messages[0].prompt.input_variables `查看需要在模板插入的内容



定义在模板插入的内容

```python
customer_style = """American English \
in a calm and respectful tone.
"""
```

```python
customer_email = """
Arrr,I be fuming that me blender lid \
flew off and splattered me kitchen walls \
with smoothie!And to make matters worse,\
the warranty don't cover the cost of \
cleaning up me kitchen.I need yer help \
right now,matey!
"""
```



**使用模板格式化**

```python
customer_messages = prompt_template.format_messages(
                    style=customer_style,
                    text=customer_email )
```

```python
print(type(customer_messages))  # 返回一个列表
print(type(customer_messages[0]))  #第一个就是HumanMessage模板内容
print(customer_messages[0])
```

> <class 'list'>
> <class 'langchain.schema.messages.HumanMessage'>
> content="Translate the text that is delimited by triple backticks into a style that is American English in a calm and respectful tone.\n. text: \```\nArrr,I be fuming that me blender lid flew off and splattered me kitchen walls with smoothie!And to make matters worse,the warranty don't cover the cost of cleaning up me kitchen.I need yer help right now,matey!\n```\n" additional_kwargs={} example=False



**调用GPT**

```python
customer_response = chat(customer_messages)
customer_response.content
```

> "Arrr, I'm quite upset that my blender lid flew off and splattered my kitchen walls with smoothie! And to make matters worse, the warranty doesn't cover the cost of cleaning up my kitchen. I would greatly appreciate your help at this moment, matey!"



顾客的语言可能是其他语言以及想要翻译成其他风格，只需要在模板格式化（format）时更改模板中相应内容即可





## 为什么要使用提示模板

>  为什么要使用Prompt模板而不是直接使用字符串？

随着构建复杂的应用，提示可能会变得非常长而且详细，所以模板是对提示的抽象，可以在构建复杂应用时**提高复用性**，比如下面这个例子，要让结果更精确，往往需要更细节的提示

![image-20230817142700814](./Langchain(1)/image-20230817142700814.png)

Langchain还内置了一些常用的提示，比如summarization，question answering， 连接数据库或其他API等，通过使用内置提示，可以快速构建应用而不用自己写提示



> Output Parsing输出解析

使用模板能提示语言模型按照特定格式生成输出，比如使用特定关键词

下面的示例使用的时Langchain默认的模型思维链推理框架：**ReAct框架**

**思维链**：能够给予模型一个思考过程，以达到更精确的结论（在后面的**Agent**用途很大）

![image-20230817143439909](./Langchain(1)/image-20230817143439909.png)



模板与解析器的结合可以很好的**抽象指定LLM的输入**，并且使解析器**正确解析LLM的输出**





## Ouput Parser示例

> LangCain解析LLM的Json输出
>
> 示例：从一个产品评论中提取信息并以JSON的格式，格式化输出

```python
# 一个期望输出的示例
{
    "gift": False, # 是否是别人送的礼物
    "delivery_days": 5,   #所需交付天数
    "price_value": "pretty affordable" # 价格怎么样
}
```



```python
customer_review = """\
This leaf blower is pretty amazing.  It has four settings:\
candle blower, gentle breeze, windy city, and tornado. \
It arrived in two days, just in time for my wife's \
anniversary present. \
I think my wife liked it so much she was speechless. \
So far I've been the only one using it, and I've been \
using it every other morning to clear the leaves on our lawn. \
It's slightly more expensive than the other leaf blowers \
out there, but I think it's worth it for the extra features.
"""

review_template = """\
For the following text, extract the following information:

gift: Was the item purchased as a gift or present for someone else? \
Answer True if yes, False if not or unknown.

delivery_days: How many days did it take for the product \
to arrive? If this information is not found, output -1.

price_value: Extract any sentences about the value or price,\
and output them as a comma separated Python list.

Format the output as JSON with the following keys:
gift
delivery_days
price_value

text: {text}
"""
```



```python
from langchain.prompts import ChatPromptTemplate

prompt_template = ChatPromptTemplate.from_template(review_template)
print(prompt_template)
```

```python
messages = prompt_template.format_messages(text=customer_review)
chat = ChatOpenAI(temperature=0.0)
response = chat(messages)
print(response.content)
```

检查响应的type， `type(response.content)` 其实是一个字符串，如果用键值对索引方式会报错



>  将LLM输出字符串解析为Python字典

```python
from langchain.output_parsers import ResponseSchema
from langchain.output_parsers import StructuredOutputParser
```

```python
gift_schema = ResponseSchema(name="gift",
                             description="Was the item purchased\
                             as a gift for someone else? \
                             Answer True if yes,\
                             False if not or unknown.")
delivery_days_schema = ResponseSchema(name="delivery_days",
                                      description="How many days\
                                      did it take for the product\
                                      to arrive? If this \
                                      information is not found,\
                                      output -1.")
price_value_schema = ResponseSchema(name="price_value",
                                    description="Extract any\
                                    sentences about the value or \
                                    price, and output them as a \
                                    comma separated Python list.")

response_schemas = [gift_schema, 
                    delivery_days_schema,
                    price_value_schema]
```

```python
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()
print(format_instructions)
```



新的格式化模板

```python
review_template_2 = """\
For the following text, extract the following information:

gift: Was the item purchased as a gift for someone else? \
Answer True if yes, False if not or unknown.

delivery_days: How many days did it take for the product\
to arrive? If this information is not found, output -1.

price_value: Extract any sentences about the value or price,\
and output them as a comma separated Python list.

text: {text}

{format_instructions}
"""

prompt = ChatPromptTemplate.from_template(template=review_template_2)

messages = prompt.format_messages(text=customer_review, 
                                format_instructions=format_instructions)
```

```python
print(messages[0].content)
```

```python
response = chat(messages)
print(response.content)
```

> \```json
> {
> 	"gift": false,
> 	"delivery_days": "2",
> 	"price_value": "It's slightly more expensive than the other leaf blowers out there, but I think it's worth it for the extra features."
> }
>
> \```

转化为字典：

```python
output_dict = output_parser.parse(response.content)
output_dict
```

这样就可以用键查找到值了

```python
output_dict.get('delivery_days')
```













