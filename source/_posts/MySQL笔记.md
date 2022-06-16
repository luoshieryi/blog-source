---
title: MySQL基础
date: 2021-09-10
tags: [mysql, sql]
math: true
---

# 初识~09.06~

## 数据库术语

- 数据库 database : 保存有组织的数据的容器
- 表 table : 某种特定类型数据的结构化清单
  - 相同数据库不能使用相同表名
- 模式 schema : 关于数据库和表的布局及特性的信息 (有时用作数据库的同义词)
- 列 column : 表中的一个字段
  - 表由一个或多个列组成
- 数据类型 datatype : 所容许的数据的类型
- 行 row : 表中的一个记录
- 主键 primary key :  一列（或一组列），其值能够唯一区分表 中每个行
  - 任意两行主键值不同 (一组列时, 所有列值的组合唯一)
  - 每行都有主键值(不允许NULL)
  - 好习惯 : 
    - 不更新主键列中的值
    - 不重用主键列的值
    - 不在主键列中使用可能会更改的值
- 外键 foreign key : 某个表中的一列, 它包含另一个表的主键值, 定义了两个表之间的关系 
  - 列名可以与原主键不同, 数据类型要相同

## SQL简介

sequel是结构化查询语言 Structured Query Language 的缩写, 一种专门用来与数据库通信的语言

## MySQL

MySQL是一种DBMS, 即一种数据库软件

# 使用MySQL

选择数据库: `USE 数据库名`

返回可用数据库列表: SHOW DATABASES

- 包含MySQL内部使用的数据库

`SHOW` : 查看数据库, 表和内部信息

# 检索数据-SELECT~09.08~

## SELECT基础

`SELECT 列名 FROM 表名`

- SQL语句应以 ; 分隔\结束
- SQL语句不区分大小写, 但是推荐大写
- SQL语句中的所有空格都被忽略

检索多个列: `SELECT 列名,列名 FROM 表名`

检索所有列: `SELECT * FROM 表名`

- 这样可以检索未知列

只检索具有不同值的列表: `SELECT DISTINCT 列名 FROM 表名`

限制不多于五行: `SELECT 列名 FROM 表名 LIMIT 5`
返回从第四行开始的五行: `SELECT 列名 FROM 表名 LIMIT 4,5` , 同义于`SELECT 列名 FROM 表名 LIMIT 5 OFFSET 4`

- 检索出来的**第一行是行0**

使用完全限定的列名, 表名: `SELECT 表名.列名 FROM 连接名.表名`

## 排序检索数据-ORDER BY

- 如果不排序, 数据一般以它在底层表中出现的顺序显示 (更新或删除后会受到影响)

使用 OREDR BY : `SELECT 列名1 FROM 表名 ORDER BY 列名2`

- 可以用`,`分隔以多列排序
- 默认以升序排序(ASC), 在列名后附加` DESC`指定该列以降序排序

## 条件过滤数据-WHERE

使用 WHERE : `SELECT 列名1 FROM 表名 WHERE 条件`

