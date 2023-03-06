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
