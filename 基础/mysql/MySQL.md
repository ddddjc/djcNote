# MySQL

## 1. 语法：

### 1.检索不同的行（DISTINCT）：

- 只返回不同的行，必须放在列名前

- 例

    ```mysql
    SELECT DISTINCT name,password from user
    ```

### 2.限制结果（LIMIT）:

- LIMIT n :返回不多于n行

- LIMIT m，n ：m是开始的位置，n是检索的行数

    =>LIMIT n OFFSET m;

### 3.完全限定表名

- ```mysql
    SELECT user.username from djc.user
    ```

### 4.排序（ORDER BY)

- ```mysql
    select price from product order by price limit 1;
    ```

### 5.过滤数据（where）

| 操作符  | 说明           |
| ------- | -------------- |
| =       | 等于           |
| <>      | 不等于         |
| !=      | 不等于         |
| <       | 小于           |
| <=      | 小于等于       |
| >       | 大于           |
| >=      | 大于等于       |
| BETWEEN | 在指定两个之间 |
|         |                |

``` mysql
SELECT * FROM user where age between 8 and 18 ORDER BY age LIMIT 3;
```

#### 5.1.AND

```mysql
SELECT * FROM user where age between 8 and 18 AND age<16 ORDER BY age desc LIMIT 3;
```

#### 5.2.OR

```mysql
SELECT * FROM user where age between 8 and 18 or age>24 ORDER BY age desc LIMIT 3;
```

- **优先计算and，后or**，使用时最好加括号

#### 5.4.IN

- in （   ,    ）
- 可包含其他的select语句

#### 5.5.NOT

```mysql
SELECT * FROM user where age not between 8 and 18 or not age>24 ORDER BY age;
```

### 6.用通配符过滤

- %

- 任何字符，任意次数(不能匹配null)

    ```mysql
    SELECT * FROM user where addr like "%xxx%";
                                        "a%b";
    ```

- _

- 单个字符

### 7.正则表达式

### 8.拼接字段（Concat（））

- **多少DBMS使用+或||实现拼接，MySQL使用Concat（）**

- ```mysql
    SELECT CONCAT(name,'有',age,'岁') from user;
    ```

    - **RTrim() ,  LTrim() ,Trim()  (去掉右，左，两边空格)**

    - ```mysql
        SELECT CONCAT(Rtrim(name),'有',Trim(age),'岁') from user;
        ```

- 别名

- ```mysql
    SELECT CONCAT(name,'有',age,'岁') AS nameAndAge from user;
    ```

### 8.执行算术计算

- ```mysql
    SELECT age,age+10 AS Ageyears from user;
    ```

### 9.函数

#### 1.Concat（）、R/LTrim（）

#### 2.文本处理函数

  ```mysql
select UPPER(user.username) AS big FROM djc.user
  ```

| 函数             | 说明              |
| ---------------- | ----------------- |
| Left（str，len） | 返回串右边字符    |
| Length（）       | 返回长度          |
| Locate（）       | 找出串的一个子串  |
| Lower（）        | 将串转换成小写    |
| LTrim（）        | 去掉串左侧空格    |
| Right（）        | 返回串右边字符    |
| RTrim（）        | 去掉串右边空格    |
| Soundex（）      | 返回串的SOUNDEX值 |
| SubString（）    | 返回子串的字符    |
| Upper（）        | 将串z换为大写     |

#### 3.时间处理函数

| 函数         | 说明                         |
| ------------ | ---------------------------- |
| AddDate（）  | 增加一个日期（天、周等）     |
| AddTime（）  | 再加一个时间（时、分等）     |
| CurDate（）  | 返回当前日期                 |
| CurTime（）  | 返回当前时间                 |
| DateDiff（） | 计算两个日期之差             |
| Date（）     | 返回日期时间的日期部分       |
| Date_Add()   | 高度灵活的日期运算函数       |
| Date_Format  | 返回一个格式化的日期或时间串 |
| Day（）      | 返回一个日期的天数部分       |
| DAYOfWeek()  | 对于一个日期，返回对应星期几 |
| Hour（）     | 返回一个时间的小时部分       |
| Minute（）   | 返回一个时间的分钟部分       |
| Month（）    | 返回一个日期的月份部分       |
| Now（）      | 返回当前日期的时间           |
| Second（）   | 返回一个时间的秒部分         |
| Time（）     | 返回一个日期时间的时间部分   |
| Year（）     | 返回一个时间的年部分         |

