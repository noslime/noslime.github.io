---
title: Markdown表格中代码块竖线问题
date: 2020-12-07 16:20:02
author: noslime
top: true
cover: true
categories: blog
tags: 
	- Markdown
	- case
keywords: Markdown
---

当我们编写markdown格式的文档，不可避免的会用到表格，有时候我们会在表格中用到 `|` 符号，由于这个符号属于markdown表格定义符号，所以不可避免会遇到一些显示问题。当我用typora编写markdown文件通过hexo生成网页时，由于在表格里写了代码块，恰好发生了格式显示错误，本文记录解决过程。

## 一、首行代码块中存在竖线

### 问题表现

#### Tpory编辑器显示：

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/vertical_line02.png)

#### Hexo网页显示

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/vertical_line03.png)

### 解决方案

#### 方案一

首先查看markdown源码

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/vertical_line01.png)

通过源码可知，markdown编辑器默认帮我们给非代码部分的 `|`字符添加了转义字符 `\`  ，所以可以手动给代码块部分添加

#### 方案二

通过标签插件提供的{%raw%} {%raw%} {%endraw%}{%endraw%} 标签包裹不需要解析的位置。

#### 方案三

直接使用HTML标签\<code\>替代反引号包裹含有竖线的代码块

## 二、非首行代码块中存在竖线

### 问题表现

#### Tpory编辑器显示：

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/vertical_line04.png)

#### Hexo网页显示

![](https://cdn.jsdelivr.net/gh/noslime/noslime.github.io@master/source/images/vertical_line05.png)

>  虽然导致错误形式不同，但解决方案同问题一，此处不在赘述
