---
title: Linux三剑客之sed
date: 2017-12-21 19:55:53
tags: [linux,sed]
categories: Linux
---
Sed本质上是一个编辑器，但是它是非交互式的，这点与VIM不同；同时它又是面向字符流的，输入的字符流经过Sed的处理后输出。这两个特性使得Sed成为命令行下面非常有用的一个处理工具。
<!--more-->

## 命令格式

>sed [options] script filename
>sed -f scriptfile filename

sed会以此读取文件中的行并且加入到缓存当中去并用script来处理该内容，并且不会影响到原有的文件（使用-i参数是会影响到原有文件）一般多指令的情况下会使用`;`分隔(也可以使用参数-e)

## 模式空间
sed的处理流程如下图显示:
![](https://ws1.sinaimg.cn/large/65ca5a5cly1fmoo6jcun5j20hz0f0gm7.jpg)

1. 读入新的一行内容到缓存空间；
2. 从指定的操作指令中取出第一条指令，判断是否匹配pattern；
3. 如果不匹配，则忽略后续的编辑命令，回到第2步继续取出下一条指令；
4. 如果匹配，则针对缓存的行执行后续的编辑命令；完成后，回到第2步继续取出下一条指令；
5. 当所有指令都应用之后，输出缓存行的内容；回到第1步继续读入下一行内容；
6. 当所有行都处理完之后，结束

由次可以，sed不是将一个命令用于每一行，而是一个行的方式来单个处理，同时每一行被读取到缓存中，并且每次只会缓存一行，我们称之为`模式空间`（pattern space）每个应用到的命令都会相互影响，因为每个命令处理之后都会应用到模式空间,所以说命令之间的顺序就很重要了,这种由于只会缓存一行到模式空间中去所以，用sed处理大文件就不会有什么问题了。如下图是一个简单的模式空间转换的流程图

![](https://ws1.sinaimg.cn/large/65ca5a5cly1fmpmyorywej20hp0almxr.jpg)

1. 先读取一行 `The Unix System`到模式空间中
2. 应用第一条命令将 `Unix` 替换成 `UNIX`， 这个时候这一行内容变成 `The UNIX System` 并存到模式空间中区
3. 然后应用第二条命令将 `UNIX System` 替换成 `UNIX Operating System` 这时候这一行的内容变成 `The UNIX Operating System` 并存储到模式空间
4. 此时执行完所有的命令，输出模式空间中的行

## 地址匹配
地址匹配在sed操作中和非常重要，因为sed是某人处理每一行，对每一行处理所有的指令，这个时候就需要用地址匹配来限制sed处理的行,例如：

```
sed '/root/s/sa/saaaa/g' /etc/passwd
```

这时候就只会匹配包含向/root/是一个正则的行才会将sa替换为saaaa 类似 cook sa book中的sa就不会被替换。
sed中的地址匹配，可以是0个，1个或者1个地址对（2个地址）。当然地址可以是`正则`（如\root\），`行号`，或者`行号符` (如最后一行$)。
例如

```
#通过地址对
sed '/lda/,/krbtgt/s/var/super star/g' /etc/passwd

#通过行号
sed '2s/sa/saaaaaaa/g' temp
sed '1,2s/sa/saaaaaaa/g' temp

#通过特定符号
sed '$s/sa/saaaaaaa/g' temp
```

* 没有地址匹配将应用到所有的行上
* 如果指定了地址匹配，命令将只应用到匹配的行上
* 如果指定了地址对（addr1,addr2）命令将应用到包括起始行之间的所有行,如果只匹配到了addr1，则开启命令开关了，直到匹配到addr2结束,如果没有匹配到addr2就会一直到最后行
* 如果地址后面有一个感叹号（！），则将编辑命令应用到不匹配该地址的所有行；

```
sed '/root/!s/sa/saaaaaaa/g' temp
```

总结一下：
1. 对于addr1,addr2地址对的匹配，如果addr1匹配上回开启匹配开关，直到addr2匹配完成。如果没有匹配到addr2就会默认直到最后一行
2. 如果对于addr1是行号的话，如果新读入的行大于等于addr1行号，则匹配。如果小于addr1行号的话则不匹配，同理addr2是行号的话，新入得行小于addr2则匹配，大于addr2的话则不匹配。

## 基础命令

名称|命令|语法|说明
---|---|---|---
替换|s|[address]s/pattern/replacement/flags|替换匹配的内容这里的 `/`分隔符是可以替换成其他的
删除|d|[address]d|删除匹配的行
插入|i|[line-address]i\text|在匹配行的前方插入文本
追加|a|[line-address]a\text|在匹配行的后方插入文本
行替换|c|[address]c\text|将匹配的行替换成文本text
打印行|p|[address]p|打印在模式空间中的行
打印行号|=|[address]=|打印当前行行号
打印行|l|[address]l|打印在模式空间中的行，同时显示控制字符
转换字符|y|[address]y/SET1/SET2/|将SET1中出现的字符替换成SET2中对应位置的字符
读取下一行|n|[address]n|将下一行的内容读取到模式空间
读文件|r|[line-address]r file|将指定的文件读取到匹配行之后
写文件|w|[address]w file|将匹配地址的所有行输出到指定的文件中
退出|q|[line-address]q|读取到匹配的行之后即退出

### 替换命令：s

>[address]s/pattern/replacement/flags

address是指的地址，s是替换命令，pattern表示匹配表达式，replacement表示需要替换成的内容，flags表示替换的标志位，标志位一般有没有一种或多种如：
1. n: 一个数字（取值范围1-512），表明仅替换前n个被pattern匹配的内容
2. g: 表示全局替换，替换所有被pattern匹配的内容；
3. p: 仅当行被pattern匹配时，打印模式空间的内容
4. w file：仅当行被pattern匹配时，将模式空间的内容输出到文件file中；

当flag没有的时候默认匹配第一个后面的不会替换
```
echo "1,1,1,1,"|sed 's/,/*/'
1*1,1,1,
```

当flag为n数字的时候表示第几次匹配的时候替换其他匹配不替换
```
echo "1,1,1,1,"|sed 's/,/*/2'
1,1*1,1,
```

当flag为g标志位时表示全局替换
```
echo "1,1,1,1,"|sed 's/,/*/g'
1*1*1*1*
```

### 删除命令：[address]d

删除命令用于删除一条或多条视频例如`1,3d`就会删除1-3行的内容。删除命令的原理是删除模式空间中的内容，所以一旦前面的命令是删除命令的时候，后面的命令将不会执行了，因为模式空间中的内容已经被清空了。这时候就会读取新的行到模式空间中来。

删除1-5行
```
 sed '1,5d' temp
```

删除匹配root的行
```
sed '/root/d' temp
```

删除地址对之间的行
```
sed '/root/,/mysql/d' temp
```

### 插入/追加/替换行命令：i/a/c
>[line-address]i\text
>[line-address]a\text
>[address]c\text

这三个当中插入和追加只能使用当个地址，替换可以使用多个地址
```
sed '2a\
--------------------------------------
' temp
this is root page sa
i am not sa
--------------------------------------
haha
sdfasdf
sadfsadfdsfqwerqwer
wwqrewrq
```

### 打印命令：p/l/=
>[address]p
>[address]l
>[address]=

下面的-n是指只显示匹配的行
```
sed -n '2p' temp
i am not sa

sed '=' temp
1
this is root page sa
2
i am not sa
3
haha
4
sdfasdf
5
sadfsadfdsfqwerqwer
6
wwqrewrq
```

### 特殊字符
1. & 字符这个字符是存储匹配模式里的内容

在匹配的内容后面加ok
```
sed 's/root/& ok/g' /etc/passwd
```

2. ^ $ 开头结尾

```
#将3到最后一行删除然后将std替换成wangchao
sed -e '3,$d' passwd  -e 's/std/wangchao/g'

查找到wangchao在该行的开头加一个hello ^这个表示开头 同理$表示结尾
sed '/wangchao/s/^/ hello/' /etc/passwd 
```


