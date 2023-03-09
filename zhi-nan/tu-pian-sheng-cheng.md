# 图片生成

学习如何使用我们的DALL·E模型生成或编辑图像

## 介绍

&#x20;Images API提供了三种与图像交互的方法：

* 基于文本提示从头开始创建图像
* 根据新的文本提示对现有图像进行编辑
* 创建现有图像的变化版本

本指南介绍了使用这三个API端点的基础知识，并提供了有用的代码示例。要看它们的实际效果，请查看我们的DALL·E预览应用程序。

Images API目前处于测试版。在此期间，API和模型将根据您的反馈不断发展。为了确保所有用户都能轻松进行原型设计， 默认的速率限制是每分钟50张图像。如果您想增加速率限制，请查看此帮助中心文章。随着我们了解更多有关使用和容量要求的信息，我们将增加默认速率限制。

用法： 生成图像： 图像生成端点允许您根据文本提示创建原始图像。生成的图像可以具有256x256、512x512或1024x1024像素的尺寸。较小的尺寸生成速度更快。您可以使用n参数一次请求1-10个图像。

```python
response = openai.Image.create(
  prompt="a white siamese cat",
  n=1,
  size="1024x1024"
)
image_url = response['data'][0]['url']
```

描述得越详细，你或你的最终用户得到想要的结果的可能性就越大。你可以在 DALL·E 预览应用程序中探索更多提示灵感的示例。这里是一个快速的例子：

| 提示                                  | 生成                                          |
| ----------------------------------- | ------------------------------------------- |
| 一只白色暹罗猫                             | ![](<../.gitbook/assets/image (1) (2).png>) |
| 一张特写的白色暹罗猫的摄影画作，瞪大好奇的眼睛，耳朵被背景的光线照亮。 | ![](<../.gitbook/assets/image (5).png>)     |

每张图片都可以通过response\_format参数以URL或Base64数据的形式返回。URL会在一小时后过期。

### 编辑&#x20;

图像编辑端点允许您通过上传蒙版来编辑和扩展图像。蒙版的透明区域指示图像应该被编辑的位置，提示应该描述完整的新图像，而不仅仅是被擦除的区域。这个端点可以实现像我们的DALL·E预览应用程序中的编辑器那样的体验。

```python
response = openai.Image.create_edit(
  image=open("sunlit_lounge.png", "rb"),
  mask=open("mask.png", "rb"),
  prompt="A sunlit indoor lounge area with a pool containing a flamingo",
  n=1,
  size="1024x1024"
)
image_url = response['data'][0]['url']
```

![](<../.gitbook/assets/image (14).png>)

> 提示词：一个阳光明媚的室内休息区，里面有一个装着火烈鸟的游泳池。

上传的图像和掩模必须都是小于4MB的正方形PNG图像，并且它们必须具有相同的尺寸。生成输出时，掩模的非透明区域不会被使用，因此它们不一定需要与原始图像匹配，就像上面的示例一样。

### 变体&#x20;

图像变体端点允许您生成给定图像的一个变体。

```python
response = openai.Image.create_variation(
  image=open("corgi_and_cat_paw.png", "rb"),
  n=1,
  size="1024x1024"
)
image_url = response['data'][0]['url']
```

![](<../.gitbook/assets/image (13).png>)

与编辑端点类似，输入图像必须是小于4MB的正方形PNG图像。

### 内容审核&#x20;

基于我们的内容政策，提示和图片会被过滤并在标记时返回错误。如果您对误报或相关问题有任何反馈，请通过我们的[帮助中心](https://help.openai.com/)与我们联系。

## 语言特定提示(只翻译NodeJS相关)

### 使用内存中的图像数据&#x20;

上面指南中的Node.js示例使用fs模块从磁盘读取图像数据。在某些情况下，您可能已经将图像数据存储在内存中。这是一个使用存储在Node.js缓冲区对象中的图像数据的API调用示例：

```javascript
// This is the Buffer object that contains your image data
const buffer = [your image data];
// Set a `name` that ends with .png so that the API knows it's a PNG image
buffer.name = "image.png";
const response = await openai.createImageVariation(
  buffer,
  1,
  "1024x1024"
);
```

### 使用TypeScript

如果您正在使用TypeScript，可能会遇到一些与图像文件参数有关的怪异问题。以下是通过显式转换参数来解决类型不匹配的示例：

```typescript
// Cast the ReadStream to `any` to appease the TypeScript compiler
const response = await openai.createImageVariation(
  fs.createReadStream("image.png") as any,
  1,
  "1024x1024"
);
```

这里有一个类似的例子，用于内存中的图像数据：

```typescript
// This is the Buffer object that contains your image data
const buffer: Buffer = [your image data];
// Cast the buffer to `any` so that we can set the `name` property
const file: any = buffer;
// Set a `name` that ends with .png so that the API knows it's a PNG image
file.name = "image.png";
const response = await openai.createImageVariation(
  file,
  1,
  "1024x1024"
);
```

### 错误处理

API请求可能由于无效输入、速率限制或其他问题而返回错误。这些错误可以通过try...catch语句处理，错误详细信息可以在error.response或error.message中找到：

```typescript
try {
  const response = await openai.createImageVariation(
    fs.createReadStream("image.png"),
    1,
    "1024x1024"
  );
  console.log(response.data.data[0].url);
} catch (error) {
  if (error.response) {
    console.log(error.response.status);
    console.log(error.response.data);
  } else {
    console.log(error.message);
  }
}
```