#### 4.数值处理函数

- 是DBMS的函数中最一致统一的函数

    | 函数     | 说明                 |
    | -------- | -------------------- |
    | Abs（）  | 返回一个数的绝对值   |
    | Cos（）  | 返回一个角度的余弦值 |
    | Exp（）  | 返回一个数的指数值   |
    | Mod（）  | 返回一个操作数的余数 |
    | Pi（）   | 返回圆周率           |
    | Rand（） | 返回一个随机数       |
    | Sin（）  | 返回一个角度的正弦   |
    | Sqrt（） | 返回一个数的平方根   |
    | Tan（）  | 返回一个角度的正切   |

### 10.汇总数据

- 聚集函数

    | 函数                                     | 说明             |
    | ---------------------------------------- | ---------------- |
    | AVG（）//忽略null行                      | 返回某列的平均值 |
    | COUNT（） //（*）所有 （column）忽略null | 返回某列的行数   |
    | MAX（）                                  | 返回某列的最大值 |
    | MIN（）                                  | 返回某列的最小值 |
    | SUM（）                                  | 返回某列值之和   |

- 聚集不同值

    ALL：默认 **AVG（ALL   price）**

    DISTINCT：**AVG（DISTINCT  price）**

- ```sql
    SELECT sum(price) as sum ,max(price) as max ,AVG(DISTINCT price) AS avg from product;
    ```

### 11.数据分组

- GROUP BY

- ```sql
    select DISTINCT username, COUNT(*) from buyed GROUP BY username, id;
    ```

    必须在where后，ORDER BY前

- 过滤分组 HAVING

- **WHERE过滤行，HAVING过滤分组**很多地方可替代WHERE

- ```sql
    select id, COUNT(*) as nums from buyed GROUP BY id HAVING nums>1;
    ```

- ```sql
    select id, SUM(num) as nums from buyed where YEAR(date)=2022 GROUP BY id HAVING nums>1;
    ```

- 分组和排序

    ```sql
    select id, SUM(num) as nums from buyed where YEAR(date)>=2022 GROUP BY id HAVING nums>1 ORDER BY nums DESC;
    ```

| 子句     | 说明               | 是否必须使用             |
| -------- | ------------------ | ------------------------ |
| SELECT   | 要返回的列或表达式 | 是                       |
| FROM     | 从中检索数据的列表 | 仅在从列表选择数据时使用 |
| WHERE    | 行级过滤           | 否                       |
| GROUP BY | 分组说明           | 仅在按组计算聚集时使用   |
| HAVING   | 组级过滤           | 否                       |
| ORDER BY | 输出排序顺序       | 否                       |
| LIMIT    | 要检索的行数       | 否                       |

### 12.子查询

- 利用子查询过滤

```sql
SELECT * from product WHERE id in (select id as nums from buyed where YEAR(date)>=2022                                    GROUP BY id HAVING nums>1 ORDER BY nums DESC)
```

- 作为计算字段使用

    ```sql
    SELECT name,
    			(SELECT COUNT(*) from buyed where buyed.username=user.username) as ord
    			from user
    ```

### 13.连结表

- ```sql
    select name,id,num from user,buyed where user.username=buyed.username
    ```

- 不写where。。。会返回笛卡尔积

- 内部链接（等值联结）

    - ```sql
        select name,id,num from buyed INNER JOIN user on buyed.username=user.username
        ```

    - 规范首选**INNER JOIN**，**使用明确的联结语法能够确保不会忘记联结条件**

- **创建高级联结**

    1. 自连接 

        ```sql
        select b1.username,b1.num from buyed as b1,buyed as b2 where b1.bid=b2.bid and b2.id='111';
        ```

    2. 自然联结

        排除多次出现某一列，每一列只返回一次

        - 一个通配符（*），其他注明

        - ```sql
            select b1.*,u.name,p.pname from buyed as b1,user as u,product as p
            where b1.id=p.id and b1.username=u.username;
            ```

    3. 外部联结

        包含没有关联行的那些行

        - ```sql
            select buyed.*,product.pname from product left outer join buyed
            on buyed.id=product.id;
            ```

        - **right/left指包含右/左边表的所有行（以那边为主体）**

    4. 使用带聚合函数的联结

        ```sql
        select pname,COUNT(buyed.id) from product left outer join buyed
        on buyed.id=product.id
        group by product.id;
        ```


