---
title: Leetcode-Database题解
date: 2019-05-16
tag: [Leetcode]
categories: [Leetcode,数据库]
---


## 175. Combine Two Tables

### Description

```sql
Table: Person

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
```

```sql
Table: Address

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.
```

> Write a SQL query for a report that provides the following information for each person in the Person table, regardless if there is an address for each of those people:

```sql
FirstName, LastName, City, State
```

### Analysis
不一定每个人都有地址，所以要使用`外连接(outer join)`，而不是`内链接(inner join)`。

### Solution

```sql
# Write your MySQL query statement below
SELECT FirstName, LastName, City, State
FROM Person P LEFT JOIN Address A
ON P.PersonId = A.PersonId;
```


## 176. Second Highest Salary

### Description

```sql
Table: Employee

+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

> Write a SQL query to get the second highest salary from the Employee table.

For example, given the above Employee table, the query should return 200 as the second highest salary. If there is no second highest salary, then the query should return null.

```sql
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

### Analysis
使用`MAX`可选取最高的工资，那么使用嵌套选择比最高工资低的最高工资即可以选到第二高的工资。


### Solution

```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM employee
WHERE salary < (SELECT MAX(salary)
                 FROM employee); 
```


## 177. Nth Highest Salary

### Description

```sql
Table: Employee

+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

> Write a SQL query to get the nth highest salary from the Employee table.

For example, given the above Employee table, the nth highest salary where n = 2 is 200. If there is no nth highest salary, then the query should return null.

```sql
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```

### Analysis
当`N`比较大用嵌套循环就不太合适了，考虑使用`LIMIT`，`LIMIT`两个参数，第一个参数是表示`OFFSET`，第二个参数表示选取多少条记录。


### Solution

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE m int;
SET m = N - 1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT DISTINCT Salary
      FROM Employee
      ORDER BY Salary DESC 
      LIMIT m, 1
  );
END
```
> 引入新变量`N`是因为`LIMIT`中不能写`n-1`。



## 178. Rank Scores

### Description

```sql
Table: Scores

+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```
> Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks.

For example, given the above Scores table, your query should generate the following report (order by highest score):

```sql
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

### Analysis
可以把`rank`是大于等于本成绩的成绩个数。

### Solution

```sql
SELECT Score, 
    (SELECT COUNT(DISTINCT Score) 
     FROM Scores 
     WHERE Score >= s.Score) Rank 
FROM Scores s 
ORDER BY Score DESC;
```

## 180. Consecutive Numbers

### Description

> Write a SQL query to find all numbers that appear at least three times consecutively.

```sql
+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+
```

For example, given the above Logs table, 1 is the only number that appears consecutively for at least three times.

```sql
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```


### Analysis
查找出`Id`连续且`Num`相等的数字即可。

### Solution
```sql
SELECT DISTINCT L1.num ConsecutiveNums
FROM Logs L1, Logs L2, Logs L3
WHERE L1.id = l2.id - 1
AND L2.id = L3.id - 1
AND L1.num = L2.num
AND l2.num = l3.num;
```


## 181. Employees Earning More Than Their Managers

### Description

```sql
Table: Employee

+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```
> Given the Employee table, write a SQL query that finds out employees who earn more than their managers. For the above table, Joe is the only employee who earns more than his manager.

```sql
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

### Solution
```sql
SELECT e1.Name as Employee
FROM Employee e1 , Employee e2
WHERE e1.ManagerId = e2.Id AND e1.Salary > e2.Salary
```


## 182. Duplicate Emails

### Description

```sql
Table: Person

+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```
> Write a SQL query to find all duplicate emails in a table named Person.

For example, your query should return the following for the above table:

```sql
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

> All emails are in lowercase.

### Analysis
可以使用`distinct`找出`Id`不同但是`Email`相同的，或使用`count`找出次数大于2的即为重复。

### Solution

```sql
SELECT DISTINCT (p.Email)
FROM Person p, Person p1
WHERE (p.Id<>p1.Id and p.Email=p1.Email);
```

或

```sql
SELECT DISTINCT p.Email
FROM Person p
GROUP BY p.Email
HAVING COUNT(p.Email)>1;
```


## 183. Customers Who Never Order

### Description
```sql
Table: Customers
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+


Table: Orders
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

> Write a SQL query to find all customers who never order anything.

