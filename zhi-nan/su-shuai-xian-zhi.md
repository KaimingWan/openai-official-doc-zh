# 速率限制

## 概述&#x20;

### 什么是速率限制？&#x20;

速率限制是API对用户或客户端在指定时间内访问服务器的次数施加的限制。&#x20;

### 为什么我们有速率限制？

&#x20;速率限制是API的常见实践，它们出于几个不同的原因而被设置：

* &#x20;它们有助于防止滥用或误用API。例如，恶意行为者可能会通过请求来淹没API，试图使其超载或导致服务中断。通过设置速率限制，OpenAI可以防止这种活动发生。
* 速率限制有助于确保每个人都能公平地访问API。如果一个人或组织进行过多的请求，可能会拖累其他所有人使用API。通过调节单个用户可以进行的请求数量，OpenAI确保最多数量的人有机会使用API而不经历减缓。&#x20;
* 速率限制可以帮助OpenAI管理其基础设施上的总负载。如果对API的请求急剧增加，则可能会给服务器带来压力并导致性能问题。通过设置速率限制，OpenAI可以帮助所有用户维护平稳一致体验。

&#x20;请完整阅读本文档以更好地了解OpenAI 的 速度极值系统如何工作。我们提供代码示例和处理常见问题所需解决方案，请在填写“极值增长申请表”之前遵循此指南，并详细说明如何在最后一部分填写该表格。&#x20;

### 我们 API 的极值是什么？

我们根据使用特定端点以及您拥有哪种类型账户，在组织级别而非用户级别强化极值控管 。 极值按两种方式测量：RPM（每分钟请求数）和TPM（每分钟词元数）。下表突出显示了我们 API 的默认极值 ，但这些极值可根据您的用例在填写“Rate Limit Increase Request” 表格后进行增加 。

TPM（每分钟标记数）单位因模型而异：

| 类型      | 1 TPM 等价于 |
| ------- | --------- |
| davinci | 1 词元/分钟   |
| curie   | 25 词元/分钟  |
| babbage | 100 词元/分钟 |
| ada     | 200 词元/分钟 |

从实际角度来看，这意味着您可以每分钟向ada模型发送大约200倍的令牌，而相对于davinci模型则更多。

|                  | TEXT & EMBEDDING                    | CHAT                               | CODEX                          | EDIT                            | IMAGE           | AUDIO  |
| ---------------- | ----------------------------------- | ---------------------------------- | ------------------------------ | ------------------------------- | --------------- | ------ |
| 免费试用用户           | <p>•20 RPM <br>•150,000 TPM</p>     | <p>•20 RPM <br>•40,000 TPM</p>     | <p>•20 RPM <br>•40,000 TPM</p> | <p>•20 RPM <br>•150,000 TPM</p> | 50 images / min | 50 RPM |
| 按使用量付费用户（前48小时）  | <p>•60 RPM <br>•250,000 TPM*</p>    | <p>•60 RPM <br>•60,000 TPM*</p>    | <p>•20 RPM <br>•40,000 TPM</p> | <p>•20 RPM <br>•150,000 TPM</p> | 50 images / min | 50 RPM |
| 按使用量付费用户（48小时以后） | <p>•3,500 RPM <br>•350,000 TPM*</p> | <p>•3,500 RPM <br>•90,000 TPM*</p> | <p>•20 RPM <br>•40,000 TPM</p> | <p>•20 RPM <br>•150,000 TPM</p> | 50 images / min | 50 RPM |

需要注意的是，速率限制可以由任一选项触发，取决于哪个先发生。例如，您可能会向Codex端点发送20个请求，并仅使用100个代币来填充您的限制，即使在这些20个请求中没有发送40k代币。

### 速率限制如何工作？&#x20;

如果您的速率限制为每分钟60个请求和每分钟150k davinci代币，则将受到两者之一的限制，无论哪种情况先发生。例如，如果您的最大请求数/分为60，则应能够每秒发送1个请求。如果您每800毫秒发送1次请求，在达到速率限制后，只需让程序休眠200毫秒即可再次发送一个请求；否则后续请求将失败。对于默认值3,000 requests/min，默认值下客户可以有效地每20ms或0.02秒发送1次请求。

### &#x20;如果我遇到了速率限制错误会怎样？&#x20;

速率限制错误看起来像这样： Rate limit reached for default-text-davinci-002 in organization org-{id} on requests per min. Limit: 20.000000 / min. Current: 24.000000 / min. 如果你遇到了速度上线问题，则意味着你在短时间内进行了过多的申请，并且API拒绝履行进一步申请直至经过指定时间。&#x20;

