账号和密码已经告诉你了，实际上要做的就是提权
1.sudo提权
使用sudo -l查看可以用于sudo提权的命令，然后可以逐个尝试是否能行。这里就不详细讲了
2.hash密码爆破
理论上passwd应该存储正常密码，而shadow存储hash加密密码，但是这个靶机里面insecurity账户的密码存储在passwd，但是是hash加密的密码，而shadow文件打不开，那么我们尝试使用 John the Ripper（约翰）  进行爆破，首先把完整的insecurity一行存储在文件里，然后执行john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 进行爆破，很快得到密码P@ssw0rd，可以直接登录
