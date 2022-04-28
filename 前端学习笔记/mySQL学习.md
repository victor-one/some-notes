# 01 数据库的基础知识

## 数据库的概念

**数据库：**database（DB），是一种存储数据的仓库。具有如下特性：

* 数据库是根据数据结构组织、存储和管理数据。
* 数据库能够长期、高效的管理和存储数据。
* 数据库的目的就是能够存储（写）和提供（读）数据。

## 数据库分类

数据库分为两类：

* **关系型数据库：**把复杂的数据结构归结为简单的二元关系，即二维表格形式（二维表）。注重数据存储的持久性。
* **非关系型数据库：**没有具体模型的数据结构。英文简称 NoSQL （Not Only SQL），意为“不仅仅是SQL”。注重数据读取的效率。

### 1、关系型数据库

**关系型数据库**：把复杂的数据结构归结为简单的二元关系，即二维表格形式（二维表）。

关系型数据库有四层结构：

- 数据库管理系统（DBMS）：DataBase Management System。
- 数据库（DB）：数据存储的管理者。
- 数据表（Table）：数据关系管理者。
- 数据字段（Field）：实际数据存储者。

常见的关系型数据库产品：

- 大型：Oracle
- 中型：MySQL、SQL Server
- 小型：Sybase、Access

### 2、非关系型数据库

**非关系型数据库**：没有具体模型的数据结构。英文简称 NoSQL（Not Only SQL )，意为"不仅仅是SQL"。

常见的非关系型数据库产品：MongoDB、Redis、Memcached。

## SQL 的介绍

**SQL**：全称 **Structured Query Language**，译为**结构化查询语言**。

**SQL**：是一种针对关系型数据库的标准化编程语言，能够实现用户数据库的查询和程序设计。

通俗来讲，**SQL 是关系型数据库的操作指令**。

根据操作类型不同，SQL 可分为几类：

- DQL：Data Query Language，数据查询语言，用于查询和检索数据
- DML：Data Manipulation Language，数据操作语言，用于数据的写操作（增删改）
- DDL：Data Definition Language，数据定义语言，用于创建数据结构
- DCL：Data Control Language，数据控制语言，用于用户权限管理
- TPL：Transaction Process Language，事务处理语言，辅助DML进行事务操作（因此也归属于DML）

补充：

- SQL 虽然是编程语言，但通常只用来进行数据管理，逻辑部分交给其他编程语言。
- SQL 是针对关系型数据库的**通用语言**，所有关系型数据库都是基于SQL进行数据操作；而不同的数据库产品，在 SQL 操作指令上略有差异。

## MySQL 的介绍

### MySQL 访问原理

MySQL是一种C/S结构的软件，因此我们需要安装 MySQL 的客户端来访问远程的服务端。也就是说，数据是存放在服务器上的，客户端通过执行 sql 指令来操作服务端的数据。

具体步骤是：

（1）客户端通过 主机（host） + 端口号（port） 服务端。

（2）输入 username 和 password 验证身份。

（3）客户端和服务端连接成功，通过 sql 指令开始操作数据库。

# 02 MySQL的安装和Navicat使用

## MySQL安装

