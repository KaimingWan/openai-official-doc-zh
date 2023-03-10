# 微调

学习如何为您的应用程序定制模型。

## 简介&#x20;

通过提供以下功能，Fine-tuning（微调）可以让您从API中提供的模型中获得更多：

比提示设计更高质量的结果 能够训练更多的例子，而这些例子无法适应提示 由于提示更短而节省的令牌 更低的延迟请求 GPT-3已经在开放互联网上的大量文本上进行了预训练。当给出一个只有几个例子的提示时，它通常可以直观地知道您正在尝试执行什么任务，并生成一个可信的完成结果。这通常被称为“少样本学习”。

Fine-tuning通过训练比提示中所能适应的更多的例子，从而在许多任务上获得更好的结果，从而改善了少样本学习。一旦模型经过微调，您就不需要在提示中提供示例了。这节省了成本并使延迟请求更低。

在高层次上，Fine-tuning包括以下步骤：

准备并上传训练数据 训练一个新的微调模型 使用您的微调模型 访问我们的定价页面，了解有关微调模型培训和使用的更多信息。

## 哪些模型可以进行Fine-tuning？

目前，Fine-tuning仅适用于以下基本模型：davinci、curie、babbage和ada。这些是没有任何训练后指令的原始模型（例如，text-davinci-003就有指令）。您还可以继续微调微调模型以添加其他数据，而无需从头开始。

## 安装

&#x20;我们建议使用我们的OpenAI命令行界面（CLI）。要安装此程序，请运行：

```
pip install --upgrade openai
```

（以下说明适用于版本0.9.4及以上。此外，OpenAI CLI需要Python 3。） 通过将以下行添加到您的shell初始化脚本（例如.bashrc、zshrc等）或在微调命令之前在命令行中运行它来设置OPENAI\_API\_KEY环境变量：

```
export OPENAI_API_KEY="<OPENAI_API_KEY>"
```

## 准备训练数据&#x20;

训练数据是教GPT-3说出你想要的话的方法。 您的数据必须是JSONL文档，其中每行都是与一个训练示例相对应的提示-完成对。您可以使用我们的CLI数据准备工具轻松将您的数据转换为此文件格式。

```
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
...
```

我们开发了一种工具，用于验证、提供建议和重新格式化您的数据：

```
openai tools fine_tunes.prepare_data -f <LOCAL_FILE>
```

该工具接受不同的格式，唯一的要求是它们包含一个提示列/键和一个完成列/键。您可以传递CSV、TSV、XLSX、JSON或JSONL文件，它将在指导您进行建议更改的过程后将输出保存到一个准备好进行Fine-tuning的JSONL文件中。

## 创建Fine-tuning模型&#x20;

以下假设您已经按照上述说明准备了训练数据。

使用OpenAI CLI启动Fine-tuning作业：

```
openai api fine_tunes.create -t <TRAIN_FILE_ID_OR_PATH> -m <BASE_MODEL>
```

其中，BASE\_MODEL是您要基于的基础模型的名称（ada、babbage、curie或davinci）。您可以使用suffix参数自定义Fine-tuning模型的名称。

运行上述命令会执行以下几个操作：

使用文件API上传文件（或使用已上传的文件） 创建Fine-tune作业 流式传输事件，直到作业完成（这通常需要几分钟，但如果队列中有许多作业或您的数据集很大，则可能需要几个小时） 每个Fine-tuning作业都是从一个基础模型开始的，默认为curie。模型的选择影响模型的性能和运行Fine-tuning模型的成本。您的模型可以是：ada、babbage、curie或davinci。请访问我们的价格页面了解有关Fine-tuning费率的详细信息。

在启动Fine-tuning作业后，可能需要一些时间才能完成。您的作业可能会排在我们系统中的其他作业后面，训练我们的模型可能需要几分钟或几个小时，具体取决于模型和数据集大小。如果由于任何原因中断事件流，您可以通过运行以下命令恢复它：

```
openai api fine_tunes.follow -i <YOUR_FINE_TUNE_JOB_ID>
```

## 使用fine-tuned模型&#x20;

当一个任务成功后，fine\_tuned\_model字段将被填充为模型的名称。现在，您可以将此模型指定为完成API的参数，并使用Playground进行请求。

