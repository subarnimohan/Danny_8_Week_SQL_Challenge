## Solutions to StrataScratch Free Easy Questions

### Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments. Output just the absolute difference in salaries.

WITH tt AS(SELECT DISTINCT dt.department,
MAX(db.salary) OVER(partition by db.department_id ORDER BY db.salary DESC) as sal
FROM db_employee db
INNER JOIN db_dept dt
ON db.department_id=dt.id
WHERE dt.department IN ('marketing','engineering'))

select 
lead(sal,1) OVER (ORDER BY sal)-sal
from tt
LIMIT 1