Using the above tables as example, return the following:

```sql
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

### Analysis
可使用`NOT IN`或`NOT EXISTS`做排除。

### Solution

```sql
SELECT C.Name
FROM Customers C
WHERE NOT EXISTS
(SELECT Id
FROM Orders O
WHERE C.Id = O.CustomerId)
```
或

```sql
SELECT C.Name
FROM Customers C
WHERE C.Id NOT IN
(SELECT O.CustomerId
FROM Orders O)
```

## 184. Department Highest Salary

### Description
```sql
Table: Employee

+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```

### Solution

```sql
select
d.Name, e.Name, e.Salary
from
Department d,
Employee e,
(select MAX(Salary) as Salary,  DepartmentId as DepartmentId from Employee GROUP BY DepartmentId) h
where
e.Salary = h.Salary and
e.DepartmentId = h.DepartmentId and
e.DepartmentId = d.Id;
```

## 185. Department Top Three Salaries

### Description
```sql
Table: Employee
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+

Table: Department
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

> Write a SQL query to find employees who earn the top three salaries in each of the department. For the above tables, your SQL query should return the following rows (order of rows does not matter).


Explanation:

In IT department, Max earns the highest salary, both Randy and Joe earn the second highest salary, and Will earns the third highest salary. There are only two employees in the Sales department, Henry earns the highest salary while Sam earns the second highest salary.


### Analysis

满足部门`id`相同，表示在同一个部门，且比当前高的工资个数要小于 3 个即可。

### Solution

```sql
SELECT d.name AS Department, e.Name AS Employee, e.Salary 
FROM Employee AS e INNER JOIN Department AS d 
ON e.DepartmentId = d.Id 
WHERE (SELECT count(DISTINCT Salary) 
       FROM Employee 
       WHERE DepartmentId = d.Id AND Salary > e.Salary ) < 3
ORDER BY e.Salary DESC;
```

## 196. Delete Duplicate Emails

### Description
```sql
Table: Person
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id is the primary key column for this table.
```

> Write a SQL query to delete all duplicate email entries in a table named Person, keeping only unique emails based on its smallest Id.

For example, after running your query, the above Person table should have the following rows:

```sql
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+

```

### Solution

```sql
DELETE p1 FROM Person p1 ,Person p2
WHERE p1.Email =p2.Email AND p1.Id > p2.Id 
```



## 197. Rising Temperature

### Description
```sql
Table: Weather
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```

> write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.

For example, return the following Ids for the above Weather table:

```sql
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```


### Analysis
使用`DATEDIFF`，使两个日期的天数差为1。

### Solution
```sql
SELECT DISTINCT a.id as Id 
FROM weather a ,weather b 
WHERE a.temperature > b.temperature 
AND datediff(a.recorddate, b.recorddate) = 1
```

## 262. Trips and Users

### Description
```sql
Table: Trips
+----+-----------+-----------+---------+--------------------+----------+
| Id | Client_Id | Driver_Id | City_Id |        Status      |Request_at|
+----+-----------+-----------+---------+--------------------+----------+
| 1  |     1     |    10     |    1    |     completed      |2013-10-01|
| 2  |     2     |    11     |    1    | cancelled_by_driver|2013-10-01|
| 3  |     3     |    12     |    6    |     completed      |2013-10-01|
| 4  |     4     |    13     |    6    | cancelled_by_client|2013-10-01|
| 5  |     1     |    10     |    1    |     completed      |2013-10-02|
| 6  |     2     |    11     |    6    |     completed      |2013-10-02|
| 7  |     3     |    12     |    6    |     completed      |2013-10-02|
| 8  |     2     |    12     |    12   |     completed      |2013-10-03|
| 9  |     3     |    10     |    12   |     completed      |2013-10-03| 
| 10 |     4     |    13     |    12   | cancelled_by_driver|2013-10-03|
+----+-----------+-----------+---------+--------------------+----------+


Table: Users
+----------+--------+--------+
| Users_Id | Banned |  Role  |
+----------+--------+--------+
|    1     |   No   | client |
|    2     |   Yes  | client |
|    3     |   No   | client |
|    4     |   No   | client |
|    10    |   No   | driver |
|    11    |   No   | driver |
|    12    |   No   | driver |
|    13    |   No   | driver |
+----------+--------+--------+

```