在任务完成后，您的模型可能需要几分钟才能准备好处理请求。如果完成请求超时，则可能是因为您的模型仍在加载中。如果发生这种情况，请稍后再试几分钟。

您可以通过将模型名称作为完成请求的模型参数来开始发出请求：

OpenAI CLI:

```
openai api completions.create -m <FINE_TUNED_MODEL> -p <YOUR_PROMPT>
```

cURL

```
curl https://api.openai.com/v1/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": YOUR_PROMPT, "model": FINE_TUNED_MODEL}'
```

python:

```python
import openai
openai.Completion.create(
    model=FINE_TUNED_MODEL,
    prompt=YOUR_PROMPT)
```

Node.js

```
const response = await openai.createCompletion({
  model: FINE_TUNED_MODEL
  prompt: YOUR_PROMPT,
});
```

您可以继续在这些请求中使用所有其他完成参数，例如温度、频率惩罚、存在惩罚等，以对微调模型进行优化。&#x20;

## 删除微调模型&#x20;

要删除微调模型，您必须在组织内被指定为“所有者”。&#x20;

OpenAI CLI:

```bash
openai api models.delete -i <FINE_TUNED_MODEL>
```

cURL:

```bash
curl -X "DELETE" https://api.openai.com/v1/models/<FINE_TUNED_MODEL> \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

Python:

```python
import openai
openai.Model.delete(FINE_TUNED_MODEL)
```

## 准备数据集

Fine-tuning是一种创建针对特定用例的新模型的强大技术。在fine-tuning您的模型之前，我们强烈建议您阅读以下最佳实践和特定用例的指南。

### 数据格式

要对模型进行fine-tuning，您需要一组训练示例，每个示例都由一个单独的输入（"提示"）和其相关输出（"完成"）组成。这与使用我们的基础模型有明显不同，基础模型中您可能会输入详细的说明或多个示例。

* 每个提示应以固定的分隔符结尾，以通知模型提示何时结束并完成何时开始。一个通常很有效的简单分隔符是\n\n###\n\n。分隔符不应在任何提示中出现。&#x20;
* 由于我们的token化方法，每个完成都应以空格开头，大多数单词都会在前面加上空格。
* 每个完成都应以固定的停止序列结束，以通知模型完成何时结束。停止序列可以是\n、###或任何其他不出现在任何完成中的标记。&#x20;
* 对于推断，您应以与创建训练数据集时相同的方式格式化您的提示，包括相同的分隔符。还要指定相同的停止序列以正确截断完成。

### 一般最佳实践

微调在有更多高质量的例子时表现更好。为了微调一个比使用我们的基础模型和高质量提示表现更好的模型，您应该提供至少几百个高质量的例子，最好由人类专家审核过。从那里开始，性能往往会随着每倍增加示例数量而线性增加。增加示例数量通常是改善性能的最佳和最可靠方法。

分类器是入门最容易的模型。对于分类问题，我们建议使用ada，在微调后通常只比更强大的模型略差一点，同时速度和成本显著降低。&#x20;

如果您正在对预先存在数据集进行微调而不是从头编写提示，请务必手动检查数据以查找冒犯或不准确内容（如果可能），或者尽可能审查数据集中许多随机样本（如果它很大）。

### 具体指南&#x20;

微调可以解决各种问题，而最优的使用方式可能取决于您的具体用例。下面，我们列出了微调的最常见用例和相应的指南。

* 分类&#x20;
  * 模型是否发表了不真实的陈述？&#x20;
  * 情感分析&#x20;
  * 电子邮件分类&#x20;
* 条件生成&#x20;
  * 基于维基百科文章编写吸引人的广告&#x20;
  * 实体提取&#x20;
  * 客服聊天机器人&#x20;
  * 基于技术属性列表的产品描述&#x20;

### 分类&#x20;

在分类问题中，每个提示输入应该被分类到预定义的类别之一。对于这种类型的问题，我们建议：

* 在提示的末尾使用分隔符，例如 \n\n###\n\n。当您最终向模型发出请求时，也要附加此分隔符。
* &#x20;选择映射到单个词元的类别。在推断时，指定 max\_tokens=1，因为您只需要分类的第一个词元。
* 确保提示+完成不超过2048个词元，包括分隔符。&#x20;
* 针对每个类别至少有 \~100 个示例。&#x20;
* 在使用模型时，如果需要类别日志概率，则可以指定 logprobs=5（用于5个类别）。&#x20;
* 确保用于微调的数据集在结构和任务类型上与模型将要使用的相似。&#x20;

#### 案例研究：模型是否发表了不真实的陈述？&#x20;

假设您想确保网站广告的文本提到了正确的产品和公司。换句话说，您希望确保模型没有捏造内容。您可以微调一个分类器来过滤不正确的广告。

```
{"prompt":"公司：BHFF保险\n产品：全方位保险\n广告：满足您所有保险需求的一站式服务！\n支持:", "completion":" 是"}
{"prompt":"公司：阁楼改建专家\n产品：-\n广告：几周内拥有整齐的牙齿！\n支持:", "completion":" 否"}
```

在上面的示例中，我们使用了一个结构化输入，其中包含公司名称、产品和相关广告。作为分隔符，我们使用了\nSupported:来清楚地将提示与完成分开。有足够数量的示例时，分隔符并不会产生太大影响（通常少于0.4%），只要它不出现在提示或完成中。

&#x20;对于这种用例，我们微调了ada模型，因为它速度更快、更便宜，并且性能可与较大的模型相媲美，因为这是一个分类任务。

&#x20;现在我们可以通过发出“Completion”请求来查询我们的模型。

```
curl https://api.openai.com/v1/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -d '{
  "prompt": "Company: Reliable accountants Ltd\nProduct: Personal Tax help\nAd:Best advice in town!\nSupported:",
  "max_tokens": 1,
  "model": "YOUR_FINE_TUNED_MODEL_NAME"
}'
```

这将返回是或否。

#### 案例研究：情感分析

假设您想要获得一条推文的积极或消极程度，数据集可能如下所示：

```
{"prompt":"对新iPhone感到非常高兴！->", "completion":"积极"}
{"prompt":"@lakers 连续第三个晚上让人失望 https://t.co/38EFe43 ->", "completion":"消极"}
```

一旦模型微调完成，您可以通过在完成请求上设置logprobs=2来获取第一个完成token的对数概率。正类别的概率越高，情感相对就越高。 现在我们可以通过发出完整请求来查询我们的模型。

```
curl https://api.openai.com/v1/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -d '{
  "prompt": "https://t.co/f93xEd2 Excited to share my latest blog post! ->",
  "max_tokens": 1,
  "model": "YOUR_FINE_TUNED_MODEL_NAME"
}'
```

将返回：

```json
{
  "id": "cmpl-COMPLETION_ID",
  "object": "text_completion",
  "created": 1589498378,
  "model": "YOUR_FINE_TUNED_MODEL_NAME",
  "choices": [
    {
      "logprobs": {
        "text_offset": [
          19
        ],
        "token_logprobs": [
          -0.03597255
        ],
        "tokens": [
          " positive"
        ],
        "top_logprobs": [
          {
            " negative": -4.9785037,
            " positive": -0.03597255
          }
        ]
      },

      "text": " positive",
      "index": 0,
      "finish_reason": "length"
    }
  ]
}
```

#### 案例研究：电子邮件分类&#x20;

假设您想将收到的电子邮件归类为大量预定义的类别之一。对于大量类别的分类，我们建议您将这些类别转换为数字，这在 \~500 个类别以下效果良好。我们观察到，在数字前添加一个空格有时会略微提高性能，因为它可以进行分词处理。您可能希望按以下方式构建培训数据：

```
{"prompt":"主题：<email_subject>\n来自：<customer_name>\n日期：<date>\n内容：<email_body>\n\n###\n\n", "completion":"<numerical_category>"}
```

例如：

```
{"prompt":"主题：更新我的地址\n发件人：乔·多伊\n收件人：support@ourcompany.com\n日期：2021年06月03日\n内容：\n您好，\n我想要更新我的账单地址以匹配我的送货地址。\n请在完成后告知我。\n谢谢，\n乔。\n###\n", "completion":"4"}
```

在上面的示例中，我们使用了一个最多包含2043个词元的传入电子邮件作为输入。（这允许4个标记分隔符和一个标记完成，总计为2048。）作为分隔符，我们使用了\n\n###\n\n，并删除了电子邮件中任何出现的###。

### 条件生成&#x20;

条件生成是一个问题，需要在给定某种输入的情况下生成内容。这包括改写、摘要、实体提取、根据规格编写产品描述、聊天机器人等。对于这种类型的问题，我们建议：

* &#x20;在提示末尾使用分隔符，例如\n\n###\n\n。
* 记得在最终向模型发出请求时也附加此分隔符。&#x20;
* 在完成末尾使用结束标记，例如END。&#x20;
* 记得将结束标记添加为推理期间的停止序列，例如stop=\[" END"]。&#x20;
* 目标至少达到\~500个示例&#x20;
* 确保提示+完成不超过2048个词元（包括分隔符）
* &#x20;确保示例具有高质量并遵循相同的所需格式&#x20;
* 确保用于微调的数据集与模型将要用于的任务结构和类型非常相似 对于这些用例，使用较低学习率和仅1-2个时代通常效果更好

#### &#x20;案例研究：基于维基百科文章撰写引人入胜的广告&#x20;

这是一种生成性用例，因此您希望确保提供的样本具有最高质量，因为经过微调的模型将尝试模仿给定示例的风格（和错误）。一个良好起点大约是500个示例。样本数据集可能如下所示：

```
{"prompt":"<产品名称>\n<Wikipedia 描述>\n\n###\n\n",
"completion":" <吸引人的广告> 结束"}
```

例如：

```
{"prompt":"三星Galaxy Feel\n三星Galaxy Feel是由三星电子专门为日本市场开发的Android智能手机。该手机于2017年6月发布，由NTT Docomo销售。它运行在Android 7.0（Nougat）上，拥有4.7英寸显示屏和3000毫安时电池。\n软件\n三星Galaxy Feel运行在Android 7.0（Nougat）上，但可以后期更新到Android 8.0（Oreo）。\n硬件\n三星Galaxy Feel配备了一块4.7英寸超级AMOLED高清显示屏、1600万像素后置摄像头和500万像素前置摄像头。它还配备了一个3000毫安时电池、1.6 GHz八核ARM Cortex-A53 CPU和一个ARM Mali-T830 MP1 700 MHz GPU。它内置32GB存储空间，可通过microSD扩展至256GB。除了其软件和硬件规格外，三星还推出了独特的手机壳孔以适应日本人个性化移动电话的偏好。 Galaxy Feel的电池也被誉为主要卖点之一，因为市场青睐续航时间更长的手持设备。该设备还具有防水功能，并支持使用单独销售天线进行1seg数字广播。\n\n###\n\n",
"completion":"正在寻找一款全能型智能手机？不用再找了！来试试我们最新款的Samsung Galaxy Feel吧！这款机身轻薄、设计精美的智能手机拥有高质量图片和视频功能，并且获得过奖项，在续航方面表现优异。END"}
```

在这里，我们使用了多行分隔符，因为维基百科文章包含多个段落和标题。我们还使用了一个简单的结束标记，以确保模型知道何时完成。&#x20;

#### 案例研究：实体提取&#x20;

这类似于语言转换任务。为了提高性能，最好按字母顺序或与原始文本中出现的顺序相同地排序不同的提取实体。这将帮助模型跟踪需要按顺序生成的所有实体。数据集可能如下所示：

```
{"prompt":"<任意文本，例如新闻文章>\n\n###\n\n", "completion":"<实体列表，用换行符分隔> END"}
```

例如：

```
{"prompt":"由于新冠病例上升和对所谓的印度变异体尼泊尔突变的担忧，自周二起葡萄牙将被从英国的绿色旅行名单中移除。它将加入琥珀名单，这意味着度假者不应该前往，并且回国人员必须隔离10天...\n\n###\n\n",
"completion":" 葡萄牙\n英国\n尼泊尔突变\n印度变异体 结束"}
```

多行分隔符效果最佳，因为文本可能包含多行。理想情况下，输入提示的类型应具有高度的多样性（新闻文章、维基百科页面、推特、法律文件等），这反映了在提取实体时可能遇到的文本类型。

#### &#x20;案例研究：客户支持聊天机器人

&#x20;聊天机器人通常会包含与对话相关的上下文（订单详情）、迄今为止对话的摘要以及最近的消息。对于这种用例，同一过去的对话可以生成数据集中多个行，在每次代理生成完成时都带有稍微不同的上下文。由于它可能涉及不同类型的请求和客户问题，因此该用例将需要几千个示例。为确保性能高质量，我们建议审核对话样本以确保代理消息质量。摘要可以使用单独调整后文字转换模型来生成。数据集如下所示：

```
{"prompt":"摘要：<交互至今的概述>\n\n具体信息：<例如自然语言中的订单详情>\n\n###\n\n客户：<消息1>\n代理人：<响应1>\n客户：<消息2>\n代理人：", "completion":"<响应2> \ n"}
{"prompt":"摘要：<交互至今的概述>\n\n具体信息：<例如自然语言中的订单详情>\n\n###\n\n客户：<消息1> \ n代理人： <响应1> \ n客户： <消息2> \ n代理人： <响应2> \ n客户： <消息3> \ n代理人:", "completion": "<response3 >\ N"}
```

在这里，我们有意地将不同类型的输入信息分开，但保持了客户代理对话框在提示和完成之间的相同格式。所有完成只应由代理人完成，并且在进行推断时可以使用 \n 作为停止序列。&#x20;

#### 案例研究：基于技术属性清单的产品描述&#x20;

在这里，将输入数据转换为自然语言非常重要，这可能会导致更好的性能。例如以下格式：

```
{"prompt":"物品=手提包，颜色=军绿色，价格=$99，尺码=S->", "completion":"这款时尚的小绿色手提包将为您的造型增添独特的风格，而不会花费您太多钱。"}
```

不会像如下方式一样有效：

```
{"prompt":"物品是手提包。颜色为军绿色。价格属于中档。尺寸较小。->", "completion":"这款时尚的小型绿色手提包将为您的造型增添独特的风格，而不会花费太多钱。"}
```

为了获得高性能，请确保完成是基于提供的描述。如果经常查阅外部内容，则以自动化方式添加此类内容将改善性能。如果描述基于图像，则使用算法提取图像的文本描述可能会有所帮助。由于完成只有一句话长，因此我们可以在推理过程中使用“。”作为停止序列。

## 高级用法

### 自定义模型名称

您可以使用后缀参数将最多40个字符的后缀添加到您的微调模型名称中。

OpenAI CLI:

```
openai api fine_tunes.create -t test.jsonl -m ada --suffix "custom model name"
```

生成的名称将是：

```
ada:ft-your-org:custom-model-name-2022-02-15-04-21-04
```

### 分析您的微调模型

我们为每个作业附加一个结果文件，一旦它完成，它将被列出。检索微调时，结果文件ID将被列出，并在查看微调事件时列出。您可以下载这些文件：

OpenAI CLI:

```bash
openai api fine_tunes.results -i <YOUR_FINE_TUNE_JOB_ID>
```

CURL:

```
curl https://api.openai.com/v1/files/$RESULTS_FILE_ID/content \
  -H "Authorization: Bearer $OPENAI_API_KEY" > results.csv
