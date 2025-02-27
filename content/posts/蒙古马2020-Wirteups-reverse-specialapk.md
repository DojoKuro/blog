---
title: '蒙古马2020 Wirteups:reverse-specialapk'
date: 2020-09-13 14:05:06
cover: cover.jpg
tags:
- reverse
- writeup
categories:
- writeups
---

**题目：specialapk**

程序下载:[specialapk](special_apk.zip)

<!-- more -->

jadx查看apk查看MainActivity

```java
package com.example.easyapk;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    private Button btn;
    public c ccc = new c();
    /* access modifiers changed from: private */
    public EditText edt;

    public native String a();

    static {
        System.loadLibrary("native-lib");
    }

    /* access modifiers changed from: protected */
    @Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, androidx.appcompat.app.AppCompatActivity, androidx.fragment.app.FragmentActivity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView((int) R.layout.activity_main);
        this.edt = (EditText) findViewById(R.id.flag);
        this.btn = (Button) findViewById(R.id.btn);
        this.btn.setOnClickListener(new View.OnClickListener() {
            /* class com.example.easyapk.MainActivity.AnonymousClass1 */

            public void onClick(View v) {
                System.out.println(MainActivity.this.a());
                System.out.println(MainActivity.this.edt.getText().toString());
                System.out.println(MainActivity.this.ccc.cc(MainActivity.this.edt.getText().toString(), MainActivity.this.a()));
                if (MainActivity.this.ccc.cc(MainActivity.this.edt.getText().toString(), MainActivity.this.a())) {
                    Toast.makeText(MainActivity.this, "正确！", 0).show();
                } else {
                    Toast.makeText(MainActivity.this, "错误！", 0).show();
                }
            }
        });
    }
}
```

显然，是java+so的套路，输入与native-lib数据做比对，看关键代码

```java
package com.example.easyapk;

public class c {
    private char[] ee = {'@', 9, 1, 10, '1', 3, 8, 18, 10, ')', 26, 26, 18, 30, 3, 13}; //key

    /* JADX INFO: Multiple debug info for r4v2 char[]: [D('j' int), D('dd' char[])] */
    public boolean cc(String a1, String a2) {
        char[] aa = new char[16];
        // 验证长度
        if (a1.length() != 16 || a2.length() != 16) {
            return false;
        }
        System.out.println(a1); //输入字符串
        System.out.println(a2); //jni生成字符串
        for (int i = 0; i < 16; i++) {
            aa[i] = (char) (a1.charAt(i) ^ a2.charAt(i)); //jni生成字符串与输入xor
        }
        System.out.println(aa);
        char[] bb = new char[16];
        for (int j = 0; j < 8; j++) {
            bb[j * 2] = aa[(j * 2) + 1]; //交换奇偶
            bb[(j * 2) + 1] = aa[j * 2]; //
        }
        char[] dd = new char[16];
        for (int s = 15; s >= 0; s--) {
            dd[s] = bb[15 - s];   //逆序
        }
        for (int y = 0; y < 16; y++) {
            if (dd[y] != this.ee[y]) { //比对
                return false;
            }
        }
        return true;
    }
}
```

加密方法有了，key也知道了，找到加密后的字符串解密就可以了，这题直接有输出，看输出就可以。没有模拟器和logcat环境的直接ida里看也可以。用了NewStringUTF8(a1,"easyapk_is_great")这个function。

```bash
com.example.easyapk I/System.out: 1234567890123456
com.example.easyapk I/System.out: easyapk_is_great
```

直接跑脚本

```python
#!/usr/bin/env python
#coding:utf-8

ee = [64, 9, 1, 10, 49, 3, 8, 18, 10, 41, 26, 26, 18, 30, 3, 13] # 方便起见把char都转换成了int
encStr = 'easyapk_is_great'

def deFlag(encStr):
    aa = [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
    a1 = []
    a2 = []
    flag = ''
    for i in range(len(encStr)):
        a2.append(ord(encStr[i]))
    bb = ee[::-1]
    for i in range(8):
        aa[i*2] = bb[(i*2)+1]
        aa[(i*2)+1] = bb[i*2]
    for i in range(16):
        a1.append(aa[i]^a2[i])
    for a in a1:
        flag += ''.join(chr(a))
    return flag

def main():
    print deFlag(encStr)

if __name__ == '__main__':
    main()
```

```bash
$ ./solve.py
flag{javaandso!}
```
