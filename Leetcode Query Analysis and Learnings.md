
### 569. Median Employee Salary (Hard)

Idea:        
- The trick in almost all Median problems in programs to know two things:
    - First find total number of entries =  n      
    - Next, the row numbers that is/are median row numbers can be defined as either of the two methods (I preferred the first one in the SQL query as it allows BETWEEN concept), otherwise to find exact ROW numbers you can use method 2.                
            - METHOD 1: median are row numbers BETWEEN n/2 and (n/2)+1             
            - METHOD 2: median are row numbers FLOOR((n+1)/2), FLOOR((n+2)/2)               
    - Both above methods work for even and odd values of n.               
        
```sql
WITH t1 as (

    SELECT 
        e.id,
        e.company,
        e.salary,
        COUNT(e.id) OVER(PARTITION BY e.company) AS companyCount,
        ROW_NUMBER() OVER(PARTITION BY e.company ORDER BY e.salary) AS position
    FROM
        Employee e
)
SELECT
    t1.id,
    t1.company,
    t1.salary
FROM
    t1
WHERE
    t1.position  BETWEEN ((t1.companyCount)/2) AND (((t1.companyCount)/2) + 1);
```

```sql
WITH t1 as (

    SELECT 
        e.id,
        e.company,
        e.salary,
        COUNT(e.id) OVER(PARTITION BY e.company) AS companyCount,
        ROW_NUMBER() OVER(PARTITION BY e.company ORDER BY e.salary) AS position
    FROM
        Employee e
)
SELECT
    t1.id,
    t1.company,
    t1.salary
FROM
    t1
WHERE
    t1.position  = FLOOR((companyCount + 1)/2)
    OR t1.position = FLOOR((companyCount + 2)/2);
    
```



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


### 511. Game Play Analysis I (Easy)

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

### 512. Game Play Analysis II (Easy)

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
### 534. Game Play Analysis III (Medium)

Ideas:
    - Basically the question needs running totals for unique playerid and date. It means using SUM() as window function is one of the obvious choices
    - SUM() when used as window function along with an ORDER BY takes the frame as UNBOUNDED PRECEDING to CURRENT ROW by default, producing running totals. However, it does not hurt to mention it explicitly.
    - Usage of RANGE seems to be appropriate to specify frame as there could be multiple entries with the same date. ALthough the test case does not have such entries.
    - Some of the solutions in the discussion may suggest that it could be done without a CTE, but they assume that it is not possible for the same player to play using multiple devices on the same day. Or the user having multiple logins on the same day on the same device. The solution below, on the other hand, is robust to such scenarions. 
    - Used GROUP BY instead of SELECT DISTINCT in the actual query to make it more efficient.
    
```sql
WITH t1 as (SELECT
    a.player_id,
    a.event_date,
    a.games_played,
    SUM(a.games_played) 
        OVER (PARTITION BY a.player_id ORDER BY a.event_date
             RANGE UNBOUNDED PRECEDING) AS games_played_so_far
FROM
    Activity a)
SELECT 
    t1.player_id,
    t1.event_date,
    t1.games_played_so_far
FROM
    t1
GROUP BY t1.player_id, t1.event_date, t1.games_played_so_far
ORDER BY t1.player_id, t1.event_date;
```


### 550. Game Play Analysis IV (Medium)

Idea:
    - The solution is the division of TWO subqueries in the SELECT clause of the main query itself. This makes this query unique
    - Usage of the DATEDIFF function to JOIN in one of the subqueries is a very handy trick to choose anything "consecutive" or with a constant difference.
    - Note, when a aggreagte function column is used without an alias, it takes the name of the full formula, so use it always with alias.
    - Whenever calculating fractions and rates, be flexible to calculate numerators and denominators as separate subqueries

```sql
WITH t1 as (
    SELECT
        a.player_id,
        MIN(a.event_date) as min_event_date
    FROM
        Activity a
    GROUP BY a.player_id
)
SELECT
    ROUND((SELECT COUNT(DISTINCT(t1.player_id))
           FROM t1 JOIN Activity a1 
                ON t1.player_id=a1.player_id 
                AND DATEDIFF(a1.event_date, t1.min_event_date) = 1)/
        (SELECT COUNT(DISTINCT a2.player_id) FROM Activity a2),2) AS fraction;

```

