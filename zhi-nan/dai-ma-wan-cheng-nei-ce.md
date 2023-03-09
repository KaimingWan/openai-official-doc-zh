# 代码完成(内测)

学习如何生成或操作代码 介绍 Codex 模型系列是我们 GPT-3 系列的后代，它经过了自然语言和数十亿行代码的训练。它在 Python 中最为强大，在包括 JavaScript、Go、Perl、PHP、Ruby、Swift、TypeScript、SQL 甚至 Shell 在内的十多种语言中也很熟练。在这个初始有限的测试期间，Codex 的使用是免费的。了解更多信息。 您可以使用 Codex 完成各种任务，包括：

* &#x20;将注释转换为代码&#x20;
* 在上下文中完成下一行或函数&#x20;
* 为应用程序查找有用库或 API 调用等知识带来便利&#x20;
* 添加注释&#x20;
* 重写代码以提高效率&#x20;

要查看 Codex 的实际运作情况，请查看我们的 [Codex JavaScript 沙盒](https://platform.openai.com/codex-javascript-sandbox)或其他[演示视频](https://www.youtube.com/playlist?list=PLOXw6I10VTv\_FhQbbvYh1FvbiaPf43Ve2)。

![](<../.gitbook/assets/image (12).png>)

## 快速开始

以下是一些可以在 [Playground](https://platform.openai.com/playground) 中测试的使用 Codex 的示例(一般可以输入到instruction中)。

**Saying "Hello" (Python)**

```python
"""
Ask the user for their name and say "Hello"
"""
```

**Create random names (Python)**

```python
"""
1. Create a list of first names
2. Create a list of last names
3. Combine them randomly into a list of 100 full names
"""
```

**Create a MySQL query (Python)**

```python
"""
Table customers, columns = [CustomerId, FirstName, LastName, Company, Address, City, State, Country, PostalCode, Phone, Fax, Email, SupportRepId]
Create a MySQL query for all customers in Texas named Jane
"""
query =
```

**Explaining code (JavaScript)**

```javascript
// Function 1
var fullNames = [];
for (var i = 0; i < 50; i++) {
  fullNames.push(names[Math.floor(Math.random() * names.length)]
    + " " + lastNames[Math.floor(Math.random() * lastNames.length)]);
}

// What does Function 1 do?
```

### 更多示例&#x20;

访问我们的[示例库](https://platform.openai.com/examples?category=code)，探索为Codex设计的更多提示。

### 最佳实践

#### **从英文翻译成中文（简体）**

从注释、数据或代码开始。您可以在我们的playgroud中使用Codex模型之一进行实验（必要时将样式指令作为注释）。要让Codex创建有用的完成，有助于考虑程序员执行任务所需的信息。这可能只是一个清晰的注释或编写有用函数所需的数据，例如变量名称或函数处理哪个类别。

```
创建一个名为'nameImporter'的函数，将名字和姓氏添加到数据库中。
```

在这个例子中，我们告诉Codex该如何命名函数以及它将要执行的任务。 这种方法甚至可以扩展到您可以向Codex提供注释和数据库模式示例的程度，从而使其编写各种数据库有用的查询请求。

```
# 表 albums, columns = [AlbumId, Title, ArtistId]
# 表 artists, columns = [ArtistId, Name]
# 表 media_types, columns = [MediaTypeId, Name]
# 表 playlists, columns = [PlaylistId, Name]
# 表 playlist_track, columns = [PlaylistId, TrackId]
# 表 tracks, columns = [TrackId, Name, AlbumId, MediaTypeId, GenreId, Composer, Milliseconds, Bytes, UnitPrice]

# 创建一个查询，以获取Adele的所有专辑。
```

当您向Codex展示数据库模式时，它能够对如何格式化查询做出明智的猜测。&#x20;

#### **指定语言**

Codex理解数十种不同的编程语言。许多共享类似的注释、函数和其他编程语法约定。通过在注释中指定语言和版本，Codex更能够提供您想要的完成功能。话虽如此，Codex在样式和语法方面相当灵活。

```
# R 语言
# 计算点阵数组中的平均距离
# Python 3
# 计算点阵数组中的平均距离
```

用你想要的方式提示Codex。如果你想让Codex创建一个网页，在注释后面放置HTML文档中的第一行代码:

```
 <!DOCTYPE html>
```

告诉Codex接下来应该做什么。同样的方法也适用于从注释中创建函数（在注释后面加上以func或def开头的新行）。

```
<!-- 创建一个标题为“Kat Katman律师”的网页 -->
<!DOCTYPE html>
```

在我们的注释后面放置\<! DOCTYPE html>可以让Codex非常清楚地知道我们想要它做什么。

```
# 创建一个函数来计数到100
def counter
```

如果我们开始编写函数，Codex 将理解接下来需要做什么。&#x20;

#### **指定库将有助于 Codex 理解您想要的内容**

Codex 知道大量的库、API 和模块。通过告诉 Codex 使用哪些库，可以通过注释或将它们导入到您的代码中，Codex 将基于这些库而不是替代方案提出建议。

```
<!-- 使用 A-Frame 版本 1.2.0 创建一个3D网站。 -->
<!-- https://aframe.io/releases/1.2.0/aframe.min.js -->

```

通过指定版本，您可以确保Codex使用最新的库。 注意：Codex可以建议有用的库和API，但一定要进行自己的研究，以确保它们对您的应用程序是安全的。&#x20;

#### **注释风格可能会影响代码质量**

对于某些语言，注释样式可以提高输出质量。例如，在使用Python时，在某些情况下使用文档字符串（三引号包装的注释）比使用井号（#）符号产生更高质量的结果。

```
"""
创建一个用户和电子邮件地址的数组
"""
```

#### **在函数内部放置注释可能会有所帮助**

推荐的编码标准通常建议将函数的描述放在函数内部。使用这种格式可以帮助Codex更清楚地理解您想要函数执行什么操作。

```
def getUserBalance(id):
    """
    在数据库“UserData”中查找用户并返回其当前账户余额。
    """
```

#### **提供更多的例子可以得到更精确的结果**

如果您需要 Codex 使用特定的样式或格式，请在请求的前面提供示例或演示，这将有助于 Codex 更准确地匹配您所需的内容。

```
"""
Create a list of random animals and species
"""
animals  = [ {"name": "Chomper", "species": "Hamster"}, {"name":
```

#### **较低的温度可以得到更精确的结果**

将API温度设置为0或接近于零（如0.1或0.2）通常在大多数情况下会产生更好的结果。与GPT-3不同，高温可以提供有用的创意和随机结果，但是使用Codex时，较高的温度可能会给您带来真正随机或不稳定的响应。 在需要Codex提供不同潜在结果的情况下，请从零开始逐步增加0.1直到找到合适变化。

#### **将任务组织成函数**

我们可以通过注释尽可能精确地指定函数应该执行什么操作来让Codex编写函数。通过编写以下注释，Codex创建了一个JavaScript计时器函数，在用户按下按钮时触发： 一个简单的JavaScript计时器

```
// 创建一个计时器，在10秒后创建一个警报
```

我们可以使用Codex来执行常见任务，例如使用众所周知的库创建Stripe API中的客户： 在Python中创建一个Stripe客户。

```
# 从电子邮件地址创建Stripe客户
```

#### **创建示例数据**

测试应用程序通常需要使用示例数据。因为Codgen是一种语言模型，它可以理解和编写自然语言，所以您可以要求Codex创建像虚构名称、产品和其他变量的数组等数据。

```
/* 创建一个旧金山天气温度数组 */
```

请求Codex执行此任务将生成如下表格：

```
var weather = [
  { month: 'January', high: 58, low: 48 },
  { month: 'February', high: 61, low: 50 },
  { month: 'March', high: 64, low: 53 },
  { month: 'April', high: 67, low: 55 },
  { month: 'May', high: 70, low: 58 },
  { month: 'June', high: 73, low: 61 },
  { month: 'July', high: 76, low: 63 },
  { month: 'August', high: 77, low: 64 },
  { month: 'September', high: 76, low: 63 },
  { month: 'October', high: 73, low: 61 },
  { month: 'November', high: 68, low: 57 },
  { month: 'December', high: 64, low: 54 }
];
```

#### **复合函数和小应用程序**

我们可以向Codex提供一个由复杂请求组成的注释，例如创建随机名称生成器或使用用户输入执行任务，只要有足够的词元，Codex就可以生成其余部分。

```
 /* 创建动物列表 创建城市列表 使用这些列表来生成关于我在每个城市动物园看到的事情的故事 */
```

#### **限制完成大小以获得更精确的结果或降低延迟**

在Codex中请求较长的完成可能会导致不准确的答案和重复。通过减少max\_tokens并设置停止标记来限制查询大小。例如，添加\n作为停止序列可将完成限制为一行代码。较小的完成还会产生较少的延迟。&#x20;

#### **使用流式传输以减少延迟**

大型Codex查询可能需要数十秒才能完成。为了构建需要更低延迟（如执行自动完成功能） 的应用程序，请考虑使用流式传输。模型完成整个完成之前将返回响应。只需要部分完成的应用程序可以通过编程方式或使用停止符号进行切断来减少延迟。 用户可以结合流媒体和重复使用来缩短延迟时间，并从API中请求多个解决方案，并使用返回第一个响应结果 。通过设置n>1 来实现此操作 。这种方法消耗更多令牌配额，因此请谨慎使用（例如 ，通过对max\_tokens 和stop 使用合理设置）。

#### **利用Codex解释代码** 。

Codex 创建和理解代码 的能力使我们能够将其用于执行诸如解释文件中代码所做内容之类 的任务 。实现此目标有一种方法是在函数后面放置一个以“This function” 或 “This application is” 开头 的注释 。 Codex通常会将其解释为说明开始并完整其余文本。 / 解释上一个功能正在做什么：

```
/* Explain what the previous function is doing: It
```

#### **解释一个 SQL 查询**

在这个例子中，我们使用 Codex 以人类可读的格式来解释一个 SQL 查询正在做什么。

```sql
SELECT DISTINCT department.name
FROM department
JOIN employee ON department.id = employee.department_id
JOIN salary_payments ON employee.id = salary_payments.employee_id
WHERE salary_payments.date BETWEEN '2020-06-01' AND '2020-06-30'
GROUP BY department.name
HAVING COUNT(employee.id) > 10;
-- 针对以上SQL提供人类可读格式的解释
--
```

#### **编写单元测试**

在Python中，只需添加注释“单元测试”并启动函数即可创建一个单元测试。

```
# Python 3
def sum_numbers(a, b):
  return a + b

# 单元测试
def
```

检查代码中的错误。通过使用示例，您可以向Codex展示如何识别代码中的错误。在某些情况下不需要示例，但是展示提供描述所需的级别和详细信息可以帮助Codex了解要查找什么以及如何解释它。（由Codex进行错误检查不应替代用户的仔细审查。）

```
/* 解释为什么之前的函数不起作用。*/
```

#### **使用源数据编写数据库函数**

就像人类程序员从了解数据库结构和列名称中受益一样，Codex可以利用这些数据帮助您编写准确的查询请求。在此示例中，我们插入数据库模式并告诉Codex要查询什么。

```
# Table albums, columns = [AlbumId, Title, ArtistId]
# Table artists, columns = [ArtistId, Name]
# Table media_types, columns = [MediaTypeId, Name]
# Table playlists, columns = [PlaylistId, Name]
# Table playlist_track, columns = [PlaylistId, TrackId]
# Table tracks, columns = [TrackId, Name, AlbumId, MediaTypeId, GenreId, Composer, Milliseconds, Bytes, UnitPrice]

# 创建一个查询，以获取Adele的所有专辑。
```

#### **语言转换**

您可以通过遵循一个简单的格式，将Codex从一种语言转换为另一种语言，其中您在注释中列出要转换的代码的语言，然后是代码和希望将其翻译成的语言的注释。

```
# 将这个从Python转换为R。
# Python 版本

[ Python code ]

# End

# R 版本
```

#### **重写库或框架的代码**

如果您想让Codex使一个函数更有效率，您可以提供需要重写的代码，并附上使用哪种格式的说明。

```
// 将此重写为一个React组件
var input = document.createElement('input');
input.setAttribute('type', 'text');
document.body.appendChild(input);
var button = document.createElement('button');
button.innerHTML = 'Say Hello';
document.body.appendChild(button);
button.onclick = function() {
  var name = input.value;
  var hello = document.createElement('div');
  hello.innerHTML = 'Hello ' + name;
  document.body.appendChild(hello);
};

// React 版本:
```

## 插入代码

“completions” 端点还支持通过提供后缀提示来在代码中插入代码，除了前缀提示。这可以用于在函数或文件的中间插入完成(下图绿色部分由AI完成补全)。&#x20;

![](<../.gitbook/assets/image (8).png>)

通过为模型提供额外的上下文，它可以更加可控。然而，这对于模型来说是一个更受限制和具有挑战性的任务。

### 最佳实践

在测试版中，插入代码是一个新功能，您可能需要修改使用API的方式以获得更好的结果。以下是一些最佳实践：&#x20;

* **使用max\_tokens > 256**。模型更擅长插入较长的完成内容。如果max\_tokens太小，则模型可能会在连接后缀之前被截断。请注意，即使使用较大的max\_tokens，在生成令牌数量时也只会收取费用。&#x20;
* **优先选择finish\_reason == "stop"**。当模型到达自然停止点或用户提供的停止序列时，它将设置finish\_reason为“stop”。这表明该模型已成功连接到后缀，并且是完成质量良好的良好信号。这对于在n>1或重新采样（见下一点）时选择几个完成之间进行选择尤其相关。&#x20;
* **重新采样3-5次**。虽然几乎所有完成都与前缀相连，但在更难的情况下，模型可能会难以连接后缀。我们发现重新采样3或5次（或者使用k = 3,5 的best\_of），并挑选具有“stop”作为其finish\_reason标记的样本可以成为解决此类问题有效方法之一 。在重新采样时，通常希望温度较高以增加多样性。&#x20;

注意：如果返回的所有示例都具有finish\_reason == “length”，则很可能max\_tokens太小，并且模型在自然地连接提示和后缀之前耗尽了令牌数，请考虑增加max\_tokens再进行重试.

## 编辑代码

编辑端点可用于编辑代码，而不仅仅是完成它。您提供一些代码和修改指令，code-davinci-edit-001模型将尝试相应地进行编辑。这是重构和微调代码的自然界面。在此初始测试期间，使用编辑端点是免费的。

### 例子&#x20;

迭代构建程序 编写代码通常是一个需要逐步完善文本的迭代过程。通过编辑使得持续改进模型输出变得更加自然，直到最终结果被打磨出来为止。在这个例子中，我们以斐波那契数列作为示例来说明如何逐步构建代码。&#x20;

1. 编写一个函数

![](<../.gitbook/assets/image (10).png>)

2. 重构代码

![](<../.gitbook/assets/image (11).png>)

3. 重命名函数

![](<../.gitbook/assets/image (1) (1).png>)

4. 增加文档

![](<../.gitbook/assets/image (3) (1).png>)

### 最佳实践&#x20;

编辑端点仍处于 alpha 阶段，我们建议遵循以下最佳实践。&#x20;

* 考虑使用空提示！在这种情况下，编辑可以类似于完成。
* 尽可能具体地说明指示。&#x20;
* 有时，模型无法找到解决方案并会导致错误。我们建议重新措辞您的指示或输入。