> Write a SQL query to find the cancellation rate of requests made by unbanned users between Oct 1, 2013 and Oct 3, 2013. For the above tables, your SQL query should return the following rows with the cancellation rate being rounded to two decimal places.


### Analysis
可以使用`Case When ... Then ... Else ...End`关键字来做，使用`cancelled%`来表示开头是`cancelled`的所有项(包括`client`和`driver`)。

### Solution

```sql
SELECT t.Request_at Day, 
ROUND(SUM(CASE WHEN t.Status 
          LIKE 'cancelled%' 
          THEN 1 ELSE 0 END)/COUNT(*), 2) 'Cancellation Rate'
FROM Trips t JOIN Users u 
ON t.Client_Id = u.Users_Id AND u.Banned = 'No' 
WHERE t.Request_at BETWEEN '2013-10-01' AND '2013-10-03' 
GROUP BY t.Request_at;
```

## 595. Big Countries

### Description
```sql
Table: World
+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
```

A country is big if it has an area of bigger than 3 million square km or a population of more than 25 million.

> Write a SQL solution to output big countries' name, population and area.

For example, according to the above table, we should output:
```sql
+--------------+-------------+--------------+
| name         | population  | area         |
+--------------+-------------+--------------+
| Afghanistan  | 25500100    | 652230       |
| Algeria      | 37100000    | 2381741      |
+--------------+-------------+--------------+
```


### Solution
```sql
SELECT name,population, area
FROM World WHERE area>3000000 
OR population>25000000
```

## 596. Classes More Than 5 Students

### Description
```sql
Table: courses
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```
> Please list out all classes which have more than or equal to 5 students.

```sql
+---------+
| class   |
+---------+
| Math    |
+---------+
```
### Solution
```sql
SELECT class FROM courses 
GROUP BY class 
HAVING count(distinct student) >= 5;
```


## 601. Human Traffic of Stadium

### Description
```sql
Table: stadium
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
```
> Please write a query to display the records which have 3 or more consecutive rows and the amount of people more than 100(inclusive).

For the sample data above, the output is:

```sql
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
```

> Each day only have one row record, and the dates are increasing with id increasing.

### Solution

```sql
SELECT DISTINCT a.*
FROM stadium a, stadium b, stadium c
WHERE (a.id=b.id+1 AND b.id=c.id+1
    OR a.id+1=b.id AND b.id+1=c.id
    OR a.id+1=b.id AND a.id=c.id+1)
AND a.people >= 100 AND b.people>= 100 AND c.people >= 100
ORDER BY a.id
```


## 620. Not Boring Movies


### Description
```sql
Table: cinema
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
```

> Please write a SQL query to output movies with an odd numbered ID and a description that is not 'boring'. Order the result by rating.

For the example above, the output should be:

```sql
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

### Solution
```sql
SELECT id, movie, description, rating FROM cinema 
WHERE description!='boring' AND id%2!=0 
ORDER BY rating desc;
```


## 626. Exchange Seats

### Description
```sql
Table: seat
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
```

>  change seats for the adjacent students

For the sample input, the output is:

```sql
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
```

### Analysis

可以通过交换`id`来交换座位。如果id是奇数，就加1作为新`id`。如果是偶数，就减1作为新`id`。另外，如果学生数为奇数，就不改变最后一个`id`。

### Solution

```sql
SELECT (CASE
            WHEN id % 2 = 1 AND id != C THEN id+1
            WHEN id % 2 = 1 AND id  = C THEN id
       ELSE id-1 END) AS id,student
FROM seat,(SELECT COUNT(1) AS C FROM seat) AS T
ORDER BY id;
```


## 627. Swap Salary

### Description
```sql
Table: salary
+----+------+-----+--------+
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
+----+------+-----+--------+
```

> Swap all f and m values (i.e., change all f values to m and vice versa) with a `single update statement` and no intermediate temp table.

After running your update statement, the above salary table should have the following rows:

```sql
+----+------+-----+--------+
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
+----+------+-----+--------+
```

### Analysis
可以使用`Case When ... Then ... Else ...End`或`IF`来做。

### Solution

```sql
UPDATE salary SET sex = CASE sex WHEN 'm' then 'f' ELSE 'm' end
```

或

```sql
UPDATE salary SET sex = IF(sex = 'm','f','m')
```