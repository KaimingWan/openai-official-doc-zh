# 如何构建一个能够回答关于你的网站问题的人工智能

本教程演示了一个简单的网站爬取示例（在此示例中，是 OpenAI 网站），使用嵌入式(embedding) API 将爬取的页面转换为嵌入式，并创建基本搜索功能，允许用户提出有关embedding信息的问题。这旨在成为更复杂应用程序的起点，这些应用程序利用自定义知识库。

## 开始

一些Python和GitHub的基本知识对于这个教程是有帮助的。在开始之前，请确保设置了OpenAI API密钥并完成了快速入门教程。这将使您对如何充分利用API有一个良好的直觉。 Python作为主要编程语言，与OpenAI、Pandas、transformers、NumPy和其他流行包一起使用。如果您在学习此教程时遇到任何问题，请在OpenAI社区论坛上提问。 要开始编码，请克隆GitHub上此教程的完整代码。或者，跟着每个部分复制到Jupyter笔记本中，并逐步运行代码，或者只是阅读。避免任何问题的好方法是设置一个新的虚拟环境，并通过运行以下命令安装所需包：

```
python -m venv env

source env/bin/activate

pip install -r requirements.txt
```

### 建立网络爬虫

本教程的主要重点是OpenAI API，因此如果您愿意，可以跳过如何创建网络爬虫的内容，直接下载源代码。否则，请展开以下部分以了解实现网络爬取机制的详细步骤。

#### 学习如何构建网络爬虫

