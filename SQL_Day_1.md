
### 175. Combine Two Tables (Easy)

```sql
SELECT
    P.firstName,
    P.lastName,
    A.city,
    A.state
FROM
    Person P
    LEFT JOIN 
    Address A ON A.personId = P.personId
;
```

### 177. Nth Highest Salary (Medium)

There are different ways of doing this.

Questions to ask:
- Do we just need the salary value or do we need the names of the employees who have that salary
- Lets assume we just need the salary value for this question. That meaans only 1 row (and 1 column) will be returned from the query.
- If there are no values in the table, If there is only a single value in the table, if there are less than N values in the table, if there are less than N unique saary values in the table - In all these cases, the Nth highest salary does not exist. What should the resultset say - Blank resultset (0 rows returned) or NULL value returned.
- Lets assume we need to return NULL in case any of the above edge conditions arise.

Things to know:
If N was given, say 3rd Highest,then -       
**Approach 1 :**       
Simplest way to think about this is :

```sql
SELECT DISTINCT
    salary
FROM
    Employee 
ORDER BY salary DESC LIMIT 1 OFFSET 2;
```
However, the above query does not work in case of the edge conditions above. It does not return NULL.

**Approach 2 (Using ORDER BY, OFFSET N-1, LIMIT 1) :**   

So there is just one trick to do this, wrap the sql statement befoer inside another SELECT :          

```sql
SELECT(SELECT DISTINCT
    salary
FROM
    Employee 
ORDER BY salary DESC LIMIT 1 OFFSET 2) AS ThirdHighestSalary;
```

**Approach 3 (Using Correlated Subquery) :**      

For 3rd highest salary, there are 2 distinct salaries which the required 3rd highest salary is less than.           
For 2rd highest salary, there is 1 distinct salary which the required 2nd highest salary is less than.         
For Nth highest salary, there are N-1 distinct salaries which the required Nth highest salary is less than.         
Make use of above fact to write a correlated subquery.           


```sql
SELECT DISTINCT
    e1.salary
FROM 
    Employee e1
 WHERE N-1 = (SELECT
                COUNT(DISTINCT(e2.salary))
              FROM
                Employee e2
              WHERE e1.salary < e2.salary
             )
;
```

**Approach 4 :**   

So approach 2 works if N is known. Lets say N is variable. This essentially means we need to create a FUNCTION to do this that takes N as parameter.
So we write approach 2 query, but inside a Function. Note OFFSET has to N-1, which somehow does not work directly, so we need to declare and set another variable to N-1.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    DECLARE M INT;
    SET M = N-1;
    RETURN(
    SELECT(SELECT DISTINCT
    salary
    FROM
    Employee 
    ORDER BY salary DESC LIMIT 1 OFFSET M) AS NthHghestSalary;
    )
END
```
**Approach 5 :**          
This is just the N-version of Approach 3.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INotT
BEGIN
RETURN (
  select distinct e1.salary from Employee e1 where N-1 = (select count(distinct e2.Salary) from Employee e2 where e1.Salary < e2.Salary)
);
END
```

### 178. Rank Scores (Medium)

The question is a simple DENSE_RANK(question). But the trick is that they expect that the output column be named as *rank*. The trick to do that is to use quotes when creating the alias, otherwise MySQL throws an error as rank is a keyword.

```sql
SELECT
    s.score,
    DENSE_RANK() OVER(ORDER BY s.score DESC) AS 'rank'
FROM
    Scores s;
```

However, another option is do below, but not only is this verbose, but also 50% slower if the below 

```sql
select score, (select count(distinct score) from scores s2 where s2.score >= s1.score) as "Rank"
from scores s1
order by score desc;
```

### 181. Employees Earning More Than Their Managers (Easy)

```sql
SELECT
    e1.name as 'Employee'
FROM
    Employee e1
WHERE
    e1.managerId IS NOT NULL 
    AND e1.salary > (SELECT e2.salary FROM Employee e2 WHERE e2.id=e1.managerId)
;
```
### 182. Duplicate Emails (Easy)

This is a case where not all the elements in a row are duplicate.
Also this is a case where we just need to find the duplicates. Not remove them.
So a simple Count() HAVING will work with GROUP BY.
```sql
SELECT
    p1.email
FROM
    Person p1
GROUP BY p1.email
HAVING COUNT(*) > 1;

```

### 183. Customers Who Never Order (Easy)
```sql
SELECT
    c.name as Customers
FROM
    Customers c
    LEFT JOIN Orders o ON o.customerId = c.id
WHERE
    o.id IS NULL;
```

### 184. Department Highest Salary (Medium)

Pretty tricky question, mainly because of the format of the resultset expected output which requires columns from both the tables.
Note:
1) The resultset output contains columns from both tables. So a JOIN is inevitable. So do a JOIN. Since you are joining, do a GROUP BY and put depertmentid and departmentname as part of the GROUP BY and get the Max salary. Now you have departmentId, department name and Max salary of that department.
2) Use the above as a CTE. You have got all the rows except the employee name(s) having that max salary in the table. So do a JOIN with the CTE, and get the name from the employee where the salary is matching.

```sql
WITH t1 AS
(SELECT
    e1.departmentId, d.name, MAX(e1.salary) as Max_Salary
FROM
    Employee e1
    JOIN Department d ON d.id = e1.departmentId
GROUP BY e1.departmentId, d.name
SELECT 
    t1.name as Department, 
    e2.name as Employee,
    e2.salary as Salary
FROM 
    t1
    JOIN Employee e2 ON t1.departmentId = e2.departmentId
                            AND t1.Max_Salary = e2.salary;
```

