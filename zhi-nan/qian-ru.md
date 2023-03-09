# 嵌入

OpenAI的文本嵌入（embeddings）可以测量文本字符串之间的相关性。嵌入通常用于：

* 搜索（结果按查询字符串的相关性排序）&#x20;
* 聚类（将文本字符串按相似性分组）&#x20;
* 推荐（推荐具有相关文本字符串的物品）&#x20;
* 异常检测（识别具有较少相关性的异常值）&#x20;
* 多样性测量（分析相似性分布）&#x20;
* 分类（将文本字符串按其最相似的标签分类）&#x20;

嵌入是由浮点数组成的向量（列表）。两个向量之间的距离可以衡量它们之间的相关性。小距离表明高相关性，大距离则表明低相关性。

请访问我们的[定价](https://openai.com/api/pricing/)页面了解嵌入的价格。请求基于输入中的token数量计费。

要了解嵌入的实际运用，请查看本文中的示例：

* 分类&#x20;
* 主题聚类
* 搜索
* 推荐

## 如何获取嵌入&#x20;

要获取嵌入，将您的文本字符串发送到嵌入API端点，并选择一个嵌入模型ID（例如text-embedding-ada-002）。响应将包含一个嵌入，您可以提取、保存和使用。

示例请求:



```shell
# 获取嵌入
curl https://api.openai.com/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{"input": "Your text string goes here",
       "model":"text-embedding-ada-002"}'
```

示例响应:

```
{
  "data": [
    {
      "embedding": [
        -0.006929283495992422,
        -0.005336422007530928,
        ...
        -4.547132266452536e-05,
        -0.024047505110502243
      ],
      "index": 0,
      "object": "embedding"
    }
  ],
  "model": "text-embedding-ada-002",
  "object": "list",
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 5
  }
}
```

在[OpenAI Cookbook](https://github.com/openai/openai-cookbook/)中查看更多Python代码示例。 使用OpenAI嵌入时，请记住它们的限制和风险。

## 嵌入模型

&#x20;OpenAI 提供了一个第二代嵌入模型（在模型 ID 中标记为 -002）和 16 个第一代模型（在模型 ID 中标记为 -001）。 我们建议几乎所有用例都使用 text-embedding-ada-002。它更好、更便宜、更简单易用。请阅读博客文章公告。

| 模型版本 | 分词器          | 最大输入词元 | 知识截止时间   |
| ---- | ------------ | ------ | -------- |
| V2   | cl100k\_base | 8191   | Sep 2021 |
| V1   | GPT-2/GPT-3  | 2046   | Aug 2020 |

使用按输入词元计价，每1000个词元的费率为0.0004美元，或者大约1美元可以翻译3000页（假设每页有800个词元）：

| 模型                     | 每美元的粗糙页面数 | BEIR 搜索评估的示例性能 |
| ---------------------- | --------- | -------------- |
| text-embedding-ada-002 | 3000      | 53.9           |
| _-davinci-_-001        | 6         | 52.8           |
| _-curie-_-001          | 60        | 50.9           |
| _-babbage-_-001        | 240       | 50.4           |
| _-ada-_-001            | 300       | 49.0           |

### 第二代模型

| 模型                     | 分词器          | 最大词元数 | 输出维度 |
| ---------------------- | ------------ | ----- | ---- |
| text-embedding-ada-002 | cl100k\_base | 8191  | 1536 |

### 第一代模型（不推荐）

官方也不推荐使用其他的第一代模型，这边就不翻译了

## 使用案例

这里我们展示一些代表性的使用案例。接下来的例子中，我们将使用[亚马逊美食评论数据集](https://www.kaggle.com/snap/amazon-fine-food-reviews)。

### 获取嵌入

该数据集包含截至2012年10月亚马逊用户留下的共568,454条食品评论。我们将使用最近1,000条评论的子集进行说明。这些评论是用英语编写的，往往是积极或消极的。每个评论都有一个ProductId、UserId、Score、评价标题（Summary）和评价正文（Text）。例如：

| 产品 ID      | 用户 ID          | 得分 | 总结     | 文本               |
| ---------- | -------------- | -- | ------ | ---------------- |
| B001E4KFG0 | A3SGXH7AUHU8GW | 5  | 优质狗粮   | 我已经买了几罐活力饮料...   |
| B00813GRG4 | A1D87F6ZCVE5NK | 1  | 不如广告所述 | 产品标签上写着巨型盐腌花生... |

我们将把评论摘要和评论文本合并成一个组合文本。模型将对这个组合文本进行编码，并输出一个单一的向量嵌入。

[Obtain\_dataset.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Obtain\_dataset.ipynb)

```python
def get_embedding(text, model="text-embedding-ada-002"):
   text = text.replace("\n", " ")
   return openai.Embedding.create(input = [text], model=model)['data'][0]['embedding']
 
df['ada_embedding'] = df.combined.apply(lambda x: get_embedding(x, model='text-embedding-ada-002'))
df.to_csv('output/embedded_1k_reviews.csv', index=False)
```

要从保存的文件中加载数据，您可以运行以下命令：

```python
import pandas as pd
 
df = pd.read_csv('output/embedded_1k_reviews.csv')
df['ada_embedding'] = df.ada_embedding.apply(eval).apply(np.array)
```

### 数据2D可视化

[Visualizing\_embeddings\_in\_2D.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Visualizing\_embeddings\_in\_2D.ipynb)

嵌入的大小取决于底层模型的复杂性。为了可视化这个高维数据，我们使用t-SNE算法将数据转换成二维。 我们根据评论者给出的星级评分来着色每个单独的评论：&#x20;

* 1星：红色&#x20;
* 2星：深橙色&#x20;
* 3星：金色&#x20;
* 4星：青绿色&#x20;
* 5星：深绿色

![](<../.gitbook/assets/image (7).png>)

可视化似乎产生了大约3个聚类，其中一个主要是负面评价。

```python
import pandas as pd
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
import matplotlib
 
df = pd.read_csv('output/embedded_1k_reviews.csv')
matrix = df.ada_embedding.apply(eval).to_list()
 
# Create a t-SNE model and transform the data
tsne = TSNE(n_components=2, perplexity=15, random_state=42, init='random', learning_rate=200)
vis_dims = tsne.fit_transform(matrix)
 
colors = ["red", "darkorange", "gold", "turquiose", "darkgreen"]
x = [x for x,y in vis_dims]
y = [y for x,y in vis_dims]
color_indices = df.Score.values - 1
 
colormap = matplotlib.colors.ListedColormap(colors)
plt.scatter(x, y, c=color_indices, cmap=colormap, alpha=0.3)
plt.title("Amazon ratings visualized in language using t-SNE")
```

### 将嵌入作为文本特征编码器用于机器学习算法

[Regression\_using\_embeddings.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Regression\_using\_embeddings.ipynb)

嵌入可以作为机器学习模型中通用的自由文本特征编码器。如果一些相关输入是自由文本，那么加入嵌入将提高任何机器学习模型的性能。嵌入也可以作为ML模型中分类特征编码器使用。如果分类变量名称有意义且数量众多（例如职位名称），则此方法最具价值。相似度嵌入通常比搜索嵌入在此任务上表现更好。&#x20;

我们观察到，通常情况下，嵌入表示的信息密度很高。例如，使用SVD或PCA降低输入维数即使只有10％，在特定任务的下游性能方面通常会导致更差的结果。

&#x20;该代码将数据分成训练集和测试集，并将被以下两个用例使用：回归和分类。

```python
from sklearn.model_selection import train_test_split
 
X_train, X_test, y_train, y_test = train_test_split(
    list(df.ada_embedding.values),
    df.Score,
    test_size = 0.2,
    random_state=42
)
```

#### 使用嵌入特征的回归&#x20;

嵌入提供了一种优雅的方法来预测数值。在这个例子中，我们根据评论文本预测评论者的星级评分。由于嵌入所包含的语义信息非常丰富，即使只有很少的评论，也可以得到不错的预测结果。&#x20;

我们假设得分是1到5之间的连续变量，并允许算法预测任何浮点值。机器学习算法将预测值与真实得分之间的距离最小化，并取得了0.39 的平均绝对误差，这意味着平均而言，预测偏差不到半颗星。

```python
from sklearn.ensemble import RandomForestRegressor
 
rfr = RandomForestRegressor(n_estimators=100)
rfr.fit(X_train, y_train)
preds = rfr.predict(X_test)
```

### 使用嵌入特征进行分类

[Classification\_using\_embeddings.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Classification\_using\_embeddings.ipynb)

这一次，我们不再让算法预测1到5之间的任意值，而是尝试将评论中的星级精确分类为5个档次，范围从1星到5星。&#x20;

经过训练后，模型学会了更好地预测1和5星评价，而对于更微妙的评价（2-4星），由于情感表达不够极端可能效果较差。

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, accuracy_score
 
clf = RandomForestClassifier(n_estimators=100)
clf.fit(X_train, y_train)
preds = clf.predict(X_test)
```

### 零样本分类

[Zero-shot\_classification\_with\_embeddings.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Zero-shot\_classification\_with\_embeddings.ipynb)

我们可以使用嵌入来进行零样本分类，而无需任何标记的训练数据。对于每个类别，我们将类名或类的简短描述嵌入其中。为了以零样本方式对一些新文本进行分类，我们将其嵌入与所有类别嵌入进行比较，并预测相似度最高的类别。

```python
from openai.embeddings_utils import cosine_similarity, get_embedding
 
df= df[df.Score!=3]
df['sentiment'] = df.Score.replace({1:'negative', 2:'negative', 4:'positive', 5:'positive'})
 
labels = ['negative', 'positive']
label_embeddings = [get_embedding(label, model=model) for label in labels]
 
def label_score(review_embedding, label_embeddings):
   return cosine_similarity(review_embedding, label_embeddings[1]) - cosine_similarity(review_embedding, label_embeddings[0])
 
prediction = 'positive' if label_score('Sample Review', label_embeddings) > 0 else 'negative'
```

### 获取用户和产品嵌入以进行冷启动推荐

[User\_and\_product\_embeddings.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/User\_and\_product\_embeddings.ipynb)

我们可以通过对用户所有评论进行平均来获得用户嵌入。同样地，我们可以通过对有关该产品的所有评论进行平均来获得产品嵌入。为了展示这种方法的实用性，我们使用50k个评论的子集以涵盖更多用户和产品的评论。&#x20;

我们在一个单独的测试集上评估这些嵌入的实用性，在那里我们绘制用户和产品嵌入相似度作为评分函数。有趣的是，基于这种方法，即使在用户收到产品之前，我们也能比随机预测他们是否会喜欢该产品。

![](<../.gitbook/assets/image (3).png>)

```
user_embeddings = df.groupby('UserId').ada_embedding.apply(np.mean)
prod_embeddings = df.groupby('ProductId').ada_embedding.apply(np.mean)
```

### 聚类

[Clustering.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Clustering.ipynb)\


聚类是理解大量文本数据的一种方法。嵌入对于这个任务非常有用，因为它们提供了每个文本的语义向量表示。因此，在无监督的情况下，聚类将揭示我们数据集中隐藏的分组。&#x20;

在这个例子中，我们发现四个不同的簇：一个专注于狗粮，一个专注于负面评论，另外两个则是关于正面评论。

![](../.gitbook/assets/image.png)

```python
import numpy as np
from sklearn.cluster import KMeans
 
matrix = np.vstack(df.ada_embedding.values)
n_clusters = 4
 
kmeans = KMeans(n_clusters = n_clusters, init='k-means++', random_state=42)
kmeans.fit(matrix)
df['Cluster'] = kmeans.labels_
```

### 使用嵌入进行文本搜索

[Semantic\_text\_search\_using\_embeddings.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Semantic\_text\_search\_using\_embeddings.ipynb)

为了检索出最相关的文档，我们使用查询嵌入向量和每个文档之间的余弦相似度，并返回得分最高的文档。

```python
from openai.embeddings_utils import get_embedding, cosine_similarity
 
def search_reviews(df, product_description, n=3, pprint=True):
   embedding = get_embedding(product_description, model='text-embedding-ada-002')
   df['similarities'] = df.ada_embedding.apply(lambda x: cosine_similarity(x, embedding))
   res = df.sort_values('similarities', ascending=False).head(n)
   return res
 
res = search_reviews(df, 'delicious beans', n=3)
```

### 使用嵌入进行代码搜索

[Code\_search.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/Code\_search.ipynb)

代码搜索与基于嵌入式文本搜索类似。我们提供一种从给定存储库中的所有Python文件中提取Python函数的方法。然后，每个函数都由text-embedding-ada-002模型进行索引。&#x20;

要执行代码搜索，我们使用相同的模型将查询以自然语言形式进行嵌入。然后，我们计算结果查询嵌入和每个函数嵌入之间的余弦相似度。最高余弦相似度结果最相关。

```python
from openai.embeddings_utils import get_embedding, cosine_similarity
 
df['code_embedding'] = df['code'].apply(lambda x: get_embedding(x, model='text-embedding-ada-002'))
 
def search_functions(df, code_query, n=3, pprint=True, n_lines=7):
   embedding = get_embedding(code_query, model='text-embedding-ada-002')
   df['similarities'] = df.code_embedding.apply(lambda x: cosine_similarity(x, embedding))
 
   res = df.sort_values('similarities', ascending=False).head(n)
   return res
res = search_functions(df, 'Completions API tests', n=3)
```