### 14.组合查询

- select语句之间用UNION分隔
- **UNION ALL包含重复的行**
- 只能使用一条ORDER BY 在最后一条select语句里

### 15.全文搜索

### 16.插入数据

### 17.更新和删除数据

- **IGNORE** 更新多行时，若某一行或多行发生错误，使其忽略错误继续更新
- ​                （原本会取消操作，错误发生前更新的行恢复）
- delete不加限制删除所有行
- **TRUNCATE table** 比delete更快，（实际上是删除原来的表并重新创建）

### 18.创建表和操纵表

- 多个列作为主键
    - 例如：PRIMARY KEY（计算机2102，18号）
    - 这两个列组合唯一
- last_insert_id() 获取最后插入的ATTO_INCREMENT
- DEFAULT **   **为默认值
- 引擎类型（ENGINE)
    - MySQL有多个引擎
    - 不声明这多少会使用MyISAM,
    - InnoDB是一个可靠的事务处理引擎，但不支持全文本搜索
    - MEMORY在功能等同于MyISAM，但是由于数据存储在内存（不是磁盘）中，速度很快（特别适合于临时表）；
    - MyISAM是一个性能极高的引擎，它支持全文本搜索，不支持事务处理
    - 可以混用，但外键不能跨引擎
- 更新表（ALTER：改变） 常用于定义外键
    - 
- 删除表：DROP TABLE name
- 重命名表 RENAME TABLE name TO name；

### 19.视图

1. 视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。

2. 不包含表中应有的任何列或数据，包含的是SQL查询

3. 应用：

    - 重用SQL语句
    - 简化SQL操作而不必知道他的基本查询细节
    - 使用表的组成部分而不是这个表
    - 保护数据
    - 更改数据格式和表示

4. **多个联结和过滤创建了复杂的视图或者嵌套了视图，可能会发现性能下降得很厉害。**

5. 语法：

    - CREATE VIEW 语句创建

    - SHOW CREATE VIEW viewname；查看

    - DROP 删除视图  DROP VIEW viewname；

    - 更新视图时，可以先用DROP再用CREATE，也可以直接用CREATE OR REPLACE VIEW。

    - ```SQL
        SELECT product.id,pname,num
        from product,buyed
        where product.id=buyed.id
        ```

6. 更新一个视图将更新其基表（一般用于检索，不更新，许多情况不能更新，如分组，联结。。。）

### 20.使用存储过程（类型封装函数）

- **在控制台‘；’后回车会直接执行，造成错误，使用*DELIMTER*（分号），**

- ```sql
    DELIMITER "符号" （类似于起别名，回车不会运行）
    ```

- ```sql
    CREATE PROCEDURE prod()
    BEGIN
      SELECT AVG(product.price) as avg
    	from product;
    	SELECT * from buyed;
    END;
    ```

- ```sql
    CALL prod();
    ```

- ```sql
    DROP PROCEDURE prod     !!!不要'()
    ```

- **使用参数** ！！！！

    - IN 只可以读取变量，不会改

    - OUT 只能改，不能读

    - INTO 均可

        ```sql
        DROP PROCEDURE pro
        CREATE PROCEDURE pro(
            out p1 DECIMAL(8,2),
        		in p2 int,
        		INOUT p3 DOUBLE)
        BEGIN
            select p2
        		into p1;
        		select p3*p3
        		into p3;
        		SELECT 0
        		into p2;
        end;
        SET @pp1=0;
        set @pp2=3;
        SET @pp3=7;
        call pro(@pp1,@pp2,@pp3);
        SELECT @pp1;
        SELECT @pp2;
        SELECT @pp3;
        ```

        

- **检查存储过程**

    - ```sql
        SHOW CREATE PROCEDURE name；
        ```

    - ```sql
        show PROCEDURE status like 'prod'
        ```

### 21.游标！！！

- 是一种结果集

### 22.触发器

- 

