---
title: Linux三剑客之awk
date: 2017-12-20 16:10:25
tags: [linux,awk]
categories: Linux
---
因为经常使用awk，这次抽出时间把awk总结一下，正好使用了hexo，就准备写一篇吧，下面是搜搜到的说明：
awk是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势
<!--more-->

## 命令格式

>awk [-F|-f|-v] 'BEGIN{} // {command1; command2} END{}' file

执行流程是读取文件的时候先执行BEGIN初始代码块的内容->执行`//`里的匹配代码块，这里可以是字符串或者正则表达式->执行`{command1; command2}`里代码块，多条命令用；分隔->再执行END代码块里的命令。
这里需要说明的事，BEGIN和END模块都是可以单独存在的，在BEGIN模块中主要是对一个变量的赋值初始化定义。
在`\\`匹配代码块中 纯字符匹配：`\string\` ，纯字符不匹配：`!\string\` ，字段匹配`~\string1\` ，字段不匹配：`!~\string1|string2\`。字段匹配的一般都是`$n ~ /string/` $n表示某段，$1整段或的其他段 
同时在`{}`的结构体中还可以使用`if`里的条件条件，使用`（）`包裹起来


## 常用参数

1. -F 置顶分隔符，它可以是一个字符串或者正则 单个可以直接使用`-F:` 或者`-F ':'` 多个分隔符用`[]`包含 如：`[#:/]`



## 需要说明的点

在awk中有很多内置的变量都有这不同含义，也会方便简化我们的一个操作
1. NF 字段数量变量 $NF则表示最后一位 当然$(NF-1)表示倒数第二个 如同$0表示当前行 $1表示每行的第一个字段
2. NR 每行的行号
3. OFS 输出域分隔符，主要使用在BEGIN中定义 H
4. print 打印命令

## 实例

```
#以分隔符 ：打印第一列
awk -F: '{print $1}' /etc/passwd

#以：和\分隔打印第一第二列并且以*分隔符分隔
awk -F "[:/]" '{print $1,$2}' OFS="*" /etc/passwd

#查看有多少个字段和输出最后一位
awk -F ":" '{print NF,$NF}' /etc/passwd

#输入字段大于6的最后一位
awk -F ":" 'NF>6{print NF,$NF}' /etc/passwd

#查找含有root的行
awk '/root/{print $0}' /etc/passwd

#查找不含root的行
awk '!/root/{print $0}' /etc/passwd

#匹配含有多字段的行
awk '/root|mysql/{print $0}' /etc/passwd

#匹配两个查找字段之间的行
awk '/root/,/db/{print $0}' /etc/passwd

//第一个字段匹配wang才显示
awk -F: '$1~/wang/' /etc/passwd

#匹配是也可以使用正则表达式
awk '/^w/' /etc/passwd

#第一段中不含有root的行
awk '{if($1!~/root/)print $0}' /etc/passwd

#只显示一行
seq 1 5 | awk 'NR==1 {print}'

#if ... else ...查找一个段包含fpsd打印出来没有的打印 not found
awk -F: '{if ($1~/fpsd/) {print $1} else {print "not found"} }' /etc/passwd

#查找第三行是否大于100并且记录总数
awk -F: 'BEGIN {A=0;B=0}{if($3>100){A++;print "biger"} else {B++;print "small"}} END{print A,"\t",B}' /etc/passwd

#当前目录中文件的大小
ls -l|awk 'BEGIN{total=0}!/^d/{total+=$5}END{print total/1024/1024,"MB"}'
```