```

\_results.csv文件包含每个训练步骤的一行，其中一步骤指的是在一批数据上进行前向和后向传递的操作。除了步数，每行还包含以下与该步骤相对应的字段：

* elapsed\_tokens：模型迄今为止看到的词元数（包括重复）&#x20;
* elapsed\_examples：模型迄今为止看到的示例数（包括重复），其中一个示例是批次中的一个元素。例如，如果batch\_size = 4，则每个步骤将使elapsed\_examples增加4。
* training\_loss：训练批次的损失&#x20;
* training\_sequence\_accuracy：在训练批次中，模型预测的标记与真实的标记完全匹配的完成百分比。例如，如果您的数据包含完成\[\[1, 2]，\[0, 5]，\[4, 2]]，并且模型预测\[\[1, 1]，\[0, 5]，\[4, 2]]，则准确性将为2/3 = 0.67&#x20;
* training\_token\_accuracy：模型正确预测的训练批次中的标记百分比。例如，如果您的数据包含完成\[\[1, 2]，\[0, 5]，\[4, 2]]，并且模型预测\[\[1, 1]，\[0, 5]，\[4, 2]]，则准确性将为5/6 = 0.83

### 分类特定的度量标准&#x20;

我们还提供了生成结果文件中的其他分类特定度量标准的选项，例如准确性和加权F1分数。这些指标定期针对整个验证集进行计算，并在微调结束时计算。您将在结果文件中看到它们作为其他列。

要启用此功能，请设置--compute\_classification\_metrics参数。另外，您必须提供验证文件，并设置classification\_n\_classes参数（用于多类分类）或classification\_positive\_class参数（用于二进制分类）。

OpenAI CLI:

```bash
# For multiclass classification
openai api fine_tunes.create \
  -t <TRAIN_FILE_ID_OR_PATH> \
  -v <VALIDATION_FILE_OR_PATH> \
  -m <MODEL> \
  --compute_classification_metrics \
  --classification_n_classes <N_CLASSES>

