# Subqueries (Correlated & Non-Correlated) — FAANG SQL Interview Guide

> A complete interview-oriented guide built around **real non-premium LeetCode SQL questions**. Instead of learning theory separately, every subquery pattern is introduced through practical interview problems.

---

# Table of Contents

- [Question 1 — Employees Earning More Than Their Managers](#question-1--employees-earning-more-than-their-managers)
- [Question 2 — Department Highest Salary](#question-2--department-highest-salary)
- [Question 3 — Rising Temperature](#question-3--rising-temperature)
- [Question 4 — Customers Who Never Order](#question-4--customers-who-never-order)
- [Question 5 — Second Highest Salary](#question-5--second-highest-salary)

---

# Question 1 — Employees Earning More Than Their Managers

**LeetCode:** https://leetcode.com/problems/employees-earning-more-than-their-managers/

**Difficulty:** Easy

## Companies Asking

Google • Amazon • Microsoft • Apple • Meta • Adobe • Oracle • Salesforce

---

## Problem Statement

Given an `Employee` table:

| id | name | salary | managerId |
|----|------|--------|-----------|

Return employees whose salary is **strictly greater than their manager's salary**.

Output:

| Employee |

---

## Interview Pattern

This is the classic **Correlated Relationship Problem**.

Although many candidates immediately use a JOIN, interviewers often ask:

> "Can you solve it using a subquery?"

This question introduces **correlated subqueries**.

---

## Approach & Technique

### Correlated Subquery

The inner query depends on the outer row.

For every employee:

1. Find that employee's manager
2. Compare salaries
3. Return employee if salary is larger

Visualization

```
Employee Row
      │
      ▼
ManagerId = 3
      │
      ▼
Inner Query executes
      │
      ▼
Manager Salary
      │
      ▼
Compare
```

The inner query executes once for every employee.

---

## SQL Solution

```sql
SELECT name AS Employee
FROM Employee e
WHERE salary >
(
    SELECT salary
    FROM Employee
    WHERE id = e.managerId
);
```

---

## Explanation

Outer query

```sql
SELECT name
FROM Employee e
```

Iterates through every employee.

---

Inner query

```sql
SELECT salary
FROM Employee
WHERE id = e.managerId
```

Notice:

```
e.managerId
```

comes from the outer query.

That makes it a **Correlated Subquery**.

---

Comparison

```sql
salary >
(
    ...
)
```

Returns only employees earning more than their managers.

---

## Alternative (JOIN)

Many interviewers expect both solutions.

```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m
ON e.managerId = m.id
WHERE e.salary > m.salary;
```

---

## Complexity

### Correlated Subquery

Time

```
O(N × Lookup)
```

Without indexes

```
O(N²)
```

With PK index

```
≈ O(N log N)
```

---

JOIN

Usually faster because the optimizer transforms it efficiently.

---

## Common Pitfalls

### Forgetting alias

Wrong

```sql
managerId=id
```

Correct

```sql
id = e.managerId
```

---

### Using >=

Question asks

```
strictly greater
```

Use

```sql
>
```

---

### Returning manager instead of employee

Very common interview mistake.

---

## LLM-Proof Follow-Up Questions

### Follow-Up 1

Return employees earning at least **20% more** than their managers.

---

### Follow-Up 2

Return manager names alongside employee names.

---

### Follow-Up 3

Return employees whose salary exceeds the **average salary of all managers**.

---

# Interview Takeaway

Whenever the inner query references the outer query,

```
Outer Row
      ↓
Inner Query
      ↓
Comparison
```

you are dealing with a **Correlated Subquery**.

---

# Question 2 — Department Highest Salary

**LeetCode:** https://leetcode.com/problems/department-highest-salary/

**Difficulty:** Medium

---

## Companies Asking

Google • Amazon • Meta • Microsoft • Bloomberg • Oracle • Apple • Uber

---

## Problem Statement

Tables

Department

| id | name |

Employee

| id | name | salary | departmentId |

Return every employee having the **highest salary inside their department**.

Output

| Department | Employee | Salary |

If multiple employees tie for highest salary, return all of them.

---

## Interview Pattern

This problem introduces one of the most common interview patterns:

> Correlated Aggregate Subquery

The aggregate changes depending on the current department.

---

## Approach & Technique

For every employee

```
Employee
    │
    ▼
Find maximum salary
inside same department
    │
    ▼
Compare employee salary
```

Since the department changes for every employee,

the MAX query depends on the outer query.

---

Visualization

```
Employee (HR)
        │
        ▼
MAX salary in HR
        │
        ▼
Compare

Employee (IT)
        │
        ▼
MAX salary in IT
        │
        ▼
Compare
```

---

## SQL Solution

```sql
SELECT
    d.name AS Department,
    e.name AS Employee,
    e.salary AS Salary
FROM Employee e
JOIN Department d
ON e.departmentId = d.id
WHERE salary =
(
    SELECT MAX(salary)
    FROM Employee
    WHERE departmentId = e.departmentId
);
```

---

## Explanation

Outer query

Gets every employee.

---

Inner query

```sql
SELECT MAX(salary)
FROM Employee
WHERE departmentId = e.departmentId
```

This computes the highest salary **inside the employee's own department**.

The condition

```sql
departmentId = e.departmentId
```

makes it correlated.

---

Comparison

```sql
salary =
(
   MAX(...)
)
```

returns everyone tied for first place.

---

## Why NOT LIMIT 1?

Many beginners write

```sql
ORDER BY salary DESC
LIMIT 1
```

That returns only one employee overall.

The question needs

```
Highest salary
inside EACH department
```

---

## Alternative Solution Using Derived Table

This is usually more efficient.

```sql
SELECT
    d.name,
    e.name,
    e.salary
FROM Employee e
JOIN Department d
ON e.departmentId=d.id
JOIN
(
    SELECT
        departmentId,
        MAX(salary) AS max_salary
    FROM Employee
    GROUP BY departmentId
) t
ON e.departmentId=t.departmentId
AND e.salary=t.max_salary;
```

This uses a **Non-Correlated Subquery** because the inner query runs once.

---

## Correlated vs Non-Correlated

| Correlated | Non-Correlated |
|------------|----------------|
| Runs per employee | Runs once |
| Simpler | Faster |
| Depends on outer row | Independent |
| Common in interviews | Common in production |

---

## Complexity

Correlated

```
O(N²)
```

without indexes.

---

Derived Table

```
O(N log N)
```

typically better.

---

## Common Pitfalls

### Using LIMIT 1

Wrong.

Need highest salary **per department**.

---

### Missing ties

Using

```sql
ROW_NUMBER()
```

without handling duplicates.

---

### Forgetting department correlation

Wrong

```sql
MAX(salary)
```

Correct

```sql
MAX(salary)
WHERE departmentId=e.departmentId
```

---

## LLM-Proof Follow-Up Questions

### Follow-Up 1

Return top **3 salaries per department**.

---

### Follow-Up 2

Return employees earning above department average.

---

### Follow-Up 3

Return second highest salary in every department without window functions.

---

# Patterns Learned So Far

| Pattern | Question |
|----------|----------|
| Simple Correlated Subquery | Employees Earning More Than Managers |
| Correlated Aggregate | Department Highest Salary |
| Derived Table (Non-Correlated) | Department Highest Salary Optimization |

---

# Question 3 — Rising Temperature

**LeetCode:** https://leetcode.com/problems/rising-temperature/

**Difficulty:** Easy

---

## Companies Asking

Google • Amazon • Meta • Microsoft • Bloomberg • Apple • Oracle • Uber

---

## Problem Statement

Given the `Weather` table:

| id | recordDate | temperature |
|----|------------|-------------|

Return the IDs of all dates where the temperature is **higher than the previous day's temperature**.

Output:

| id |

---

## Interview Pattern

This question is usually solved with a self-join, but interviewers frequently ask:

> "Can you solve it using a correlated subquery?"

This tests whether you understand how to retrieve information from a related row without using joins.

---

## Approach & Technique

For every weather record:

1. Find the record exactly **one day before**
2. Retrieve that day's temperature
3. Compare temperatures

Since the previous day's record depends on the current row, the inner query references the outer query.

---

### Visualization

```
Current Row
     │
     ▼
recordDate = 2023-05-08
     │
     ▼
Find record where
recordDate = 2023-05-07
     │
     ▼
Compare temperatures
```

---

## SQL Solution (Correlated Subquery)

```sql
SELECT id
FROM Weather w
WHERE temperature >
(
    SELECT temperature
    FROM Weather
    WHERE recordDate = DATE_SUB(w.recordDate, INTERVAL 1 DAY)
);
```

---

## Explanation

Outer query

```sql
SELECT id
FROM Weather w
```

Processes every day's weather.

---

Inner query

```sql
SELECT temperature
FROM Weather
WHERE recordDate = DATE_SUB(w.recordDate, INTERVAL 1 DAY)
```

The previous date depends on

```sql
w.recordDate
```

which comes from the outer query.

Therefore, this is a **Correlated Subquery**.

---

Comparison

```sql
temperature >
(
    previous day's temperature
)
```

Returns only warmer days.

---

## Alternative Solution (Self Join)

```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

Interviewers often prefer this version because joins are generally more efficient.

---

## Complexity

Correlated Subquery

```
O(N × Lookup)
```

With indexed dates:

```
≈ O(N log N)
```

Without indexes:

```
O(N²)
```

---

Self Join

Usually optimized better by modern SQL engines.

---

## Common Pitfalls

### Comparing IDs instead of dates

Wrong

```sql
id - 1
```

IDs are not guaranteed to represent consecutive dates.

---

### Forgetting missing dates

Some datasets skip dates.

Always compare using

```sql
DATE_SUB()
```

rather than assuming yesterday's ID exists.

---

### Using >=

Question asks

```
strictly higher
```

Use

```sql
>
```

---

## LLM-Proof Follow-Up Questions

### Follow-Up 1

Return dates where today's temperature is at least **5° higher** than yesterday.

---

### Follow-Up 2

Return the largest temperature increase between consecutive days.

---

### Follow-Up 3

Return all streaks of increasing temperatures.

---

# Interview Takeaway

Whenever you must compare the current row with a previous or next row, consider:

- Correlated Subquery
- Self Join
- Window Functions (if allowed)

---

# Question 4 — Customers Who Never Order

**LeetCode:** https://leetcode.com/problems/customers-who-never-order/

**Difficulty:** Easy

---

## Companies Asking

Amazon • Google • Meta • Microsoft • Apple • Oracle • IBM

---

## Problem Statement

Tables

Customer

| id | name |

Orders

| id | customerId |

Return all customers who have **never placed an order**.

Output:

| Customers |

---

## Interview Pattern

This is one of the most important interview questions for understanding

- Non-Correlated Subqueries
- `NOT IN`
- `NOT EXISTS`

Interviewers often ask all three approaches.

---

## Approach & Technique

The idea is simple.

Retrieve every customer whose ID is **not present** in the Orders table.

The inner query runs only once.

```
Orders
    │
    ▼
List of customer IDs
    │
    ▼
Outer query removes them
```

Since the inner query does **not** depend on the outer query, it is a **Non-Correlated Subquery**.

---

## SQL Solution (NOT IN)

```sql
SELECT name AS Customers
FROM Customers
WHERE id NOT IN
(
    SELECT customerId
    FROM Orders
);
```

---

## Explanation

Inner query

```sql
SELECT customerId
FROM Orders
```

Runs only once.

Produces

```
1
3
4
5
```

Outer query removes every matching customer.

---

## Better Solution (NOT EXISTS)

```sql
SELECT name AS Customers
FROM Customers c
WHERE NOT EXISTS
(
    SELECT 1
    FROM Orders o
    WHERE o.customerId = c.id
);
```

---

## Why Many Interviewers Prefer NOT EXISTS

`NOT IN` behaves unexpectedly when NULL values exist.

Example

Orders

| customerId |
|------------|
| NULL |

Then

```sql
NOT IN
```

may return **no rows**.

`NOT EXISTS` avoids this issue.

---

## NOT IN vs NOT EXISTS

| NOT IN | NOT EXISTS |
|---------|------------|
| Simple syntax | Safer |
| Problems with NULL | Handles NULL correctly |
| Good for small datasets | Preferred in interviews |
| Non-Correlated | Correlated |

---

## Complexity

NOT IN

```
O(N + M)
```

if optimized.

---

NOT EXISTS

```
O(N × Lookup)
```

With indexes:

```
≈ O(N log M)
```

---

## Common Pitfalls

### Ignoring NULL

This is one of the most common SQL interview mistakes.

---

### Forgetting alias

Wrong

```sql
customerId=id
```

Correct

```sql
o.customerId = c.id
```

---

### Assuming NOT IN is always faster

Performance depends on

- database engine
- indexes
- optimizer

---

## LLM-Proof Follow-Up Questions

### Follow-Up 1

Return customers who placed **exactly one** order.

---

### Follow-Up 2

Return customers who ordered in 2024 but not in 2025.

---

### Follow-Up 3

Return customers who ordered every available product.

---

# Patterns Learned So Far

| Question | Pattern | Subquery Type |
|----------|---------|---------------|
| Employees Earning More Than Their Managers | Salary comparison | Correlated |
| Department Highest Salary | Aggregate comparison | Correlated |
| Rising Temperature | Previous-row lookup | Correlated |
| Customers Who Never Order | Membership filtering | Non-Correlated (`NOT IN`) / Correlated (`NOT EXISTS`) |

---

# Question 5 — Second Highest Salary

**LeetCode:** https://leetcode.com/problems/second-highest-salary/

**Difficulty:** Medium

---

## Companies Asking

Google • Amazon • Meta • Microsoft • Apple • Bloomberg • Oracle • Adobe • Salesforce

---

## Problem Statement

Given the `Employee` table:

| id | salary |
|----|--------|

Return the **second highest distinct salary**.

If there is no second highest salary, return `NULL`.

Output:

| SecondHighestSalary |

---

## Interview Pattern

This is one of the most frequently asked SQL interview questions because it evaluates your understanding of:

- Non-Correlated Subqueries
- Aggregate Functions
- Nested Queries
- DISTINCT
- NULL handling

Interviewers often extend this into "Nth Highest Salary" as a follow-up.

---

## Approach & Technique

The key observation is:

```
Second Highest Salary
=
Largest salary
smaller than
Maximum salary
```

Visualization

```
All Salaries

9000
7500   ← Answer
6000
5000
4000

Maximum Salary
      │
      ▼
Ignore it
      │
      ▼
Find MAX again
```

Instead of sorting the table, we remove the highest salary first.

The inner query runs **once**, so this is a **Non-Correlated Subquery**.

---

## SQL Solution (Aggregate Subquery)

```sql
SELECT
(
    SELECT MAX(salary)
    FROM Employee
    WHERE salary <
    (
        SELECT MAX(salary)
        FROM Employee
    )
) AS SecondHighestSalary;
```

---

## Explanation

### Step 1

Find the highest salary.

```sql
SELECT MAX(salary)
FROM Employee
```

Suppose it returns

```
9000
```

---

### Step 2

Ignore that salary.

```sql
WHERE salary < 9000
```

Remaining salaries

```
7500
6000
5000
```

---

### Step 3

Find the maximum again.

```sql
MAX(salary)
```

Returns

```
7500
```

---

## Alternative Solution (ORDER BY)

```sql
SELECT
(
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

---

## Why Use DISTINCT?

Suppose salaries are

```
9000
9000
7500
```

Without `DISTINCT`

```
OFFSET 1
```

still returns

```
9000
```

which is incorrect.

---

## Complexity

Aggregate Subquery

```
Time: O(N)
Space: O(1)
```

---

ORDER BY

```
Time: O(N log N)
```

because sorting is required.

---

## Common Pitfalls

### Forgetting DISTINCT

Example

```
9000
9000
8000
```

Incorrect answer:

```
9000
```

Correct answer:

```
8000
```

---

### Returning no rows

The problem requires

```
NULL
```

not an empty result.

Wrapping the subquery inside

```sql
SELECT (...)
```

ensures one row is always returned.

---

### Using LIMIT 2

`LIMIT 2` returns two rows instead of one value.

---

## LLM-Proof Follow-Up Questions

### Follow-Up 1

Return the **Nth highest salary**.

---

### Follow-Up 2

Return the third highest salary without using `LIMIT`.

---

### Follow-Up 3

Return the top three distinct salaries.

---

# Interview Pattern Summary

## Correlated vs. Non-Correlated Subqueries

| Feature | Correlated Subquery | Non-Correlated Subquery |
|----------|---------------------|-------------------------|
| Depends on outer query | Yes | No |
| Executes | Once per outer row | Once |
| Typical use cases | Comparisons, EXISTS, row-by-row filtering | Aggregates, IN, scalar values |
| Performance | Usually slower | Usually faster |
| Interview frequency | Very High | Very High |

---

## Decision Tree

```text
Need information from the current row?
        │
        ├── Yes
        │      │
        │      ▼
        │ Correlated Subquery
        │
        └── No
               │
               ▼
      Non-Correlated Subquery
```

---

## Recognizing Common Patterns

| Requirement | Pattern | Example |
|------------|---------|----------|
| Compare with related row | Correlated Subquery | Employees earning more than managers |
| Compare with department statistics | Correlated Aggregate | Department Highest Salary |
| Compare with previous/next row | Correlated Subquery | Rising Temperature |
| Filter using another table | `NOT EXISTS` / `NOT IN` | Customers Who Never Order |
| Compute a single aggregate value | Non-Correlated Scalar Subquery | Second Highest Salary |

---

# FAANG Interview Cheat Sheet

| If you see... | Think... |
|--------------|----------|
| "For each employee..." | Correlated Subquery |
| "Inside the same department..." | Correlated Aggregate |
| "Previous/Next row..." | Correlated Subquery or Self Join |
| "Exists / Doesn't exist..." | `EXISTS` / `NOT EXISTS` |
| "Highest/Second Highest/Nth Highest..." | Aggregate or Scalar Subquery |
| "Compare against a global value..." | Non-Correlated Subquery |

---

# Final Takeaways

1. A **Correlated Subquery** references columns from the outer query and is evaluated for each outer row.
2. A **Non-Correlated Subquery** is independent and typically executes once, making it easier for the optimizer to reuse.
3. Before writing a subquery, identify whether the inner query needs information from the current row.
4. For existence checks, prefer `EXISTS`/`NOT EXISTS` over `IN`/`NOT IN` when `NULL` values may appear.
5. Many correlated subqueries can be rewritten as joins or window-function queries for better performance, but interviewers often expect you to understand both approaches and their trade-offs.




