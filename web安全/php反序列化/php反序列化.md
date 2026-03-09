**序列化     serialize（）**
序列化是将变量或对象转换成字符串的过程，用于存储或传递 PHP 的值的过程中，同时不丢失其类型和结构。  
**反序列化  unserialize（）**
 反序列化是将字符串转换成变量或对象的过程  

```php
highlight_file($this->x)
输出该文件的文件内容  
```
1.序列化字母标识
a：array 数组     i：integer 整数    b：boolean  bool值
d：double  双精度浮点数     O：class 对象    s：string 字符串
N: null 空值     r：reference  对象或数组引用   C：custom object  自定义对象序列化    S：encodeed string    U:unicode string
2.魔术方法
__construct 当一个对象创建时被调用，即类被实例化时触发
__destruct 当一个对象销毁时被调用，
__wakeup() 使用unserialize时触发   当序列化字符串中表示对象属性，即成员属性的个数大于真实存在的成员个数时，可以绕过__wakeup（）函数
__sleep() 使用serialize()时触发
__call() 对不存在的方法或者不可访问的方法进行调用就自动调用
__get() 用于从不可访问的属性读取数据即私有属性或不存在属性，如private
__set() 在给不可访问的(protected或者private)或者不存在的属性赋值的时候，会被调用
__isset() 在不可访问的属性上调用isset()或empty()触发
__unset() 在不存在的或不可访问属性上外部使用unset()时触发
__toString() 把对象当作字符串使用时触发,返回值需要为字符串
$a=new A();
$a->name=new B();
__invoke() 当将对象调用为函数时触发
3.对象序列化
本质上讲，对象是根据类的定义创建的实例
对象（o）：类名长度：“类名”：成员属性个数
{成员名类型：成员名长度：“成员名”；成员值类型：成员值长度：“成员值”……}
其中， 在PHP序列化对象时，`private`和`protected`成员变量会被特殊处理。具体来说，对于`private`成员变量，序列化时会添加类名作为前缀，而对于`protected`成员变量，则会添加一个星号（*）作为前缀。  

```plain
//对象序列化
 
<?php
class test
{
    private $a1="haha";
    protected $a2="dada";
    public $a3="sasa";
    public $b=true;
    public $c=123;
}
$d=new test();
echo serialize($d);
?>
//输出为：
O:4:"test":5:{s:8:" test a1";s:4:"haha";s:5:" * a2";s:4:"dada";s:2:"a3";s:4:"sasa";s:1:"b";b:1;s:1:"c";i:123;}
```
4.pop链序列化(对象的成员属性为另一个对象)
相当于成员本身的名称没变，值变成了宁一个对象，序列化时改变其值，使其指向一个对象即可
对象：对象名长度 ：“对象名”：  成员个数：
{成员类型：成员名长度：成员名 ；对象：对象名长度 对象名 对象成员个数{}}
这里有一个很重要的点，每个魔术方法生效都是基于自己所在的对象内生效，比如to_string()方法，是被当成字符串调用的那个对象里面存在这个方法才会调用to_string()这个方法，否则就会报错

```plain
//pop链序列化
 
<?php
class test1
{
    public $a="haha";
    public $b=true;
    public $c=123;
}
class test2
{
    public $h="hhh";
    public $d;
}
 
$m=new test1();
$n=new test2();
$n->d=$m;
echo serialize($n);
 
?>
//输出：
O:5:"test2":2:{s:1:"h";s:3:"hhh";s:1:"d";O:5:"test1":3:{s:1:"a";s:4:"haha";s:1:"b";b:1;s:1:"c";i:123;}}
//对象的成员属性为另一个对象，序列化值出现如上嵌套
```
5.数组的序列化
数组（a）：数组元素个数
{元素下标：元素类型 元素长度 元素值……}

```plain
//数组序列化
<?php
$ha=array("haha",123,true,"ggg");
echo serialize($ha);
?>
//输出：
a:4:{i:0;s:4:"haha";i:1;i:123;i:2;b:1;i:3;s:3:"ggg";}
//解释：a表示这是一个数组的序列化，成员属性名为数组的下标，格式"i:数组下标;"
```

```plain
//__toString()和__invoke()
 
<?php
class test
{
    public $a="haha";
    public function __toString()
    {
        return "被当成字符串了--";
    }
    public function __invoke()
    {
        echo "被当成函数了";
    }
}
 
$a=new test();
echo $a;
$a();
 
?>
 
//输出：
被当成字符串了--ss被当成函数了
 
//ehco $a 把对象当成字符串输出触发了__toString(),$a() 把对象当成
//函数执行触发了__invoke()
```
 私有属性的属性名应该是 `\00类名\00属性名`（`\00` 是 ASCII 值为 0 的空字符，不是空格）。例如 `Name` 类的 `username` 序列化后是：`s:14:"\00Name\00username"`（长度 14，包含两个空字符）。  

```python
//__call（）和其他魔术方法
 
<?php
class test
{
    public $h="haha";
 
    public function __call($arg1,$arg2)
    {
        echo "你调用了不存在的方法";
    }
}
 
$a=new test();
$a->h();
 
?>
 
//输出：
你调用了不存在的方法
 
//$a->h()调用了不存在的方法触发了__call()方法，其他魔术方法类似不再演示
```

```python
<?php
class w44m
{
    private $admin = 'w44m';
    protected $passwd = '08067';
}
class w22m
{
    public $w00m;
}
class w33m
{
    public $w00m;
    public $w22m="Getflag";
}
 
$a=new w22m();
$b=new w33m();
$c=new w44m();
$b->w00m=$c;
$a->w00m=$b;
 
$payload=serialize($a);
echo "?w00m=".urlencode($payload);  //存在private和protected属性要url编码
 
?>
 
//输出为：
?w00m=O%3A4%3A%22w22m%22%3A1%3A%7Bs%3A4%3A%22w00m
%22%3BO%3A4%3A%22w33m%22%3A2%3A%7Bs%3A4%3A%22w00m%22%
3BO%3A4%3A%22w44m%22%3A2%3A%7Bs%3A11%3A%22%00w44m%00ad
min%22%3Bs%3A4%3A%22w44m%22%3Bs%3A9%3A%22%00%2A%00pas
swd%22%3Bs%3A5%3A%2208067%22%3B%7Ds%3A4%3A%22w22m%22%
3Bs%3A7%3A%22Getflag%22%3B%7D%7D
```
