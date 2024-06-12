## 重要的SQL常识

### 聚合函数

当使用聚合函数时，所有非聚合字段必须出现在`GROUP BY`子句中

聚合函数的子查询或`having`语句

### 空值

1. `select count(*)`**会**计算空(NULL)行
2. `select count(name)`**不会**计算空(NULL)行

**例子：**

> 有三个人的薪水：10000， 20000， NULL
>
> 使用`AVG(sal)`计算三个人的平均薪水， 会计算成15000

**使用函数：**

```sql
select coalesce(success_cnt, 1) from tableA
```

当`success_cnt` 为`NULL`时，将返回1，否则将返回success_cnt的真实值。

## 字符串处理

### 遍历字符串

SQL没有Loop循环功能，我们需要**数据透视表**

引入一张表t1, ..., t10，做一个笛卡尔积

| str  | id   |
| ---- | ---- |
| King | 1    |
| King | 2    |
| King | 3    |
| King | 4    |



### 嵌入引号

两边单引号`'`， 中间`"`

### 统计字符出现的次数

替换+长度相减

```sql
length('10,CLARK,MANAGER')-length(replace('10,CLARK,MANAGER', ',', '') / length(','))
```

### 删除不需要的字符

删除元音字母AEIOU

不同数据库有不同解决方案，MySQL：replace嵌套😭

### 分离数字和字符数据

### 判断含有字母和数字的字符串

MySQL：正则表达式 

> MySQL赢！

### 提取姓名的首字母

## 数值处理

最大值和最小值不受空值影响

### 累计求和

`order by 2`表示按select里面的第2个排序

### 中位数

Oracle： median函数

窗口函数：limit配合offset

### 去掉

NOT IN

## 日期处理

### 年月日加减法

### 计算两个日期之间的天数

MySQL: `datediff(day1, day2)`函数

Oracle and PostgreSQL：之间用`-`

MySQL: current_date