# For binary classification
openai api fine_tunes.create \
  -t <TRAIN_FILE_ID_OR_PATH> \
  -v <VALIDATION_FILE_OR_PATH> \
  -m <MODEL> \
  --compute_classification_metrics \
  --classification_n_classes 2 \
  --classification_positive_class <POSITIVE_CLASS_FROM_DATASET>
```

\
如果你设置了--compute\_classification\_metrics，以下指标将在你的结果文件中显示：

#### 对于多类分类：

* classification/accuracy：准确率&#x20;
* classification/weighted\_f1\_score：加权F1分数

#### 对于二元分类

以下指标基于分类阈值为0.5（即当概率> 0.5时，将一个示例分类为属于正类）。

* classification/accuracy：准确率
* &#x20;classification/precision：精确率&#x20;
* classification/recall：召回率
* &#x20;classification/f{beta}：F-beta分数&#x20;
* classification/auroc：AUROC
* &#x20;classification/auprc：AUPRC

请注意，这些评估假定您使用文本标签来表示分词为单个词元的类，如上所述。如果这些条件不成立，您得到的数字可能会错误。

### 验证

&#x20;您可以为验证保留一些数据。验证文件与训练文件具有完全相同的格式，您的训练和验证数据应互不重叠。

如果您在创建微调作业时包含验证文件，则生成的结果文件将包括在训练期间定期评估微调模型在验证数据上的表现。\
OpenAI CLI:

```bash
openai api fine_tunes.create -t <TRAIN_FILE_ID_OR_PATH> \
  -v <VALIDATION_FILE_ID_OR_PATH> \
  -m <MODEL>
