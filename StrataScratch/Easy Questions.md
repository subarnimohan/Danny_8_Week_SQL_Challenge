## Solutions to StrataScratch Free Easy Questions

Overview:
I have attempted the questions from Strata Scratch which is a good resource to test and improve SQL skills. Here are my solutions. Do chat with me if you have any questions

### Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments. Output just the absolute difference in salaries.

```sql
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

```
![image](https://github.com/user-attachments/assets/9f714216-19a1-441e-a98e-0a7069ec4b6e)



