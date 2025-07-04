---
title: Full ttys
date: 2025-07-03 16:20:24
description: Linux提权
tags:
- Linux
- Privillege Escalation
categories:
- CTF
---

```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
# or
python -c 'import pty; pty.spawn("/bin/bash")'
# or
python2 -c 'import pty; pty.spawn("/bin/bash")'
# or
SHELL=/bin/bash script -q /dev/null

# make our shell even more perfect
Ctrl+z
stty raw -echo;fg
Enter x2
export TERM=xterm
```
