## MySQL必知必会笔记

### 第4章 检索数据

#### 4.4 检索所有列

- 一般，**除非你确实需要表中的每个列，否则最好别使用*通配符**。因为**检索不需要的列通常会降低检索和应用程序的性能**。

#### 4.5 检索不同的行

- **DISTINCT**关键字指示MySQL**只返回不同的值**；

- ```mysql
  SELECT DISTINCT vend_id FROM products;
  # DISTINCT关键字告诉MySQL只返回不同的vend_id行
  # 如果使用DISTINCT关键字，它必须直接放在列名的前面
  ```

- **不能部分使用DISTINCT**：**DISTINCT关键字应用于所有列而不仅是前置它的列**。即如果给出SELECT DISTINCT vend_id, prod_price，除非指定的两个列都不同，否则所有行都将被检索出来。

#### 4.6 限制结果

- **为了返回第一行或前几行**，可使用**LIMIT子句**；

- ```mysql
  # LIMIT 5指示MySQL返回不多于5行
  SELECT prod_name FROM products LIMIT 5;
  # LIMIT 5,5指示MySQL返回从行5开始的5行，第一个数为开始位置（从0算），第二个数为要检索的行数
  SELECT prod_name FROM products LIMIT 5,5;
  ```

- **检索出来的第一行为行0而不是行1**，因此，LIMIT 1将检索出第二行而不是第一行。



### 第5章 排序检索数据

#### 5.1 排序数据

- **为了明确地排序用SELECT语句检索出的数据，可使用ORDER BY子句**。ORDER BY子句**取一个或多个列**的名字，据此对输出进行排序；

- ```mysql
  # 指示MySQL对prod_name列以字母顺序排序数据
  SELECT prod_name FROM products ORDER BY pro_name;
  ```

- **通过非选择列进行排序：**通常，ORDER BY子句中使用的列将是为显示所选择的列。但是，实际上并不一定要这样，**用非检索的列排序数据是完成合法的**。

#### 5.2 按多个列排序

- 为了**按多个列排序，只要指定列名，列名之间用逗号分开即可**；

- ```mysql
  # 检索3个列，并按其中两个列对结果进行排序——首先按价格，当价格一致时，再按名称排序
  SELECT prod_id, prod_price, prod_name
  FROM products
  ORDER BY prod_price, prod_name;
  ```

#### 5.3 指定排序方向

- 数据**排序默认为升序**。**为了进行降序排序，必须指定DESC关键字**；

- ```mysql
  SELECT prod_id, prod_price, prod_name
  FROM products
  ORDER BY prod_price DESC;
  ```

- **在多个列上降序排序：**DESC关键字只应用到直接位于其前面的列名。**如果想在多个列上进行降序排序，必须对每个列指定DESC关键字**；

- ```mysql
  # 使用ORDER BY和LIMIT的组合，能够找出一个列中最高或最低的值
  SELECT prod_price
  FROM products
  ORDER BY prod_price DESC
  LIMIT 1;
  ```

- **ORDER BY子句的位置：**在给出ORDER BY子句时，应该保证它位于FROM子句之后。如果使用LIMIT，它必须位于ORDER BY之后。**使用子句的次序不对将产生错误消息**。



### 第6章 过滤数据

#### 6.1 使用WHERE子句

- 在SELECT语句中，**数据根据WHERE子句中指定的搜索条件进行过滤**。WHERE子句在表明（FROM子句）之后给出；

- ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE prod_price = 2.50;
  ```

- **SQL过滤与应用过滤：**尽可能**使用SQL过滤**。如果在应用层面处理数据库工作将会极大地影响应用的性能，并且使所创建的应用完全不具备可伸缩性。此外，如果在客户机上过滤数据，服务器不得不通过网络发送多余的数据，这将导致网络带宽上的浪费；

- **WHERE子句的位置**：在同时使用ORDER BY和WHERE子句时，**应该让ORDER BY位于WHERE之后**，否则将会产生错误。

#### 6.2 WHERE子句操作符

- | 操 作 符 |       说 明        |
  | :------: | :----------------: |
  |    =     |        等于        |
  |    <>    |       不等于       |
  |    !=    |       不等于       |
  |    <     |        小于        |
  |    <=    |      小于等于      |
  |    >     |        大于        |
  |    >=    |      大于等于      |
  | BETWEEN  | 在指定的两个值之间 |

  **注意：**

  - MySQL在**执行匹配时默认不区分大小写**；
  - 字符串匹配时需要用**单引号限定字符串**。

- **范围值检查：**

  - ```mysql
    # 检索价格在5美元和10美元之间的所有产品
    SELECT prod_name, prod_price
    FROM products
    WHERE prod_price BETWEEN 5 AND 10;
    ```

  - 在使用BETWEEN时，必须指定两个值——所需范围的低端值和高端值。这**两个值必须用AND关键字分隔**。**BETEEN匹配范围中所有的值，包括指定的开始值和结束值**。

- **空值检查：**

  - SELECT语句中有一个**特殊的WHERE子句，可用来检查具有NULL值的列**。这个WHERE子句就是**IS NULL子句**；

  - ```mysql
    SELECT prod_name
    FROM products
    WHERE prod_price IS NULL;
    ```

  - **NULL与不匹配**：**在通过过滤选择出不具有特定值的行时，不会返回具有NULL值的行**。因为未知具有特殊的含义，数据库不知道它们是否匹配。因此，在过滤数据时，一定要验证返回数据中确实给出了被过滤列具有NULL的行。



### 第7章 数据过滤

#### 7.1 组合WHERE子句

- **MySQL允许给出多个WHERE子句**。这些**子句可以两种方式使用：以AND子句的方式或OR子句的方式使用**；

- ```mysql
  SELECT prod_id, prod_price, prod_name
  FROM products
  WHERE vend_id = 1003 AND prod_price <= 10;
  ```

- **计算次序：**WHERE可包含任意数目的AND和OR操作符，**SQL在处理OR操作符前，优先处理AND操作符**。因此，任何时候使用具有AND和OR操作符时的WHERE子句，都**应该使用圆括号明确地分组操作符**；

#### 7.2 IN操作符

- **IN操作符用来指定条件范围，范围中的每个条件都可以进行匹配**；

- ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id IN(1002, 1003, 1004)
  ORDER BY prod_name;
  ```

