---
layout: post
title: "正则表达式"
date: 2019-01-19 
author: "WangX"
catalog: true
header-style: text
tags:
    - 正则表达式
---

## 正则表达式

#### 概述

* 正则表达式(Regular Expression)使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。Regular Expression即“描述某种规则的表达式”。
* 最初的正则表达式出现于理论计算机科学的自动控制理论和形式化语言理论中。1940年，Warren S. McCulloch 与 Walter Pitts 将神经系统中的神经元描述成小而简单的自动控制单元。1950年代，数学家 Stephen Cole Kleene利用称之为“正则集合”的数学符号描述此模型。
* 1960年代，Kenneth Lane Thompson 将此符号系统引入编辑器QED，和后来由他编写的Unix上的编译器ed，并最终在ed中引入grep。 (B语言，Unix，UTF-8, GO)
* grep原先只是ed下面的一个应用程序，名称来自于 g/re/p (globally search a regular expression and print) 在ed下输入 g/re/p这个命令后，会将所有匹配样式的字符串，以行为单位打印出来。1973，unix第四版，grep首次出现在man页面中。
* 正则表达式主要遵从POSIX BRE或者POSIX ERE标准。(POSIX Portable Operating System Interface 可移植操作系统接口) ERE是BRE的扩展版本，具体更强的处理能力。grep,vi,sed都属于BRE，是历史最早的正则表达式。元字符必须转义之后才才具有特殊含义。egrep,awk则属于ERE，元字符不用转义。

#### 语法

* 正则表达式是由普通字符和元字符组成的文字模式。可以将小的表达式结合在一起来创建更大的表达式。表达式的组件可以是单个的字符、字符集合、字符范围、字符间的选择或者所有这些组件的任意组合。
* 


* 普通字符：可打印字符和不可打印字符。可打印字符：所有大小写字符、所有数字、所有标点符号和一些其他字符。不可打印字符如下图所示：
!["不可打印字符"](/img/regular-expression/print-char.png "不可打印字符")

#### 元字符

* 正则表达式有多种不同的风格。下面介绍的是PCRE（Perl兼容正则表达式，Perl Compatible Regular Expressions，一个由Philip Hazel开发的，为很多现代工具所使用的库）适用于Perl或者Python编程语言。（grep或者egrep的正则表达式文法是PCRE的子集）

**数量限定符 6个**  指定正则表达式的一个给定组件必须要出现多少次才能满足匹配。
!["数量限定符"](/img/regular-expression/numlimit.png "数量限定符")

**定位符4个**  将正则表达式固定到行首或行尾，或描述字符串或单词的边界。
!["定位符"](/img/regular-expression/location.png "定位符")

**字符集合 []**  []内不论有几个字符，它都只代表一个字符。
!["字符集合"](/img/regular-expression/charset.png "字符集合")

**子表达式(pattern)**
**向后引用：** (pattern)将相关匹配存储到一个临时缓冲区中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 \n 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。
!["子表达式"](/img/regular-expression/subexpression.png "子表达式")

**其他元字符：**
!["其他元字符"](/img/regular-expression/other.png "其他元字符")

**运算符优先级**
!["优先级"](/img/regular-expression/priority.png "优先级")

