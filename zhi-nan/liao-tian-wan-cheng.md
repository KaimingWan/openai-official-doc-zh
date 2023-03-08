# 聊天完成

ChatGPT由gpt-3.5-turbo驱动，这是OpenAI最先进的语言模型。 使用OpenAI API，您可以构建自己的应用程序，并使用gpt-3.5-turbo执行以下操作：&#x20;

* 起草电子邮件或其他写作&#x20;
* 编写Python代码&#x20;
* 回答有关一组文档的问题&#x20;
* 创建对话代理&#x20;
* 为软件提供自然语言界面&#x20;
* 在各种学科中进行辅导&#x20;
* 翻译语言&#x20;
* 为视频游戏模拟角色等等。&#x20;

本指南介绍了如何调用基于聊天的语言模型API，并分享了获取良好结果的技巧。您还可以在OpenAI Playground中尝试新的聊天格式。

## 介绍

聊天模型将一系列消息作为输入，并返回一个由模型生成的消息作为输出。 虽然聊天格式旨在使多轮对话变得容易，但它同样适用于没有任何对话的单轮任务（例如以前由指令跟随模型（如text-davinci-003）提供服务的任务）。 一个示例API调用如下所示：

```python
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai

openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}
    ]
)
```

主要输入是消息参数。消息必须是一个消息对象数组，其中每个对象都有一个角色（“系统”、“用户”或“助手”）和内容（消息的内容）。对话可以只有1条信息，也可以填满许多页面。

&#x20;通常情况下，会先显示系统信息，然后是交替出现的用户和助手信息。&#x20;

系统信息帮助设置助手的行为。在上面的示例中，“You are a helpful assistant.”指示了该助手应如何操作。&#x20;

用户信息帮助指导助手。它们可以由应用程序最终用户生成，也可以由开发人员作为指令设置。&#x20;

助手信息帮助存储以前的响应。它们还可以由开发人员编写以提供所需行为的示例。 包括对话历史记录可在用户说明引用之前的消息时提供帮助。

在上面的示例中，“Where was it played?”这个问题只有在关于2020年世界大赛之前的消息背景下才有意义。因为模型没有过去请求方面记忆力，所有相关信息必须通过对话提供。如果一次对话无法适合模型标记限制，则需要以某种方式缩短它。

## 响应格式

API响应格式如下：

```json
{
 'id': 'chatcmpl-6p9XYPYSTTRi0xEviKjjilqrWU2Ve',
 'object': 'chat.completion',
 'created': 1677649420,
 'model': 'gpt-3.5-turbo',
 'usage': {'prompt_tokens': 56, 'completion_tokens': 31, 'total_tokens': 87},
 'choices': [
   {
    'message': {
      'role': 'assistant',
      'content': 'The 2020 World Series was played in Arlington, Texas at the Globe Life Field, which was the new home stadium for the Texas Rangers.'},
    'finish_reason': 'stop',
    'index': 0
   }
  ]
}
```

在Python中，可以使用response\['choices']\[0]\['message']\['content']提取助手的回复。 每个响应都将包括一个finish\_reason。 finish\_reason的可能值为： stop：API返回完整的模型输出 length：由于max\_tokens参数或令牌限制而导致不完整的模型输出 content\_filter：由于我们内容过滤器中的标志而省略内容 null：API响应仍在进行中或不完整

## 管理词元

语言模型以称为词元的块读取文本。在英语中，一个标记可以短至一个字符或长至一个单词（例如 a 或 apple），而在某些语言中，词元甚至可以比一个字符更短或比一个单词更长。&#x20;

例如，“ChatGPT is great！”字符串被编码成六个词元：\[“Chat”，“G”，“PT”，“ is”，“ great”，“!”]。 API 调用中的总词元数会影响以下内容：您支付每个词元的费用；写入更多词元需要更多时间；总词元必须低于模型的最大限制（gpt-3.5-turbo-0301 的最大限制为 4096 个词元）。&#x20;

输入和输出词元都计入这些数量。例如，如果您的 API 调用在消息输入中使用了 10 个词元，并且您收到了消息输出中的 20 个词元，则将向您收取30个代币。&#x20;

要查看 API 调用使用了多少代币，请检查 API 响应中的 usage 字段（例如 response\['usage']\['total\_tokens']）。

