---
title: 倒排索引
date: 2021-05-07 15:34:05
tags: 倒排
categories: 算法
---

在搜索引擎中每个文件都对应一个文件ID，文件内容被表示为一系列关键词的集合（实际上在搜索引擎索引库中，关键词也已经转换为关键词ID）。例如“文档1”经过分词，提取了20个关键词，每个关键词都会记录它在文档中的出现次数和出现位置。


<!--more-->

![](https://mstacks.oss-cn-beijing.aliyuncs.com/mstacks/article/2020-04-12/20200412193553806.png)

搜索引擎会将正向索引重新构建为倒排索引，即把文件ID对应到关键词的映射转换为关键词到文件ID的映射，每个关键词都对应着一系列的文件，这些文件中都出现这个关键词。

![](https://mstacks.oss-cn-beijing.aliyuncs.com/mstacks/article/2020-04-12/20200412175602593.png)

稍作转化，搜索引擎的索引其实就是实现“单词-文档矩阵”的具体数据结构。

![](https://mstacks.oss-cn-beijing.aliyuncs.com/mstacks/article/2020-04-12/20200412175801844.png)

可以有不同的方式来实现上述概念模型，比如“倒排索引”、“签名文件”、“后缀树”等方式。但是各项实验数据表明，“倒排索引”是实现单词到文档映射关系的最佳实现方式。

-  **倒排索引(Inverted Index)**：倒排索引是实现“单词-文档矩阵”的一种具体存储形式，通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：“单词词典”和“倒排文件”。
-  **倒排列表(PostingList)**：倒排列表记载了出现过某个单词的所有文档的文档列表及单词在该文档中出现的位置信息，每条记录称为一个倒排项(Posting)。根据倒排列表，即可获知哪些文档包含某个单词。
- **倒排文件(Inverted File)**：所有单词的倒排列表往往顺序地存储在磁盘的某个文件里，这个文件即被称之为倒排文件，倒排文件是存储倒排索引的物理文件。
- **单词词典(Lexicon)**：搜索引擎的通常索引单位是单词，单词词典是由文档集合中出现过的所有单词构成的字符串集合，单词词典内每条索引项记载单词本身的一些信息以及指向“倒排列表”的指针。

![](https://mstacks.oss-cn-beijing.aliyuncs.com/mstacks/article/2020-04-12/20200412192756360.png)

ES就是使用的是倒排索引，单词词典使用了稀疏hash。


