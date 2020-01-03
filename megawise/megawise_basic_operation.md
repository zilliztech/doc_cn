---
id: "megawise_basic_operation"
lang: "cn"
title: "基本数据操作"
label1: "用户手册"
label2: "MegaWise"
---

# 基本数据操作


## 约定

我们对命令格式使用以下约定：方括弧（[和]）表示可选的部分（在 Tcl 命令里，使用的是问号 （?），就像通常的 Tcl 一样）。 花括弧（{和}）和竖线（|）表示你必须选取一个候选。 点（...）表示它前面的元素可以被重复。

如果能提高清晰度，那么 SQL 命令前面会放上提示符=>， 而 shell 命令前面会放上提示符 $。不过，提示符通常不被显示。


## 基本操作


### 创建一个数据库

MegaWise 系统可以管理多个数据库。 通常我们会为每个项目和每个用户单独创建一个数据库。数据库名必须是以字母开头并且长度小于 63 个字符。

通过下面的命令创建一个自定义名称的数据库，本例中数据库名为`mydb`：

```bash
$ create database mydb
```

如果不产生任何响应则表示该步骤成功。


如果在创建数据库时得到下面的信息：

```bash
ERROR:  permission denied to create database
```
说明当前用户没有创建数据库的权限，请登录管理员账号并为该用户账号进行提权操作。


### 切换数据库
通过下面的命令切换到一个已经创建的数据库，本例中数据库名为`mydb`：

```bash
$ \c mydb
```

### 删除数据库
你可以用下面的命令删除一个数据库：

```bash
$ drop database mydb
```

