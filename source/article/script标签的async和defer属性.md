---
title: script标签的async和defer属性
tags: [JavaScript]
category: Frontend
filename: async and defer attribute of script tag
createdTime: 2019-10-23 18:07
---
首先，async和defer属性的作用都是指示浏览器加载外联脚本的具体方式，所以两个属性都必须用在外联脚本的script的标签上，否则无效.
```js
<script src="./a.js" async></script>
<script src="./b.js" defer></script>
```
### 不使用async和defer
正常情况下，script标签会阻塞浏览器对HTML文档的解析，遇到script标签，浏览器会遵循下列步骤：
下载脚本 → 运行脚本 → 继续解析后面的文档内容.

### async
async指示浏览器以异步的方式并行下载脚本，脚本下载过程中不会阻塞浏览器对HTML文档的解析，脚本下载完成后立即执行，因此脚本的执行顺序不确定，与script标签在HTML文档的顺序不一定保持一致.

### defer
defer指示浏览器先以并行方式下载脚本，也不会阻塞浏览器，与async不同的是，使用defer属性的脚本下载完成之后不会立即执行，而是在所有的defer脚本下载完成之后，并在DOMContentLoaded事件触发前按照script标签在HTML文档中的顺序执行.

### 总结
对script标签使用async和defer属性可以减少页面脚本总的下载时间，进而减少页面的白屏时间.
