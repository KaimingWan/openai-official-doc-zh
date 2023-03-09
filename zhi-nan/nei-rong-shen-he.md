# 内容审核

## 概述&#x20;

Moderation 端点是一个工具，您可以使用它来检查内容是否符合 OpenAI 的使用政策。开发人员因此可以识别违反我们使用政策的内容，并采取行动，例如通过过滤它。&#x20;

该模型分类以下类别：

| 类别                 | 描述                                            |
| ------------------ | --------------------------------------------- |
| `hate`             | 表达、煽动或宣传基于种族、性别、民族、宗教、国籍、性取向、残疾状态或种姓的仇恨内容。    |
| `hate/threatening` | 包括针对目标群体的暴力或严重伤害的仇恨内容。                        |
| `self-harm`        | 促进、鼓励或描绘自我伤害行为，如自杀，割伤和饮食障碍等。                  |
| `sexual`           | 旨在引起性兴奋的内容，例如描述性活动或推广性服务（不包括性教育和健康）。          |
| `sexual/minors`    | 包含未满18岁个体的性内容。                                |
| `violence`         | 推广或美化暴力，庆祝他人遭受苦难或屈辱的内容。                       |
| `violence/graphic` | <p>描述死亡, 暴力, 或极端图形细节中造成严重身体损伤.</p><p><br></p> |

当监控 OpenAI API 的输入和输出时，可免费使用审核端点。我们目前不支持第三方流量监控。

我们正在不断努力提高分类器准确度，并特别致力于改善对仇恨言论，自我伤害以及图形暴力等类型文章进行分类。我们目前对非英语语言支持有限。

## 快速入门

要获取文本片段的分类，请像以下代码片段中演示的那样向审核端点发出请求:

```python
curl https://api.openai.com/v1/moderations \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{"input": "Sample text goes here"}'
```

以下是端点的示例输出。它返回以下字段：&#x20;

* flagged：如果模型将内容分类为违反OpenAI使用政策，则设置为true，否则为false。&#x20;
* categories：包含每个类别二进制使用政策违规标志的字典。对于每个类别，如果模型将相应的类别标记为违规，则该值为true，否则为false。&#x20;
* category\_scores：包含由模型输出的每个类别原始分数的字典，表示模型对输入是否违反OpenAI有关该类别的政策的信心。该值介于0和1之间，其中较高的值表示更高的置信度。不应将得分解释为概率。

```python
{
  "id": "modr-XXXXX",
  "model": "text-moderation-001",
  "results": [
    {
      "categories": {
        "hate": false,
        "hate/threatening": false,
        "self-harm": false,
        "sexual": false,
        "sexual/minors": false,
        "violence": false,
        "violence/graphic": false
      },
      "category_scores": {
        "hate": 0.18805529177188873,
        "hate/threatening": 0.0001250059431185946,
        "self-harm": 0.0003706029092427343,
        "sexual": 0.0008735615410842001,
        "sexual/minors": 0.0007470346172340214,
        "violence": 0.0041268812492489815,
        "violence/graphic": 0.00023186142789199948
      },
      "flagged": false
    }
  ]
}
```

OpenAI将持续升级调节端点的基础模型。因此，依赖于类别分数的自定义策略可能需要随时间重新校准。