(对于这条命令而言，数据库名不是默认的用户名，因此你就必须声明它） 。这个动作将在物理上把所有与该数据库相关的文件都删除并且不可取消，因此在删除之前一定要考虑清楚。


### 访问数据库

连接数据库后，在 `psql` 中，你将看到下面的欢迎信息：

```bash
psql (11.1)
Type "help" for help.

mydb=>
```

最后一行也可能是：

```bash
mydb=#
```

这个提示符意味着你是数据库超级用户。 超级用户在操作数据库时不受访问控制的限制。

`psql` 打印出的最后一行是提示符，表示 `psql` 正在监听当前会话，在提示符后可以敲入 SQL 语句进行查询操作。尝试下面的命令：

```bash
mydb=> SELECT version();
                                         version
------------------------------------------------------------------------------------------
 MegaWise 11.1 on x86_64-pc-linux-gnu, compiled by gcc （Ubuntu 5.4.0-6ubuntu1~16.04.10）5.4.0 20160609 4.9.2, 64-bit
(1 row)

mydb=> SELECT current_date;
    date
------------
 2019-09-15
(1 row)

mydb=> SELECT 2 + 2;
 ?column?
----------
        4
(1 row)
```

`psql`支持一些内部命令。内部命令以反斜线开头，“`\`”。 欢迎信息中列出了部分内部命令。你可以用下面的命令获取 MegaWise 的 SQL 命令的帮助语法：

```bash
mydb=> \h
```

要退出`psql`，输入：

```bash
mydb=> \q
```


### 创建一个新表
通过指定表名、列名及列的数据类型来创建表∶

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- 最低温度
    temp_hi         int,           -- 最高温度
    prcp            real,          -- 湿度
    date            date
);
```
> 注意：psql 将分号作为一条命令的结束符。

通过两个连续划线 `--` 引入注释。 任何跟在它后面直到行尾的东西都会被忽略。SQL 对关键字和标识符大小写不敏感，只有在标识符被双引号包围时才能保留大小写。

varchar(80)指定了一个可以存储最长 80 个字符的字符串数据类型。int 指整数，real 指单精度浮点数，date 指日期类型。

MegaWise 支持标准的 SQL 数值类型 bool、int、smallint、bigint、float、real、double precision、decimal、numericchar(N)、varchar(N)、text、name、date、time、timestamp、interval，还支持其他的通用功能的类型。



第二个例子将保存城市和它们相关的地理位置：
```sql
CREATE TABLE cities (
   name            varchar(80),
   area            real
);
```




### 删除表

可以用下面的命令删除一个表：

```sql
DROP TABLE tablename;
```


### 在表中增加行
使用 `INSERT` 语句向表中添加行：

```sql
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');　　　　　　　　　　　　　
```

```sql
INSERT INTO cities VALUES ('San Francisco', 88.65);　　　　　　
```

请注意各数值的顺序需要与创建表时 `CREATE TABLE` 命令中各列的顺序保持一致，且数值类型以外的常量值通常必须用单引号 `'` 包围，如上面的例子所示。

另一种添加行的方法要求明确指定出列名，该方法要求插入数据与列名一一对应，各列的顺序无需与创建表时 `CREATE TABLE` 命令中各列的顺序一致：

```sql
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

还可使用 `COPY` 命令从文本文件中批量导入数据，如：

```sql
COPY weather FROM '/home/user/weather.txt';
```
>注意：该命令要求数据文件位于 MegaWise 所在服务器上， 而不是客户端所在的服务器上。


### 查询一个表

你可以使用 `SELECT` 语句对数据库的各表的数据进行查询。 该语句分为选择列表（需要返回的列）、表列表（查询相关的所有表）以及对输入或结果数据的限制条件。比如，通过以下命令查询表 weather 中的所有数据：
```
SELECT * FROM weather;
```

此处*代表“所有列”，因此上述命令等价于：

```sql
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

该命令输出类似如下格式：

```sql
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 11-27-1994
 San Francisco |      43 |      57 |    0 | 11-29-1994
 Hayward       |      37 |      54 |    0 | 11-29-1994
(3 rows)

```
你可以在选择列表中写任意表达式，而不仅仅是列的列表。比如，你可以执行以下命令：

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

得到以下结果：

```sql
     city      | temp_avg |    date    
---------------+----------+------------
 San Francisco |       48 | 11-27-1994
 San Francisco |       50 | 11-29-1994
 Hayward       |       45 | 11-29-1994
(3 rows)

```
命令中的 `AS` 子句是用于对输出列进行重命名。

一个查询可以使用 `WHERE` 子句对输入或者结果数据进行过滤。`WHERE` 子句包含一个布尔表达式，只有那些使布尔表达式为真的行才会被返回。在条件中可以使用常用的布尔操作符`AND`、`OR` 和 `NOT` 。 比如：
```sql
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
```
结果：
```sql
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
(1 row)
```
使用 `ORDER BY` 关键字可对查询结果进行排序：
```sql
SELECT * FROM weather
    ORDER BY city;
```
```sql

     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 Hayward       |      37 |      54 |    0 | 11-29-1994
 San Francisco |      46 |      50 | 0.25 | 11-27-1994
 San Francisco |      43 |      57 |    0 | 11-29-1994
(3 rows)

```

以上命令根据 “city” 列对结果进行排序。使用类似的方法可以根据多个列的值对结果进行排序：

```sql
SELECT * FROM weather
    ORDER BY city, temp_lo;
```

使用 `DISTINCT` 关键字可消除结果中的重复数据：

```sql
SELECT DISTINCT city
    FROM weather;
```

```sql
     city      
---------------
 Hayward
 San Francisco
(2 rows)
```

一条命令中可以同时使用关键字 `DISTINCT` 和 `ORDER BY` ：

```sql
SELECT DISTINCT city
    FROM weather
    ORDER BY city;
```



### 表的连接查询

在 SQL 中查询可以一次访问多个表，该类查询被称为连接查询。举例来说，如需列出所有天气记录以及相关的城市位置，需要拿 weather 表每行的 city 列和 cities 表所有行的 name 列进行比较，并选取出那些值相匹配的行对进行组合。该查询可使用以下命令实现：


```sql
SELECT *
    FROM weather, cities
    WHERE city = name;
```

```sql
     city      | temp_lo | temp_hi | prcp |    date    |     name      | area  
---------------+---------+---------+------+------------+---------------+-------
 San Francisco |      46 |      50 | 0.25 | 11-27-1994 | San Francisco | 88.65
 San Francisco |      43 |      57 |    0 | 11-29-1994 | San Francisco | 88.65
(2 rows)

```

返回结果中有两个列均包含了城市名字。因为 `weather` 和 `cities` 表的所有列被组合在了一起。可在命令中明确列出输出列而不使用 `*` 来设置结果中出现的列以及其出现顺序：

```sql
SELECT city, temp_lo, temp_hi, prcp, date, area
    FROM weather, cities
    WHERE city = name;
```

如果在两个表里有重名的列，需要限定列名来说明该列来自的表名，如：

```sql
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.area
    FROM weather, cities
    WHERE cities.name = weather.city;
```
上述连接查询也可以用下面这样的形式写出来：

```sql
SELECT *
    FROM weather INNER JOIN cities ON (weather.city = cities.name);
```
通过以下命令可以执行外连接操作，该查询扫描 weather 表， 并且对每一行都找出匹配的 cities 表行。如果没有找到匹配的行，则使用空值代替 cities 表中的相应列。


```sql
SELECT *
    FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);
```
```sql
     city      | temp_lo | temp_hi | prcp |    date    |     name      |    area     
---------------+---------+---------+------+------------+---------------+-------------
 San Francisco |      46 |      50 | 0.25 | 11-27-1994 | San Francisco |       88.65
 San Francisco |      43 |      57 |    0 | 11-29-1994 | San Francisco |       88.65
 Hayward       |      37 |      54 |    0 | 11-29-1994 |               | 
(3 rows)
```

上述查询是一个左外连接，连接操作符左部的表中的行在输出中至少出现一次，而在右部的表的行只有在能找到匹配的左部表的时候才能出现在输出中。如果输出的左部表中的行没有对应匹配的右部表中的行，那么右部表内的相应列将填充空值（null）。

自连接查询把一个表和自己连接起来,比如：

```sql
SELECT W1.city, W1.temp_lo AS low, W1.temp_hi AS high,
    W2.city, W2.temp_lo AS low, W2.temp_hi AS high
    FROM weather W1, weather W2
    WHERE W1.temp_lo < W2.temp_lo and w1.temp_hi < w2.temp_hi and w1.prcp = w2.prcp;
```

```sql
  city   | low | high |     city      | low | high 
---------+-----+------+---------------+-----+------
 Hayward |  37 |   54 | San Francisco |  43 |   57
(1 row)
```

上述命令把 weather 表重新标记为 W1 和 W2 以区分连接的左部和右部。

此外，可以用别名在连接查询中指代表名，比如：

```sql
SELECT *
    FROM weather w, cities c
    WHERE w.city = c.name;
```


*******

**注意**

在所有表连接操作中， `WHERE` 子句中必须含有 “＝” 的连接条件。

****




### 聚集函数

一个聚集函数从多个输入行中计算出一个结果。比如，在一个行集合上计算 `count`（计数）、`sum`（和）、`avg`（均值）、`max`（最大值）和 `min`（最小值）。

通过以下语句可查询所有记录中最低温度的最大值：

```sql
SELECT max(temp_lo) FROM weather;
```
```sql
 max 
-----
  46
(1 row)
```

> 注意：聚集函数不能用于 `WHERE` 子句中。例如，以下查询出现最低温度最大值的城市的命令不符合 SQL 语法规范。

```sql
SELECT city FROM weather WHERE temp_lo = max(temp_lo);   
```
可以使用子查询实现上述查询：

```sql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
```

```sql
      city       
-----------------
 San Francisco
 (1 row)
```

子查询可以被视为一次独立的数据查询操作。

聚集函数经常会和 `GROUP BY` 子句组合使用。比如，通过以下命令查询每个城市观测到的最低温度的最大值：

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;
```

```sql
     city      | max 
---------------+-----
 Hayward       |  37
 San Francisco |  46
(2 rows)

```

可以用 `HAVING` 过滤对结果进行过滤：

```sql
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```
```sql
  city   | max 
---------+-----
 Hayward |  37
(1 row)
```
该命令只返回temp_lo的最大值低于40的城市。

理解聚集函数和 SQL 的 `WHERE` 以及 `HAVING` 子句之间的关系非常重要。`WHERE` 和 `HAVING` 的基本区别如下：`WHERE` 在分组和聚集计算之前对输入行进行过滤， 而 `HAVING` 在分组和聚集之后对结果进行过滤。因此，`WHERE` 子句不能包含聚集函数； 而 `HAVING` 子句通常总是包含聚集函数（严格说来，可以写不含聚集函数的 `HAVING` 子句， 但该过滤操作在 `WHERE` 子句中使用时查询执行将更加高效）。

在前面的例子里，在 `WHERE` 子句中对城市名称过滤的效率更高，因为可以避免那些不符合条件的行参与到分组和聚集计算中。
