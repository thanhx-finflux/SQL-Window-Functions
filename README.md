# SQL-Window-Functions
This repository explores window functions in PostgreSQL, including 'ROW_NUMBER ()', 'LEAD ()', 'LAG ()',  and 'NTH_VALUE()'.

### Project Overview
- Assign row numbers to data.
- Partition data by specific columns for grouped calculations.
- Use LEAD () and LAG() to access values from the next or previous rows.
- Retrieve specific values like the first or nth record in a partition.
- Combine window functions with Common Table Expressions (CTEs) for complete analyses.
```sql
### Queries
###############################################################
###############################################################
-- Project: SQL Window Functions
###############################################################
###############################################################
#############################
-- Retrieve all the data in the project_db database
#############################
SELECT * FROM employees;

SELECT * FROM departments;

SELECT * FROM regions;

SELECT * FROM customers;

SELECT * FROM sales;

#############################
-- The ROW_NUMBER() and OVER() to assign numbers to each row
#############################
-- Assign numbers to each row of the departments table
SELECT department, division, ROW_NUMBER() OVER () AS Row_N
FROM departments
ORDER BY Row_N;

-- Assign numbers to each row of the department for the Entertainment division
SELECT *, ROW_NUMBER() OVER () AS Row_N
FROM departments
WHERE
    division = 'Entertainment'
ORDER BY Row_N ASC;

-- Retrieve all the data from the employees table
SELECT * FROM employees;

-- Order by inside OVER()
-- Retrieve a list of employee_id, first_name, hire_date, and department of all employees in the sports department, ordered by the hire date
SELECT
    employee_id,
    first_name,
    hire_date,
    department,
    ROW_NUMBER() OVER (
        ORDER BY hire_date ASC
    ) AS Row_N
FROM employees
WHERE
    department = 'Sports'
ORDER BY Row_N ASC;

-- Order by multiple columns
SELECT
    employee_id,
    first_name,
    hire_date,
    salary,
    department,
    ROW_NUMBER() OVER (
        ORDER BY hire_date ASC, salary DESC
    ) AS Row_N
FROM employees
WHERE
    department = 'Sports'
ORDER BY Row_N ASC;

-- Ordering inside and outside the OVER() clause
SELECT
    employee_id,
    first_name,
    hire_date,
    salary,
    department,
    ROW_NUMBER() OVER (
        ORDER BY hire_date ASC, salary DESC
    ) AS Row_N
FROM employees
WHERE
    department = 'Sports'
ORDER BY employee_id;

#############################
-- PARTITION BY clause inside OVER()
#############################
-- Retrieve the employee_id, first_name, hire_date of employees for different departments
SELECT
    employee_id,
    first_name,
    department,
    hire_date,
    ROW_NUMBER() OVER (
        PARTITION BY
            department
    ) AS Row_N
FROM employees
ORDER BY department ASC;

-- Order by the hire_date
SELECT
    employee_id,
    first_name,
    department,
    hire_date,
    ROW_NUMBER() OVER (
        PARTITION BY
            department
        ORDER BY hire_date ASC
    ) AS Row_N
FROM employees
ORDER BY department ASC;

#############################
-- PARTITION BY with CTE
#############################
-- Retrieve all data from the sales and customers tables
SELECT * FROM sales;

SELECT * FROM customers;

--Create a common table expression to retrieve the customer_id, customer_name, segment and how many times the customer has purchased from the mall
WITH
    customer_purchase AS (
        SELECT s.customer_id, c.customer_name, c.segment, COUNT(*) AS purchase_count
        FROM sales s
            JOIN customers c ON s.customer_id = c.customer_id
        GROUP BY
            s.customer_id,
            c.customer_name,
            c.segment
        ORDER BY customer_id
    )
SELECT *
FROM customer_purchase
ORDER BY customer_id;

-- Number each customer by how many purchases they've made, Same CTE as above
WITH
    customer_purchase AS (
        SELECT s.customer_id, c.customer_name, c.segment, COUNT(*) AS purchase_count
        FROM sales AS s
            JOIN customers AS c ON s.customer_id = c.customer_id
        GROUP BY
            s.customer_id,
            c.customer_name,
            c.segment
        ORDER BY customer_id
    )
SELECT
    customer_id,
    customer_name,
    segment,
    purchase_count,
    ROW_NUMBER() OVER (
        ORDER BY purchase_count DESC
    ) AS Row_N
FROM customer_purchase
ORDER BY purchase_count DESC;

-- Number each customer by their customer segment and by how many purchases they've made in descending order
WITH
    customer_purchase AS (
        SELECT s.customer_id, c.customer_name, c.segment, COUNT(*) AS purchase_count
        FROM sales s
            JOIN customers c ON s.customer_id = c.customer_id
        GROUP BY
            s.customer_id,
            c.customer_name,
            c.segment
        ORDER BY customer_id
    )
SELECT
    customer_id,
    customer_name,
    segment,
    purchase_count,
    ROW_NUMBER() OVER (
        PARTITION BY
            segment
        ORDER BY purchase_count DESC
    ) AS Row_N
FROM customer_purchase
ORDER BY segment, purchase_count DESC;

#############################
-- Fetching: LEAD() & LAG() clauses
#############################
-- Retrieve all employees' first name, department, salar,y and the salary after that employee
SELECT
    first_name,
    department,
    salary,
    LEAD(salary, 1) OVER () AS next_salary
FROM employees;

-- Retrieve all employees' first name, department, salary, and the salary before that employee
SELECT
    first_name,
    department,
    salary,
    LAG(salary, 1) OVER () AS previous_salary
FROM employees;

-- Retrieve all employees' first name, department, salary, and the salary after that employee in order of their salaries
SELECT
    employee_id,
    first_name,
    department,
    salary,
    LEAD(salary, 1) OVER (
        ORDER BY salary DESC
    ) AS next_lower_salary -- Fetches the next lower salary
FROM employees;

SELECT
    first_name,
    department,
    salary,
    LEAD(salary, 1) OVER (
        ORDER BY salary DESC
    ) AS next_lower_salary,
    salary - LEAD(salary, 1) OVER (
        ORDER BY salary DESC
    ) AS salary_difference
FROM employees;

-- Retrieve all employees' first name, department, salary, and the salary before that employee in order of their salaries in descending order. Call the new column closest_higher_salary
SELECT
    first_name,
    department,
    salary,
    LAG(salary, 1) OVER (
        ORDER BY salary DESC
    ) AS closest_higher_salary,
    salary - LAG(salary, 1) OVER (
        ORDER BY salary DESC
    ) AS salary_difference
FROM employees;

-- Retrieve all employees' first name, department, salary, and the salary after that employee for each department in descending order of their salaries. Call the new column closest_lowest_salary
SELECT
    first_name,
    department,
    salary,
    LEAD(salary, 1) OVER (
        PARTITION BY
            department
        ORDER BY salary DESC
    ) AS closest_lowest_salary,
    salary - LEAD(salary, 1) OVER (
        PARTITION BY
            department
        ORDER BY salary DESC
    ) AS salary_difference
FROM employees;

-- What do you think this query will return?
SELECT
    first_name,
    department,
    salary,
    LEAD(salary, 1) OVER (
        ORDER BY salary DESC
    ) closest_salary,
    LEAD(salary, 2) OVER (
        ORDER BY salary DESC
    ) next_closest_salary
FROM employees
WHERE
    department = 'Clothing';

#############################
-- FIRST_VALUE() clause with the OVER() clause
#############################
-- Retrieve the first_name, last_name, department, and hire_date of all employees. Add a new column called first_emp_date that returns the hire date of the first hired employee
SELECT
    first_name,
    last_name,
    department,
    hire_date,
    FIRST_VALUE(hire_date) OVER (
        ORDER BY hire_date
    ) AS first_emp_date
FROM employees;

-- Find the difference between the hire date of the first employee hired and every other employee
SELECT
    first_name,
    last_name,
    department,
    hire_date,
    FIRST_VALUE(hire_date) OVER (
        ORDER BY hire_date
    ) AS first_emp_date
FROM employees
ORDER BY hire_date;

-- Partition by department
SELECT
    first_name,
    last_name,
    department,
    hire_date,
    FIRST_VALUE(hire_date) OVER (
        PARTITION BY
            department
        ORDER BY hire_date
    ) AS first_emp_date
FROM employees
ORDER BY department, hire_date;

-- Find the difference between the hire date of the first employee hired and every other employee, partitioned by department
SELECT *, AGE (hire_date, first_emp_date) AS date_difference
FROM (
        SELECT
            first_name, last_name, department, hire_date, FIRST_VALUE(hire_date) OVER (
                PARTITION BY
                    department
                ORDER BY hire_date
            ) AS first_emp_date
        FROM employees
    ) AS a
ORDER BY department;

-- Alternative way
SELECT
    first_name,
    last_name,
    department,
    hire_date,
    FIRST_VALUE(hire_date) OVER (
        PARTITION BY
            department
        ORDER BY hire_date
    ) AS first_emp_date,
    hire_date - FIRST_VALUE(hire_date) OVER (
        PARTITION BY
            department
        ORDER BY hire_date
    ) AS date_difference
FROM employees
ORDER BY department;

-- Or
SELECT
    first_name,
    last_name,
    department,
    hire_date,
    MIN(hire_date) OVER (
        PARTITION BY
            department
        ORDER BY hire_date
    ) AS first_emp_date,
    hire_date - MIN(hire_date) OVER (
        PARTITION BY
            department
        ORDER BY hire_date
    ) AS date_difference
FROM employees
ORDER BY department;

-- Return the first salary for different departments. Order by salary in descending order
SELECT
    *,
    first_salary - salary AS salary_difference
FROM (
        SELECT
            first_name, email, department, salary, FIRST_VALUE(salary) OVER (
                PARTITION BY
                    department
                ORDER BY salary DESC
            ) first_salary
        FROM employees
    ) AS a;

-- OR
SELECT
    *,
    first_salary - salary AS salary_difference
FROM (
        SELECT
            first_name, email, department, salary, MAX(salary) OVER (
                PARTITION BY
                    department
                ORDER BY salary DESC
            ) first_salary
        FROM employees
    ) AS a;

-- Return the first salary for different departments ordered by the first_name in ascending order
SELECT
    *,
    first_salary - salary AS salary_difference
FROM (
        SELECT
            first_name, email, department, salary, FIRST_VALUE(salary) OVER (
                PARTITION BY
                    department
                ORDER BY first_name ASC
            ) AS first_salary
        FROM employees
    ) AS a;

-- Return the fifth salary for different departments ordered by the first_name in ascending order
SELECT
    first_name,
    email,
    department,
    salary,
    NTH_VALUE(salary, 5) OVER (
        PARTITION BY
            department
        ORDER BY first_name ASC
    ) AS fifth_salary
FROM employees;
```
### Why Window Functions
Window functions are a powerful SQL feature for performing calculations across rows related to the current row without collapsing the result set using GROUP BY.
- Ranking and numbering rows.
- Calculating running totals or differences.
- Accessing data from adjacent rows.
- Analyzing trends within groups.

### Contact
Thanh Xuyen, Nguyen

LinkedIn: [xuyen-thanh-nguyen-0518](https://www.linkedin.com/in/xuyen-thanh-nguyen-0518/)

Email: thanhxuyen.nguyen@outlook.com
