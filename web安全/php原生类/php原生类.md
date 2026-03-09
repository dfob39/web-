1.目录遍历
#### DirectoryIterator 类

```plain
<?php
$a = new DirectoryIterator("/");
echo $a;

<?php
$a = new DirectoryIterator("glob:///secret*");
echo $a;#结合glob协议读取所要的文件名
```
#### GlobIterato类
可以使用模式匹配找到文件名，不用在用glob协议

```plain
<?php
$a = new GlobIterator("/");
echo $a;
<?php
$dir=new GlobIterator("/fl*");
echo $dir;
```
2.文件读取
##### SplFileObject 类

```plain
<?php
$dir=new SplFileObject("/flag.txt");
echo $dir;#读取一行
<?php
    $dir = new SplFileObject("/flag.txt");
    foreach($dir as $tmp){
        echo ($tmp.'<br>');
    }#全部读取
```
