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



```python
from pydub import AudioSegment

song = AudioSegment.from_mp3("good_morning.mp3")

# PyDub handles time in milliseconds
ten_minutes = 10 * 60 * 1000

first_10_minutes = song[:ten_minutes]

first_10_minutes.export("good_morning_10.mp3", format="mp3")
```