```

如果您提供了一个验证文件，在训练期间我们会定期计算验证数据批次上的指标。在结果文件中，您将看到以下额外的指标：

* validation\_loss：验证批次的损失值。&#x20;
* validation\_sequence\_accuracy：验证批次中完成度百分比，其中模型预测的标记与真实标记完全匹配。例如，如果您的数据包含完成度\[\[1，2]，\[0，5]，\[4，2]]，而模型预测为\[\[1，1]，\[0，5]，\[4，2]]，则准确率为2/3 = 0.67（假设batch\_size为3）。&#x20;
* validation\_token\_accuracy：模型正确预测的验证批次中标记的百分比。例如，如果您的数据包含完成度\[\[1，2]，\[0，5]，\[4，2]]，而模型预测为\[\[1，1]，\[0，5]，\[4，2]]，则准确率为5/6 = 0.83（假设batch\_size为3）。&#x20;

### 超参数&#x20;

我们已选择默认超参数，可在各种用例中运行良好。唯一需要的参数是训练文件。

但是，调整用于微调的超参数通常可以产生生成更高质量输出的模型。特别是，您可能想配置以下内容：

* model：要微调的基础模型的名称。您可以选择“ada”、“babbage”、“curie”或“davinci”中的一个。要了解有关这些模型的更多信息，请参见模型文档。&#x20;
* n\_epochs - 默认值为4。训练模型的时期数。一个时期指的是一次完整的通过训练数据集的循环。&#x20;
* batch\_size - 默认值为训练集中示例数的约0.2％，上限为256。批处理大小是用于训练单个前向和后向传递的训练示例数。通常，我们发现更大的批量大小对于更大的数据集效果更好。
* learning\_rate\_multiplier - 默认值为0.05、0.1或0.2，具体取决于最终批量大小。微调学习率是用于预训练的原始学习率乘以此乘数。我们建议尝试在0.02到0.2的范围内的值，以查看哪个产生最佳结果。从经验上看，我们发现更大的学习率往往在处理更大的批量大小时表现更好。&#x20;
* compute\_classification\_metrics - 默认值为False。如果为True，则针对分类任务的微调，在每个时期结束时在验证集上计算分类特定的指标（准确性、F-1分数等）。 要配置这些额外的超参数，请通过OpenAI CLI的命令行标志传递它们，例如：

```
openai api fine_tunes.create \
  -t file-JD89ePi5KMsB3Tayeli5ovfW \
  -m ada \
  --n_epochs 1
