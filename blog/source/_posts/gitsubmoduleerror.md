---
title: "记一次fatal: Pathspec 'blog/themes/next/1' is in submodule 'blog/themes/next'"
date: 2017-12-30 21:47:46
categories: stackoverflow
tags: [git,stackoverflow]
---
最近捣鼓hexo博客，发现next主题不在版本库中无法被add进去所以就想强制添加发现报错了如下：

```
fatal: Pathspec 'blog/themes/next/1' is in submodule 'blog/themes/next'
```
<!--more-->
然后各种资料发现如果当你在你的当前的git版本中clone了另一个扩展或者说事版本库的时候我们为了能将这个扩展添加在本git版本中去会删了他原有.git目录，但是如果你删了，并且add后git会创建一个子模块就会抱这样的一个错误，这时候就需要处理这样一个情况需要一步即可：
```
git rm --cached file
```
然后add 即可
