## SQL练习

### SQL基础

#### 去重

```sql
# 去重-使用distinct关键字
select distinct university from user_profile;

# 去重-使用`GROUP BY`
select university from user_profile
group by university;
```

**补充：**多个字段进行`distinct`去重

```sql
SELECT round((SELECT COUNT(DISTINCT requester_id, accepter_id) from accepted_requests)
     / (SELECT COUNT(DISTINCT sender_id, send_to_id) from friend_requests), 2) AS accept_rate;
```

#### 列的别名

使用`AS`， 其中`AS`可省略

```sql
select device_id AS user_infos_example from user_profile 
LIMIT 2;
```

#### 查询满足条件的元组

![图片说明](https://uploadfiles.nowcoder.com/images/20210928/170531835_1632759782879/3211A9C2DBBEC176C7BFA1F02F91FB8B)

```sql
select device_id, university from user_profile
where university = '北京大学';

select device_id, gender, age from user_profile
where age between 20 and 23;

# 常见判断空值方法
select device_id, gender, age, university from user_profile
where age IS NOT NULL and age <> "";

# 确定集合
select device_id, gender, age, university, gpa from user_profile
where university in ('北京大学', '复旦大学', '山东大学');
```

> SQL标准说字符串是大小写敏感且只支持单引号，但是MySQL支持双引号，且不区分大小写。

### 字符匹配 LIKE

查找学校中包含“北京”的

```sql
select device_id, age, university from user_profile
where university like '%北京%';
```

四种通配符：

- `_`：匹配任意一个字符；
- `%`：匹配0个或多个字符；
- `[ ]`：匹配`[ ]`中的任意一个字符(若要比较的字符是连续的，则可以用连字符“-”表达)；
- `[^ ]`：不匹配`[ ]`中的任意一个字符。

### 使用函数

```sql
select MAX(gpa) from user_profile
where university = '复旦大学';

# 与下面写法等价
select gpa from user_profile
where university = '复旦大学'
order by gpa desc limit 1;

# 保留一位小数
select COUNT(*) AS male_num, round(AVG(gpa), 1) AS avg_gpa from user_profile
where gender = 'male';
```

### 使用Group by 和 Having

```sql
select gender, university, count(*) as user_num, round(avg(active_days_within_30), 1) as avg_active_day, round(avg(question_cnt), 1) as avg_question_cnt
from user_profile
group by gender, university;

select university, round(avg(question_cnt), 3) as avg_question_cnt, round(avg(answer_cnt), 3) as avg_answer_cnt
from user_profile
group by university
having round(avg(question_cnt), 3) < 5 or round(avg(answer_cnt), 3) < 20;

# Group by和order by 连用
select university, avg(question_cnt) as avg_question_cnt
from user_profile
group by university
order by avg_question_cnt;
```

### 多列排序

取出用户信息表中对应的数据，并先按照gpa、年龄降序排序输出

```sql
select device_id, gpa, age from user_profile
order by gpa desc, age desc;
```

#### 关于LIMIT

在MySQL 8.0中 `IN/ALL/ANY/SOME`下的子查询是不能使用`LIMIT`的

即下面的语句不能被正确执行

```sql
select * from table where id in (select id from table limit 10);
```

但是，只要在` limit`子查询语句的外面再来一层就行

```sql
select * from table where id in (select t.id from (select * from table limit 10)as t);
```

#### Limit和offset连用

```sql
select (select distinct Salary from Employee
                order by Salary desc
                limit 1 offset 1)
AS SecondHighestSal
```

### 连表查询

#### 多表连接

```sql
select a.device_id, question_id, result 
from question_practice_detail as a, user_profile as b
where a.device_id = b.device_id and university = '浙江大学'
order by question_id;
```

还可以使用`join`语法

```sql
SELECT q.device_id, question_id, result
FROM   question_practice_detail q
JOIN   user_profile u
ON     q.device_id = u.device_id 
WHERE  u.university = '浙江大学';
```

- inner join：有效连接，两张表中都有的数据才会显示
- left join：有左显示，比如`on a.field=b.field`，则显示a表中存在的全部数据及a、b中都有的数据，a中有、b中没有的数据以`null`显示
- right join：有右显示，比如`on a.field=b.field`，则显示b表中存在的全部数据及a、b中都有的数据，b中有、a中没有的数据以`null`显示
- full join：全连接，两张表中所有数据都显示

**统计每个学校的答过题的用户的平均答题数**

```sql
select university, 
    count(question_id) / count(distinct a.device_id) as avg_answer_cnt
from question_practice_detail as a
inner join user_profile as b
on a.device_id = b.device_id
group by university;
```

#### 自连接

查询**每一个**部门中薪水最高的职工

```sql
SELECT e.dno, e.eno, e.salary FROM employees e
JOIN (SELECT dno, MAX(salary) AS max_salary
     FROM employees
     GROUP BY dno) sub
ON e.dno = sub.dno and e.salary = sub.max_salary
ORDER BY dno;
```

### Union

![图片说明](https://uploadfiles.nowcoder.com/images/20210928/170531835_1632838203569/612E97B815C9A88A6D6E2C8896095175)

```sql
select device_id, gender, age, gpa
from user_profile
where university = '山东大学'

union all

select device_id, gender, age, gpa
from user_profile
where gender = 'male';
```

### IF和CASE 语句

If 语句

```sql
IF(age<25 OR age IS NULL,'25岁以下','25岁以及上') age_cut
```

Case语句：

```sql
select case when age < 25 or age is null then '25岁以下'
            when age >= 25 then '25岁及以上'
            end age_cut, count(*) as number
from user_profile
group by age_cut;
```

`IFNULL`函数的用法示例：

```sql
select p.product_id, IFNULL(ROUND(SUM(u.units * p.price) / SUM(u.units), 2), 0) AS average_price
from UnitsSold u
RIGHT JOIN Prices p ON p.product_id = u.product_id and u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY product_id;
```

### 谓词逻辑

查询所有职工都参与的项目，注意存在的关键词是`EXISTS`

注意条件是`eno NOT in (select ...)`

```sql
SELECT projects.pno FROM projects
WHERE NOT exists (SELECT eno from employees WHERE 
              eno NOT IN (select eno from works where works.pno = projects.pno));
```

### 时间相关SQL

#### 解析时间

使用`year`, `month`, `day`函数解析时间：

```sql
select day(date) as day, count(question_id) as question_cnt
from question_practice_detail
where month(date) = 8 and year(date) = 2021
group by day;
```

date_format函数: `date_format(date, "%Y-%m")="202108" `

#### 计算用户留存率（自连接和left join）

日期计算函数（增加一天）：`date_add(q1.date, interval 1 day)` 

```sql
select count(q2.device_id) / count(q1.device_id) as avg_ret
from (select distinct device_id, date from question_practice_detail) as q1
left join
(select distinct device_id, date from question_practice_detail) as q2
on q1.device_id = q2.device_id and q2.date = date_add(q1.date, interval 1 day);
```

#### 星期几

`WEEKDAY()`：对应周几的引索（0=周一，1=周二 ... 6=周日）

> 要求解 2020 年 2 月内的总数，可以使用以下两种方式：
>
> 1. ```
>    SELECT COUNT(*) FROM your_table WHERE your_date_column BETWEEN '2020-02-01' AND '2020-02-29';
>    ```
>
>    这种方式是在日期范围内包含了 2 月的所有日期，即从 2020 年 2 月 1 日到 2020 年 2 月 29 日。
>
> 2. ```
>    SELECT COUNT(*) FROM your_table WHERE your_date_column >= '2020-02-01' AND your_date_column < '2020-03-01';
>    ```
>
>    这种方式是利用了日期比较运算符，将日期范围限定在 2020 年 2 月 1 日到 2020 年 3 月 1 日之间，但不包含 2020 年 3 月 1 日。

### 字符串处理  

#### 字符串长度

使用`length`函数

```sql
select tweet_id
from Tweets
where length(content) > 15;
```

#### 字符串切割

`substring_index(str, delim, count)`，这个函数是截取一部分字符串（拆成两半），而不是split的效果。

```sql
select substring_index(profile, ',', -1) as gender, count(*) from user_submit
group by gender;
```

两次切割取出中间的一段：

```sql
select substring_index(substring_index(profile, ',', 3), ',', -1) as age, count(*) as number
from user_submit
group by age;
```

#### 字符串的截取、删除、替换

```sql
replace(string, '被替换部分','替换后的结果')
substr(string, start_point, length*可选参数*)
trim('被删除字段' from 列名)
```

#### 正则表达式

```sql
# Write your MySQL query statement below
select user_id, name, mail
from Users
where mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*\\@leetcode\\.com$'
```

### 窗口函数

> MySQL 5.7不支持窗口函数

```sql
select device_id, university, gpa
from (select *, rank() over (partition by university order by gpa) as rk
    from user_profile) as univ_min
where rk = 1
order by university;
```

### 综合练习

去重`count(distinct 字段名)`+时间函数

```sql
select count(distinct device_id) as did_cnt, count(*) as question_cnt
from question_practice_detail
where year(date) = 2021 and month(date) = 8;
```

`sum`函数和`if`函数组合

```sql
select qd.difficult_level, sum(if(qpd.result = 'right', 1, 0)) / count(*) as correct_rate
from 
    user_profile
join question_practice_detail as qpd
on user_profile.device_id = qpd.device_id

join question_detail as qd
on qpd.question_id = qd.question_id

where user_profile.university = '浙江大学'
group by qd.difficult_level
order by correct_rate;
```

 还有`COUNT`和`IF`函数的组合，这时要使用`NULL`

```sql
count(if(state = 'approved', 1, NULL))
```