- **`ORDER BY`应位于`WHERE`之后**
- WHERE 支持的子句操作符: =, <>(不等于), !=(不等于), <, <=, >, >=, BETWEEN(在指定的两个数据之间
- 字符串以单引号括住, 数字不需要
- BETWEEN 示例: `SELECT 列名1 FROM 表名 WHERE 列名2 BETWEEN 数值a AND 数值b`
  - BETWEEN 的范围是>=a , <=b
- 空值检查: `WHERE 列名 IS NULL`
  - 不匹配搜索(使用不等号)不会返回具有NULL值的行

组合 WHERE 语句, 使用AND, OR : `WHERE 条件1 AND 条件2`

- AND 的优先级比 OR 高, 需要使用圆括号正确组合操作符

通过 IN 指定条件范围: `WHERE 列名 IN (值1, 值2)`

- IN 比等效的 OR 执行更快
- IN 可以包含其他 SELECT 语句

通过 NOT 否定后跟条件的关键字: `WHERE 列名 NOT IN (值1, 值2)`

- MySQL支持 NOT 对IN, BETWEEN, EXISTS取反
- NOT 只否定后边紧跟的一个条件

通过 LIKE 使用通配符: `WHERE 列名 LIKE 'jet%'` , 找出所有词以jet起头的内容

- 通配符 % 表示任意字符出现任意次数(包括0个字符), 此处即接受jet后的任意字符
  - `LIKE '%'`不能匹配NULL 
- 通配符可以在任意位置出现任意次
- 通配符 _ 只匹配单个字符, 不包括0个字符
- 通配符运行效率较差, 尽量使用其他语句
  使用多个条件搜索时应当将通配符相关的搜索条件放在靠后执行的位置

### 正则表达式-REGEXP

REGEXP 后所跟的东西作为正则表达式

- 例: `WHERE 列名 REGEXP '1000'`, 检索列中包含文本 1000 的所有行
- `.`是正则表达式语言中一个特殊的字符, 表示匹配任意一个字符
- 默认不区分大小写, 使用`BINARY`关键字区分大小写

OR 匹配: `WHERE 列名 REGEXP '1000 | 2000 | 3000'`

[] 匹配一组字符中的任意一个: `REGEXP [123]000`, 等效上一句

- `[^123]`匹配除1, 2, 3以外的任何内容
- `[0-9]`匹配 0~9 任意数字

特殊匹配规则:

- 匹配"-"与"."等特殊字符: 使用\\为前导, 进行转义 `\\-` 

- `\\ `也用来引用元字符: 如`\\n`表示换行

- 匹配字符类: 如`[:alnum:]`表示任意字母和数字, 同`[a-zA-Z0-9]`

匹配多个实例: 默认只对其前边的一个字符有效

- `*` : 0个或多个匹配
- `+` : 1个或多个匹配, 等于`{1, }`
- `?` : 0个或一个匹配, 等于 `{0,1}`
- `{数值}` : 指定数目的匹配
- `{数值, }` : 不少于指定数目的匹配
- `{数值1, 数值2}` : 匹配数目的范围 (数值2不超过255)
- 示例: 
  1. `REGEXP '\\([0-9] sticks?\\)'` : 此句中 ? 使最后的 s 成为可选字符, 有没有s都会被检出, 有其他字符不会
  2. `REGEXP '[[:digit:]]{4}` : 匹配连在一起的任意四位数字, 等效于`'[0-9][0-9][0-9][0-9]'`

匹配特定位置的文本: 

- `^` : 文本的开始
- `$` : 文本的结尾
- `[[:<:]]` : 词的开始
- `[[:>:]]` : 词的结尾
- 示例: `REGEXP '^[0-9\\.]` 在 . 或任意数字为串中第一个字符时匹配
- 通过^开始每个表达式, $结束每个表达式, 可以使REGEXP的作用与LIKE一样

## 创建计算字段

字段: 基本与 列 相同

Concat()函数 拼接: 将值连在一起构成单个值

- 示例: `SELECT Concat(列名1, ' (', 列名2, ')')`
  会输出`值1 (值2)`
- Concat 的多个串以","分隔

删除数据多余的空格: Trim()函数

- `RTrim(列名)`去掉值右边的空格
- `LTrim(列名)`去掉值左边的空格
- `Trim(列名)`去掉左右两边的空格

AS 使用别名: 新计算的列没有名字无法引用, 赋予别名解决这个问题

- 示例: `SELECT Concat(列名1, ' (', 列名2, ')') AS 别名`
- 在原名不合法时重命名它, 易误解时扩充它
- 别名也称导出列

计算: 支持 +, -, *, / 四个基本操作运算符, 可用圆括号区分优先顺序

- 示例: `SELECT 列名1 + 列名2 AS 别名`

## 数据处理函数

*函数的可移植性较差, 使用时建议做好注释*

文本处理函数:

- 示例 `WHERE Soundex(列名) = Soundex('Lie')`, 匹配发音与Lie类似的值(如Lee)

日期和时间处理函数:

- 日期格式应为 `yyyy-mm-dd`
- 基本日期条件过滤: `WHERE 列名 = 'yyyy-mm-dd'`
  - 默认具体时间为`00:00:00`, 如果表中值为`yyyy-mm-dd 11:30:05`不会匹配
  - 使用Date()函数, 仅比较给出的日期(即只比较`yyyy-mm-dd`), 相应的Time函数只比较时间: `WHERE Date(列名) = 'yyyy-mm-dd'`
  - 使用Year(), Month()等只返回年份, 月份

数值处理函数: 一般用于代数, 三角或几何运算

## 汇总数据

> 有时返回具体数据是对时间和资源的浪费

SQL的五个聚集函数:

<table border="2">
    <tr>
        <th>函数</th>
        <td>AVG()</td>
        <td>COUNT()</td>
        <td>MAX()</td>
        <td>MIN()</td>
        <td>SUM()</td>
    </tr>
    <tr>
    	<th>功能</th>
        <td>平均值</td>
        <td>行数</td>
        <td>最大值</td>
        <td>最小值</td>
        <td>总和</td>
    </tr>
</table>
AVG() : `SELECT AVG(列名) AS 别名 FROM 表名` , 得到这一列的平均值

COUNT() : 

- `SELECT COUNT(*)` : 对表中包括空值的所有行计数
- `COUNT(列名)` : 对特定列中具有值的行计数, 不计算NULL

MAX() 与 MIN() 与 SUM() : 要求指定列名, 会忽略NULL

**使用`DISTINCT`参数, 只考虑不同的内容: **

- 示例: `AVG(DISTINCT 列名)`, 计算平均值时只取不同的值
- 必须指定列名
- *有一个对应的参数`ALL`, 是没什么用的默认值*

组合聚集参数: 使用","隔开不同语句, 可以换行好看些

## 分组数据-GROUP BY~09.09~

把数据分成多个逻辑组, 以便对每个组进行聚集计算

- 示例: `SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id`
  输出: ![image-20210909130806621](https://raw.githubusercontent.com/luoshieryi/images/main/markdown/image-20210909130806621.png)

- GROUP BY子句可以包含任意数目的列, 使得可以通过对分组嵌套达到更细致的控制
- GROUP BY子句中不能使用别名, 若在 SELECT 中使用表达式, GROUP BY 子句中须指定相同的表达式
- 除聚集计算语句外, SELECT 中每个列都需要在GROUP BY 子句中给出
- 一个或多个NULL值也会被分为一组
- **GROUP BY 子句应在 WHERE 之后, ORDER BY 之前**
- **使用 GROUP BY 子句时, 应当给出 ORDER BY 子句保证正确排序**
- 后边附加`WITH ROLLUP`, 可以获得所有分组汇总的值

### 过滤分组-HAVING

HAVING 非常类似于 WHERE (WHERE 在分组前进行过滤, HAVING 在分组后进行过滤)

- 示例: `SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >=2`
  过滤两个以上的 COUNT(*) 的分组
- 不能使用别名
- **HAVING支持所有WHERE操作符**

## SELECT 子句顺序

SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT

## 子查询-嵌套

嵌套在其他查询中的查询 : 将一条 SELECT 语句返回的结果用于另一条 SELECT 语句的 WHERE 子句

- <a name="子查询1">示例</a>:

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                  FROM orders
                  WHERE order_num IN (SELECT order_num
                                      FROM orderitems
                                      WHERE prod_id ='TNT2'))

```

- 上边的例子在性能上不一定是最好的, [之后的联结查询](#联结查询1)会给出一个更好的方法
- 子查询一般与 IN 操作符结合使用, 也可以用 =, <> 等
- 作为计算字段使用子查询示例:
  - 这里嵌套内的 WHERE 使用了完全限定列名, 告诉 SQL 比较 orders 表中的 cust_id 与当前正从customers 表中检索的 cust_id
  - 这种称为相关子查询, 只要列名可能有多义性, 就必须使用这种语法

```sql
SELECT cust_name,
			 cust_state,
			 (SELECT COUNT(*)
        FROM orders
        WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

> 建议先用硬编码数据建立和测试外层查询, 并且仅在确认它正常后才嵌入子查询

## 联结表(划重点)

联结是一种机制, 用来在一条 SELECT 语句中关联表, 可以联结多个表返回一组输出

- <a name="示例-联结表1">示例</a>: 

  ```sql
  SELECT vend_name, prod_name, prod_price
  FROM vendors, products
  WHERE vendors.vend.id = products.vend.id
  ORDER BY vend_name, prod_name;
  ```

  这里三个列分别在两个表中, 通过 WHERE 语句正确联结两个表, 需要完全限定列名

- 没有联结条件的表关系返回结果为笛卡尔积, 检索出的行数是第一个表中行数乘以第二个表中行数

- 叉联结是一种笛卡尔积的联结类型

### 内部联结(等值联结)

即[上边示例](#示例-联结表1)的联结方式, 可以这样明确指定联结的类型:

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend.id
```

联结条件用特定的 ON 子句而不是 WHERE 子句给出

### 联结多个表

联结太多表性能下降会很严重

<a name="联结查询1">示例1</a>: 使用联结查询优化[上边的子查询](#子查询1)

```sql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num
	AND prod_id='TNT2';
```

### 高级联结

#### 使用表别名

- 缩短SQL语句
- **允许在单条 SELECT 语句中多次使用相同的表** `FROM employee AS a, employee AS b`

#### 自联结

以下两种解决方案结果相同, 第二种自联结方式效率高于第一种子查询

```sql
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
                 FROM products
                WHERE prod_id = 'DTNTR')
```

```sql
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
	AND p2.prod_id = 'DTNTR'
```

#### 自然联结

自然联结排除多次出现, 使每个列只返回一次

- 需要自己手动完成

#### 外部联结

包含了在相关表中没有关联行的行

- 必须使用 RIGHT 或 LEFT 关键字指定包括其所有行的表 (包括OUTER JOIN 右边或左边的表) , 即从右边或左边表中选择所有行

- 内部, 外部联结的区别示例|
  ![image-20210909183350173](https://raw.githubusercontent.com/luoshieryi/images/main/markdown/image-20210909183350173.png)![image-20210909183441236](https://raw.githubusercontent.com/luoshieryi/images/main/markdown/image-20210909183441236.png)

#### 带聚集函数的联结

```sql
SELECT customers.cust_name,
			 customers.cust_id,
			 COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

## 组合查询-UNION

组合使用多条 SELECT 语句, 等效于具有多个 WHERE 子句的单条 SELECT 语句. 两种查询性能优劣不一定

通过在各 SELECT 语句之间加入 UNION 实现

- 示例: 

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price < 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

等效于

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
OR vend_id IN (1001,1002);
```

- UNION 中的每个查询必须包含相同的列, 表达式或聚集函数, 顺序可以不同
- UNION 从查询结果集中自动去除重复的行
  - 使用 UNION ALL 返回不去重的全部行 (此时必须使用 UNION 而不能 WHERE)
- 组合查询可以应用于不同的表

### 对组合查询结果排序

使用 UNION 时, 只能使用一条 ORDER BY 语句, 出现在最后一条 SELECT 之后,

## 全文本搜索

*并非所有引擎都支持全文本搜索*'

对指定列中各词创建一个索引, 针对这些词进行搜索.

- 为进行全文本搜索, 必须索引被搜索的列, 且随着数据的改变不断进行索引(自动进行)
- 索引后, SELECT 可与 Match() 和 Against() 一起使用执行搜索

全文本搜索的优点:

- 性能更好
- 明确控制
- 智能化的结果

启用全文本搜索支持: 在创建表时通过 FULLTEXT 子句, 示例如下

```sql
CREATE TABLE productnotes
(
    note_id		int			NOT NULL AUTO_INCREMENT,
    prod_id		char(10)	NOT NULL,
    note_date	datetime	NOT NULL,
    note_text	text		NULL,
    PRIMARY KEY(note_id),
    FULLTEXT(note_text)
)ENGING=MyISAM
```

- 这里根据`FULLTEXT(note_text)`对note_text进行索引
- 定义之后, MySQL会自动维护索引

进行全文本搜索: 使用`Match()`和`Against()`两个函数进行全文本搜索, 其中Match()指定被搜索的列, Against()指定要使用的搜索表达式, 示例如下

```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');
```

等效于

```sql
SELECT note_text
FROM productnotes
WHERE note_text LIKE '%rabbit%';
```

- 指定 rabbit 作为搜索文本, 搜索 note_text 列中包含 rabbit 的行
- 前者以文本匹配良好程度排序的数据, 后者以不特别有用的顺序返回数据

注意事项: 

- 传递给 Match() 的值必须与 FULLTEXT() 中定义的相同, 多个列时必须列出他们且次序正确

- 搜索默认不区分大小写 (除非使用 BINARY 方式)

- 指定多个搜索项时, 包含多数匹配词的行将具有更高的等级值

### 使用查询扩展

流程: `Against('值' WITH QUERY EXPANSION)`

1. 先进行一个基本的全文本搜索, 找出与搜索条件匹配的所有行
2. MySQL 检查这些匹配行并选择有用的词
3. 再次进行全文本搜索, 使用原来的词 + 有用的词

### 布尔文本搜索

以布尔方式, 提供如下关于内容的细节: 

- 要匹配的词
- 要排斥的词
- 排列提示(指定某些词比其他词更重要)
- 表达式分组
- 另外一些内容

*即使没有FULLTEXT也可以使用布尔方式, 但会非常缓慢*

示例: `WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE)`

- `-rope*`指示排除包含rope*的词(以rope开始的词)
- ![image-20210910205020402](https://raw.githubusercontent.com/luoshieryi/images/main/markdown/image-20210910205020402.png)

### 使用说明

> - 在索引全文本数据时，短词被忽略且从索引中排除。短词定义为 那些具有3个或3个以下字符的词（如果需要，这个数目可以更改）。
> - MySQL带有一个内建的非用词（stopword）列表，这些词在索引全文本数据时总是被忽略。如果需要，可以覆盖这个列表（请参 阅MySQL文档以了解如何完成此工作）。
> - 许多词出现的频率很高，搜索它们没有用处（返回太多的结果）。因此，MySQL规定了一条50%规则，如果一个词出现在50%以上的行中，则将它作为一个非用词忽略。50%规则不用于IN BOOLEAN MODE。
> - 如果表中的行数少于3行，则全文本搜索不返回结果（因为每个词 或者不出现，或者至少出现在50%的行中）。
> - 忽略词中的单引号。例如，don't索引为dont。
> - 不具有词分隔符（包括日语和汉语）的语言不能恰当地返回全文本搜索结果。
> - 如前所述，仅在MyISAM数据库引擎中支持全文本搜索。

# 插入/更新/删除~09.10~

## 插入数据-INSERT

通常方法:

```sql
INSERT INTO 表1
VALUES(NULL,
      '值1',
      '值2',
      NULL,
      '值3');
```

- 插入一个新内容到表1中, 按照表中列的顺序输入
- 若某一列没有值, 需要使用NULL(允许的话)

更安全的方法:

```sql
INSERT INTO 表2(列名1,
              列名2,
              列名3)
              VALUES('值1',
                     NULL,
                     '值2');
```

- 在表名后给出列名, 与之后的值顺序对应
- 不需要提供值的列直接不写出列名 (NO NULL且没有默认值(如自增值)的必须赋值)

插入多行: 同时使用以";"分隔的多条 INSERT 语句或者单条 INSERT 有多组值(更快)

```sql
INSERT INTO 表3(列名1,
               列名2)
               VALUES(
                   '值1',
                   '值2'
               ),
               (
                   '值3',
                   '值4'
               );
```

插入检索出的数据:

```sql
INSERT INTO 表4(列1,
               列2)
               SELECT 列3,
               		  列4
               FROM 表5;
```

## 更新数据-UPDATE

示例:

```sql
UPDATE 表1
SET 列1 = '值1',
	列2 = '值2'
WHERE 条件1;
```

- 使用 WHERE 限定要更新的行, 不然它会更新所有行
- 更新时如果出现错误, 则整个UPDATE操作被取消, 使用`UPDATE IGNORE 表1`, 即使发生错误也继续更新
- 通过`SET 列1 = NULL`删去某列的值(假设允许NULL)

## 删除数据-DELETE

示例:

```sql
DELETE FROM 表2
WHERE 条件2;
```

- 删除指定条件的行
  - 删除指定的列使用UPDATE
- 不使用WHERE会删除全部行

删除表中所有行使用 TRUNCATE TABLE 更快(删除原来的表并重新创建一个)

**更新/删除前应现用 SELECT 对 WHERE 条件进行测试**

# 创建和操纵表

## 创建表-CREATE

使用交互式工具, 或直接使用MySQL语句

```sql
CREATE TABLE customers
(
    cust_id 			int 		NOT NULL AUTO_INCREMENT,
    cust_name 		char(50) 	NOT NULL,
    cust_address 	char(50) 	NULL,
  	cust_num			int				NOT NULL DEFAULT 1,
    PREMARY KEY (cust_id)
) ENGINE=InnoDB;
```

- 使用`PREMARY KEY (列1, 列2)`指定主键, 主键不能有 NULL 与重复值
- 使用`NOT NULL`指定不允许 NULL , 否则默认为允许
- 使用`AUTO_INCREMENT`指定对该列自动增量
  - `SELECT last_insert_id`返回最后一个 AUTO_INCREMENT 值
- 使用`DEFAULT 值1`指定默认值
- 使用`ENGINE=引擎名`指定引擎类型, 不指定时使用默认引擎
  - `InnoDB` : 不支持全文本搜索, 可靠的事务处理引擎
  - `MEMORY` : 数据存储在内存中, 速度很快, 适合临时表, 功能等同于 `MyISAM`
  - `MyISAM` : 性能极高, 支持全文本搜索, 不支持事务处理
  - **外键不能跨引擎**, 引擎混用的缺点

## 更新表-ALTER

理想状态下不应当更新表, 常见用途是定义外键

```sql
ALTER TABLE 表名1
ADD 列名1 数据类型1; 			-- 添加列1
DROP COLUMN 列名2;			 -- 删除列2
ADD CONSTRAINT 外键约束名 FOREIGN KEY (列名3)		-- 添加外键
REFERENCES 表名2 (列名4)
ADD CONSTRAINT fk_products_vendors FOREIGN KEY (vend_id)		-- 添加外键示例
REFERENCES vendors (vend_id)
```

## 删除表-DROP

`DROP TABLE 表名`, 没有确认, 不能撤销

## 重命名表-RENAME

`RENAME TABLE 表名1 TO 表名2`, 可以同时对多个表重命名

# 其他功能~09.11~

## 使用视图

视图示例① : 简化复杂的联结

```sql
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = order.cust_id
	AND orderitems.order_num = orders.order_num;
```

- 创建了一个名为 `productcutomers` 的视图, 联结三个表, 返回其中的三列
- 使用 `SELECT * FROM productcutomers` 返回这个视图的三列

视图示例② : 重新格式化检索出的数据

```sql
CREATE VIEW vendorlocations AS
SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country), ')')
		AS vend_title