### 570. Managers with at Least 5 Direct Reports (Medium)

Idea:       
- Joining on a field that has null values will ignore the nulls
- In problems related to Manager and Employee being in the same table, you can take self-join on ManagerId and Id
- After that simple GROUP BY and count should solve the problem. 
- The only need to add a subquery is the fact that we have to produce only name in the output and we cannot do operations based on just name in the inner query as it could be duplicate for different people, we have to use id there.

```sql
SELECT t1.name 
FROM
    (SELECT
        e1.id,
        e1.name
    FROM
        Employee e1
        JOIN Employee e2 ON e1.id = e2.managerId
    GROUP BY
        e1.id, e1.name
    HAVING COUNT(e2.id) >= 5) t1;
```

### 571. Find Median Given Frequency of Numbers (Hard)

Idea:       
- Do not actually try to decompress this in SQL. Hope that is clear.
- Again, when it comes to median, you know you want to find 2 things - total(n) and positions floor(n+1)/2, floor (n+2)/2
- Generally in median problems it is good to append columns to the table with total(n) and in this case the cumulative fequency
- Median lies in the slot where cumulative frequency of the slot is minimum cumulative freq greater than floor(n+1)/2 and minimum cum freq where cum freq is greater than floor (n+2)/2. Then just take average of the num in those two rows.

```sql
WITH t1 as (

    SELECT
        n.num,
        n.frequency,
        SUM(n.frequency) OVER() AS total_frequency,
        SUM(n.frequency) OVER(ORDER BY n.num) AS cumulative_frequency
    FROM
        Numbers n
)

SELECT
    AVG(t1.num) as median
FROM
    t1
WHERE   
    t1.cumulative_frequency = (SELECT MIN(t1.cumulative_frequency) FROM t1 WHERE t1.cumulative_frequency >= FLOOR((total_frequency + 1)/2))
    OR
    t1.cumulative_frequency = (SELECT MIN(t1.cumulative_frequency) FROM t1 WHERE t1.cumulative_frequency >= FLOOR((total_frequency + 2)/2)) ;
```

### 574. Winning Candidate (Medium)

Idea :     
- The first solution is the one given in leetcode.
- The second solution is what I created.
- In case you can do all computation in 1 table, but you need a column from another, do all calculation in one table and just use it as key in the other table. This involves subquery. JOIN can also be used though.

```sql
WITH t1 as (SELECT
    c.id,
    c.name,
    COUNT(v.id) AS total_votes
FROM
    Vote v
    JOIN Candidate c ON c.id = v.candidateId
GROUP BY c.id, c.name)
SELECT
    t1.name
FROM
    t1
ORDER BY t1.total_votes DESC
LIMIT 1;
```


```sql
SELECT
    name AS 'Name'
FROM
    Candidate
        JOIN
    (SELECT
        Candidateid
    FROM
        Vote
    GROUP BY Candidateid
    ORDER BY COUNT(*) DESC
    LIMIT 1) AS winner
WHERE
    Candidate.id = winner.Candidateid
;
```

### 577. Employee Bonus (Easy)

A very deceptive question indeed!

This is one of those questions that show you the need to consider LEFT JOINs and possibility of null values.

The trick is to realize the business scenario of the situation and understand that some employees may have 0 bonus, and are not present in the bonus table. So employees with bonus lower than 1000 would include those employees as well who do not have a bonus at all.

```sql
SELECT
    e.name,
    b.bonus
FROM
    Employee e
    LEFT JOIN 
    Bonus b ON b.empId = e.empId
WHERE
    b.bonus < 1000 
    OR b.bonus IS NULL;
```


### 579. Find Cumulative Salary of an Employee (Hard)

Idea :
- Note the tricky part of the question is that there are some missing months where employee has not worked. otherwise you could have thought of a window function sum() over 2 PRECEDING and CURRENT ROW     
- The right solution is do a DOUBLE SELF JOIN and use the condition to specify the previous and previos-to-previous month. These will ensure null values for the employees who did not work in those months.           
- Then you just need to add those salary columns, use IFNULL() or COALESCE() to ensure that column values added do not have to deal with NULL, instead 0 if not present
- Mind the fact that you should not use SUM(col1,col2,col3) as SUM() is an aggregate function used for rows, cannot sum across columns, for that you just need to use + operator,                  
- Lastly, an added layer of complication is added by asking to remove the most recent month. The trick is to not try removing it at first, rather than once all above is done, create a separate CTE with list of the months to be filtered out. Once you have it, just do a NOT EXISTS in the CTE in the main query to filter them out.