```

从已经进行微调的模型继续微调 如果您已经对模型进行了微调，并且现在有额外的训练数据需要加入，您可以从模型进行继续微调。这样创建的模型能够从所有训练数据中学习，而不需要重新从头开始训练。

为此，在创建新的微调作业时，传递微调模型名称（例如 -m curie:ft-\<org>-\<date>）。其他训练参数无需更改，但是如果您的新训练数据比以前的训练数据小很多，您可能会发现将learning\_rate\_multiplier减少2到4倍是有用的。

### 权重和偏置&#x20;

您可以将微调结果与Weights & Biases同步，以跟踪实验、模型和数据集。

要开始使用，您需要一个Weights & Biases帐户和一个付费的OpenAI计划。为确保您使用的是最新版本的openai和wandb，请运行:

```bash
pip install --upgrade openai wandb
```

要将您的微调与Weights＆Biases同步，请运行：

```bash
openai wandb sync
```

## 示例笔记本&#x20;

### 分类&#x20;

[finetuning-classification.ipynb ](https://github.com/openai/openai-cookbook/blob/main/examples/Fine-tuned\_classification.ipynb)

此笔记本将演示如何微调模型，以对输入文本是否与棒球或曲棍球相关进行分类。我们将在笔记本中执行以下四个步骤：&#x20;

* 数据探索将概述数据源和示例的外观。
* 数据准备将把我们的数据源转换为可用于微调的jsonl文件。&#x20;
* 微调将启动微调作业并解释所得到的模型性能。&#x20;
* 使用该模型将演示如何向经过微调的模型发出请求以获取预测结果。

### 回答问题

[olympics-1-collect-data.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/fine-tuned\_qa/olympics-1-collect-data.ipynb)

[olympics-2-create-qa.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/fine-tuned\_qa/olympics-2-create-qa.ipynb)

[olympics-3-train-qa.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/fine-tuned\_qa/olympics-3-train-qa.ipynb)

该项目的想法是创建一个问答模型，基于提供的几段文本。基于GPT-3模型在回答问题时表现良好，当答案包含在段落中时，但如果答案不包含在其中，则基础模型往往会尽力回答，导致混淆的答案。 为了创建仅在有足够上下文情况下才回答问题的模型，我们首先创建了一个基于文本段落的问题和答案数据集。为了训练模型只有当存在答案时才回答，在这些情况下我们还添加对抗性示例，其中问题与上下文不匹配。在这些情况下，我们要求模型输出“没有足够的上下文来回答问题”。 我们将通过三个笔记本执行此任务：&#x20;

* 第一个笔记本专注于收集最近数据，在预训练期间GPT-3没有看到过这些数据。 我们选择了2020年奥运会（实际上发生在2021年夏季）作为主题，并下载了713个独特页面。 我们按单独部分组织数据集，并将其用作询问和回复问题的背景。&#x20;
* 第二个笔记本将利用Davinci-instruct根据Wikipedia章节提出一些问题，并根据该章节回答回应那些问题。
* &#x20;第三个笔记本将利用上下文、问句和解释对数据集来额外地创造对抗性问句和背景对, 在这种情况下, 该模型被提示以"无足够背景信息来解析此类问题" 来进行响应. 我们还将训练鉴别器模型, 以预测是否可以根据环境或其他因素来解析某一类特定类型或者所有类型 的相关内容.
