---
title: Linux三剑客之grep
date: 2017-12-21 14:55:01
tags: [linux,grep]
categories: Linux
---
Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户
<!--more-->
## 命令格式

>grep [option] pattern file

## 主要参数
1. -v 反向匹配，即显示不配的行
2. -B <行数> 匹配处理的行以外 再显示之前的n行
3. -A <行数> 同上 之后的N行
4. -C <行数> 同上 显示匹配行前后N行
5. -n 显示行号
6. -c 只显示匹配的行数即匹配了多少行
7. -o 只显示匹配的部分
8. -l 查询多文件时只输出包含匹配字符的文件名
9. -i 忽略大小写
10. -r 递归查询 目录及子目录的文件

## 实例

```
#查找不含root的行
grep -v "root" /etc/passwd

#在文件中查找匹配行并显示行号
grep -n "root" /etc/passwd

#在多文件中查找显示匹配的文件
grep -l "root" /etc/passwd /etc/group

#递归查询所有含root的行
grep -r "root" /etc/

#查找包含root行的前5行 -C -A同理
grep -B 5 "root" /etc/passwd

#查找包含root开头的行
grep "^root" /et/passwd
```