```sql
WITH t1 AS (SELECT
    e1.id,
    e1.month,
    e1.salary + COALESCE(e2.salary,0) + COALESCE(e3.salary,0) as salary
FROM
    Employee e1
    LEFT JOIN Employee e2 ON e1.id =  e2.id AND e2.month = e1.month - 1
    LEFT JOIN Employee e3 ON e1.id =  e3.id AND e3.month = e1.month - 2
    ),

t2 AS (
    SELECT
        e4.id,
        MAX(e4.month) as recent_month
    FROM
        Employee e4
    GROUP BY e4.id
)
SELECT 
    t1.id,
    t1.month,
    t1.salary
FROM
    t1
WHERE 
    NOT EXISTS (
        SELECT 
            t2.id,
            t2.recent_month
        FROM
            t2
        WHERE 
            t2.id = t1.id
            AND t2.recent_month = t1.month 
    )
ORDER BY t1.id, t1.month DESC;

```

### 578. Get Highest Answer Rate Question (Medium)

Idea:
- The Leetcode solution is slightly better. And it shows that you can conditionally SUM up a group (based on IF(condition, 1, 0)). Basically it utilizes your idea of adding an indicator variable for rate calculation problems. However, in this case a simple binary variables added by CASE WHEN won't work as the denominator is not all the entries. But they have still done it well. 
- Remember you have IF() in your toolkit as a conditional tool apart from CASE WHEN. Also IFNULL is another null handling version of IF
- Try to name ctes as cte1 and cte2 rather than t1 and t2. Will avoid confusion in future.

Leetcode Solution (recommended):      
```sql
select question_id as survey_log from
(select question_id, 
 SUM(IF(action = "answer", 1, 0)) / SUM(IF(action = "show", 1, 0)) as answer_rate
 from surveylog
 group by question_id
 order by answer_rate DESC, question_id
 limit 1
)
```

My Solution:
```sql
WITH cte1 AS (SELECT
    s.question_id,
    s.action,
    COUNT(*) AS action_count
FROM
    SurveyLog s
WHERE
    s.action <> 'skip'
GROUP BY s.question_id, s.action),

cte2 AS (SELECT
    c1.question_id,
    c1.action_count / c2.action_count AS answer_rate
FROM
    cte1 c1
    JOIN
    cte1 c2 ON c1.question_id = c2.question_id AND c1.action < c2.action 
ORDER BY answer_rate DESC, c1.question_id
LIMIT 1)

SELECT 
    cte2.question_id AS survey_log
FROM
    cte2;
```
### 180. Consecutive Numbers

Idea: 
- Using Lag() or Lead() window functions the solution should be easy
- The Leetcode solution uses JOINs only, which is nice too. It is useful to remember that you don't always have to use JOIN ON condition, you can also use JOIN table1, table2 WHERE some condition, although it seems that if you can handle with ON condition it is probably more efficient.

My Solution:
```sql
WITH cte1 AS (SELECT
    l.id,
    l.num as first_num,
    LEAD(l.num, 1) OVER(ORDER BY l.id) AS second_num,
    LEAD(l.num, 2) OVER(ORDER BY l.id) AS third_num
FROM
    Logs l)

SELECT
    cte1.first_num AS ConsecutiveNums
FROM
    cte1
WHERE
    cte1.first_num = cte1.second_num
    AND cte1.second_num = cte1.third_num
GROUP BY cte1.first_num;
```

Leetcode Solution:
```sql
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```

### 580. Count Student Number in Departments (Medium)