详情操作见：[MySQL安装教程](https://blog.csdn.net/qq_39652397/article/details/124022739)

## Navicat 使用

详情操作见：[ Navicat 使用教程](https://blog.csdn.net/qq_45069279/article/details/105919312)

# 03 MySQL的基本操作

## SQL 的一些简单语法规则

### 结束符

SQL 指令需要语句结束符，默认是英文分号`;`。

当然，还有另外两个结束符：

- `\g` 与英文分号`;`等效。
- `\G`：将查到的结构旋转90度变成纵向。

### 反引号``

SQL语句中如果用到了关键字或者保留字，需要使用反引号``（Tab键上面的符号）来包裹，让系统忽略。

## MySQL 数据库的操作分类

根据数据库的对象层级，可以将**SQL的基础操作分为四类**：

- 数据库（DB）操作。
- 数据表（Table）操作。
- 数据字段（Field）操作。
- 数据操作。

## 一、数据库（DB）的基本操作

在终端的任何位置，输入如下命令，即可进入 mysql 命令的执行窗口：

```shell
mysql -u root -p
```

### 1、创建数据库

**语法格式**：

```mysql
create database 数据库名称 [数据库选项];
```

**数据库名称的命名规范**：

- 由数字、字母和下划线组成。
- 不区分大小写。
- 不能以数字开头。
- 建议使用下划线法创建复杂的数据库名字。比如 `db_qianguyihao`。

**举例**：

创建一个名为 db_qianguyihao1 的数据库：

```mysql
create database db_qianguyihao1;
```

创建一个指定字符集的数据库：

```mysql
create database db_qianguyihao2 charset utf8MB4;
```

创建一个指定校对集的数据库：

```mysql
create database db_qianguyihao3 charset utf8MB4 collate utf8mb4_general_ci
```

### 2、查看数据库

查看有哪些数据库：(显示所有的数据库列表)

```mysql
show databases;
```

查看 `db_qianguyihao1` 这个数据库的具体创建指令是怎样的：

```mysql
show create database db_qianguyihao1;
```

备注：由于系统会加工，所以看到的结果不一定是真实的创建指令。

### 3、使用指定的数据库

使用指定的数据库：（也可以理解成：进入指定的数据库）

```mysql
# 语法格式
use database_xxx;

# 举例
use db_qianguyihao;
```

假设当前服务器连接中有很多个数据库（db_qianguyihao1、db_qianguyihao2），此时，我输入 `use db_qianguyihao2`则代表我想使用 `db_qianguyihao2` 这个数据库。

### 4、修改数据库的参数

我们一般很少修改数据库的名称，一般是去修改数据库的一些选项，比如：

- 修改字符集
- 修改校对集

**语法格式**：

```mysql
alter database 数据库名称 [库选项]
```

举例1、修改数据库的字符集为gbk：

```mysql
alter database db_qianguyihao1 charset gbk;
```

举例2、修改数据库的校对集：

```mysql
alter database db_qianguyihao2 charset gbk collate gbk_chinese_ci;
```

备注：因为校对集是和字符集有关的，所以上方指令是在修改字符集的同时，修改校对集。

### 5、删除指定的数据库

**语法格式**：

```mysql
drop database 数据库名称;
```

备注：删除数据库时，会清空当前数据库里的所有数据表，所以删除数据库的操作一定要谨慎。

## 二、数据表（Table）的基本操作

注意，我们最好先通过 `use xxx_database` 命令进入指定的数据库（DB），然后在当前数据库下，进行数据表（Table）的操作。

### 1、创建数据表

**语法格式**：

```mysql
create table [数据库名].[表名] (
	字段名1 字段类型,
	...
    ...
    字段名2 字段类型
) 表选项;
```

**举例**：

1、在当前数据库中创建数据表 `table_qiangu1`，并新增**主键** id 字段：

```mysql
CREATE TABLE table_qiangu1 (
  id int NOT NULL AUTO_INCREMENT PRIMARY KEY
);
```

2、在当前数据库中创建数据表 `t_student1`，并新增 name、age这连个字段：

```mysql
create table t_student1(
    name varchar(255),
    age int
);
```

3、在指定的数据库 `db_2` 中创建数据表 `t_student2`：

```mysql
create table db_2.t_student2(
    name varchar(255),
    age int
);
```

4、在当前数据库中创建数据表 `t_student3`（含表选项）：

```mysql
create table t_student3(
    name varchar(255),
    age int
)engine Innodb charset utf8MB4;
```

举例4中的代码涉及到存储引擎，这里解释一下：

**存储引擎**是指数据存储和管理的方式，MySQL中提供了多种存储引擎，一般使用默认存储引擎 InnoDB。

- InnoDB：默认存储引擎；支持事务处理和外键；数据统一管理。
- MyIsam：不支持事务和外键；数据、表结构、索引独立管理；MySQL5.6以后不再维护。

6、扩展：如果想创建一个与已有表一样的数据表，MySQL提供了一种便捷的复制模式

### 2、复制数据表

如果想创建一个与已有表一样的数据表，MySQL提供了一种便捷的**复制**模式。

**语法格式**：（复制现有的表 `table_xx1` 到 `table_xx2`）

```mysql
create table table_xx1 like 数据库名.table_xx2;
```

注意，这种复制模式，`table_xx2` 只会复制表 `table_xx1` 中的字段，不会复制表`table_xx1`中的数据。

**举例**：

```mysql
# 在当前数据库下，复制现有的表`t_qianguyihao1` 到表 `t_qianguyihao2`
create table t_qianguyihao1 like t_qianguyihao2;

# 复制现有的表`t_qianguyihao1` 到表 `t_qianguyihao2`，是复制到 `db2`这个数据库中
create table t_qianguyihao1 like db2.t_qianguyihao2;
```

### 3、显示数据表的名称

在当前数据库下，显示所有的数据表：

```mysql
show tables;
```

在指定的数据库中，显示所有的数据表：

```mysql
show tables from db_qianguyihao1;
```

显示数据表的创建指令：(查看 `t_qianguyihao1` 这个数据表的具体创建指令是怎样的)

```mysql
show create table t_qianguyihao1; # 备注：由于系统会加工，所以看到的结果不一定是真实的创建指令。
```

### 4、查询（查找）数据表的名称

> 根据 表名称 查询数据表，也可以理解成：按条件显示部分数据表。

根据数据表的**表名称**查找数据表时，需要用到关键词`like`，而且还要涉及到两个符号：

- `%` 表示匹配任意**多个字符**。
- `_` 表示匹配任意**一个字符**（固定位置）。

上面这两个模糊查询的符号，大家要牢记。我们来看看具体的例子。

语法举例：

```mysql
show tables like '%like_';	# _表示匹配一个字符（固定位置），%表示匹配N个字符
```

**`%` 符号举例**：

```mysql
# 查询表名称中，包含 “qiangu” 这个关键字的表（“qiangu”这个关键字的前后可能都有内容）
show tables like '%qiangu%';

# 查询表名称以“qiangu”开头的表（这个命令应该很实用）
show tables like 'qiangu%';
```

**`_`符号举例**：

```mysql
# 根据 表名称 来查询表，查询条件是：表名称以“qiangu”开头，而且要确保 qiangu 的后面有三个字符（因为我在 qiangu 的后面写了三个下划线）。
show tables like 'qiangu___';
```

### 5、desc：查看数据表的表结构

查看数据表的表结构，就是**查看这张表中定义了哪些字段**，以及这些字段是如何定义的。通过这种方式，我们可以清晰地了解数据的存储形式。

项目开发中，领导在检查我们的工作时，首先看的就是我们的表中定义了哪些字段。所以说，这种方式，还是很实用的。

**语法格式**：

```mysql
# 方式1
desc 表名称;

# 方式2
describe 表名称;

# 方式3
show columns from 表名称;
```

上面的三种方式，效果都一样，三选一即可。

### 6、修改数据表的表名称和表选项

**修改数据表的表名称**：

在当前数据库下，修改数据表的表名称：

```mysql
rename table 原表名 to 新表名;
```

指定某个数据库，然后修改数据表的表名称：

```mysql
rename table 数据库名.原表名 to 数据库名.新表名;
```

**修改数据表**：

```mysql
alter table table1 charset gbk;
```

### 7、删除数据表

语法格式：

```mysql
drop table 数据表名称;
```

## 三、字段（Field）的基本操作

数据表 table 创建好了之后，我们就可以开始在这张表中新增字段了。

### 1、新增字段

**语法格式**：

```mysql
alter table 表名 add [column] 字段名 字段类型 [字段属性] [字段位置];
```

注意事项：

- 新增字段时，必须制指定字段类型。
- [column]、 [字段属性]、[字段位置] 这几个都是选填，其他是必填。
- 追加字段时，这个字段的顺序默认排在最后。

**举例**：

新增字段 `name`:

```mysql
alter table table_qiangu1 add name varchar(255);
```

新增字段 `age`：

```mysql
alter table table_qiangu1 add age int;
```

### 2、新增字段时，设置字段的位置（顺序）

在新增字段时，它的顺序是默认放在最后面的，当然，我们也可以人工指定它的顺序。

在修改字段的位置时，我们可以用到下面这两个关键字：

- `first` 放到最前面
- `after` 放到某个字段的后面

**语法格式**：

```mysql
alter table 表名 add 新字段名 字段类型 字段位置;
```

**举例1**：

在 `name`字段的后面，新增一个 `sex` 字段：

```mysql
alter table t_qiangu1 add sex varchar(255) default null comment '性别' after name;
```

注意，上方举例中，如果是新建 varchar 类型的字段，一定要指定 varchar 的长度（比如255），否则报错。

**举例2**：

新增一个 `id` 字段，放到最前面：

```mysql
alter table t_qiangu1 add id int first;
```

### 3、change：修改现有字段的字段名

> 修改现有字段的字段名，是通过 change 关键字，不是通过 modify 关键字（后者会报错，执行失败）。

**语法格式**：

```mysql
# 格式1（精简版）
alter table 表名 change 原字段名 新字段名 字段类型;

# 格式2（完整版）
alter table 表名 change 原字段名 新字段名 字段类型 [字段属性] [位置];
```

注意：

- 修改字段名时，一定要设置新字段的字段类型。
- 虽然 change 关键字也可以修改现有字段的字段属性、字段位置，但我们一般是通过 modify 关键字来做（下面会讲）。

**举例**：

修改字段名 `sex` 为 `sexy`：

```mysql
alter table t_qiangu2 change sex sexy varchar(255);
```

### 4、modify：修改现有字段的的字段类型、字段位置、字段属性

**语法格式**：

```mysql
alter table 表名 modify 现有字段的字段名 现有字段的字段类型 [字段属性] [位置]；
```

**举例1**、针对现有的字段 `name` 和 `age`，更换这两个字段的顺序：

```mysql
# 注意，这里的 age 后面一定要跟上它的字段类型，否则执行失败
alter table t_qiangu1 modify age int after name;
```

**修改字段的默认值**

```mysql
# 若本身存在默认值，则先删除
alter table 表名 alter column 字段名 drop default;

# 若本身不存在则可以直接设定
alter table 表名 alter column 字段名 set default 默认值;
```

### 5、删除字段

> 删除字段的同时，会删除字段对应的数据。删除字段的操作不可逆，请谨慎操作。

语法格式：

```mysql
alter table 表名 drop 字段名;
```

举例：（删除 t_qiangu1 表中的 age 这个字段）

```mysql
alter table t_qiangu1 drop age;
```

## 四、数据的基本操作

### 1、新增数据

**方式1、全字段插入**：

语法格式：

```mysql
insert into 表名 values(值1, 值2, ... 最后一个值);
```

解释：

- 值的顺序必须与所有字段的顺序一致。
- 值的数据类型也必须与字段定义的数据类型一致。

举例（给 t_qiangu1 这个表中插入一条完整的数据）：

```mysql
insert into t_qiangu1 values(3, 'qianguyihao', 28);
```

**方式2、部分字段插入**：

语法格式：

```mysql
insert into 表名 (字段1, 字段2, 字段3) values(值1, 值2, 值3);
```

解释：

-字段的顺序可以随意，但值的顺序必须要与前面的字段顺序**一一对应**，数据类型也要一致。

举例（给 t_qiangu1 这个表中的指定字段插入数据）：

```mysql
insert into t_qiangu1 (id, name) values(4, 'xusong');
```

### 2、查询数据

> 查询数据的操作，占sql日常操作的95%以上。

**语法格式**：

```mysql
select xxx from 表名;
```

**举例**：

查询表中的所有数据：

```mysql
select * from t_qiangu1;
```

查询表中 name、age 这两个字段的数据：

```mysql
select name, age from t_qiangu2;
```

查询表中 id=2 的数据：

```mysql
select * from t_qiangu3 where id = 2;
```

### 3、修改数据

**语法格式**：

```mysql
update 表名 set (字段1 = 新值1, 字段2 = 新值2) [where 条件筛选];
```

**解释**：

- 我们通常是结合 where 条件语句来修改数据。
- **修改数据之前，要先保证表里面有数据**。如果这张表是空表，那么，执行这个命令后，等于没执行。

**举例**：

将表中，name 这个字段的值全部修改为`千古壹号`：

```mysql
update t_qiangu1 set name = '千古壹号';
```

id = 3 的这条记录中，修改 name 和 age 这两个字段的值：

```mysql
update t_qiangu1 set name = '许嵩', age = '34' where id = 3;
```

### 4、删除数据

> 删除字段的操作不可逆，请谨慎操作。

**语法格式**：

```mysql
delete from 表名 [where 条件];
```

**解释**：

- 执行删除操作之后，匹配到的**整条记录**，都会删除。
- **删除数据之前，要先保证表里面有数据**。如果这张表是空表，那么，执行这个命令后，等于没执行。

**举例**：

删除表中`id = 2`的记录：

```mysql
delete from t_qiangu1 where id = 2;
```

# 04 MySQL字段的数据类型

MySQL 中的字段，主要有四种数据类型：

- 整型（整数）
- 小数
- 字符串类型
- 时间日期类型

## 整数类型

### 整数类型的分类

MySQL中，整型有五种：

- 迷你整型：tinyint，使用**1个字节**存储整数，最多存储256个整数（-128~127）。
- 短整型：smallint，使用**2个字节**存储整数。
- 中整型：mediumint，使用**3个字节**存储整数。
- 标准整型：int，使用**4个字节**存储整数。
- 大整型：bigint，使用**8个字节**存储。

**强调**：

（1）如无特殊情况使用`int`即可。

（2）整型在 MySQL 中默认是有符号的，即有正负；无符号需要使用 `unsigned` 修饰整型，代表正整数。

**举例**：

在指定的表中新增 age 字段，要求 age 是正整数：

```mysql
alter table table_qiangu1 add age int unsigned;
```

### 设计思路

1. 确定需要存储的数据是整数；
2. 预估整数的范围，选择合适的整数类型；
3. 确定是否包含负数。

### 整数类型的取值范围

查看[此处](https://blog.csdn.net/slyjit/article/details/54290486)

### 整数类型的显示宽度、零填充

> 我们在很多设计表中，可能会看到比如 `int(11)`这种数据类型，这里面的 `11`代表的就是`显示宽度`。

所谓的**显示宽度**，其实就是显示的时候，看到的**最少**数字个数。如果没有满足显示宽度，做**零填充**；如果超过了显示宽度，则直接显示原始值，不会做**零填充**。

**显示宽度的注意事项**：

- 显示宽度只适用于 MySQL 的整数类型。
- 显示宽度只是指明 MySQL 整数类型最少显示的数字个数（可以通过desc查看表字段显示）。
- **显示宽度只是在显示的时候改变数值的样式，不会对原本的值进行更改**。
- 显示宽度和数值类型的取值范围无关。例如int(10) 他的取值范围依然是(-2 147 483 648，2 147 483 647)。

**零填充的注意事项**：

- 要想让显示宽度自动进行**零填充**，必须要配合 `ZEROFILL`这个关键字一起使用。
- **零填充只能针对正整数**，也就是说，`ZEROFILL` 要求整型为无符号型。

**举例**：

1、新建一张表，然后在这张表中新增 num1 字段，要求 num1 显示3位，不够3位的自动进行零填充：

```mysql
# 新建一张表
CREATE TABLE table_qiangu1 (
  id int NOT NULL AUTO_INCREMENT PRIMARY KEY
);

# 显示宽度有效（正确写法）
alter table table_qiangu1 add num1 int(3) zerofill;

# 对比：普通写法，显示宽度无效
alter table table_qiangu1 add num2 int(3);

# 对比：普通写法
alter table table_qiangu1 add num3 int;
```

上述命令中，如果把 `zerofill` 这个关键字去掉，是达不到显示宽度的效果的。

## 小数

MySQL 中的小数分为两大类：

浮点型的数据分为两种：

- 单精度：float，使用4个字节存储，精度范围为6-7位有效数字。
- 双精度：double，使用8个字节存储，精度范围为14-15位有效数字。

注意：

- 浮点数超过精度范围会自动进行四舍五入。
- 精度可以指定整数和小数部分。

**指定小数位数：：**

```mysql
# m必须大于n,m表示总位数，n表示小数点后的位数
alter table 表名 add 列名 float(m, n)
```

## 字符串类型

* 定长字符串：char，0~255
* 变长字符串：varchar，0~65535
* 短文本字符串：tinytext，2^8-1
* 文本字符串：text，2^16-1

## 时间日期类型

* 日期格式：date，YYYY-MM-DD
* 时间格式：time，HH: mm： ss
* 最常用的时间格式：datetime，YYYY-MM-DD HH: mm: ss
* 时间戳：timestamp，1970.1.1到现在的毫秒数
* 年份表示：year

# 05 MySQL数据库的常用命令

## MySQL的一些简单命令

**以 root 身份进入命令行**：

```shell
mysql -u root -p
```

**查看有哪些数据库**：

```mysql
show databases;
```

**选择进入指定的数据库**：

```mysql
use xxx_database;

# 举例
use qianguyihao_database;
```

**在当前数据库中，查看有哪些表**：

```mysql
show tables;
```

**在当前数据库中，查询指定表的全部数据**：

```mysql
SELECT * FROM xxx_table;

# 举例
SELECT * FROM qianguyihao_student_table
```

**删除指定的表**：

```mysql
drop table xxx;

# 举例
drop table qianguyihao_student_table;
```

**删除指定的数据库**：

```mysql
drop database qianguyihao_student_table;
```

**创建一个数据库**：

```mysql
create database qianguyihao_database2;
```

## where 条件查询

使用 `where` 子句可以对表中的数据进行筛选，结果为 true 的行会出现在查询结果中。

语法格式如下：

```mysql
SELECT * FROM 表名 where 条件;
```

上面的语法格式中，`条件` 具体要怎么写呢？这个可能有很多种情况。我们继续往下看。

### 比较运算符

- `=` 等于
- `>` 大于
- `>=` 大于等于
- `<` 小于
- `<=` 小于等于
- `!=`：不等于

**举例**：`age > 20`：查询 age 大于 20 的数据

```mysql
# 查询 age 大于 20 的数据
SELECT * FROM qianguyihao_table WHERE age > 20;
```

### 逻辑运算符

- AND
- OR
- NOT

**举例**：

```mysql
# 查询 age 在20至30之间的数据
SELECT * FROM qianguyihao_table WHERE age BETWEEN 20 AND 30;
```

### 范围查询

- `in` 表示在一个非连续的范围内。
- `between ... and ...` 表示在一个连续的范围内

举例：

```mysql
# 查询 name 为 千古壹号 或者 许嵩的数据
SELECT * FROM qianguyihao_table WHERE name IN ['千古壹号', '许嵩'];

SELECT * FROM qianguyihao_table WHERE age BETWEEN 20 AND 30;
```

### 模糊查询

- `like`
  - `%` 表示任意多个任意字符
  - `_` 表示一个任意字符

`%` 符号举例：

```mysql
# 查询标题中包含“前端”这两个字的数据（“前端”这两个字的前后可能都有内容）
select * from qianguyihao_table where `title` like "%前端%";

# 查询标题以“前端”开头的数据
select * from qianguyihao_table where `title` like "前端%";
```

`_`符号举例：

```mysql
# 查询标题，查询条件是：标题中有五个字符，而且，这五个字符中，前两个字符一定是“千古”开头的。
SELECT * FROM qianguyihao_table WHERE `title` LIKE "千古___";
```

### NULL 的判断

- `is null` 判断为空
- `is not null` 判断为非空

注意，`is null` 和**空字符串**`""` 是有区别的。学过 js 基础之后，你应该知道：空字符串并非 null，只不过是里面的值为空而已；空字符串也是会占有内存空间的。

举例：

```mysql
select * from qianguyihao_table where name is not NULL;
```

## join 联表查询

### 联表查询命令

- `tableA inner join tableB`：表 A 与表 B 匹配的行会出现在结果中。
- `tableA left join tableB`：表 A 与表 B 匹配的行会出现在结果中。表 A 中独有的数据，对应表 B 中用 null 填充。
- `tableA right join tableB`：表 A 与表 B 匹配的行会出现在结果中。表 B 中独有的数据，对应表 A 中用 null 填充。

### 举例

现在有下面这两张表：作者表 author、图书表 book。

**表 1**、作者表 author：

| id   | authorId | authorName |
| ---- | -------- | ---------- |
| 1    | 11       | 王小波     |
| 2    | 12       | 吴军       |
| 3    | 88       | 莫言       |

**表 2**、图书表 book：

| id   | bookId | bookName   | authorId |
| ---- | ------ | ---------- | -------- |
| 1    | 201    | 黄金时代   | 11       |
| 2    | 202    | 白银时代   | 11       |
| 3    | 203    | 青铜时代   | 11       |
| 4    | 204    | 浪潮之巅   | 12       |
| 5    | 205    | 硅谷之谜   | 12       |
| 6    | 206    | 数学之美   | 12       |
| 7    | 777    | 设计心理学 | 99       |

注意，表2中的每本图书都有对应的 authorId，这个 authorId 就是对应表1中的 authorId。**通过 authorId 把两张表关联起来**。

通过联表查询上面的两张表，我们来对比一下查询结果。

**情况 0**：（inner join）

```mysql
SELECT * FROM author INNER JOIN book;
```

查询结果：

| id | authorId | authorName | id | bookId | bookName   | authorId |
|---|---|---|---|---|---|---|
|  3 |       88 | 莫言       |  1 |    201 | 黄金时代   |       11 |
|  2 |       12 | 吴军       |  1 |    201 | 黄金时代   |       11 |
|  1 |       11 | 王小波     |  1 |    201 | 黄金时代   |       11 |
|  3 |       88 | 莫言       |  2 |    202 | 白银时代   |       11 |
|  2 |       12 | 吴军       |  2 |    202 | 白银时代   |       11 |
|  1 |       11 | 王小波     |  2 |    202 | 白银时代   |       11 |
|  3 |       88 | 莫言       |  3 |    203 | 青铜时代   |       11 |
|  2 |       12 | 吴军       |  3 |    203 | 青铜时代   |       11 |
|  1 |       11 | 王小波     |  3 |    203 | 青铜时代   |       11 |
|  3 |       88 | 莫言       |  4 |    204 | 浪潮之巅   |       12 |
|  2 |       12 | 吴军       |  4 |    204 | 浪潮之巅   |       12 |
|  1 |       11 | 王小波     |  4 |    204 | 浪潮之巅   |       12 |
|  3 |       88 | 莫言       |  5 |    205 | 硅谷之谜   |       12 |
|  2 |       12 | 吴军       |  5 |    205 | 硅谷之谜   |       12 |
|  1 |       11 | 王小波     |  5 |    205 | 硅谷之谜   |       12 |
|  3 |       88 | 莫言       |  6 |    206 | 数学之美   |       12 |
|  2 |       12 | 吴军       |  6 |    206 | 数学之美   |       12 |
|  1 |       11 | 王小波     |  6 |    206 | 数学之美   |       12 |
|  3 |       88 | 莫言       |  7 |    777 | 设计心理学 |       99 |
|  2 |       12 | 吴军       |  7 |    777 | 设计心理学 |       99 |
|  1 |       11 | 王小波     |  7 |    777 | 设计心理学 |       99 |

上面这种查询，没有意义，因为没有加任何查询条件。

**情况 1**：（inner join）

```mysql
SELECT * FROM author INNER JOIN book ON author.authorId = book.authorId;
```

查询结果：
| id | authorId | authorName | id | bookId | bookName | authorId |
|---|---|---|---|---|---|---|
|  1 |       11 | 王小波     |  1 |    201 | 黄金时代 |       11 |
|  1 |       11 | 王小波     |  2 |    202 | 白银时代 |       11 |
|  1 |       11 | 王小波     |  3 |    203 | 青铜时代 |       11 |
|  2 |       12 | 吴军       |  4 |    204 | 浪潮之巅 |       12 |
|  2 |       12 | 吴军       |  5 |    205 | 硅谷之谜 |       12 |
|  2 |       12 | 吴军       |  6 |    206 | 数学之美 |       12 |

上面这行命令，跟下面这行命令等价：

```mysql
SELECT * FROM author, book WHERE author.authorId = book.authorId;
```

**情况 2**：（left join）

```mysql
SELECT * FROM author LEFT JOIN book ON author.authorId = book.authorId;
```

查询结果：
| id | authorId | authorName | id   | bookId | bookName | authorId |
|---|---|---|---|---|---|---|
|  1 |       11 | 王小波     |    3 |    203 | 青铜时代 |       11 |
|  1 |       11 | 王小波     |    2 |    202 | 白银时代 |       11 |
|  1 |       11 | 王小波     |    1 |    201 | 黄金时代 |       11 |
|  2 |       12 | 吴军       |    6 |    206 | 数学之美 |       12 |
|  2 |       12 | 吴军       |    5 |    205 | 硅谷之谜 |       12 |
|  2 |       12 | 吴军       |    4 |    204 | 浪潮之巅 |       12 |
|  3 |       88 | 莫言       | NULL | NULL   | NULL     | NULL     |

**情况 3**：（right join）

```mysql
SELECT * FROM author RIGHT JOIN book ON author.authorId = book.authorId;
```

查询结果：
| id   | authorId | authorName | id | bookId | bookName   | authorId |
|------|----------|------------|----|--------|------------|----------|
|    1 |       11 | 王小波     |  1 |    201 | 黄金时代   |       11 |
|    1 |       11 | 王小波     |  2 |    202 | 白银时代   |       11 |
|    1 |       11 | 王小波     |  3 |    203 | 青铜时代   |       11 |
|    2 |       12 | 吴军       |  4 |    204 | 浪潮之巅   |       12 |
|    2 |       12 | 吴军       |  5 |    205 | 硅谷之谜   |       12 |
|    2 |       12 | 吴军       |  6 |    206 | 数学之美   |       12 |
| NULL | NULL     | NULL       |  7 |    777 | 设计心理学 |       99 |

## 自关联查询

涉及到层级关系时可以用自关联查询。

## 子查询

当一个查询结果是另一个查询的条件时，这个查询称之为子查询。

# MySQL设计三大范式

## MySQL 设计三大范式

### 第一范式（1NF）：原子性

表的每一列具有原子性，不可再分。

### 第二范式：唯一性

第二范式是建立在第一范式基础上的；外要求所有非主键字段必须完全依赖主键，而不是部分依赖。

### 第三范式

第三范式是建立在第二范式基础上的；且要求没有传递依赖。

## 参考链接

- [MySql--数据库设计三范式](https://www.jianshu.com/p/3e97c2a1687b)