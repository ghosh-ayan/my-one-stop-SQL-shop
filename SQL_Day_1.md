
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