This is an easy one, but you could take some time trying to remember whether  if you use group by and the grouping column does not have any rows in the other table (have nulls in those columns), does COUNT() return null. The answer is COUNT(the other table's column) will return 0.
```sql
SELECT
    d.dept_name,
    COUNT(student_id) as student_number
FROM
    Department d
    LEFT JOIN Student s ON s.dept_id = d.dept_id
GROUP BY dept_name
ORDER BY student_number DESC, dept_name;
```
### 585. Investments in 2016

Mildly irritating question. However, exhibits the usage of EXISTS and NOT EXISTS subqueries.

```sql
SELECT
    ROUND(SUM(i1.tiv_2016),2) AS tiv_2016

FROM
    Insurance i1

WHERE
    EXISTS (
        SELECT i2.pid
        FROM
        Insurance i2
        WHERE i2.pid <> i1.pid
        AND i2.tiv_2015 = i1.tiv_2015
    )
    
    AND NOT EXISTS (
        SELECT i3.pid
        FROM
        Insurance i3
        WHERE i3.pid <> i1.pid
        AND i3.lat = i1.lat
        AND i3.lon = i1.lon
    )
;
```
### 586. Customer Placing the Largest Number of Orders (Easy)

This is the classic problem where you check usage of a CTE, subquery, etc.
Although this query can avoid usage of a CTE, as MySQL allows to ORDER BY COUNT(\*) without using it in the select clause, still I think in situations where more than 1 maximum can exist, the below query is better.
Also, it is memory efficient as for the second set of scans it uses the temporary table instead of the actual table.

```sql
WITH cte1 AS (SELECT
    o1.customer_number,
    COUNT(order_number) AS order_count
FROM Orders o1
GROUP BY o1.customer_number)

SELECT 
    cte1.customer_number
FROM cte1
WHERE cte1.order_count = (SELECT MAX(order_count) FROM cte1);
```

### 601. Human Traffic of Stadium (Hard)

The nice thing to note here is:
    - The CTEs themselves can be aliased many times in multiple subqueries, which is cool.
    - In such problems of consecutive rows, we need to focus on selection criteria for 1 single row, and write our logic around it.
    - For example, a row is selected (after filtering everything based on >= 100) if 
    1. Another row exists with id = id + 1 and another row exists with id = id + 2 OR   -- this row is the beginning of 3-series
    2. Another row exists with id = id - 1 and another row exists with id = id -2 OR    -- this row is the end of 3-series
    3. Another row exists with id = id - 1 and another row exists with id = id + 1 OR   -- this row is the middle of 3-series
    - Note that, the problem states 3 or more... but anything that is part of a 4-series or 5-series or n-series is also part of a 3-series, so above logic works.
    - The Leetcode solution uses a 3-cross join to something similar. One could think of that too.

```sql
WITH cte1 AS (SELECT *

FROM
    Stadium S1
WHERE 
    S1.people >= 100)
    
SELECT
    *
FROM
    cte1 x
WHERE
    (EXISTS (
        SELECT a.id
        FROM cte1 a
        WHERE a.id = x.id + 1 
             ) 
     AND EXISTS
            (
        SELECT b.id
        FROM cte1 b
        WHERE b.id = x.id + 2
             ) 
    )
    OR
    (EXISTS (
        SELECT c.id
        FROM cte1 c
        WHERE c.id = x.id - 1 
             ) 
     AND EXISTS
            (
        SELECT d.id
        FROM cte1 d
        WHERE d.id = x.id - 2
             ) 
    )
    OR (EXISTS (
        SELECT e.id
        FROM cte1 e
        WHERE e.id = x.id + 1
                )
       AND EXISTS(
        SELECT f.id
        FROM cte1 f
        WHERE f.id = x.id - 1        
                )
       )
ORDER BY x.visit_date;
```


### 602. Friend Requests II: Who Has the Most Friends (Medium)

```sql
WITH cte1 AS (SELECT
    t1.requester_id AS id
FROM
    RequestAccepted t1
    
UNION ALL

SELECT
    t2.accepter_id AS id
FROM
    RequestAccepted t2)

SELECT cte1.id, COUNT(*) as num
FROM
    cte1
GROUP BY cte1.id
ORDER BY num DESC
LIMIT 1
```

- Key point above is usage of "UNION ALL", and not UNION. We need to combine data from both the columns to see how many 'friend' relationships a user has. But we do not want to remove duplicates while Unioning the columns.
- Also, to find the one with the maximum, we use the simple idea of COUNT(\*), GROUP BY, ORDER BY COUNT(\*) and LIMIT 1. If the question asked to remove the count column, we would have taken the above as another cte and selected only the user column from that.

### 603. Consecutive Available Seats (Easy)

```sql
SELECT
    c1.seat_id
FROM
    Cinema c1

WHERE 
    c1.free = 1
    AND EXISTS (
        SELECT
                c2.seat_id
        FROM
            Cinema c2
        WHERE 
            (c2.seat_id = c1.seat_id + 1
                OR
             c2.seat_id = c1.seat_id - 1)
             AND c2.free = 1
    )
ORDER BY c1.seat_id;
```

Given Solution

```sql
select distinct a.seat_id
from cinema a join cinema b
  on abs(a.seat_id - b.seat_id) = 1
  and a.free = true and b.free = true
order by a.seat_id
;
```

Few things to note here, mostly tricks:
-   The logic to say that x and y are consecutive can be expressed as x=y-1 OR x=y+1 or simply as abs(x-y)=1. The second one seems to be more succinct and stylish, however, not sure whether calling a function abs() affects performance or not.
-   I have used correlated subquery. It could impact performance. On the other hand, the leetcode solution uses a cross-join instead. This is a handy technique. Sometimes in situations where you want to filter based on conditions of "interrelated rows", one can think of getting a cartesian product first, and then doing a simpler filtering from the cartesian product table rather than subqueries or window functions.


### 607. Sales Person (Easy)

This is a nice problem! Although labelled as easy, this one is very tricky.
- If one uses simply joining on three tables and filtering on not equal to RED, it will fail. 
- One also realises confusing stuff that happens when using both LEFT/RIGHT (OUTER) and INNER JOINS in the same bunch of join statements, note that JOINS are executed in pairs in the order they are mentioned.
- Best way is to use Subqueries for this problem. either use NOT IN or NOT EXISTS subquery.

```sql
SELECT 
    s1.name
FROM
    SalesPerson s1
WHERE
    NOT EXISTS (
        SELECT o1.sales_id
        FROM
         Orders o1
         INNER JOIN Company c1 ON c1.com_id = o1.com_id
        WHERE
            c1.name = 'RED'
            AND o1.sales_id = s1.sales_id
    );
```

### 608. Tree Node (Medium)

Not very complex when you can figure out the simple conditions of a
    - Root Node - parent node field is NULL
    - Leaf Node - it itself is not present in the parent node column
    - Inner - Everything else

```sql
SELECT
    t1.id,
    CASE 
    WHEN t1.p_id IS NULL THEN 'Root'
    WHEN (SELECT COUNT(*) FROM Tree t2 WHERE t2.p_id = t1.id) = 0 THEN 'Leaf'
    ELSE 'Inner' 
    END AS type
FROM
    Tree t1
ORDER BY t1.id;
```

### 610. Triangle Judgement (Easy)

Just evaluate the triangle inequality using an CASE WHEN or an IF statement.

```sql
SELECT
    t.x,
    t.y,
    t.z,
    CASE
        WHEN ((t.x + t.y > t.z) AND (t.x + t.z > t.y) AND (t.y + t.z > t.x))
            THEN 'Yes'
        ELSE
            'No'
    END AS triangle
FROM
    Triangle t;
```

### 612. Shortest Distance in a Plane (Medium)

Nice Problem. Things to consider :
    - A classic case when CROSS JOIN is needed. 
    - Note that MySQL supports power calculations with POW(x,y) function and not with operators such as ^ or \*\*.
    - Note that MySQL supports SQRT function.
    - Do not forget to ROUND off
    - Most importantly, you need a way to rule out the rows that are same points, either specify that in the JOIN condition sopmehow or using a WHERE clause like I used below.
    - There is scope for optimization. We could make this more optimized if we do not calculate for all possible combinations of cross join, including same points.
```sql
WITH cte1 AS (
SELECT
    p1.x as x1,
    p1.y as y1,
    p2.x as x2,
    p2.y as y2,
    SQRT(POW((p1.x - p2.x),2) + POW((p1.y - p2.y),2)) AS distance
FROM    
    Point2D p1, Point2D p2)
SELECT 
    ROUND(MIN(cte1.distance),2) AS shortest
FROM cte1
WHERE distance <> 0;
```

### 614. Second Degree Follower (Medium)

The solution involves a subquery in the WHERE clause which can be written as 'IN' subquery or 'EXISTS' subquery. IN is simpler in this case.

```sql
SELECT
    f1.followee AS follower,
    COUNT(f1.follower) AS num
FROM
    Follow f1
WHERE f1.followee IN (
    SELECT
        f2.follower
    FROM
        Follow f2
)
GROUP BY f1.followee
ORDER BY f1.followee;
```


### 615. Average Salary: Departments VS Company (Hard)

This is again one of those problems that need to be broken down in processing stages (in order to provide output in the manner provided in leetcode output).

First, you need the date_format function to get the pay_month across all rows
Second, you need to compute the monthly company and dept wise averages across those rows
Third, you need to select distinct pay_month, department_id and whether or not the dept is higher/lower/same as company monthly avg by CASE WHEN.

There seems to be a trickier solution provided in leetcode that is hard to think in terms of, by JOINs.

```sql
WITH cte1 AS (
    SELECT
        s1.id,
        s1.employee_id,
        e1.department_id,
        s1.amount,
        s1.pay_date,
        DATE_FORMAT(s1.pay_date, '%Y-%m') AS pay_month 
    FROM
        Salary s1
        INNER JOIN Employee e1 ON e1.employee_id = s1.employee_id
    ),
cte2 AS (
    
    SELECT
        cte1.id,
        cte1.employee_id,
        cte1.department_id,
        cte1.amount,
        cte1.pay_date,
        cte1.pay_month,
        AVG(cte1.amount) OVER(PARTITION BY cte1.department_id, cte1.pay_month) AS dept_monthly_avg,
        AVG(cte1.amount) OVER(PARTITION BY cte1.pay_month) AS company_monthly_avg 
    FROM
        cte1
)
    
SELECT DISTINCT
    cte2.pay_month,
    cte2.department_id,
    CASE
        WHEN cte2.dept_monthly_avg = cte2.company_monthly_avg THEN 'same'
        WHEN cte2.dept_monthly_avg > cte2.company_monthly_avg THEN 'higher'
        ELSE 'lower'
    END AS comparison
FROM  
    cte2
    ;
```

### 619. Biggest Single Number  (Easy)

The only important thing to remember is that to have 'null' as a possbile output, it helps to wrap it inside another query.

```sql
SELECT MAX(t1.num) as num
FROM
(SELECT 
    m.num
FROM
    MyNumbers m
GROUP BY m.num
HAVING COUNT(*) = 1) AS t1;
```

### 620. Not Boring Movies (Easy)

Remember the usage of MOD(field, integer) to check to check even/odd or multiple of a particular integer, comes in handy.

```sql
Select
    c1.*
FROM
    Cinema c1
WHERE
    MOD(c1.id,2) = 1
    AND c1.description NOT LIKE '%boring%'
ORDER BY c1.rating DESC;
```
### 626. Exchange Seats (Medium)

This is an interesting one. Things to note:
- Firstly, the problem does not say update the table, so it will be a select statement. Its important to know that.
- Secondly. its all about identifying the scenarios. There are only three and in the below order, involves conbination of CASE statement abd subqueries for each condition. When id is odd but is the last id, do nothing. When the id is odd but not the last id, pick up the name from id+1 row, when the id is even, pick up the name from id-1 row.

```sql
SELECT

    s1.id,
    CASE 
        WHEN (MOD(s1.id,2) = 1 AND s1.id = (SELECT MAX(s2.id) FROM Seat s2))
            THEN s1.student
        WHEN MOD(s1.id,2) = 0 
            THEN (SELECT s3.student FROM Seat s3 WHERE s3.id = s1.id-1)
        WHEN MOD(s1.id,2) = 1 
            THEN (SELECT s4.student FROM Seat s4 WHERE s4.id = s1.id+1)
    END AS student

FROM 
    Seat s1;
```

### 627. Swap Salary (Easy)

Another interesting problem. Things to note:

1. The problem explicitly states usage of UPDATE STATEMENT, and a single UPDATE single statement, no select statement.
2. We need to remember that fact that CASE WHEN works with UPDATE statement as well.

```sql
UPDATE Salary s1
    SET s1.sex = CASE WHEN s1.sex = 'm' THEN 'f' ELSE 'm' END;
```


