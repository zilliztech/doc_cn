---
id: "megawise_sql_datadef"
lang: "cn"
title: "数据定义"
---
# 数据定义

<!-- TOC -->

- [表的基础概念](#表的基础概念)
- [默认值](#默认值)

<!-- /TOC -->

## 表的基础概念

关系型数据库中的一个表由行和列组成。列的数量和顺序是固定的，并且每一列拥有一个名字。行的数目是变化的，它反映了在一个给定时刻表中存储的数据量。SQL 并不保证表中行的顺序。当一个表被读取时，表中的行将以非特定顺序出现，除非明确地指定需要排序。此外，SQL 不会为行分配唯一的标识符，因此在一个表中可能会存在一些完全相同的行。

MegaWise 包括了相当多的内建数据类型，大部分内建数据类型有着显而易见的名称和语义。一些常用的数据类型是：用于整数的 integer；可以用于分数的 numeric；用于字符串的 text，用于日期的 date，用于一天内时间（无日期）的 time 以及可以同时包含日期和时间（无时区）的 timestamp。

要创建一个表，我们要用到 `CREATE TABLE` 命令。在这个命令中 我们需要为新表至少指定一个名字、列的名字及数据类型。例如：

```sql
CREATE TABLE my_first_table (
    first_column text,
    second_column integer
);
```

将创建一个名为 my_first_table 的表，它拥有两个列。第一个列名为 first_column 且数据类型为 text ；第二个列名为 second_column 且数据类型为 integer 。注意列表中各列由逗号分隔并被圆括号包围。

当然，前面的例子是非常不自然的。通常，我们为表和列赋予的名称都会表明它们存储着什么类别的数据。因此让我们再看一个更现实的例子：

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```

（`numeric` 类型能够存储小数部分，典型的例子是金额。）

一个表能够拥有的列的数据是有限的，根据列的类型，这个限制介于250和1600之间。

如果我们不再需要一个表，我们可以通过使用 DROP TABLE 命令来移除它。例如：

```sql
DROP TABLE my_first_table;
DROP TABLE products;
```

移除一个不存在的表会引起错误。然而，在 SQL 脚本中创建每个表之前无条件地尝试移除表的做法很常见，即使发生错误也会忽略，因此这样的脚本无论是表存在还是不存在但时候都可以正常工作（可以使用 `DROP TABLE IF EXISTS` 变体来防止出现错误消息，但这并非标准 SQL ）。

## 默认值

一个列可以被分配一个默认值。当一个新行被创建且没有为某些列指定值时，这些列将会被它们相应的默认值填充。

如果没有显式指定默认值，则默认值是空值。空值表示未知数据。

在一个表定义中，默认值被列在列的数据类型之后。例如：

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```