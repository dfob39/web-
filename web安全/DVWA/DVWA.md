**1.brute force(暴力破解)**
low：直接使用burpsuite爆破，不过要选择集束炸弹的双变量模式进行爆破，一个是admin。密码是password，直接爆破得到结果1
medium：加了sleep(2)，只是延长了爆破时间，每次验证后需要两秒后才能进行下一次，还是爆破得到结果
high：加了一个所谓token认证，但是用burpsuite不知道怎么搞，知道原理就好了
impossible:加了验证3次的次数限制，超过后会锁定，而且还有token验证
**2.Command Injection（命令注入）**
low：就是输入ip地址，构造恶意命令进行执行，输入127.0.0.1|ipconfig或者其他常见的命令拼接的符号比如&，&&等都可以执行
medium：增加了对&&和；的限制，其他的还是一样可以执行
high：大部分命令拼接的符号都被过滤，但是仔细看发现他是把'| ',转成空格，|后面多了一个空格，所以输入|实际上仍可以执行命令，这里不知道为什么；好像都是不能执行（可能是不是linux系统？）
impossible：直接用白名单验证
**3.CSRF（跨站请求伪造）**
 CSRF，全名 Cross Site Request Forgery，跨站请求伪造。很容易将它与 XSS 混淆，对于 CSRF，其两个关键点是跨站点的请求与请求的伪造，由于目标站无 token 或 referer 防御，导致用户的敏感操作的每一个参数都可以被攻击者获知，攻击者即可以伪造一个完全一样的请求以用户的身份达到恶意目的。  
low：发现只要账户和密码输入一致那么密码就会被更改，然后执行完发现网站的URL也变成了[http://dvwa:8898/vulnerabilities/csrf/?password_new=123&password_conf=123&Change=Change#](http://dvwa:8898/vulnerabilities/csrf/?password_new=123&password_conf=123&Change=Change#)，那么可以发现实际上就是改变网站的URL中的参数的值来修改密码的，那么我可以在空白页面直接执行这个URL就可以实现改密码的功能，也可以这样理解，一个网站有CSRF漏洞，那么我可以构造一个URL让别人点击执行从而达到修改密码或者其他目的
medium：和low相比，多了一个refered头的验证，也就是说1确保请求时来自dvwa这个网站的才会执行，如果只是复制网址诱导执行是不行的，那么就可以抓包，找到dvwa网站的refer头，然后给新的url网址执行时添加refer头，重放就可以对密码进行修改了
high：本质上还是要获取token，就是获取的方式比较多，可以使用xss来获取（还不理解，回头再看），原理就是获取token，其他的一致
impossible：这里代码要求先输入密码才能进行修改密码，未知密码的情况无法进行操作
**4.file include(文件包含)**
low：没有任何过滤，可以直接进行文件包含，包含本地或者网址都可以，比如包含https://baidu.com, 或者C:\Users\lenovo\Desktop\112.txt  这种就可以包含
medium：把http://和https://进行过滤，还把../和..\进行过滤，不能进行目录转换，但是可以双写绕过过滤hthttps://tps://baidu.com就可以绕过
high：白名单绕过，只能使用以file开头的文件绕过，那么这里可以使用file协议file:///C:\Users\lenovo\Desktop\112.txt  这样可以，但是file协议也只能访问本地文件，不能包含网上的地址和文件
impossible：也是使用白名单进行防护，白名单之外的都无法包含
**5.file upload （文件上传）**
low：没有任何限制，可以随便传
medium：限制了上传的文件的文件类型是content-type ，可以上传php文件，然后使用bp改成image/jpeg就可以
high：这里使用图片马可以成功上传，然后要用到文件包含漏洞里面的low关，直接把上传后的地址[http://dvwa:8898/hackable/uploads/2.jpg](http://dvwa:8898/hackable/uploads/2.jpg)包含到里面，这里我是参数，还要一步post参数a=phpinfo();,然后就可以，这里也可以使用图片+一句话木马，因为这里加了getimagesize函数，它可以读取图片信息，不是图片返回false，所以直接用简单图片马是不行的，传人入后用蚁剑连，直接用phpinfo应该也行
impossible： 对上传的文件进行了重命名（搞了一个MD5的加密），还增加了token值的校验，对文件的内容也做了严格的检查。  
**6.Insecure CAPTCHA（不安全的验证码）**
low：通过源码分析可以分析修改密码步骤分为两步，第一个是服务器对客户端进行身份验证，不过这个验证是不安全的，只通过一个step=1一个参数进行验证，而第二步也一样，只要step=2和两个密码一致就会直接进行修改密码，那么我们甚至可以不用进行验证，直接让step=2就可以成功修改密码
medium：比low多了一个参数passed_captcha，要成功修改密码还要让这个等于1,总之这个参数要不为空就可以
high：多了下面这个，相当于两个必须有一个是符合的，这里我们无法修改resp参数，只能满足下面那个条件，就是添加一个g-recaptcha-response参数，然后在让user-agent头等于下面这个值就可以

```python
if (
		$resp || 
		(
			$_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
			&& $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
		)
	){
```
impossible：这里添加防御csrf的技术，同时要求输入原密码 利用PDO技术防护sql注入 
7.sql注入
low：一个普通的联合注入，是单引号闭合，两个字段，不过不知道为啥一直报错
medium：还是联合注入，不过这个是数字型注入，没有字符
high：加了一个session_id来进行获取id值，其他的和low等级一样
8.xss  反射型
low：什么都没有过滤。可以直接进行弹窗，<script>alert('xss')</script>
medium：把script替换掉，但是只要随便大小写就能绕过，或者是双写绕过，或者使用

```python
# 事件属性注入（无需<script>）
?name=<img src=x onerror=alert('XSS')>
?name=<a href=javascript:alert('XSS')>点击触发</a>
?name=<body onload=alert('XSS')>

# 其他标签+事件
?name=<div onclick=alert('XSS')>点我</div>

```
也可以绕过
high：尝试使用正则进行过滤，但是对onclick，onerror等属性依旧可以成功执行比如

```plain
<a href=javascript:alert('XSS')>点击触发</a>

# Payload 1：img标签+onerror（无script，直接执行）
?name=<img src=x onerror=alert('XSS绕过')>

# Payload 2：a标签+javascript伪协议（无script）
?name=<a href=javascript:alert('XSS')>点击触发</a>

# Payload 3：div标签+onclick（无script）
?name=<div onclick=alert('XSS')>点我执行</div>

```
这种就可以成功执行
9.xss dom型
 脚本不经过服务器，仅在客户端通过**DOM 操作动态执行**（服务器未参与脚本的注入与返回）  