FROM vendors
ORDER BY vend_name;
```

- 之后使用 `SELECT * FROM vendorlocations;` 即可实现

视图的用处:

- 重用 SQL 语句
- 简化复杂的 SQL 操作
- 仅使用表的部分组成, 保护数据
- 更改数据格式和表示

注意点:

- 视图不包含任何数据, 它包含的是一个 SQL 查询
- 使用视图相当于执行检索, 过于复杂或嵌套过多的视图性能会下降严重
- 视图必须唯一命名(与表一样)
- 使用视图时的 ORDER BY 会覆盖视图内部的 ORDER BY
- 视图可以与表同时使用
- 视图不能索引

更新视图: 对视图使用 INSERT, UPDATE, DELETE

- 更新视图其实是更新其基表
- 若要更新的基数据不能被确定, 则无法更新, 即视图定义中有以下操作: 分组, 联结, 子查询, 并, 聚集函数( Min(), Count(), Sum() )等, DISTINCT, 导出列
- 视图应用于检索而不是更新

## 使用存储过程

为以后的使用保存一条或多条MySQL语句的集合

用处 :

- 把处理封装, 简化复杂的操作
- 保证所有开发人员使用的代码相同(同一存储过程), 防止错误
- 简化对变动的管理 (使用存储过程的人员不需要知道这些变化)

优点 : 简单, 安全, 高性能

- 性能更好, 使用存储过程比单独的 SQL 语句更快
- 用 一些只能用在单个请求中的MySQL元素和特性 编写功能更强更灵活的代码 [示例](#智能存储示例1)

缺陷 : 编写时 更困难,需要安全访问权限 (编写与执行存储过程的权限是分开的)

### 执行存储过程-<a name="调用存储过程1">调用</a>

```sql
CALL productpricing(@pricelow,
                   	@pricehigh,
                   	@priceaverage);
