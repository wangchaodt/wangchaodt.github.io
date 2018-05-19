---
title: php-fmp到底是什么？
date: 2018-05-19 16:48:13
tags: [php,php-fpm,cgi]
categories: PHP
---
很多时候是php-fpm一直在用，但却不知道是干什么用的，所有今天会梳理一下，做一个整体的理解。整体一句话就是php-fpm是php-fastcgi的管理器，为php提供服务。

## cgi

说到php-fastcgi就要了解什么是cgi？
网上普遍说法是 `通用网关接口` 就是一个通信协议，方便外部程序与web server质检交互的数据格式，他的一个执行流程是当web server 接收到一个请求之后，会启动相应的cgi进程(php就会启动php相应的cgi进程就是Fastcgi后面会讲到)，然后cgi会进行一系列的操作，然后将处理结果通过协议规定的格式返回，然后cgi进程会退出，最后web server会将结果返回给请求方，周而复始 请求->生成cgi->执行->退出。

## FastCgi
了解完cgi我们就知道cgi，在性能上可能不是很完美，他没接收一个请求都会生成一个cgi同时也是初始化相应的环境，这个就会比较耗时，这时候FastCgi的出现就是解决这个问题，他是其实就是cgi的优化版。FastCgi首先会启动一个master进程，解析配置文件和初始化环境，同时会启动很多个worker进程，每当有请求过来时，master会请求传递给一个worker，这样就不需要重复的初始化，加载等一系列的操作，当然系统繁忙是会多启动几个worker，当系统空闲是也会kill掉一部分的worker，当然这样也会长期占用一定的内存(但作为内存不值钱的今天，这点消耗微不足道)

## php-cgi
从上面了解我们就可以知道了php-cgi其实就是一个cgi程序，php-cgi是php官方自带的一个FastCGI管理器，实现了FastCGI的协议（5.4以前，5.4后就是php-fpm），当php.ini修改后原有的php-cig不会生效（因为这些是提前fork的），只有当前的php-cgi销毁掉之后，新的才会生效，这样就不能平滑的过渡了。

## php-fpm
终于轮到我们的主角了，`php-fpm` 有人说php-fpm是php-cig的管理器，这是对的，但是好像也不全对，php-fpm其实还是一个fastcgi协议的实现，他可以看做一个实现了fastcgi协议的cig管理器。他可以动态的进行进程调度，这样的一个交互方式让php-cgi独立于httpd而存在。如果要让php-fpm以这种方式运行就需要`--enable-fpm`选项

## 结语
上面就是一个大概的php-fpm的原理的他的执行的一个流程，这一切是是互联网技术发展带来的结果。上面如有错误，欢迎大家指正。
