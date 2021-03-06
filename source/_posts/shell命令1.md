---
title: Shell详解（1） -- sort、uniq
date: 2017-12-09 20:45:23
tags:
- Shell
categories: 
- Shell
---

<!--## Shell详解（1） -- sort、uniq-->

## sort
sort命令参数据进行排序。
如果输入数据来自多个文件，那么sor会将多个文件当成一个文件处理，输出一个排序结果。
```
sort [-fbMnrtuk] [file or stdin]
选项与参数：
-f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
-b  ：忽略最前面的空格符部分；
-M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
-n  ：使用『纯数字』进行排序(默认是以文字型态来排序的)；
-r  ：反向排序；
-u  ：就是 uniq ，相同的数据中，仅出现一行代表；
-t  ：分隔符，默认是用 [tab] 键来分隔；
-k  ：以那个区间 (field) 来进行排序的意思
```

## uniq
uniq命令用于去除文件中的重复行，但去掉的数据要求重复行必须相邻，所以uniq经常与sort一起用。
```
uniq [-icu]
选项与参数：
-i  ：忽略大小写字符的不同；
-c  ：进行计数
-d  ：仅显示重复行。
-u  ：只显示唯一的行
-f<栏位>或--skip-fields=<栏位> 忽略比较指定的栏位。
-s<字符位置>或--skip-chars=<字符位置> 忽略比较指定的字符。
-w<字符位置>或--check-chars=<字符位置> 指定要比较的字符。
```