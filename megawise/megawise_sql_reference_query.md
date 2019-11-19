---
id: "megawise_sql_query"
lang: "cn"
title: "查询"
label1: "SQL 语言参考"
label2: "MegaWise"
---

## 查询

<!-- TOC -->

- [查询](#查询)
    - [概述](#概述)
    - [表表达式](#表表达式)
        - [FROM 子句](#from-子句)
            - [连接表](#连接表)
            - [表和列别名](#表和列别名)
            - [子查询](#子查询)
        - [WHERE 子句](#WHERE-子句)
        - [GROUP BY 和 HAVING 子句](#GROUP-BY-和-HAVING-子句)
    - [选择列表](#选择列表)
        - [选择列表项](#选择列表项)
        - [列标签](#列标签)
        - [使用 DISTINCT 删除结果的重复行](#使用-DISTINCT-删除结果的重复行)
    - [行排序](#行排序)
    - [使用 LIMIT 和 OFFSET 查询部分结果](#使用-LIMIT-和-OFFSET-查询部分结果)

<!-- /TOC -->

### 概述

从数据库中检索数据的过程或命令叫做查询。在 SQL 里 SELECT 命令用于指定查询。 SELECT 命令的一般语法是

```sql
[WITH with_queries] SELECT select_list FROM table_expression [sort_specification]
```

一个简单类型的查询的形式：

```sql
SELECT * FROM table1;
```

假设有一个表叫做 `table1` ，这条命令检索 `table1` 中所有行。 `*`意味着选择列表包含表内所有的列。 一个选择列表也可以是表内部分列或列的表达式。例如，如果 `table1` 有叫做 `a` 、`b` 、`c` 和 `d` 的列，那么你可以用下面的查询：

```sql
SELECT a, b + c FROM table1;
```

（假设 `b` 和 `c` 都是数字数据类型）。 

`FROM table1` 是一种非常简单的表表达式：它只读取了一个表。通常，表表达式可以是基本表、连接和子查询组成的复杂结构。 但你也可以省略表表达式而把 `SELECT` 命令当做一个计算器：

```sql
SELECT 3 * 4;
```

也可以用这种方法调用函数：

```sql
SELECT random();
```

### 表表达式

 表表达式计算一个表。该表表达式包含一个 `FROM` 子句，该子句后面可以根据需要选用 `WHERE` 、`GROUP BY` 和 `HAVING` 子句。最简单的表表达式只是引用一个基本表，但是我们可以用更复杂的表表达式以多种方法修改或组合基本表。

表表达式里可选的 `WHERE`、`GROUP BY` 和 `HAVING` 子句指定一系列对源自 `FROM` 子句的表的转换操作。所有这些转换最后生成一个虚拟表。

#### FROM 子句

`FROM` 子句通常包含一个表引用列表，该列表可以包含多张表：

```sql
FROM table_reference [, table_reference [, ...]]
```

表引用可以是一个表名字或者是一个生成的表， 例如子查询、一个 `JOIN` 结构或者这些东西的复杂组合。`FROM` 列表的结果是一个中间的虚拟表，该表可以进行由 `WHERE` 、`GROUP BY` 和 `HAVING` 子句指定的转换，并最后生成全局的表表达式结果。

##### 连接表

一个连接表是根据特定的连接规则从两个其它表中派生的表。目前支持内连接、外连接和交叉连接。一个连接表的一般语法是：

```sql
T1 join_type T2 [ join_condition ]
```

所有类型的连接都可以被链在一起或者嵌套：*T1* 和 *T2* 都可以是连接表。可以使用圆括号来控制 `JOIN` 子句的连接顺序。如果不使用圆括号，`JOIN` 子句会从左至右嵌套。

**连接类型**


```sql
T1 { [INNER] | {LEFT} [OUTER] } JOIN T2 ON boolean_expression
T1 { [INNER] | {LEFT} [OUTER] } JOIN T2 USING ( join column list )
T1 NATURAL { [INNER] | {LEFT} [OUTER] } JOIN T2
```

`INNER` 对所有连接形式都是可选的。`INNER` 是默认值；`LEFT` 指示一个外连接。

连接条件在 `ON` 或 `USING` 子句中指定， 或者用关键字 `NATURAL` 隐含地指定。连接条件决定来自两个源表中的哪些行是匹配的。

可能的条件连接类型是：

`INNER JOIN`

​             对于 T1 的每一行 R1，生成的连接表都有一行对应 T2 中的每一个满足和 R1 的连接条件的行。

`LEFT OUTER JOIN`

​            首先，执行一次内连接。然后，为 T1 中每一个无法在连接条件上匹配 T2 里任何一行的行返回一个连接行，该连接行中 T2 的列用空值补齐。因此，生成的连接表里为来自 T1 的每一行都至少包含一行。

`ON` 子句是最常见的连接条件的形式：它接收一个和 `WHERE` 子句里用的一样的布尔值表达式。 如果两个分别来自* T1* 和 *T2* 的行在 `ON` 表达式上运算的结果为真，那么它们就算是匹配的行。

`USING` 是个缩写符号，它允许你利用特殊的情况：连接的两端都具有相同的连接列名。它接受共享列名的一个逗号分隔列表，并且为其中每一个共享列构造一个包含等值比较的连接条件。例如用 `USING (a, b)` 连接 *T1* 和 *T2* 会产生连接条件 ON *T1*.a = *T2*.a AND *T1*.b = *T2*.b。

`JOIN USING` 的输出会废除冗余列。你不需要把匹配上的列都打印出来，因为它们必须具有相等的值。不过 `JOIN ON` 会先产生来自 *T1* 的所有列，后面跟上所有来自 *T2* 的列；而 `JOIN USING` 会先为列出的每一个列对产生一个输出列，然后先跟上来自 *T1* 的剩余列，最后跟上来自 *T2* 的剩余列。

最后，`NATURAL` 是 `USING` 的缩写形式：它形成一个 `USING` 列表， 该列表由那些在两个表里都出现了的列名组成。和 `USING` 一样，这些列只在输出表里出现一次。 

以下给出一些简单的示例，假设我们有一个表 `t1`：

```sql
 num | name
-----+------
   1 | a
   2 | b
   3 | c
```

和`t2`：

```sql
 num | value
-----+-------
   1 | xxx
   3 | yyy
   5 | zzz
```

然后我们用不同的连接方式可以获得各种结果：

```sql
=> SELECT * FROM t1 INNER JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   3 | c    |   3 | yyy
(2 rows)

=> SELECT * FROM t1 INNER JOIN t2 USING (num);
 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)

=> SELECT * FROM t1 NATURAL INNER JOIN t2;
 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)

=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num;
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |   3 | yyy
(3 rows)

=> SELECT * FROM t1 LEFT JOIN t2 USING (num);
 num | name | value
-----+------+-------
   1 | a    | xxx
   2 | b    |
   3 | c    | yyy
(3 rows)
```

用 `ON` 指定的连接条件也可以包含与连接不直接相关的条件。这种功能可能对某些查询很有用，但是需要仔细考虑。例如：

```sql
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num AND t2.value = 'xxx';
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |     |
(3 rows)
```

注意把限制放在 `WHERE` 子句中会产生不同的结果：

```sql
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num WHERE t2.value = 'xxx';
 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
(1 row)
```

这是因为放在 `ON` 子句中的一个约束在连接之前被处理，而放在 `WHERE` 子句中的一个约束是在连接之后被处理。这对内连接没有关系，但是对于外连接会带来麻烦。

##### 表和列别名

你可以给一个表或复杂的表引用指定一个表别名。

要创建一个表别名，我们可以写：

```sql
FROM table_reference AS alias
```

或者

```sql
FROM table_reference alias
```

`AS` 关键字是可选的。`alias` 可以是任意标识符。

表别名的典型应用是给长表名赋予比较短的标识符，好让连接子句更易读。例如：

```sql
SELECT * FROM some_very_long_table_name s JOIN another_fairly_long_name a ON s.id = a.num;
```

到这里，别名成为当前查询的表引用的新名称 — 我们不再能够用该表最初的名字引用它了。因此，下面的用法是不合法的：

```sql
SELECT * FROM my_table AS m WHERE my_table.a > 5;  
```

表别名主要用于简化符号，但是当把一个表连接到它自身时必须使用别名，例如：

```sql
SELECT * FROM people AS mother JOIN people AS child ON mother.id = child.mother_id;
```

此外，如果一个表引用是一个子查询，则必须要使用一个别名。

圆括弧用于解决歧义。在下面的例子中，第一个语句将把别名 `b` 赋给 `my_table` 的第二个实例，但是第二个语句把别名赋给连接的结果：

```sql
SELECT * FROM my_table AS a CROSS JOIN my_table AS b ...
SELECT * FROM (my_table AS a CROSS JOIN my_table) AS b ...
```

另外一种给表指定别名的形式是给表的列赋予临时名字，就像给表本身指定别名一样：

```sql
FROM table_reference [AS] alias ( column1 [, column2 [, ...]] )
```

如果指定的列别名比表里实际的列少，那么剩下的列就没有被重命名。这种语法对于自连接或子查询特别有用。

如果用这些形式中的任何一种给一个 `JOIN` 子句的输出附加了一个别名， 那么该别名就在 `JOIN` 的作用下隐去了其原始的名字。例如：

```sql
SELECT a.* FROM my_table AS a JOIN your_table AS b ON ...
```

是合法 SQL，但是：

```sql
SELECT a.* FROM (my_table AS a JOIN your_table AS b ON ...) AS c
```

是不合法的：表别名 `a` 在别名 `c` 定义后变得不再可见。

##### 子查询

子查询指定了一个派生表，它必须被包围在圆括弧里并且必须被赋予一个表别名。例如：

```sql
FROM (SELECT * FROM table1) AS alias_name
```

这个例子等效于 `FROM table1 AS alias_name` 。在子查询里面有分组或聚集的时候，子查询不能被简化为一个简单的连接。

一个子查询也可以是一个 `VALUES` 列表：

```sql
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow'))
     AS names(first, last)
```

为 `VALUES` 列表中的列分配别名是可选的，但是这样做是一个最佳实践。

#### WHERE 子句

WHERE 子句的语法如下：

```sql
WHERE search_condition
```

其中 *search_condition* 是一个返回 `boolean` 类型值的值表达式。

在完成对 `FROM` 子句的处理之后，MegaWise 会根据搜索条件对生成的虚拟表进行检查。如果该条件的结果是真，那么该行被保留在输出表中；否则将其舍弃。搜索条件通常至少要引用一些在 `FROM` 子句里生成的列。

------

**注意**

内连接的连接条件既可以写在 `WHERE` 子句也可以写在 `JOIN` 子句里。例如，以下表达式是等效的：

```sql
FROM a, b WHERE a.id = b.id AND b.val > 5
```

和：

```sql
FROM a INNER JOIN b ON (a.id = b.id) WHERE b.val > 5
```

以及：

```sql
FROM a NATURAL JOIN b WHERE b.val > 5
```

外部连接必须在 `FROM` 子句中完成。 外部连接的 `ON` 或 `USING` 子句不等于 `WHERE` 条件，因为它导致最终结果中行的增加（对那些不匹配的输入行）和减少。

------

以下是一些 `WHERE` 子句的例子：

```sql
SELECT ... FROM fdt WHERE c1 > 5

SELECT ... FROM fdt WHERE c1 IN (1, 2, 3)

SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2)

SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10)

SELECT ... FROM fdt WHERE c1 BETWEEN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10) AND 100

SELECT ... FROM fdt WHERE EXISTS (SELECT c1 FROM t2 WHERE c2 > fdt.c1)
```

#### GROUP BY 和 HAVING 子句

在通过了 `WHERE` 过滤器之后，生成的输入表可以使用 `GROUP BY` 子句进行分组，然后用 `HAVING` 子句删除一些分组行。

```sql
SELECT select_list
    FROM ...
    [WHERE ...]
    GROUP BY grouping_column_reference [, grouping_column_reference]...
```

`GROUP BY` 子句用来把表中在所列出的列上具有相同值的行分组在一起。 这些列的列出顺序并不影响查询结果。其效果是把每组具有相同值的行聚合为一行。例如：

```sql
=> SELECT * FROM test1;
 x | y
---+---
 a | 3
 c | 2
 b | 5
 a | 1
(4 rows)

=> SELECT x FROM test1 GROUP BY x;
 x
---
 a
 b
 c
(3 rows)
```

通常，如果一个表被分了组，那么没有在 `GROUP BY` 中列出的列都不能被引用，除非在聚集表达式中被引用。 一个用聚集表达式的例子是：

```sql
=> SELECT x, sum(y) FROM test1 GROUP BY x;
 x | sum
---+-----
 a |   4
 b |   5
 c |   2
(3 rows)
```

这里的 `sum` 是一个聚集函数，它在每个分组中对相应列进行求和操作。

以下是另一个例子：它计算每种产品的总销售额（而不是所有产品的总销售额）：

```sql
SELECT product_id, p.name, (sum(s.units) * p.price) AS sales
    FROM products p LEFT JOIN sales s USING (product_id)
    GROUP BY product_id, p.name, p.price;
```

在这个例子里，列 `product_id` 、`p.name` 和 `p.price` 必须在 `GROUP BY` 子句里， 因为它们都在查询的选择列表里被引用到。列 `s.units` 不必在 `GROUP BY` 列表里，因为它只在一个聚集表达式（`sum(...)`）里使用，它代表一组产品的销售额。对于每种产品，这个查询都返回一个该产品的所有销售额的总和行。

如果一个表已经用 `GROUP BY` 子句分了组，然后你又只对其中的某些组感兴趣，那么就可以用 `HAVING` 子句从结果中删除一些组。其语法是：

```sql
SELECT select_list FROM ... [WHERE ...] GROUP BY ... HAVING boolean_expression
```

在 `HAVING` 子句中的表达式可以引用分组的表达式和未分组的表达式。未分组的表达式必须涉及一个聚集函数。

例子：

```sql
=> SELECT x, sum(y) FROM test1 GROUP BY x HAVING sum(y) > 3;
 x | sum
---+-----
 a |   4
 b |   5
(2 rows)

=> SELECT x, sum(y) FROM test1 GROUP BY x HAVING x < 'c';
 x | sum
---+-----
 a |   4
 b |   5
(2 rows)
```

以下是另一个例子：

```sql
SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
    FROM products p LEFT JOIN sales s USING (product_id)
    WHERE s.date > CURRENT_DATE - INTERVAL '4 weeks'
    GROUP BY product_id, p.name, p.price, p.cost
    HAVING sum(p.price * s.units) > 5000;
```

在上面的例子里，`WHERE` 子句用那些非分组的列选择数据行（表达式只是对那些最近四周发生的销售为真）。 而 `HAVING` 子句限制输出为总销售收入超过 5000 的组。

如果一个查询包含聚集函数调用，但是没有 `GROUP BY` 子句，分组仍然会发生：结果是一个单一行（或者根本就没有行，如果该单一行被 `HAVING` 所消除）。


### 选择列表

#### 选择列表项

最简单的选择列表类型是*，指定选择表表达式生成的所有列。其他情况下，一个选择列表是一个由逗号分隔的值表达式列表。它可能是一个列名的列表：

```sql
SELECT a, b, c FROM ...
```

列名字 a、b 和 c 是在 `FROM` 子句里引用的表中列的实际名字或者别名。如果超过一个表有同样的列名，那么你还必须给出表名字，如：

```sql
SELECT tbl1.a, tbl2.a, tbl1.b FROM ...
```

在使用多个表时，可以使用如下方法选择一个特定表的所有列：

```sql
SELECT tbl1.*, tbl2.a FROM ...
```

可将值表达式应用于选择列表，为结果的每一行进行一次相应的表达式计算。

#### 列标签

选择列表中的项可以被指定名称，用于进一步的处理，例如为了让一个列在一个 `ORDER BY` 子句中使用或者在客户端显示。

```sql
SELECT a AS value, b + c AS sum FROM ...
```

如果没有使用 `AS` 指定列名，系统会分配一个默认的列名。对于简单的列引用，它是被引用列的名字。对于复杂表达式，系统会生成一个通用的名字。

如果新列的名称不匹配任何 MegaWise 关键词，则 `AS` 关键词是可选的。为了避免关键字的意外匹配，可以使用双引号来修饰列名。例如，`VALUE` 是一个关键字，所以下面的语句不能正常执行：

```sql
SELECT a value, b + c AS sum FROM ...
```

经过如下修改之后即可正常执行了：

```sql
SELECT a "value", b + c AS sum FROM ...
```

#### 使用 DISTINCT 删除结果的重复行

可以直接在列名前面加上 `DISTINCT` 关键字删除结果的重复行：

```sql
SELECT DISTINCT select_list ...
```

显然，如果两行里至少有一个列有不同的值，那么我们认为它是可区分的。空值在这种比较中被认为是相同的。

### 行排序

可以通过 `ORDER BY` 子句对结果顺序：

```sql
SELECT select_list
    FROM table_expression
    ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }]
             [, sort_expression2 [ASC | DESC] [NULLS { FIRST | LAST }] ...]
```

排序表达式可以是任何在查询的选择列表中合法的表达式。例如：

```sql
SELECT a, b FROM table1 ORDER BY a + b, c;
```

当多于一个表达式被指定，后面的值将被用于对前面值上相等的行排序。每一个表达式后可以选择性地放置一个 `ASC` 或 `DESC` 关键词来设置排序为升序或降序。`ASC` 顺序是默认值。升序会把较小的值放在前面，而“较小”则由 `<` 操作符定义。相似地，降序则由 `>` 操作符定义。 

`NULLS FIRST` 和 `NULLS LAST` 选项可以用来决定在排序顺序中，空值是出现在非空值之前或者之后。默认情况下，排序时空值被认为比任何非空值都要大，即 `NULLS FIRST` 是 `DESC` 顺序的默认值。

>注意：顺序选项是对每一个排序列独立考虑的。例如 `ORDER BY x, y DESC` 表示 `ORDER BY x ASC, y DESC` ，而非 `ORDER BY x DESC, y DESC` 。

一个 *sort_expression* 也可以是列标签或者一个输出列的编号，如以下命令均是根据第一个输出列排序：

```sql
SELECT a + b AS sum, c FROM table1 ORDER BY sum;
SELECT a, max(b) FROM table1 GROUP BY a ORDER BY 1;
```

>注意：*sort_expression*必须是一个输出列而不能是表达式。 例如，以下是不正确的：

```sql
SELECT a + b AS sum, c FROM table1 ORDER BY sum + c; 
```

### 使用 LIMIT 和 OFFSET 查询部分结果

使用 `LIMIT` 和 `OFFSET` 查询部分结果：

```sql
SELECT select_list
    FROM table_expression
    [ ORDER BY ... ]
    [ LIMIT { number | ALL } ] [ OFFSET number ]
```

如果通过 `LIMIT` 给出了一个限制计数，那么返回的结果行数不超过该计数值（但可能更少些，因为查询本身可能生成的行数就比较少）。`LIMIT ALL` 的效果和省略 `LIMIT` 子句一样。

`OFFSET` 跳过指定的行数并返回剩余的结果。`OFFSET 0` 的效果和省略 `OFFSET` 子句是一样的。

如果 `OFFSET` 和 `LIMIT` 都出现了， 那么先忽略 `OFFSET` 行再返回 `LIMIT` 个行。

由于 `LIMIT` 的结果不保证顺序，通常 `LIMIT` 和 `ORDER BY` 需要组合使用从而获得确定的结果。