### 185. Department Top Three Salaries (Hard)

If you solved the previous one, this one is just addition of a window function dense_rank().
Strategy for both questions : The second table just has one additional info - department name. So do everything you need from the first table, and then later join it with the second table to get the department name.

```sql
WITH t1 AS (SELECT
    e1.*,
    DENSE_RANK() OVER(PARTITION BY e1.departmentId ORDER BY salary DESC) as salary_rank
FROM
    Employee e1)

SELECT
    d.name as Department,
    t1.name as Employee,
    t1.salary as Salary
FROM
    t1
    JOIN Department d ON t1.departmentId =  d.id
WHERE t1.salary_rank <=3;
```
### 196. Delete Duplicate Emails (Easy)

Although labelled as easy, this one is pretty tricky.

The reason is that you have to use a DELETE clause. While you are using a DELELTE clause, you cannot use a subquery on the same table in its DELETE WHERE clause.
So the trick is that since CTE execution happens before all this, identify the ids tio be deleted usong proper JOIN Clause, make a list of those ids. Use that CTE in the WHERE clause of DELETE, which is allowed.

    
```sql
WITH t1 AS ( SELECT p1.id
                FROM
                Person p1
                JOIN Person p2 ON p1.email = p2.email
                                  AND p1.id > p2.id
                    )
DELETE
FROM
    Person p
WHERE
    p.id IN ( SELECT t1.id FROM t1);
```

### 197. Rising Temperature (Easy)

The trick is to join on a DATEDIFF. The join can itself handle all the filtering logic.
```sql
SELECT
    weather.id AS 'Id'
FROM
    weather
        JOIN
    weather w ON DATEDIFF(weather.recordDate, w.recordDate) = 1
        AND weather.Temperature > w.Temperature
;
```

### 262. Trips and Users (Hard)

Ideas :
    - The JOIN option looks a bit tricky and reads bad. Subquery option felt better.
    - CTE based solution. But breaking into CTES does two things - produces clean code, also is better for performance
    - Two step process- Removes all banned users in first CTE, uses first CTE to subquery and filter the Trips table
    - Create an additional column using CASE WHEN. Whenever there is a rate based calculation, creating a binary column and taking its average and rounding off is a good idea.
    - Final step utilizes the above CTES to simply group and order the data. Perhaps it is possible to do this using a single CTE, but since the CASE WHEN new column needs to be there for the calculation, maybe two CTEs are required.

```sql
WITH t1 as (
    SELECT 
        u.users_id,
        u.banned,
        u.role
    FROM    
        Users u
    WHERE
        u.banned = 'No'
        AND u.role IN ('client', 'driver')
),

t2 as (
    SELECT
        t.*,
        CASE 
            WHEN t.status IN ('cancelled_by_driver', 'cancelled_by_client') THEN 1
            WHEN t.status = 'completed' THEN 0
        END AS cancel_indicator
    FROM
        Trips t
    WHERE
        EXISTS (SELECT t1.users_id FROM t1 WHERE t1.users_id = t.client_id)
        AND EXISTS ((SELECT t1.users_id FROM t1 WHERE t1.users_id = t.driver_id))
        AND t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
)
SELECT 
    t2.request_at AS Day,
    ROUND(AVG(t2.cancel_indicator),2) AS 'Cancellation Rate'
FROM   
    t2
GROUP BY t2.request_at
ORDER BY Day;
```


### 511. Game Play Analysis I

Ideas:
    - The trick is to not get carried away by possibility of using window functions such as RANK, FIRST_VALUE etc as the same thing can be achieved by using a simple GROUP BY and MIN(). If the label of the problem did not say EASY, you might get into window functions. Not sure Window functions run faster in this case or not. However, from the point of view of simplicity and readability it is a small and perfect solution.

```SQL
SELECT  
    a.player_id,
    MIN(a.event_date) AS first_login
FROM
    Activity a
GROUP BY a.player_id;
```

### 512. Game Play Analysis II

Ideas:
    - 2 approaches - Use correlated subquery or use window function (either as a derived table or a CTE)
    - The correlated subquery seems to take more time. Window function is better.
    - Although it seems that the correlated subquery is easier to read and small, it seems that in such situations it is more efficient to create a CTE/derived table with a Window Function labelling our required row and then fetching that row from the CTE using the required value of the window function. 
    
All 3 versions of the solution are given below. 

```SQL
SELECT
    a.player_id,
    a.device_id
FROM
    Activity a
WHERE
    a.event_date = (SELECT MIN(a1.event_date) FROM ACTIVITY a1 WHERE a1.player_id = a.player_id);
```
```SQL
SELECT
    t1.player_id,
    t1.device_id
FROM
(SELECT
    a.player_id,
    a.device_id,
    a.event_date,
    ROW_NUMBER() OVER (PARTITION BY a.player_id ORDER BY a.event_date) AS loginorder
FROM
    Activity a) t1
WHERE
    t1.loginorder = 1
```
```SQL
WITH t1 as (SELECT
    a.player_id,
    a.device_id,
    a.event_date,
    ROW_NUMBER() OVER (PARTITION BY a.player_id ORDER BY a.event_date) AS loginorder
FROM
    Activity a) 
SELECT
    t1.player_id,
    t1.device_id
FROM
    t1
WHERE
    t1.loginorder = 1;
```