### 速度上限与max\_tokens&#x20;

我们提供的每种模型都有一个固定数量的token可以作为输入传递给它们进行处理。不能增加模型接收标记数目上界。例如： 如果使用text-ada-001，则可以向该模型发送最多2048词元每个请求。&#x20;

## 错误处置&#x20;

### 我可以采取哪些措施来减轻此类问题？

&#x20;OpenAI Cookbook有一个[Python笔记本](https://github.com/openai/openai-cookbook/blob/main/examples/How\_to\_handle\_rate\_limits.ipynb)详细说明如何避免出现频率极高（rate limit）错误。

&#x20;当提供编程式访问、批量处理功能和自动化社交媒体发布时，请谨慎考虑 - 可以考虑仅针对信任客户启用这些功能。

&#x20;为防止自动化和高容量滥用，在指定时间范围内（日、周或月），设置单个用户使用量上界 。 对于超出此上界用户，请考虑实施硬性规定或手动审核流程。&#x20;

### 通过指数回退重试&#x20;

避免频繁调用API方法导致频繁报错也很简单——随机等待并重新尝试调用API方法就好了！具体做法是：当 API 返回“429 Too Many Requests”状态码时暂停执行代码片段，并根据当前已经重试过几次计算出等待时间 t ，然后再重新尝试调用 API 方法；若依旧返回“429 Too Many Requests”状态码则再暂停 t 秒钟之后重复以上操作…… 直至成功获取数据！

&#x20;指数回退意味着在命中第一个 rate limit 错误时执行短暂休眠并重试不成功的 request 。 如果 request 仍未成功，则增加 sleep 长度并重复该过程。 这将持续到 request 成功或达到最大重试次数为止。 此方法具有许多优点：&#x20;

* 自动重试意味着您可以从 speed limit errors 中恢复而不会崩溃或丢失数据&#x20;
* 指数回退意味着首先尝试快捷方式retry ，同时仍然从较长延迟中获益retry ，因此前几次 retry 失败。&#x20;
* 添加随机抖动以延迟 retries 不同时刻击中所有 retries 的效果。

请注意，不成功的请求会影响您每分钟的限制，因此持续重新发送请求是行不通的。 以下是一些使用指数退避的 Python 示例解决方案。

#### 示例1：使用Tenacit库

Tenacity是一个Apache 2.0许可的通用重试库，使用Python编写，旨在简化将重试行为添加到几乎任何内容的任务。要向您的请求添加指数退避，请使用tenacity.retry装饰器。下面的示例使用tenacity.wait\_random\_exponential函数向请求添加随机指数退避。

```python
import openai
from tenacity import (
    retry,
    stop_after_attempt,
    wait_random_exponential,
)  # for exponential backoff
 
@retry(wait=wait_random_exponential(min=1, max=60), stop=stop_after_attempt(6))
def completion_with_backoff(**kwargs):
    return openai.Completion.create(**kwargs)
 
completion_with_backoff(model="text-davinci-003", prompt="Once upon a time,")
```

请注意，Tenacity库是第三方工具，OpenAI不对其可靠性或安全性做出任何保证。

#### 示例2：使用backoff库

另一个提供退避和重试功能修饰符的Python库是backoff：

```python
import backoff 
import openai 
@backoff.on_exception(backoff.expo, openai.error.RateLimitError)
def completions_with_backoff(**kwargs):
    return openai.Completion.create(**kwargs)
 
completions_with_backoff(model="text-davinci-003", prompt="Once upon a time,")
```

#### 示例3：手动实现指数回退

如果您不想使用第三方库，可以按照此示例实现自己的退避逻辑：

```python
# imports
import random
import time
 
import openai
 
# define a retry decorator
def retry_with_exponential_backoff(
    func,
    initial_delay: float = 1,
    exponential_base: float = 2,
    jitter: bool = True,
    max_retries: int = 10,
    errors: tuple = (openai.error.RateLimitError,),
):
    """Retry a function with exponential backoff."""
 
    def wrapper(*args, **kwargs):
        # Initialize variables
        num_retries = 0
        delay = initial_delay
 
        # Loop until a successful response or max_retries is hit or an exception is raised
        while True:
            try:
                return func(*args, **kwargs)
 
            # Retry on specific errors
            except errors as e:
                # Increment retries
                num_retries += 1
 
                # Check if max retries has been reached
                if num_retries > max_retries:
                    raise Exception(
                        f"Maximum number of retries ({max_retries}) exceeded."
                    )
 
                # Increment the delay
                delay *= exponential_base * (1 + jitter * random.random())
 
                # Sleep for the delay
                time.sleep(delay)
 
            # Raise exceptions for any errors not specified
            except Exception as e:
                raise e
 
    return wrapper
    
@retry_with_exponential_backoff
def completions_with_backoff(**kwargs):
    return openai.Completion.create(**kwargs)
```

