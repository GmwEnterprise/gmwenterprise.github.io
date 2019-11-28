---
title: 可编辑元素属性contenteditable详细用法记录
date: 2019-11-28 13:04:46
tags:
  - html
  - javascript
---

### 前言

我要为我的小网站写一个自己使用的markdown编辑器，并且想使其能够拥有网页上类似Typora般的所见即所得体验，因此决定自己花时间开发一个。`textarea`不用考虑了，无法支持如此复杂的内容，所以考虑使用`div`属性`contenteditable`。本文将记录这个属性相关的使用方式。

### 基本使用

``` html
<div contenteditable="true">
    <p>This is content</p>
</div>
```

如此便使得div以及其子元素可被编辑。

----

经过一下午的思考后，作为