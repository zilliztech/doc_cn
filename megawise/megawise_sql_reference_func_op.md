---
id: "megawise_func_op"
lang: "cn"
title: "函数和操作符"
---
# 函数和操作符

<!-- TOC -->

- [逻辑操作符](#逻辑操作符)
- [比较函数和操作符](#比较函数和操作符)
- [数学函数和操作符](#数学函数和操作符)
- [字符串函数和操作符](#字符串函数和操作符)
- [模式匹配](#模式匹配)
- [日期/时间函数和操作符](#日期时间函数和操作符)
- [条件表达式](#条件表达式)
- [聚集函数](#聚集函数)
- [子查询表达式](#子查询表达式)
    - [EXISTS 子查询](#EXISTS-子查询)
    - [IN 子查询](#IN-子查询)
    - [NOT IN 子查询](#NOT-IN-子查询)
- [系统管理函数](#系统管理函数)

<!-- /TOC -->

## 逻辑操作符

常用的逻辑操作符有 `AND`, `OR`, 和 `NOT`. SQL 的逻辑系统，包括真、假和 `null` ，`null`（表示“未知”）三种值。其逻辑运算规则见如下的真值表：

| *a*   | *b*   | *a* AND *b* | *a* OR *b* |
| ----- | ----- | ----------- | ---------- |
| TRUE  | TRUE  | TRUE        | TRUE       |
| TRUE  | FALSE | FALSE       | TRUE       |
| TRUE  | NULL  | NULL        | TRUE       |
| FALSE | FALSE | FALSE       | FALSE      |
| FALSE | NULL  | FALSE       | NULL       |
| NULL  | NULL  | NULL        | NULL       |



| *a*   | NOT *a* |
| ----- | ------- |
| TRUE  | FALSE   |
| FALSE | TRUE    |
| NULL  | NULL    |

操作符 `AND` 和 `OR` 是可交换的，也就是说，可以交换左右操作数而不影响结果。

## 比较函数和操作符

常见的比较操作符如下表所示。

**比较操作符**

| 操作符       | 描述     |
| ------------ | -------- |
| `<`          | 小于     |
| `>`          | 大于     |
| `<=`         | 小于等于 |
| `>=`         | 大于等于 |
| `=`          | 等于     |
| `<>` or `!=` | 不等于   |

所有比较操作符都是二元操作符，它们返回 `boolean` 类型；类似于 `1 < 2 < 3` 的表达式是非法的（因为 `<` 操作符不能比较一个布尔值和 `3` ）。

如下表所示，MegaWise 也支持使用比较谓词。 比较谓词的用法和操作符类似，但是具有 SQL 标准所要求的特殊语法。


| 谓词                                      | 描述                           |
| ----------------------------------------- | ------------------------------ |
| *a* `BETWEEN` *x* `AND` *y*               | 在x和y之间                     |
| *a* `NOT BETWEEN` *x* `AND` *y*           | 不在x和y之间                   |
| *a* `BETWEEN SYMMETRIC` *x* `AND` *y*     | 在对比较值排序后位于x和y之间   |
| *a* `NOT BETWEEN SYMMETRIC` *x* `AND` *y* | 在对比较值排序后不位于x和y之间 |
| *expression* `IS NULL`                    | 是空值                         |
| *expression* `IS NOT NULL`                | 不是空值                       |
| *expression* `ISNULL`                     | 是空值（非标准语法）           |
| *expression* `NOTNULL`                    | 不是空值（非标准语法）         |


```sql
a BETWEEN x AND y
```

等效于

```sql
a >= x AND a <= y
```

注意 `BETWEEN` 认为区间端点值是包含在范围内的。 `NOT BETWEEN` 可以做相反比较：

```sql
a NOT BETWEEN x AND y
```

等效于

```sql
a < x OR a > y
```

`BETWEEN SYMMETRIC` 和 `BETWEEN` 相似，不过 `BETWEEN SYMMETRIC` 不要求 `AND` 左边的参数小于或等于右边的参数。如果左参数不是小于等于右参数，这两个参数会自动被交换，这样总是会应用一个非空范围。

当有一个输入为空时，普通的比较操作符会得到空（表示“未知”）， 而不是真或假。例如，`7 = NULL`得到空，`7 <> NULL` 也一样。

可使用下面的方法检查一个值是否为空：

```sql
expression IS NULL
expression IS NOT NULL
```

或者等效，但并不标准的方法：

```sql
expression ISNULL
expression NOTNULL
```

避免使用 `expression = NULL`，因为 `NULL` 是不等于 `NULL` 的（空值代表一个未知的值，无法判断两个未知的数值是否相等）。


## 数学函数和操作符

MegaWise 为多种数值类型数据提供了数学操作符。下表展示了所有可用的数学操作符。


| 操作符 | 描述                   | 示例        | 结果  |
| ------ | ---------------------- | ----------- | ----- |
| `+`    | 加                     | `2 + 3`     | `5`   |
| `-`    | 减                     | `2 - 3`     | `-1`  |
| `*`    | 乘                     | `2 * 3`     | `6`   |
| `/`    | 除（整数除法截断结果） | `4 / 2`     | `2`   |
| `%`    | 模（取余）             | `5 % 4`     | `1`   |

下表显示了可用的数学函数。在该表中，dp 表示 double precision 。除非特别指明，函数都返回和它的参数相同的数据类型。


| 函数                          | 返回类型           | 描述                                 | 示例              | 结果               |
| ----------------------------- | ------------------ | ------------------------------------ | ----------------- | ------------------ |
| `abs(x)`                      | （和输入相同）     | 绝对值                               | `abs(-17.4)`      | `17.4`             |
| `cbrt(dp)`                    | `dp`               | 立方根                               | `cbrt(27.0)`      | `3`                |
| `ceil(dp or numeric)`         | （和输入相同）     | 不小于参数的最小整数                 | `ceil(-42.8)`     | `-42`              |
| `ceiling(dp or numeric)`      | （和输入相同）     | 不小于参数的最小整数（`ceil`的别名） | `ceiling(-95.3)`  | `-95`              |
| `log(dp or numeric)`          | （和输入相同）     | 以10为底的对数                       | `log(100.0)`      | `2`                |
| `mod(y, x)`                   | （和参数类型相同） | *y*/*x*的余数                        | `mod(9,4)`        | `1`                |
| `pi()`                        | `dp`               | “π”常数                              | `pi()`            | `3.14159265358979` |
| `power(a dp, b dp)`           | `dp`               | 求*a*的*b*次幂                       | `power(9.0, 3.0)` | `729`              |
| `power(a numeric, b numeric)` | `numeric`          | 求*a*的*b*次幂                       | `power(9.0, 3.0)` | `729`              |
| `round(dp or numeric)`        | （和输入相同）     | 圆整为最接近的整数                   | `round(42.4)`     | `42`               |
| `sqrt(dp or numeric)`         | （和输入相同）     | 平方根                               | `sqrt(2.0)`       | `1.4142135623731`  |

下表显示了三角函数：

| 函数 (弧度) |  描述  |
| :---------: | :----: |
|  `acos(x)`  | 反余弦 |
|  `asin(x)`  | 反正弦 |
|  `atan(x)`  | 反正切 |
|  `cos(x)`   |  余弦  |
|  `cot(x)`   |  余切  |
|  `sin(x)`   |  正弦  |
|  `tan(x)`   |  正切  |

## 字符串函数和操作符

MegaWise 支持的字符串函数和操作符如下表所示。


| 函数                                     | 返回类型 | 描述     | 示例                             | 结果  |
| ---------------------------------------- | -------- | -------- | -------------------------------- | ----- |
| `substring(string, int)`                 | text     | 提取子串 | substring('Thomas', 2)           | homas |
| `substring(string, int, int)`            | text     | 提取子串 | substring('Thomas', 2, 3)        | hom   |
| `substr(string, int)`                    | text     | 提取子串 | substr('Thomas', 2)              | homas |
| `substr(string, int, int)`               | text     | 提取子串 | substr('Thomas', 2, 3)           | hom   |
| `substring(string [from int] [for int])` | text     | 提取子串 | substring('Thomas' from 2 for 3) | hom   |

## 模式匹配

MegaWise 提供了 `LIKE` 操作符实现模式匹配。

```sql
string LIKE pattern [ESCAPE escape-character]
string NOT LIKE pattern [ESCAPE escape-character]
```

如果该 *string* 匹配了提供的 *pattern* ，那么 `LIKE` 表达式返回真（如果 `LIKE` 返回真，那么 `NOT LIKE` 表达式返回假， 反之亦然。一个等效的表达式是 `NOT (string LIKE pattern)`）。

如果 *pattern* 不包含百分号或者下划线，那么 `LIKE` 的行为和等号操作符相同，该模式精确匹配*pattern*对应的字符串；*pattern* 中的下划线 `_` 可匹配任何单个字符； 而百分号 `%` 匹配零或更多个字符组成的任何序列。

以下为一些模式匹配的例子：

```sql
'abc' LIKE 'abc'    true
'abc' LIKE 'a%'     true
'abc' LIKE '_b_'    true
'abc' LIKE 'c'      false
```

如果需要匹配文本中的下划线或者百分号，必须在 *pattern* 里相应的字符前添加转义字符。默认的转义字符是反斜线，也可用 `ESCAPE` 子句指定转义字符。 如要匹配转义字符本身，则需在 *pattern* 中相应位置写连续两个转义字符。

>注意：反斜线在字符串类型数据中已经有特殊含义了，所以如果要写一个包含反斜线的模式常量，则需要在 *pattern* 中匹配两个反斜线。 由于 *pattern* 中也使用反斜线作为转义字符，所以写一个匹配单个反斜线的模式实际上要在 *pattern* 里写四个反斜线。 可以通过用 ESCAPE 选择一个不同的转义字符来避免此类操作。这时反斜线不再是 LIKE 的特殊字符，但仍然是字符数据中的特殊字符，所以你还是需要两个反斜线。 此外，可以通过写 `ESCAPE ''` 的方式不选择转义字符，从而禁用转义机制。

## 日期/时间函数和操作符

下表展示了时间类型数据的基本算术操作符 （+、*等）的行为。


| 操作符 | 示例                                                         | 结果                              |
| ------ | ------------------------------------------------------------ | --------------------------------- |
| `+`    | `date '2001-09-28' + integer '7'`                            | `date '2001-10-05'`               |
| `+`    | `date '2001-09-28' + interval '1 hour'`                      | `timestamp '2001-09-28 01:00:00'` |
| `+`    | `date '2001-09-28' + time '03:00'`                           | `timestamp '2001-09-28 03:00:00'` |
| `+`    | `interval '1 day' + interval '1 hour'`                       | `interval '1 day 01:00:00'`       |
| `+`    | `timestamp '2001-09-28 01:00' + interval '23 hours'`         | `timestamp '2001-09-29 00:00:00'` |
| `+`    | `time '01:00' + interval '3 hours'`                          | `time '04:00:00'`                 |
| `-`    | `- interval '23 hours'`                                      | `interval '-23:00:00'`            |
| `-`    | `date '2001-10-01' - date '2001-09-28'`                      | `integer '3'` (days)              |
| `-`    | `date '2001-10-01' - integer '7'`                            | `date '2001-09-24'`               |
| `-`    | `date '2001-09-28' - interval '1 hour'`                      | `timestamp '2001-09-27 23:00:00'` |
| `-`    | `time '05:00' - time '03:00'`                                | `interval '02:00:00'`             |
| `-`    | `time '05:00' - interval '2 hours'`                          | `time '03:00:00'`                 |
| `-`    | `timestamp '2001-09-28 23:00' - interval '23 hours'`         | `timestamp '2001-09-28 00:00:00'` |
| `-`    | `interval '1 day' - interval '1 hour'`                       | `interval '1 day -01:00:00'`      |
| `-`    | `timestamp '2001-09-29 03:00' - timestamp '2001-09-27 12:00'` | `interval '1 day 15:00:00'`       |
| `*`    | `900 * interval '1 second'`                                  | `interval '00:15:00'`             |
| `*`    | `21 * interval '1 day'`                                      | `interval '21 days'`              |
| `*`    | `double precision '3.5' * interval '1 hour'`                 | `interval '03:30:00'`             |
| `/`    | `interval '1 hour' / double precision '1.5'`                 | `interval '00:40:00'`             |

下表展示了可用于处理日期/时间值的函数。


| 函数                            | 返回类型         | 描述       | 示例                                               | 结果 |
| ------------------------------- | ---------------- | ---------- | -------------------------------------------------- | ---- |
| `extract`(field from source) | double precision | 获得子域 | extract(hour from timestamp '2001-02-16 20:38:40') | 20   |

`extract` 函数从日期/时间值中抽取子域，例如年或者小时等。

*source* 必须是一个类型为 timestamp、time 或 interval 的值表达式（类型为 date 的表达式将被转化为 timestamp ，因此也可以使用）。

*field* 是一个标识符或者字符串，它指定从源值中抽取的域。`extract` 函数返回类型为 double precision 的值。

下列值是有效的域名∶

- hour

    小时域（0 - 23）
    ```sql
    SELECT EXTRACT(HOUR FROM TIMESTAMP '2001-02-16 20:38:40'); 
    ```

    结果：20

- minute

    分钟域（0 - 59）
    ```sql
    SELECT EXTRACT(MINUTE FROM TIMESTAMP '2001-02-16 20:38:40'); 
    ```

    结果：38

- second

    秒域，包括小数部分（0 - 59）

    ```sql
    SELECT EXTRACT(SECOND FROM TIMESTAMP '2001-02-16 20:38:40');
    ```

    结果：40

    ```sql
    SELECT EXTRACT(SECOND FROM TIME '17:12:28.5'); 
    ```

    结果：28.5

- day
 
    对于 timestamp 值，是（月份）里的日域（1-31）；对于 interval 值，是日数

    ```sql
    SELECT EXTRACT(DAY FROM TIMESTAMP '2001-02-16 20:38:40');
    ```

    结果：16

    ```sql
    SELECT EXTRACT(DAY FROM INTERVAL '40 days 1 minute');
    ```

    结果：40

- month

    对于 timestamp 值，它是一年里的月份数（1 - 12）； 对于 interval 值，它是月的数目，然后对 12 取模（0 - 11）

    ```sql
    SELECT EXTRACT(MONTH FROM TIMESTAMP '2001-02-16 20:38:40');
    ```

    结果：2

    ```sql
    SELECT EXTRACT(MONTH FROM INTERVAL '2 years 3 months'); 
    ```

    结果：3

    ```sql
    SELECT EXTRACT(MONTH FROM INTERVAL '2 years 13 months'); 
    ```

    结果：1

- year

    年份域

    ```sql
    SELECT EXTRACT(YEAR FROM TIMESTAMP '2001-02-16 20:38:40'); 
    ```

    结果：2001


## 条件表达式

MegaWise 中的 `CASE` 条件表达式类似于其它编程语言中的 if/else 语句：

```sql
CASE WHEN condition THEN result
     [WHEN ...]
     [ELSE result]
END
```

`CASE` 子句可用于任何表达式可以出现的位置。每一个 *condition* 是一个返回 `boolean` 结果的表达式。如果结果为真，那么 `CASE` 表达式的结果就是该条件所对应的*result*，并且剩下的 `CASE` 表达式不会被处理。如果结果不为真，那么以相同方式依次搜寻其后的 `WHEN` 子句。如果没有 `WHEN` *condition* 为真，那么 `CASE` 表达式的值就是在 `ELSE` 子句中的*result*。如果省略了 `ELSE` 子句而且没有 `WHEN` *condition*为真，则结果为空。

以下为条件表达式的简单示例：

```sql
SELECT * FROM test;

 a
---
 1
 2
 3


SELECT a,
       CASE WHEN a=1 THEN 'one'
            WHEN a=2 THEN 'two'
            ELSE 'other'
       END
    FROM test;

 a | case
---+-------
 1 | one
 2 | two
 3 | other
```

所有 *result* 表达式的数据类型必须可以转换成相同的数值类型。

以下 `CASE` 表达式是上述通用形式的一个变种：

```sql
CASE expression
    WHEN value THEN result
    [WHEN ...]
    [ELSE result]
END
```

*expression* 会被求值，然后依次与各个 `WHEN` 子句中的 *value* 对比，直到找到与其值相等的 *value* 。如果没有找到匹配的 *value* ，则返回在 `ELSE` 子句中的 *result* 或者 NULL 。 

之前的例子可以用改写为：

```sql
SELECT a,
       CASE a WHEN 1 THEN 'one'
              WHEN 2 THEN 'two'
              ELSE 'other'
       END
    FROM test;

 a | case
---+-------
 1 | one
 2 | two
 3 | other
```

## 聚集函数

聚集函数从一个输入值的集合计算一个单一结果。下表展示了 MegaWise 支持的聚集函数。


| 函数                | 参数类型                                                     | 返回类型                                                     | 局部模式 | 描述                             |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- | -------------------------------- |
| `avg(expression)`   | `smallint`、`int`、`bigint`、`real`、`double precision`、`numeric`或`interval` | 对于任何整数类型参数是`numeric`，对于一个浮点参数是`double precision`，否则和参数数据类型相同 | 支持      | 所有输入值的平均值（算术平均）   |
| `count(*)`          |                                                              | `bigint`                                                     | 支持   | 输入的行数                       |
| `count(expression)` | 任意参数                                                          | `bigint`                                                     | 支持     | *expression*值非空的输入行的数目 |
| `max(expression)`   | 任意数字、字符串，日期/时间，网络，或枚举类型或这些类型的数组  | 与参数数据类型相同                                           | 支持     | 所有输入值中*expression*的最大值 |
| `min(expression)`   | 任意数字、字符串，日期/时间，网络，或枚举类型或这些类型的数组  | 与参数数据类型相同                                           | 支持     | 所有输入值中*expression*的最小值 |
| `sum(expression)`   | `smallint`、`int`、 `bigint`、`real`、`double precision`、`numeric`或`interval` | 对`smallint`或`int`参数是`bigint`，对`bigint`参数是`numeric`，否则和参数数据类型相同 | 支持    | 所有输入值的*expression*的和     |

## 子查询表达式

本节描述 MegaWise 中可用的子查询表达式。所有本节中成文的表达式都返回布尔类型（真/假）的结果。

### EXISTS 子查询

```sql
EXISTS (subquery)
```

`EXISTS` 的参数是一个任意的 `SELECT` 语句， 或者说*子查询*。系统对子查询进行运算以判断它是否返回行。如果它至少返回一行，那么 `EXISTS` 的结果就为 “true” ； 如果子查询没有返回行，那么 `EXISTS` 的结果是 “false” 。

子查询可以引用来自其外部的查询中变量，这些变量在该子查询的任何一次计算中都被作为常量处理。

下面的例子类似在 `col2` 上的一次内联接，但是它为每个 `tab1` 的行生成最多一个输出，即使存在多个匹配 `tab2` 的行也如此∶

```sql
SELECT col1
FROM tab1
WHERE EXISTS (SELECT 1 FROM tab2 WHERE col2 = tab1.col2);
```

### IN 子查询

```sql
expression IN (subquery)
```

圆括弧括起来的子查询必须正好只返回一列数据。表达式的计算结果将与子查询结果逐行进行比较。如果在子查询结果中找到一行的值与表达式值相等，则 `IN` 子查询的结果为 “true” 。子查询结果中没有任何一行的值与表达式值相等或者子查询返回的结果集为空，则结果为 “false”。

如果表达式计算结果为 NULL ，或者子查询结果中没有与表达式相等的值但包含 NULL ，那么`IN`结构的结果将是 NULL ，而不是 "false" 。

### NOT IN 子查询

```sql
expression NOT IN (subquery)
```

圆括弧括起来的子查询必须正好只返回一列数据。表达式的计算结果将与子查询结果逐行进行比较。如果在子查询结果中找不到与表达式相等的值，则 `NOT IN` 子查询的结果为 “true”。 如果找到任何相等行，则结果为 “false”。

如果表达式计算结果为 NULL ，或者子查询结果中没有与表达式相等的值但包含 NULL ，那么 `NOT IN` 结构的结果将是 NULL，而不是 "true" 。

## 系统管理函数

下表展示了可以用于查询以及修改运行时配置参数的函数。


| 名称                                            | 返回类型 | 描述                   |
| ----------------------------------------------- | -------- | ---------------------- |
| `current_setting(setting_name [, missing_ok ])` | `text`   | 获得设置的当前值       |
| `set_config(setting_name, new_value, is_local)` | `text`   | 设置一个参数并返回新值 |

`current_setting` 得到 *setting_name* 设置的当前值：

```sql
SELECT current_setting('datestyle');

 current_setting
-----------------
 ISO, MDY
(1 row)
```

如果没有名为 *setting_name* 的设置， `current_setting` 会抛出错误，除非 *missing_ok* 参数被设置为 `true` 。

`set_config` 将参数 *setting_name* 设置为 *new_value*。如果 *is_local* 设置为 `true` ，那么新值将只应用于当前操作。 如果希望新值应用于当前会话的所有后续操作，那么应该使用 `false`。

```sql
SELECT set_config('log_statement_stats', 'off', false);

 set_config
------------
 off
(1 row)
```