1.select top子句
 SELECT TOP 语句用于在 SQL 中限制返回的结果集中的行数， 它通常用于只需要查询前几行数据的情况，
即返回前面几行  
`SELECT TOP` 在 SQL Server 和 MS Access 中使用，而在 MySQL 和 PostgreSQL 中使用 `LIMIT` 关键字。  
sql server/MS  access语法
```plain
SELECT TOP number/percent column1, column2, ...
FROM table_name;
```
MYSQL 语法
```plain
SELECT column1, column2, ...
FROM table_name
LIMIT number;
```
2.like操作符
`LIKE` 操作符是 SQL 中用于在 `WHERE` 子句中进行模糊查询的关键字，它允许我们根据模式匹配来选择数据，通常与 `%` 和 `_` 通配符一起使用。  
SQL like语法
```plain
SELECT column1, column2, ...
FROM table_name
WHERE column_name LIKE pattern;
```
**pattern**：搜索模式  （即所要匹配的字符串或单个字符）
**通配符**

- `%`：匹配任意字符（包括零个字符）。
- `_`：匹配单个字符。实例：

```plain
SELECT ProductName, Category
FROM Products                      使用 % 通配符找出所有以 "iPhone" 开头的产品 
WHERE ProductName LIKE 'iPhone%';   即匹配iPhone的所有字符

 使用 _ 通配符找出所有产品名称为三个字符，第一个字符为 "D"，第二个字符为 "e" 的产品：
SELECT ProductName, Category
FROM Products
WHERE ProductName LIKE 'De_';
```
使用not关键字，可以选择不匹配的字符和字符串
3.[charlist]通配符
选取name以G","F","S"开始所有网站
```plain
SELECT * FROM Websites
WHERE name REGEXP '^[GFs]';
```
name以A-H字母开头网站
```plain
SELECT * FROM Websites
WHERE name REGEXP '^[A-H]';
```
4.in操作符

```plain
SELECT column1, column2, ...
FROM table_name
WHERE column IN (value1, value2, ...);
value为所要查询的多个列的名称，可以获得所查询的多个行
```
5.between 操作符
between and句子  not between and 句子
注意：between 操作符在不同数据库内会产生不同效果，可能两边都包含，可能包含一边，可能两边都不包含
6.别名
SQL可以为表或列指定别名
列
```plain
SELECT column_name AS alias_name
FROM table_name;
```
表
```plain
SELECT column_name(s)
FROM table_name AS alias_name;
```
7.join  连接
join=inner join
```plain
SELECT column1, column2, ...（即为所返回的列）
FROM table1
JOIN table2 ON condition;
```
 INNER JOIN 与 JOIN 是相同的  
INNER JOIN 关键字在表中存在至少一个匹配时返回行。如果 "Websites" 表中的行在 "access_log" 中没有匹配，则不会列出这些行。  
left join 返回左表所有的行，即使在右表中没有匹配的行，若没有匹配则结果为null
 RIGHT JOIN 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL  
full outer join 返回左表和右表所有的行，即使没有匹配
8.union关键字
 SQL UNION 操作符合并两个或多个 SELECT 语句的结果。 
  UNION 操作符默认会去除重复的记录，如果需要保留所有重复记录，可以使用 UNION ALL 操作符。  
union 语法
```plain
SELECT column1, column2, ...
FROM table1
UNION
SELECT column1, column2, ...
FROM table2;
```
 使用 UNION 时，每个 SELECT 语句必须具有相同数量的列，且对应列的数据类型必须相似。
UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名  
9.select into  
从表中复制表或列
```plain
select * into table2 from table1;
select column1,column2 into table2 from table1;
```
9.insert into select
 INSERT INTO SELECT 语句从一个表复制数据，然后把数据插入到一个已存在的表中。目标表中任何已存在的行都不会受影响。  

```plain
INSERT INTO table2
SELECT * FROM table1;

INSERT INTO table2
(column_name(s))
SELECT column_name(s)
FROM table1;
```
所添加的列的元素存在，其他列的元素为空
10.create database+数据库名 
创建数据库
11.SQL CREATE TABLE 创建数据库表（一个空的数据表，没有数据信息，使用insert into写入信息）
 CREATE TABLE *table_name*(*column_name1 data_type*(*size*),*column_name2 data_type*(*size*),*column_name3 data_type*(*size*),....);
​