&#x20;要查看文本字符串中有多少代币而不进行 API 调用，请使用 OpenAI 的 tiktoken Python 库。示例代码可在 OpenAI Cookbook 的指南《[如何使用 tiktoken 计算代币](https://github.com/openai/openai-cookbook/blob/main/examples/How\_to\_count\_tokens\_with\_tiktoken.ipynb)》 中找到。&#x20;

传递给 API 的每条消息都会消耗内容、角色和其他字段中的代币数量，再加上一些幕后格式化所需额外代币。这可能稍微改变未来情况。&#x20;

如果对话包含太多符号以适合模型的最大限制（例如 gpt-3.5-turbo 的超过4096 符号），则必须截断、省略或缩小文本直到其符合规定范围。

请注意，如果从消息输入删除一条消息，则该模型将失去所有相关知识点信息 还要注意非常长时间地谈话很可能会收到不完整回复。例如，在长度为4090 词元时, gpt-3.5-turbo 对话只能得出6个词元作为答案

## 指导聊天模型&#x20;

最佳实践的指导方法可能会随着模型版本的更新而改变。以下建议适用于 gpt-3.5-turbo-0301，未来的模型可能不适用。

许多对话都以系统消息开始，以温和地指导助手。例如，这是一个用于 ChatGPT 的系统消息之一：

你是 ChatGPT，由 OpenAI 训练的大型语言模型。请尽可能简明扼要地回答。知识截止时间：{knowledge\_cutoff}，当前日期：{current\_date}

总的来说，gpt-3.5-turbo-0301 不会过多地关注系统消息，因此重要的指令通常更适合放在用户消息中。

如果模型没有产生您想要的输出，请随时尝试迭代并尝试潜在的改进。您可以尝试以下方法：

* 更明确地说明您的指令&#x20;
* 指定您想要的答案格式&#x20;
* 要求模型在确定答案之前逐步思考或辩论利弊&#x20;

如需更多提示工程思路，请阅读[ OpenAI Cookbook 指南](https://github.com/openai/openai-cookbook/blob/main/techniques\_to\_improve\_reliability.md)以改善可靠性的技术。

除了系统消息之外，温度和最大标记数是许多选项中的两个，可用于影响聊天模型的输出。对于温度而言，较高的值（如 0.8）会使输出更随机，而较低的值（如 0.2）则会使其更加专注和确定性。在最大标记数的情况下，如果您想将响应限制为一定长度，则最大标记数可以设置为任意数。例如，如果将最大标记数值设置为 5，则可能会出现问题，因为输出将被截断，结果对用户来说没有意义。

## 聊天与完成

除了系统提示之外，温度和最大标记数是开发人员可以用来影响聊天模型输出的多种选项中的两个。对于温度，更高的值如0.8会使输出更加随机，而较低的值如0.2会使其更加集中和确定性。在最大标记数的情况下，如果您想将响应限制在某个长度内，最大标记数可以设置为任意数字。例如，如果您将最大标记数值设置为5，则会截断输出，结果对用户来说没有意义。

聊天与完成 由于gpt-3.5-turbo的性能类似于text-davinci-003，但每个词元的价格只有10%，因此我们建议大多数情况下使用gpt-3.5-turbo。

对于许多开发人员来说，过渡非常简单，只需重新编写和重新测试提示即可。

例如，如果您使用以下完成提示将英语翻译成法语：

```
将“{text}”翻译成法语。 
```

一个等效的聊天对话可以像这样：

```
[ 
{"role": "system", "content": "您是一个有帮助的助手，可以将英语翻译成法语。"}, 
{"role": "user", "content": "将以下英文文本翻译成法语：“{text}”"} 
]
```

甚至只需用户消息：

```
 [ {"role": "user", "content": "将以下英文文本翻译成法语：“{text}”"} ]
```



## FAQ

### GPT-3.5 Turbo是否支持微调？

不支持。截至2023年3月1日，您只能对基础GPT-3模型进行微调。有关如何使用微调模型的更多详细信息，请参阅微调指南。

### 您是否会存储通过API传递的数据？

截至2023年3月1日，我们会保留您的API数据30天，但不再使用通过API发送的数据来改善我们的模型。在我们的数据使用政策中了解更多信息。

### 如何添加内容审核层？

如果您想向Chat API的输出添加审核层，可以按照我们的审核指南进行操作，以防止显示违反OpenAI使用政策的内容。
