---
title: '蒙古马2020 Wirteups:reverse-rev'
date: 2020-09-13 10:45:06
cover: cover.jpg
description:
tags:
- reverse
- writeup
categories:
- writeups
---

**题目：rev**

程序下载:[rev](rev.rar)

> hint:yyyymmdd

<!-- more -->

# 0x00 暴力破解压缩包

根据提示yyyymmdd可知解压密码为8位年份，暴力破解得到解压密码

> 20200601

# 0x01 程序分析

解压程序看一下：

```
rev.exe: PE32 executable (console) Intel 80386, for MS Windows
```

我的环境是ubuntu，wine运行发现少两个dll文件，导入运行

```bash
wine rev.exe
please enter the flag:xxx #输入xxx
wrong flag!
```

**IDA分析**

看到_main跳转到_main_0，直接跟进F5看伪代码

![ida_main](ida_main.png)

![ida_main_0](ida_main_0.png)

查看关键function

![ida_b64encode](ida_base64encode.png)

显然是base64加密，再将加密结果strcpy到Dest上，再经过一道加密算法把Dest的每一byte加j，再与Str2比对。

看一下Str2的值：

![ida_str2](ida_str2.png)

**注意：** 这里'\x84'不能被解析所以Str2的长度应为20。

```python
Str2 = 'Znzk^8zuUMLvndT\x84vBOP'
```

# 0x02 python脚本

```python
import base64

enc = 'Znzk^8zuUMLvndT\x84vBOP'
tmp = ''
for i in range(len(enc)):
     tmp+=''.join(chr(ord(enc[i])-i))
flag = base64.b64decode(tmp)
print flag
```

```bash
$ ./exp.py
flag{g00dman
```
