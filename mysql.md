# union, or, union all
```sql
SELECT
    name, population, area
FROM
    world
WHERE
    area >= 3000000 OR population >= 25000000
;

SELECT
    name, population, area
FROM
    world
WHERE
    area >= 3000000

UNION

SELECT
    name, population, area
FROM
    world
WHERE
    population >= 25000000
;
```
Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序； 即：去重+排序

Union All：对两个结果集进行并集操作，包括重复行，不进行排序； 即：不去重+不排序

对于单列来说，用or是没有任何问题的，但是or涉及到多个列的时候，每次select只能选取一个index，如果选择了area，population就需要进行table-scan，即全部扫描一遍，但是使用union就可以解决这个问题，分别使用area和population上面的index进行查询。 但是这里还会有一个问题就是，UNION会对结果进行排序去重，可能会降低一些performance

mysql5.0以前是一张表只能使用一次索引，但5.1以后引入了index merge。会自动将多个or条件索引分别根据条件检索再union。所以速度没有太大差别。

# NULL特殊判断
MySQL 提供 IS NULL 和 IS NOT NULL 两种操作来对 NULL 特殊判断。用 = NULL或者 != NULL 是不行的

# [左连接、右连接、内连接](https://blog.csdn.net/plg17/article/details/78758593)

# [case when](https://leetcode.cn/problems/calculate-special-bonus/)
```sql
select 
employee_id,
case 
when mod(employee_id, 2) <> 0 and left(name, 1) <> 'M' then salary
else 0 
end as bonus
from Employees
order by employee_id
```
当case when只有两种情况时可以和if互换：
```sql
SELECT 
    employee_id,
IF(MOD(employee_id,2)!=0 AND LEFT(name,1)!='M',salary,0) bonus
FROM Employees
ORDER BY employee_id
```

# [自连接](https://leetcode.cn/problems/delete-duplicate-emails/)
```sql
delete p1 from person p1, person p2 where p1.email = p2.email and p1.id > p2.id;
```
还可以用窗口函数：
```sql
delete from Person 
where id not in
(select keep_id from
    (select  min(Id) over(partition by Email) as keep_id from Person) t1 );
```
或者group by:
```sql
delete from person where id not in
(SELECT id from (select min(id) as id from person group by email) as t1);
```

# [相关字符串函数](https://leetcode.cn/problems/fix-names-in-a-table/)
```sql
# 一、计算字段

# 其实本题主要考察的就是计算字段的使用。
# 二、知识点
# 2.1 CONCAT() 函数

# CONCAT 可以将多个字符串拼接在一起。
# 2.2 LEFT(str, length) 函数

# 从左开始截取字符串，length 是截取的长度。
# 2.3 UPPER(str) 与 LOWER(str)

# UPPER(str) 将字符串中所有字符转为大写

# LOWER(str) 将字符串中所有字符转为小写
# 2.4 SUBSTRING(str, begin, end)

# 截取字符串，end 不写默认为空。

# SUBSTRING(name, 2) 从第二个截取到末尾，注意并不是下标，就是第二个。

# CONCAT 用来拼接字符串 ● LEFT 从左边截取字符 ● RIGHT 从右边截取字符 ● UPPER 变为大写 ● LOWER 变为小写 ● LENGTH 获取字符串长度

# select user_id, CONCAT(UPPER(left(name, 1)), LOWER(SUBSTRING(name, 2))) as name
select user_id, CONCAT(UPPER(left(name, 1)), LOWER(RIGHT(name, length(name) - 1))) as name
from Users
order by user_id
```

# [group_concat](https://www.jianshu.com/p/7a1df0ce6d00)

# [列转行](https://leetcode.cn/problems/rearrange-products-table/)
```sql
select product_id, 'store1' store, store1 price from products where store1 is not null
union
select product_id, 'store2' store, store2 price from products where store2 is not null
union
select product_id, 'store3' store, store3 price from products where store3 is not null
```

# [查询第N高 ifnull](https://leetcode.cn/problems/second-highest-salary/)
子查询数据出虚表嵌套查询虚表,如果查询不到会返回null
```sql
select 
(select distinct salary from employee order by salary desc limit 1, 1) SecondHighestSalary;
```
或者用IFNULL(expr1,expr2), 假如expr1 不为 NULL，则 IFNULL() 的返回值为 expr1; 否则其返回值为 expr2。IFNULL()的返回值是数字或是字符串，具体情况取决于其所使用的语境。
```sql
SELECT
    IFNULL(
      (SELECT DISTINCT Salary
       FROM Employee
       ORDER BY Salary DESC
        LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary
```

# datediff timestampdiff
datediff是前者减去后者，timestampdiff是小的在前面 -> start, end
```sql
SELECT DATEDIFF('2008-12-30','2008-12-29') AS DiffDate  # 1
SELECT DATEDIFF('2008-12-29','2008-12-30') AS DiffDate  # -1

SELECT TIMESTAMPDIFF(MONTH, '2018-01-01', '2018-06-01') result  # 5
SELECT TIMESTAMPDIFF(MINUTE, '2018-01-01 10:00:00', '2018-01-01 10:45:00') result   # 45
```

# group by
group by 后面的列名，只能是select后面列中没有使用过聚合函数的列

# [count(1), count(*), count(col)](https://www.cnblogs.com/hider/p/11726690.html)
COUNT函数的用法，主要用于统计表行数。主要用法有COUNT(*)、COUNT(字段)和COUNT(1)。
因为COUNT(*)是SQL92定义的标准统计行数的语法，所以MySQL对他进行了很多优化，MyISAM中会直接把表的总行数单独记录下来供COUNT(*)查询，而InnoDB则会在扫表的时候选择最小的索引来降低成本。当然，这些优化的前提都是没有进行where和group的条件查询。
在InnoDB中COUNT(*)和COUNT(1)实现上没有区别，而且效率一样，但是COUNT(字段)需要进行字段的非NULL判断，所以效率会低一些。
因为COUNT(*)是SQL92定义的标准统计行数的语法，并且效率高，所以请直接使用COUNT(*)查询表的行数！

# 求最值
考虑排序+limit

# [group by + min](https://leetcode.cn/problems/game-play-analysis-i/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=fjtkmrh)
```sql
select player_id, min(event_date) first_login from activity group by player_id;
```

# [日期处理 eg:查找2020年最后一次的登录时间](https://leetcode.cn/problems/the-latest-login-in-2020/)
```sql
select user_id, max(time_stamp) last_stamp from logins 
where datediff('2020-12-31', time_stamp) >= 0 and
      datediff('2020-01-01', time_stamp) <= 0 
group by user_id;
```
或者用year:
```sql
select user_id, max(time_stamp) last_stamp from logins 
where year(time_stamp) = 2020
group by user_id;
```

# [价格周期变化求最终值可以考虑case when + sum](https://leetcode.cn/problems/capital-gainloss/)
```sql
select stock_name, sum(
    case when operation = "Buy" then -price
    else price
    end
) capital_gain_loss 
from stocks
group by stock_name;
```

# [order by 多字段](https://leetcode.cn/problems/top-travellers/)
按距离降序，名称升序：
```sql
select name, sum(
    case when distance is null then 0
    else distance
    end
) travelled_distance 
from users left join rides on rides.user_id = users.id
group by users.id
order by travelled_distance desc, name;
```

# having 结合group by, 放group by后面
比如[查找重复的电子邮件](https://leetcode.cn/problems/duplicate-emails/):
```sql
select email Email from person group by email having count(*) > 1;
```

