提权用模块 那个工具kiwi 还有 meterpreter基础上使用run 模块进行扫描得到对应的可以进行攻击的模块 用户权限变化 可以以管理员 权限执行命令 还有migrate 提权+存活
1.nmap扫描 
这个得扫描全端口，因为有一个是8000端口，
nmap --script vuln -sV -T4 -p- 10.49.159.56   扫描时间要很久，等....
2.漏洞发现
然后发现445的Windows 7 Professional Service Pack 1 版本默认没有打补丁，所以也存在永恒之蓝漏洞，当然这不是最主要，我们可以看到8000端口的服务是icecast，可以直接查询对应的漏洞，发现是CVE-2004-1561这个漏洞。
3.msf攻击
可以直接在msf里面搜索icecast就能找到对应模块exploit/windows/http/icecast_header，然后设置完参数可以直接拿到meterpreter shell，这个时候的权限只是普通用户权限。
4.提权/权限扩展
尝试提权，这里要用到 run post/multi/recon/local_exploit_suggester（对于windows比较适用）   其实meterpreter shell相当于在攻击机和靶机之间建立一个传输通道，命令可以直接通过这个通道传输到靶机进行执行，所以这就相当于把这个脚本输入到靶机进行扫描，然后找到msf里面可以使用的提权的模块。
接着还是use `exploit/windows/local/bypassuac_eventvwr` 然后set session 2，然后设置参数，run 就可以实现权限提升，这里可以通过getprivs命令查看扩展的权限，重点关注 能够允许我们取得文件所有权的权限为：SeTakeOwnershipPrivilege** ** 不过这时候还没拿到system权限，我们还要进行进程的移动才行，也就是因为我们有了这个权限，我们才能对高权限的进程进行操作，把meterpreter shell这个进程的权限附着到具有管理员权限的进程上，这时候在使用getuid查看我们已经具有管理员权限了.
5.密码获取
然后这时候可以load wiki 加载这个东西，接着help查看用法，然后creds_all拿到目标机上的所有凭据从而拿到明文的密码，也可以hashdump拿到哈希在解密
6.后渗透
通过管理员权限然后去进行一系列其他的操作，比如实时查看远程用户的桌面，修改系统上文件的时间戳等等，甚至收集信息为横向移动等做准备
​
​