```

- 通过 `@pricelow` 等向存储过程传递参数
- 所以MySQL变量都必须以 @ 开始
- 该语句不显示任何值, 使用 `SELECT @pricelow, @pricehigh;` 调用变量

### 创建存储过程

```sql
CREATE PROCEDURE productpricing()
BEGIN
	SELECT Ave(prod_price) AS priceaverage
	FROM products;
END;
```

- 创建名为 `productpricing` 的存储过程
- 若存储过程接受参数, 在名称后的 () 中列出
- 通过 `CALL productpricing()` 调用这个存储过程

使用命令行实用程序时需要更改分隔符, `DELIMITER //` 告诉它使用//作为新的语句结束分隔符, 语句结束时再使用 `DELIMITER ;` 换回去

### 删除存储过程

`DROP PROCEDURE productpricing`

- 删除时没有后面的 () 
- 若过程不存在会返回一个错误, 使用 `DROP PROCEDURE IF EXISTS` 不会产生错误

### 使用参数

一般存储过程不显示结果, 而是把结果返回给变量

- 变量: 内存中一个特定的位置, 用来临时存储数据, 

示例 1 :

```sql
CREAT PROCEDURE productpricing(
	OUT p1 DECIMAL(8, 2),
  OUT ph DECIMAL(8, 2),
  OUT pa DECIMAL(8, 2)
)
BEGIN
	SELECT Min(prod_price)
	INTO p1
	FROM products;
	SELECT Max(prod_price)
	INTO ph
	FROM products;
	SELECT Avg(prod_price)
	INTO pa
	FROM products;
END;
```

