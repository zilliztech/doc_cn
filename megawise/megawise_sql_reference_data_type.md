---
id: "megawise_data_type"
lang: "cn"
title: "数据定义"
---
# 数据类型

<!-- TOC -->

- [数据类型](#数据类型)
    - [数字类型](#数字类型)
        - [整数类型](#整数类型)
        - [浮点类型](#浮点类型)
    - [字符类型](#字符类型)
    - [日期/时间类型](#日期时间类型)
        - [日期/时间输入](#日期时间输入)
        - [日期/时间格式](#日期时间格式)
    - [布尔类型](#布尔类型)

<!-- /TOC -->

## 数字类型

数字类型由 2、4 或 8 字节的整数以及4或8字节的浮点数和可变精度小数组成。下表列出了所有可用类型。

**数字类型**

| 名称               | 存储字节 | 描述               | 范围                                         |
| ------------------ | -------- | ------------------ | -------------------------------------------- |
| `smallint`         | 2 字节    | 小范围整数         | -32768 至 +32767                             |
| `integer`          | 4 字节    | 整数的典型选择     | -2147483648 至 +2147483647                   |
| `bigint`           | 8 字节    | 大范围整数         | -9223372036854775808 至 +9223372036854775807 |
| `decimal`          | 可变     | 用户指定精度，精确 | 最高小数点前131072位，以及小数点后16383位    |
| `numeric`          | 可变     | 用户指定精度，精确 | 最高小数点前131072位，以及小数点后16383位    |
| `real`             | 4 字节    | 可变精度，不精确   | 6位十进制精度                                |
| `double precision` | 8 字节    | 可变精度，不精确   | 15位十进制精度                               |

### 整数类型

SQL只声明了整数类型 `integer` （或 `int` ）、`smallint` 和 `bigint` 。类型 `int2` 、 `int4` 和 `int8` 都是扩展，也在许多其它 SQL 数据库系统中使用。

### 浮点类型

数据类型 `real` 和 `double precision` 是不准确的、可变精度的数字类型。这些类型是IEEE标准 754 二进制浮点算术（分别对应单精度和双精度）的一般实现。

- 如果你要求准确的存储和计算（例如计算货币金额），应使用 `numeric` 类型。

- 如果你要做重要的复杂计算，尤其是那些你对范围情况（无穷、下溢）严重依赖的事情，那你应该仔细评诂你的实现。

- 用两个浮点数值进行等值比较可能得到非预期结果。


除了普通的数字值之外，浮点类型还有几个特殊值：

```sql
`Infinity`
`-Infinity`
`NaN
```

这些值分别表示 IEEE 754 特殊值“正无穷大”、“负无穷大”以及“不是一个数字”（在不遵循 IEEE 754 浮点算术的机器上，这些值的含义可能不是预期的）。如果在 SQL 命令里把这些数值当作常量写，你必须在它们周围放上单引号。 在输入时，这些字符串是以大小写无关的方式识别的。

------

**注意**
IEEE 754 指定 `NaN` 不与任何其他浮点值（包括 `NaN` ）相等。

------


## 字符类型

下表显示了在 MegaWise 里可用的字符类型。

**字符类型**

|                 名称                 |      描述      |
| :----------------------------------: | :------------: |
| `character varying(n)`, `varchar(n)` |  有限制的变长  |
|      `character(n)`, `char(n)`       | 定长，空格填充 |
|                `text`                |    无限变长    |

SQL 定义了两种基本的字符类型： `character varying(n)` 和 `character(n)`， 其中 *n* 是一个正整数。两种类型都可以存储最多 *n* 个字符长的串。如果要存储的串比声明的长度短，类型为 `character` 的值将会用空白填满；而类型为 `character varying` 的值将只是存储原串。

如果我们明确地把一个值造型成 `character varying(n)` 或者 `character(n)` ， 那么超过长度的部分将被抛弃，而不会抛出错误（该行为遵循 SQL 标准）。

如果不带长度说明词使用 `character varying` ，那么该类型接受任何长度的串。后者是一个 MegaWise 的扩展。另外，MegaWise 提供 `text` 类型，也可以存储任何长度的串。

```sql
CREATE TABLE test1 (a character(4));
INSERT INTO test1 VALUES ('ok');
SELECT a, FROM test1; -- (1)
  a   
------
 ok   

CREATE TABLE test2 (b varchar(5));
INSERT INTO test2 VALUES ('ok');
INSERT INTO test2 VALUES ('good      ');
INSERT INTO test2 VALUES ('too long');
ERROR:  value too long for type character varying(5)
INSERT INTO test2 VALUES ('too long'::varchar(5)); -- explicit truncation
SELECT b, FROM test2;
   b   
-------
 ok    
 good  
 too l 
```

在 MegaWise 里另外还有两种定长字符类型。 name 类型长度为 64 字节（63 可用字符加结束符）。类型 "char"（注意引号）只占用一个字节的存储空间。

**特殊字符类型**

| 名称     | 存储字节 | 描述                 |
| -------- | -------- | -------------------- |
| `"char"` | 1 字节    | 单字节内部类型       |
| `name`   | 64 字节   | 用于对象名的内部类型 |

## 日期/时间类型

MegaWise 支持 SQL 中所有的日期和时间类型，如下表所示。日期根据公历来计算。

**日期/时间类型**

| 名称                                      | 存储字节 | 描述                     | 最小值       | 最大值      | 解析度       |
| ----------------------------------------- | -------- | ------------------------ | ------------ | ----------- | ------------ |
| `timestamp [ without time zone ]` | 8 字节    | 包括日期和时间（无时区） | 4713 BC      | 294276 AD   | 1 微秒|
| `date`                                    | 4 字节    | 日期（没有一天中的时间） | 4713 BC      | 5874897 AD  | 1 日          |
| `time [ without time zone ]`      | 8 字节    | 一天中的时间（无日期）   | 00:00:00     | 24:00:00    | 1 微秒|
| `interval [ fields ]`             | 16 字节   | 时间间隔                 | -178000000年 | 178000000年 | 1 微秒|

`interval`类型有一个附加选项，它可以通过写下面之一的短语来限制存储的fields的集合：

```
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```

### 日期/时间输入

日期和时间的输入可以接受几乎任何合理的格式，包括 ISO 8601、SQL 兼容的、传统 POSTGRES 的和其他的形式。 对于一些格式，日期输入里的日、月和年的顺序会让人混淆， 并且支持指定所预期的这些域的顺序。把 DateStyle 参数设置为 MDY ，就是选择“月－日－年”的解释，设置为 DMY 就是 “日－月－年”，而 YMD 是 “年－月－日”。

ＭegaWise 在处理日期/时间输入上比 SQL 标准要求的更灵活。

请记住任何日期或者时间的文字输入需要由单引号包围，就象一个文本字符串一样。SQL 要求下面的语法

```sql
type 'value'
```

### 日期/时间格式

时间/日期类型的格式可以设成四种风格之一： ISO 8601、SQL（Ingres）、传统的 POSTGRES（Unix的date格式）或 German 。默认是 ISO 格式（ISO 标准要求使用 ISO 8601 格式。ISO 输出格式的名字是历史偶然）。下表显示了每种风格的例子。date 和 time 类型通常只有日期或时间部分和例子中一致。不过，POSTGRES 风格输出的是 ISO 格式的只有日期的值。

**日期/时间输出风格**

| 风格声明   | 描述              | 示例                           |
| ---------- | ----------------- | ------------------------------ |
| `ISO`      | ISO 8601, SQL标准 | `1997-12-17 07:37:16-08`       |
| `SQL`      | 传统风格          | `12/17/1997 07:37:16.00 PST`   |
| `Postgres` | 原始风格          | `Wed Dec 17 07:37:16 1997 PST` |
| `German`   | 地区风格          | `17.12.1997 07:37:16.00 PST`   |

------

**注意**
ISO 8601指定使用大写字母 `T` 来分隔日期和时间。MegaWise 在输入上接受这种格式，但是在输出时它采用一个空格而不是 `T`，如上所示。和一些其他数据库系统一样，这是为了可读性以及与 RFC 3339 的一致性。

------

SQL 和 POSTGRES 风格中，如果 DMY 域顺序被指定，“日”将出现在“月”之前，否则“月”出现在“日”之前。下表给出了例子。

**日期顺序习惯**

| `datestyle`设置 | 输入顺序       | 示例输出                       |
| --------------- | -------------- | ------------------------------ |
| `SQL, DMY`      | *日*/*月*/*年* | `17/12/1997 15:37:16.00 CET`   |
| `SQL, MDY`      | *月*/*日*/*年* | `12/17/1997 07:37:16.00 PST`   |
| `Postgres, DMY` | *日*/*月*/*年* | `Wed 17 Dec 07:37:16 1997 PST` |

日期/时间风格可以由用户使用 `SET datestyle` 命令选取，在 `megawise_config.yaml` 配置文件里的参数 `result_config.date_order` 设置或者在 `PGDATESTYLE` 环境变量里设置。

## 布尔类型

MegaWise 提供标准的 SQL 类型 boolean，参见下表。boolean 可以有多个状态：“true（真）”、“false（假）”和第三种状态“unknown（未知）”，未知状态由 SQL 空值表示。

**布尔数据类型**

| 名称      | 存储字节 | 描述         |
| --------- | -------- | ------------ |
| `boolean` | 1字节    | 状态为真或假 |

“真”状态的有效文字值是：

- `TRUE`
- `true`
- `yes`
- `on`
- `1`



“假”状态的有效文字值是：

- `FALSE`
- `false`
- `no`
- `off`
- `0`


前导或者末尾的空白将被忽略，并且大小写也无关紧要。建议使用 `TRUE` 和 `FALSE`（SQL 兼容）。

以下代码展示了使用字母 t 和 f 输出 boolean 值的例子。

```sql
CREATE TABLE test1 (a boolean, b text);
INSERT INTO test1 VALUES (TRUE, 'sic est');
INSERT INTO test1 VALUES (FALSE, 'non est');
SELECT * FROM test1;
 a |    b
---+---------
 t | sic est
 f | non est

SELECT * FROM test1 WHERE a;
 a |    b
---+---------
 t | sic est
```