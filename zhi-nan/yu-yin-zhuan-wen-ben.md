# 语音转文本

学习如何将音频转换为文本。

## 介绍&#x20;

语音转文本 API 提供了两个端点，即基于我们最先进的开源大型-v2 Whisper 模型的转录和翻译。它们可以用于：&#x20;

* 将音频转录为任何语言。&#x20;
* 将音频翻译并转录成英语。&#x20;

目前文件上传限制为 25 MB，并支持以下输入文件类型：mp3、mp4、mpeg、mpga、m4a、wav 和 webm。&#x20;

## 快速入门&#x20;

### 转录&#x20;

转录 API 的输入是您要进行转录的音频文件以及所需输出格式的音频文字稿。我们目前支持多种输入和输出文件格式。

```python
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai
audio_file= open("/path/to/file/audio.mp3", "rb")
transcript = openai.Audio.transcribe("whisper-1", audio_file)
```

默认情况下，响应类型将是包含原始文本的 JSON。

```
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger.
....
}
```

要在请求中设置其他参数，您可以添加更多带有相关选项的--form行。例如，如果您想将输出格式设置为文本，则应添加以下行：

```
...
--form file=@openai.mp3 \
--form model=whisper-1 \
--form response_format=text
```

### 翻译

翻译API以任何支持的语言作为输入音频文件，并在必要时将音频转录成英文。这与我们的/Transcriptions端点不同，因为输出不是原始输入语言，而是被翻译成英文文本。

```python
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai
audio_file= open("/path/to/file/german.mp3", "rb")
transcript = openai.Audio.translate("whisper-1", audio_file)
```

在这种情况下，输入的音频是德语，输出的文本看起来像：

```
Hello, my name is Wolfgang and I come from Germany. Where are you heading today?

```

我们目前仅支持英语翻译。

## &#x20;支持的语言&#x20;

我们目前通过转录和翻译端点支持以下语言：

&#x20;南非荷兰语，阿拉伯语，亚美尼亚语，阿塞拜疆语，白俄罗斯语，波斯尼亚文，保加利亚文，加泰罗尼亚文，中文，克罗地亚文、捷克文、丹麦文、荷兰文、英国英语、爱沙尼亚文、芬兰文、法国法式英語, 加利西亞語, 德國語, 希臘語, 希伯來語, 印地語, 匈牙利語, 冰島icelandic 読音: \[ˈaɪsləndɪk], 印度尼西雅Indonesian 読音: \[indoneˈsia], 意大利Italian 読音: \[iːtæljən], 日本Japanese 読音: \[dʒæpəniːz], 卡纳达Kannada 読音: \[kʌn'na:dʌ] ,哈萨克Kazakh 読音:\[kɑzɑx] , 韩国Korean 读作：\[hanguk] ，拉脫維Latvian 读作：\[lætvijan] ，立陶宛Lithuanian 读作：\[liθu'einjən] ，马其顿Macedonian 读作：\[mækidouniən ] ，马来Malay 读作：\['meilei ] ，馬拉地Marathi 讀作:\[ma'rathi ],毛里求斯Maori 讀作:\[mauri ], 尼泊尔Nepali 讀作:\[ne'pa:l ],挪威Norwegian 讀作:\['no:wijiən ] ， 波斯Persian讀做\[persi'an ] , 波蘇尼Serbian讀做sǎrbijǝTagalog讀做tӕgӕ'lɔg，坦米爾Tamil讀做'tæmil, 泰Thai讀做\[tai], 土耳其Turkish讀健\[turki'sh], 烏Crainian(乌克兰)Ukrainian 讀健\[jukreinjǝn ], 烏Urdu(乌尔都)Urdu 讓你\[u:rdu:],越南Vietnamese (越南)Vietnamese 和威尔士Welsh。&#x20;

虽然底层模型是在98种不同的语言上进行了培训。但我们只列出了超过50%单词错误率（WER）的标准行业基准测试所支持的那些。该模型将返回未列出以上列表中的其他所有可能存在输入结果但质量会较低。&#x20;

## 更长输入

&#x20;默认情况下Whisper API仅支持小于25 MB 的文件。如果您有一个比这更长的音频文件，则需要将其分成每个小于25 MB 的块或使用压缩后格式。为了获得最佳性能，请避免在句子中间断开声音以避免丢失一些上下文字信息。 处理此问题的一种方法是使用PyDub开源Python软件包来拆分声频文件。

```python
from pydub import AudioSegment

song = AudioSegment.from_mp3("good_morning.mp3")

# PyDub handles time in milliseconds
ten_minutes = 10 * 60 * 1000

first_10_minutes = song[:ten_minutes]

first_10_minutes.export("good_morning_10.mp3", format="mp3")
```

OpenAI对于像PyDub这样的第三方软件的可用性或安全性不作任何保证。

## &#x20;提示&#x20;

您可以使用提示来提高Whisper API生成的转录质量。模型将尝试匹配提示的风格，因此如果提示也使用大写和标点符号，则更有可能使用它们。但是，当前的提示系统比我们其他语言模型要受限得多，并且仅提供对生成音频的有限控制。以下是一些示例，说明如何在不同情况下使用提示：&#x20;

1. 对于模型经常错误识别音频中特定单词或缩略语非常有帮助。例如，以下提示改善了DALL·E和GPT-3这些单词（以前被写成“GDP 3”和“DALI”）的转录。&#x20;

```
该转录涉及OpenAI开发类似DALL·E、GPT-3和ChatGPT等技术，并希望有一天构建一个造福人类所有人的AGI系统。
```

2. 为了保留分段文件的上下文，请使用先前片段的转录来引导模型。这将使转录更准确，因为模型将利用先前音频中相关信息。该模型只会考虑最后224个标记并忽略之前任何内容。
3. 有时候，在转录中可能会跳过标点符号。您可以通过使用包含标点符号简单提示来避免这种情况：&#x20;

```
你好，欢迎参加我的讲座。 
```

2. 该模型还可能在音频中省略常见填充词汇。如果您想在您的转录中保留填充词汇，则可以使用包含它们的指示：

```
 嗯... 让我想想, 呃... 好吧, 这就是我正在思考 的事情. 
```

3. 某些语言可以用不同方式书写，例如简体或繁体中文。默认情况下，该模型可能无法始终按照所需书写风格进行处理 。通过在首选书写风格上添加指示即可改进此问题.