获取文本数据是使用嵌入的第一步。本教程通过爬取OpenAI网站创建了一个新的数据集，您也可以使用同样的技术来获取您自己公司或个人网站上的数据。[点我查看源代码](https://github.com/openai/openai-cookbook/tree/main/apps/web-crawl-q-and-a)。

虽然可以使用开源包（如Scrapy）来帮助完成这些操作，但本教程的网络爬虫是从零开始编写的。

该爬虫将从下方代码中传入的根URL开始，访问每个页面，查找附加链接，并访问这些页面（只要它们具有相同的根域名）。为了开始，导入所需的包，设置基本URL并定义HTMLParser类。

```python
import requests
import re
import urllib.request
from bs4 import BeautifulSoup
from collections import deque
from html.parser import HTMLParser
from urllib.parse import urlparse
import os

# Regex pattern to match a URL
HTTP_URL_PATTERN = r'^http[s]*://.+'

domain = "openai.com" # <- put your domain to be crawled
full_url = "https://openai.com/" # <- put your domain to be crawled with https or http

# Create a class to parse the HTML and get the hyperlinks
class HyperlinkParser(HTMLParser):
    def __init__(self):
        super().__init__()
        # Create a list to store the hyperlinks
        self.hyperlinks = []

    # Override the HTMLParser's handle_starttag method to get the hyperlinks
    def handle_starttag(self, tag, attrs):
        attrs = dict(attrs)

        # If the tag is an anchor tag and it has an href attribute, add the href attribute to the list of hyperlinks
        if tag == "a" and "href" in attrs:
            self.hyperlinks.append(attrs["href"])
```

下一个函数以URL作为参数，打开URL并读取HTML内容。然后，它返回该页面上找到的所有超链接。

```python
# Function to get the hyperlinks from a URL
def get_hyperlinks(url):
    
    # Try to open the URL and read the HTML
    try:
        # Open the URL and read the HTML
        with urllib.request.urlopen(url) as response:

            # If the response is not HTML, return an empty list
            if not response.info().get('Content-Type').startswith("text/html"):
                return []
            
            # Decode the HTML
            html = response.read().decode('utf-8')
    except Exception as e:
        print(e)
        return []

    # Create the HTML Parser and then Parse the HTML to get hyperlinks
    parser = HyperlinkParser()
    parser.feed(html)

    return parser.hyperlinks
```

目标是遍历并索引仅在OpenAI域下的内容。为此，需要编写一个函数调用get\_hyperlinks函数，但过滤掉任何不属于指定域的URL。

```python
# Function to get the hyperlinks from a URL that are within the same domain
def get_domain_hyperlinks(local_domain, url):
    clean_links = []
    for link in set(get_hyperlinks(url)):
        clean_link = None

        # If the link is a URL, check if it is within the same domain
        if re.search(HTTP_URL_PATTERN, link):
            # Parse the URL and check if the domain is the same
            url_obj = urlparse(link)
            if url_obj.netloc == local_domain:
                clean_link = link

        # If the link is not a URL, check if it is a relative link
        else:
            if link.startswith("/"):
                link = link[1:]
            elif link.startswith("#") or link.startswith("mailto:"):
                continue
            clean_link = "https://" + local_domain + "/" + link

        if clean_link is not None:
            if clean_link.endswith("/"):
                clean_link = clean_link[:-1]
            clean_links.append(clean_link)

    # Return the list of hyperlinks that are within the same domain
    return list(set(clean_links))
```

爬取功能是网络抓取任务设置的最后一步。它跟踪访问过的URL以避免重复访问同一页，该页可能在站点上链接到多个页面。它还从页面中提取不带HTML标记的原始文本，并将文本内容写入特定于该页面的本地.txt文件。

```python
def crawl(url):
    # Parse the URL and get the domain
    local_domain = urlparse(url).netloc

    # Create a queue to store the URLs to crawl
    queue = deque([url])

    # Create a set to store the URLs that have already been seen (no duplicates)
    seen = set([url])

    # Create a directory to store the text files
    if not os.path.exists("text/"):
            os.mkdir("text/")

    if not os.path.exists("text/"+local_domain+"/"):
            os.mkdir("text/" + local_domain + "/")

    # Create a directory to store the csv files
    if not os.path.exists("processed"):
            os.mkdir("processed")

    # While the queue is not empty, continue crawling
    while queue:

        # Get the next URL from the queue
        url = queue.pop()
        print(url) # for debugging and to see the progress

        # Save text from the url to a <url>.txt file
        with open('text/'+local_domain+'/'+url[8:].replace("/", "_") + ".txt", "w", encoding="UTF-8") as f:

            # Get the text from the URL using BeautifulSoup
            soup = BeautifulSoup(requests.get(url).text, "html.parser")

            # Get the text but remove the tags
            text = soup.get_text()

            # If the crawler gets to a page that requires JavaScript, it will stop the crawl
            if ("You need to enable JavaScript to run this app." in text):
                print("Unable to parse page " + url + " due to JavaScript being required")
            
            # Otherwise, write the text to the file in the text directory
            f.write(text)

        # Get the hyperlinks from the URL and add them to the queue
        for link in get_domain_hyperlinks(local_domain, url):
            if link not in seen:
                queue.append(link)
                seen.add(link)

crawl(full_url)
```

上面示例的最后一行运行爬虫，遍历所有可访问的链接，并将这些页面转换为文本文件。根据您网站的大小和复杂性，此过程可能需要几分钟时间。

### 构建嵌入索引

CSV是存储嵌入的常见格式。你可以通过将原始文本文件（位于text目录中）转换为Pandas数据帧来使用Python处理该格式。Pandas是一个流行的开源库，可帮助您处理表格数据（以行和列存储的数据）。 空白的空行可能会使文本文件混乱，使其更难以处理。一个简单的函数可以删除这些行并整理文件。

```python
def remove_newlines(serie):
    serie = serie.str.replace('\n', ' ')
    serie = serie.str.replace('\\n', ' ')
    serie = serie.str.replace('  ', ' ')
    serie = serie.str.replace('  ', ' ')
    return serie
```

将文本转换为CSV需要遍历之前创建的text目录中的文本文件。在打开每个文件后，删除多余的空格并将修改后的文本追加到一个列表中。然后，将删除了新行的文本添加到一个空的Pandas数据帧中，并将数据帧写入CSV文件。

> 额外的空格和新行可能会使文本混乱，并复杂化嵌入过程。这里使用的代码有助于删除其中的一些，但您可能会发现第三方库或其他方法有用于去除更多不必要字符的功能。

```python
import pandas as pd

# Create a list to store the text files
texts=[]

# Get all the text files in the text directory
for file in os.listdir("text/" + domain + "/"):

    # Open the file and read the text
    with open("text/" + domain + "/" + file, "r", encoding="UTF-8") as f:
        text = f.read()

        # Omit the first 11 lines and the last 4 lines, then replace -, _, and #update with spaces.
        texts.append((file[11:-4].replace('-',' ').replace('_', ' ').replace('#update',''), text))

# Create a dataframe from the list of texts
df = pd.DataFrame(texts, columns = ['fname', 'text'])

# Set the text column to be the raw text with the newlines removed
df['text'] = df.fname + ". " + remove_newlines(df.text)
df.to_csv('processed/scraped.csv')
df.head()
```

在将原始文本保存到CSV文件后，词元化是下一步。该过程通过分解句子和单词将输入文本分成词元。可以通过查看我们文档中的Tokenizer来进行视觉演示。

一个有用的经验法则是，对于常见的英文文本，一个词元通常对应约4个字符。这相当于大约3/4个单词（因此100个词元\~= 75个单词）。 API对于嵌入的最大输入标记数有限制。为了保持在限制范围内，需要将CSV文件中的文本拆分成多个行。首先记录每行的现有长度，以确定需要拆分哪些行。

```python
import tiktoken

# Load the cl100k_base tokenizer which is designed to work with the ada-002 model
tokenizer = tiktoken.get_encoding("cl100k_base")

df = pd.read_csv('processed/scraped.csv', index_col=0)
df.columns = ['title', 'text']

# Tokenize the text and save the number of tokens to a new column
df['n_tokens'] = df.text.apply(lambda x: len(tokenizer.encode(x)))

# Visualize the distribution of the number of tokens per row using a histogram
df.n_tokens.hist()
```

![](<../../.gitbook/assets/image (7).png>)

最新的嵌入模型可以处理多达8191个输入标记的输入，因此大多数行不需要任何拆分，但对于每个爬取的子页面可能并非都是这样，因此下一个代码块将把更长的行拆分成较小的块。

```python
max_tokens = 500

# Function to split the text into chunks of a maximum number of tokens
def split_into_many(text, max_tokens = max_tokens):

    # Split the text into sentences
    sentences = text.split('. ')

    # Get the number of tokens for each sentence
    n_tokens = [len(tokenizer.encode(" " + sentence)) for sentence in sentences]
    
    chunks = []
    tokens_so_far = 0
    chunk = []

    # Loop through the sentences and tokens joined together in a tuple
    for sentence, token in zip(sentences, n_tokens):

        # If the number of tokens so far plus the number of tokens in the current sentence is greater 
        # than the max number of tokens, then add the chunk to the list of chunks and reset
        # the chunk and tokens so far
        if tokens_so_far + token > max_tokens:
            chunks.append(". ".join(chunk) + ".")
            chunk = []
            tokens_so_far = 0

        # If the number of tokens in the current sentence is greater than the max number of 
        # tokens, go to the next sentence
        if token > max_tokens:
            continue

        # Otherwise, add the sentence to the chunk and add the number of tokens to the total
        chunk.append(sentence)
        tokens_so_far += token + 1

    return chunks
    

shortened = []

# Loop through the dataframe
for row in df.iterrows():

    # If the text is None, go to the next row
    if row[1]['text'] is None:
        continue

    # If the number of tokens is greater than the max number of tokens, split the text into chunks
    if row[1]['n_tokens'] > max_tokens:
        shortened += split_into_many(row[1]['text'])
    
    # Otherwise, add the text to the list of shortened texts
    else:
        shortened.append( row[1]['text'] )
```

再次可视化更新后的直方图可以帮助确认行是否已成功拆分为缩短的部分。

![](<../../.gitbook/assets/image (2) (1).png>)

现在将内容拆分成较小的块，并且可以向OpenAI API发送简单请求，指定使用新的text-embedding-ada-002模型来创建嵌入：

```python
import openai

df['embeddings'] = df.text.apply(lambda x: openai.Embedding.create(input=x, engine='text-embedding-ada-002')['data'][0]['embedding'])

df.to_csv('processed/embeddings.csv')
df.head()
```

这应该需要大约3-5分钟，但完成后您就可以使用您的嵌入了！

### 使用您的嵌入构建一个问答系统

嵌入已经准备好了，此过程的最后一步是创建一个简单的问答系统。这个系统会接收用户的问题，创建它的嵌入，并将其与现有的嵌入进行比较，以检索从抓取的网站中最相关的文本。然后，text-davinci-003模型将根据检索到的文本生成一个自然的回答。

将嵌入转换为NumPy数组是第一步，这将提供更多的灵活性，因为可以利用许多操作NumPy数组的函数来使用它。这还将将维度压平为1-D，这是许多后续操作所需的格式。

```python
import numpy as np
from openai.embeddings_utils import distances_from_embeddings

df=pd.read_csv('processed/embeddings.csv', index_col=0)
df['embeddings'] = df['embeddings'].apply(eval).apply(np.array)

df.head()
```

现在，由于数据已准备好，需要使用一个简单的函数将问题转换为嵌入。这很重要，因为搜索依赖嵌入和使用余弦距离的数字向量（这是原始文本的转换）进行比较。如果向量在余弦距离上接近，则它们很可能相关且可能是问题的答案。OpenAI的Python包具有内置的distances\_from\_embeddings函数，在这里非常有用。

```python
def create_context(
    question, df, max_len=1800, size="ada"
):
    """
    Create a context for a question by finding the most similar context from the dataframe
    """

    # Get the embeddings for the question
    q_embeddings = openai.Embedding.create(input=question, engine='text-embedding-ada-002')['data'][0]['embedding']

    # Get the distances from the embeddings
    df['distances'] = distances_from_embeddings(q_embeddings, df['embeddings'].values, distance_metric='cosine')


    returns = []
    cur_len = 0

    # Sort by distance and add the text to the context until the context is too long
    for i, row in df.sort_values('distances', ascending=True).iterrows():
        
        # Add the length of the text to the current length
        cur_len += row['n_tokens'] + 4
        
        # If the context is too long, break
        if cur_len > max_len:
            break
        
        # Else add it to the text that is being returned
        returns.append(row["text"])

    # Return the context
    return "\n\n###\n\n".join(returns)
```

将文本分成更小的词元集合后，按升序循环遍历并继续添加文本是确保完整答案的关键步骤。如果返回的内容比期望的多，则可以将max\_len修改为较小的值。

前一步仅检索与问题语义相关的文本块，因此它们可能包含答案，但并不保证。通过返回前5个最可能的结果，可以进一步增加找到答案的机会。

然后，回答提示将尝试从检索到的上下文中提取相关事实，以制定连贯的答案。如果没有相关答案，则提示将返回“我不知道”。

使用text-davinci-003的完成端点可以创建一个听起来更为真实的答案。

```python
def answer_question(
    df,
    model="text-davinci-003",
    question="Am I allowed to publish model outputs to Twitter, without a human review?",
    max_len=1800,
    size="ada",
    debug=False,
    max_tokens=150,
    stop_sequence=None
):
    """
    Answer a question based on the most similar context from the dataframe texts
    """
    context = create_context(
        question,
        df,
        max_len=max_len,
        size=size,
    )
    # If debug, print the raw model response
    if debug:
        print("Context:\n" + context)
        print("\n\n")

    try:
        # Create a completions using the question and context
        response = openai.Completion.create(
            prompt=f"Answer the question based on the context below, and if the question can't be answered based on the context, say \"I don't know\"\n\nContext: {context}\n\n---\n\nQuestion: {question}\nAnswer:",
            temperature=0,
            max_tokens=max_tokens,
            top_p=1,
            frequency_penalty=0,
            presence_penalty=0,
            stop=stop_sequence,
            model=model,
        )
        return response["choices"][0]["text"].strip()
    except Exception as e:
        print(e)
        return ""
```

完成了！一个拥有OpenAI网站嵌入式知识的工作问答系统现在已经准备好了。可以进行一些快速测试，以查看输出的质量：

```python
answer_question(df, question="What day is it?", debug=False)

answer_question(df, question="What is our newest embeddings model?")

answer_question(df, question="What is ChatGPT?")
```

回答将类似于以下内容：

```
"I don't know."

'The newest embeddings model is text-embedding-ada-002.'

'ChatGPT is a model trained to interact in a conversational way. It is able to answer followup questions, admit its mistakes, challenge incorrect premises, and reject inappropriate requests.'
```

如果系统无法回答一个预期的问题，那么搜索原始文本文件，看看预期的信息是否确实被嵌入其中或者没有。最初进行的爬取过程是设置跳过提供的原始域之外的网站，因此如果设置了子域，可能就没有这些知识。

目前，每次回答一个问题时都会传递数据帧。对于更多生产工作流程，应该使用[矢量数据库](https://platform.openai.com/docs/guides/embeddings/how-can-i-retrieve-k-nearest-embedding-vectors-quickly)解决方案，而不是将嵌入存储在CSV文件中，但当前的方法是原型设计的一个很好的选择。