- 为什么要使用**IN操作符**？其**优点**具体如下：

  - 语法更清楚且更直观；
  - 计算的次序容易管理；
  - **IN操作符一般比OR操作符执行更快**；
  - IN的最大优点是**可以包含其他SELECT语句，使得能够更动态地建立WHERE子句**。

#### 7.3 NOT操作符

- **WHERE子句中的NOT操作符**只有一个功能：**否定它之后所跟的任何条件**；

- **MySQL支持使用NOT对IN、BETWEEN和EXISTS子句取反**；

- ```mysql
  # 在与IN操作符联合使用时，NOT使找出与条件列表不匹配的行非常简单
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id NOT IN(1002, 1003)
  ORDER BY prod_name;
  ```

  

### 第8章 用通配符进行过滤

#### 8.1 LIKE操作符

- **LIKE**指示MySQL，**后跟的搜索模式利用通配符匹配而不是直接相等匹配进行比较**；

- ```mysql
  SELECT prod_id, prod_name
  FROM products
  WHERE prod_name LIKE 'jet%';
  ```

- **通配符可在搜索模式中任意位置使用，并且可以使用多个通配符**；

- **百分号（%）**通配符：

  - 在搜索串中，**%表示任何字符出现任意次数（可以匹配0个字符）**；
  - **尾空格可能会干扰通配符匹配**，可在搜索模式最后附加一个%，更好是使用函数去掉首尾空格；
  - **%通配符不能匹配NULL**。

- **下划线（_）**通配符：**下划线只匹配单个字符而不是多个字符**。

#### 8.2 使用通配符的技巧

- **不要过度使用通配符**。如果其他操作符能达到相同的目的，应该使用其他操作符；
- 在确实需要使用通配符时，**除非绝对有必要，否则不要把它们用在搜索模式的开始处**。把通配符置于搜索模式的开始处，搜索起来是最慢的；
- **仔细注意通配符的位置**。如果放错地方，可能不会返回想要的数据。



### 第10章 创建计算字段

#### 10.1 计算字段

- **在数据库中检索出转换、计算或格式化过的数据称为计算字段**；
- 可在SQL语句内完成的转换和格式化工作也可以在应用程序内完成。但**在数据库服务器上直接完成这些操作要更快**。

#### 10.2 拼接字段

- 在MySQL的SELECT语句中，可**使用Concat()函数来拼接不同的列和值**；

- ```mysql
  SELECT Concat(vend_name, '(', vend_country, ')')
  FROM vendors
  ORDER BY vend_name;
  ```

#### 10.3 使用别名

- **别名是一个字段或值的替换名。别名用AS关键字赋予**；

- ```mysql
  SELECT Concat(vend_name, '(', vend_country, ')') AS
  vend_title
  FROM vendors
  ORDER BY vend_name;
  ```

- **别名的用途**：

  - **计算字段是未命名的列，不能被客户机应用引用，因此需要别名**；
  - 实际的表列名包含不符合规定的字符（如空格）时重新命名它；
  - 在原来的名字含混或容易误解时利用别名来扩充它。

#### 10.4 执行算术计算

- **计算字段的另一常见用途是对检索出的数据进行算术计算**；

- ```MYSQL
  SELECT prod_id, 
  quantity, 
  item_price,
  quantity*item_price AS expanded_price
  FROM orderitems
  WHERE order_num = 20005;
  ```

- MySQL支持基本算术运算符：+、-、*、/。此外，圆括号可用来区分优先顺序。



### 第12章 汇总数据

#### 12.1 聚集函数

- **聚集函数：运行在行组上，计算和返回单个值的函数**；

- | 函  数  |      说  明      |
  | :-----: | :--------------: |
  |  AVG()  | 返回某列的平均数 |
  | COUNT() |  返回某列的行数  |
  |  MAX()  | 返回某列的最大值 |
  |  MIN()  | 返回某列的最小值 |
  |  SUM()  |  返回某列值之和  |

- **COUNT()函数**有**两种使用方式**：

  - 使用**COUNT(*)对表中行的数目进行计数**，**不管表列中包含的是空值（NULL）还是非空值**；
  - 使用**COUNT(column)对特定列中具有值的行进行计数，忽略NULL值**。

- ```mysql
  SELECT COUNT(*) AS num_cust
  FROM customers;
  ```

- **在多个列上计算**：利用标准的算术操作符，所有聚集函数都可用来执行多个列上的计算。

#### 12.2 聚集不同值

- 聚集函数都可以如下使用：

  - 对所有的行执行计算，指定ALL参数或不给参数（因为ALL是默认行为）；
  - **只包含不同的值，指定DISTINCT参数**。

- ```mysql
  # 使用了DISTINCT参数，因此平均值只考虑各个不同的价格
  SELECT AVG(DISTINCT prod_price) AS avg_price
  FROM products
  WHERE vend_id = 1003;
  ```

#### 12.3 组合聚集函数

- ```mysql
  # SELECT语句可根据需要包含多个聚集函数
  SELECT COUNT(*) AS num_items,
  			 MIN(prod_price) AS price_min
  FROM products;
  ```

  