- 此过程接受三个参数: p1存储产品最低价, ph存储最高价, pa存储平均价
- 每个参数必须指定类型, 这里用  DECIMAL 指定十进制, (8, 2) 指定小数点左侧最多 8 位, 右侧最多 2 位. (右侧最多为8)
- 记录集是不允许的类型, 所以不能通过一个参数返回多行列, 此处要使用三个参数
- 通过[此处示例](#调用存储过程1)调用此过程

示例 2 :

```sql
CREATE PROCEDURE ordertotal(
	IN onumber INT,
  OUT ototal DECIMAL(8, 2)
)
BEGIN
	SELECT Sum(item_price * quantity)
	FROM orderitems
	WHERE order_num = onumber
	INTO ototal;
END;
```

- 使用 IN 表示要输入的数据
- 示例调用 `CALL ordertotal(20005, @total)`

### 建立智能存储

在存储过程内包含业务规则和智能处理, <a name="智能存储示例1">智能存储示例</a>:

```sql
-- Name: ordertotal
-- Parameters: onumber = order number
						-- taxable = 0 ifnot taxable, 1 if taxable
						-- ototal	 = order total variable
						
CREATE PROCEDURE ordertotal(
	IN onumber INT,
  IN taxable BOOLEAN,
  OUT ototal DECIMAL(8, 2)
) COMMENT 'Obtain order total, optionally adding tax'
BEGIN

	-- Declare variable for total
	DECLARE total DECIMAL(8, 2);
	-- Declare tax percentage
	DECLARE taxrate INT DEFAULT 6;
	
	-- Get the order total
	SELECT Sum(item_price*quantity)
	FROM orderitems
	WHERE order_num = onumber
	INTO total;
	
	-- Is this taxable?
	IF taxable THEN
		-- Yes, so add taxrate to the total
		SELECT total+(total/100*taxrate) INTO total;
	END IF;
	
	-- And finally, save to out variable
	SELECT total INTO ototal;
	
END;
```

- 前边放置 -- 增加注释
- 使用 DECLARE 声明两个局部变量

- 若增加营业税, 则输入 taxable 为真, 执行之后的 SELECT , 并将结果存储到局部变量 total
- 最后用一句 SELECT 将 total 保存到 ototal

- COMMENT 添加一个解释, 会在 SHOW PROCEDURE STATUS 的结果中显示

- 使用 `CALL ordertotal(20005, 0, @total);` 使用这个过程

### 检查存储过程

显示用来创建一个存储过程的CREATE语句: `SHOW CREATE PROCEDURE ordertotal;` 

显示何时何人创建等详细信息: `SHOW PROCEDURE STATUS`, 信息太多可以使用 LIKE 过滤

## 使用游标

~~*不怎么用得到...过一下看一眼*~~

*咕了*

## 使用触发器

*同上*

## 管理事务处理

事务处理是一种机制, 用来管理必须成批执行的 MySQL 操作

- 用来维护数据库的完整性, 保证成批的 MySQL 操作要么完全执行, 要么完全不执行
- 显示事务处理需要所用数据引擎支持

相关名词:

- 事务: 一组 SQL 语句
- 回退: 撤销指定 SQL 语句的过程
- 提交: 将未存储的 SQL 语句结果写入数据库表
- 保留点: 事务处理中设置的临时占位符, 可以对它发布回退(不同于回退整个事务处理)

使用 `START TRANSACTION`; 标识事务的开始

使用 `ROLLBACK` 回退 START TRANSACTION 之后的所有语句

- 不能回退 CREATE 或 DROP 操作
- 使用后事务会自动关闭

使用 `COMMIT` 提交 START TRANSACTION 之后的所有语句

- 一般情况语句会自动隐式提交, 事务处理时语句需要手动提交
- 如果前边的语句出错, COMMIT 不会提交
- 使用后事务会自动关闭

使用 `SAVEPOINT 保留点1`; 创建保留点, 支持回退部分事务处理

- 使用 `ROLLBACK TO 保留点1;` 回退到指定位置

- 保留点在事务处理完成后自动释放 (执行一条 ROLLBACK 或 COMNMIT)

### 使用 `SET autocommit=0` 取消默认提交

- 该标志针对每个连接而不是服务器

## 字符集和校对顺序

字符集: 字母和符号的集合

编码: 某个字符集成员的内部表示

校对: 规定字符如何比较的指令

使用 `SHOW CHARACTER SET;` 显示所有可用的字符集以及每个字符集的描述和默认校对

使用 `SHOW COLLATION;` 查看所支持校对的完整列表以及它们适用的字符集

- 许多校对出现两次, 一次区分大小写(以 `_cs` 表示), 一次不区分大小写(以 `_ci` 表示)

给表指定字符集和校对: 在 `CREATE TABLE()` 语句后附加 `DEFAULT CHARACTER SET 字符集1 COLLATE 校对1;`

还支持给某个列指定字符集和校对, 

也支持在 SELLECT 语句末 `COLLATE 校对2`指定校对 (用于临时区分大小写)

## 安全管理

创建用户, 管理用户权限等

## 数据库维护

备份数据, 数据库维护, 诊断启动问题, 查看日志文件等













事务处理

设置保留点

中途出错回退到保留点

之后的语句呢?