再次声明，OpenAI 对此解决方案的安全性或效率不作任何保证，但它可以成为您自己解决方案的良好起点。

### 批量请求

&#x20;OpenAI API 对每分钟的请求数和令牌数有单独的限制。&#x20;

如果您达到了每分钟请求次数的限制，但是在每分钟令牌方面有可用容量，则可以将多个任务分批处理到每个请求中，以增加吞吐量。这将允许您处理更多的令牌，特别是对于我们较小的模型。&#x20;

发送一批提示与正常 API 调用完全相同，只需将字符串列表传递给 prompt 参数即可。

#### 不使用批处理的例子：

```python
import openai
 
num_stories = 10
prompt = "Once upon a time,"
 
# serial example, with one story completion per request
for _ in range(num_stories):
    response = openai.Completion.create(
        model="curie",
        prompt=prompt,
        max_tokens=20,
    )
    # print story
    print(prompt + response.choices[0].text)
```

使用批处理的例子

```python
import openai  # for making OpenAI API requests
 
 
num_stories = 10
prompts = ["Once upon a time,"] * num_stories
 
# batched example, with 10 story completions per request
response = openai.Completion.create(
    model="curie",
    prompt=prompts,
    max_tokens=20,
)
 
# match completions to prompts by index
stories = [""] * len(prompts)
for choice in response.choices:
    stories[choice.index] = prompts[choice.index] + choice.text
 
# print stories
for story in stories:
    print(story)
```

> 警告：响应对象可能不会按提示的顺序返回完成情况，因此请始终记住使用索引字段将响应与提示匹配。

## 请求增加&#x20;

### 我应该在什么时候考虑申请速率限制增加？&#x20;

我们的默认速率限制有助于最大化稳定性并防止滥用我们的API。我们会增加限制以启用高流量应用程序，因此申请速率限制增加的最佳时间是当您认为您拥有必要的流量数据来支持提高速率限制的强有力理由时。没有支持数据的大幅度速率限制增加请求不太可能被批准。如果您正在准备产品发布，请通过10天分阶段发布获得相关数据。

请记住，速率限制增加有时需要7-10天，因此如果存在支持当前增长数字将达到您的速率限制所需数据，则尽早计划并提交是明智之举。&#x20;

### 我的速率限制增加请求会被拒绝吗？

&#x20;一个常见原因是缺乏证明其合理性所需数据而导致拒绝。下面提供了数值示例，展示如何最好地支持一个速率上升请求，并尽力批准所有符合安全策略和显示支持数据要求的请求。我们致力于使开发人员能够使用我们的API进行规模化和成功。

### &#x20;我已经为我的文本/代码API实现了指数退避算法，但仍然出现错误。如何提高我的频次上线？

&#x20;目前，我们不支持提高免费测试端点（例如编辑端点）等功能。 我们也不会提高ChatGPT频次上线，但你可以参与ChatGPT专业版访问列表。

我们知道受到频次上线约束可能带来多大挫败感，并且很想为每个人都提高默认值。 但是由于共享容量约束，在Rate Limit Increase Request表单中只能批准付费客户证明需要通过审查后方可进行频次上线调整 。 为了帮助评估您真正需要哪些内容，请在“分享需求证据”部分中提供关于当前使用情况或基于历史用户活动预测 的统计信息 。 如果没有这些信息，则建议采取逐步释放方法：首先以当前比例释放服务给一小部分用户，在10个工作日内收集使用情况数据 ，然后根据该数据提交正式频次上线调整请求以供审核和批准。&#x20;

如果您提交了申请并获得批准，则在7-10个工作日内通知您审批结果。 以下是填写此表格的一些示例：

| 模型         | 预估词元数/分钟 | 预估请求数 | 用户数    | 需要的证据                                                                         | 1小时最大吞吐成本 |
| ---------- | -------- | ----- | ------ | ----------------------------------------------------------------------------- | --------- |
| DALL-E API | N/A      | 50    | 1000   | 我们的应用目前正在生产中，根据过去的流量情况，我们每分钟大约发出10个请求。                                        | $60       |
| DALL-E API | N/A      | 150   | 10,000 | 我们的应用在App Store中越来越受欢迎，我们开始遇到速率限制。我们能否获得默认限制的三倍，即每分钟50个图像？如果需要更多，我们将提交新表格。谢谢！ | $180      |
