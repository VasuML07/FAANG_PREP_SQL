# Common Table Expressions (CTEs) — FAANG SQL Interview Guide (Part 1)

> GitHub Ready • LeetCode Focused • Interview Preparation

---

# Table of Contents

- [Problem 1 — Employees Earning More Than Their Managers (Easy)](#problem-1--employees-earning-more-than-their-managers-easy)
- [Problem 2 — Department Top Three Salaries (Hard)](#problem-2--department-top-three-salaries-hard)
- [Problem 3 — Human Traffic of Stadium (Hard)](#problem-3--human-traffic-of-stadium-hard)
- [Problem 4 — Consecutive Numbers (Medium)](#problem-4--consecutive-numbers-medium)
- [Problem 5 — Tree Node (Medium)](#problem-5--tree-node-medium)
- [Problem 6 — Exchange Seats (Medium)](#problem-6--exchange-seats-medium)
- [LeetCode Question Patterns](#leetcode-question-patterns)
- [LLM-Proof Questions](#llm-proof-questions)
- [Advanced Tricks & Performance](#advanced-tricks--performance)

---

# Problem 1 — Employees Earning More Than Their Managers (Easy)

**LeetCode #181**

Difficulty: **Easy**

Companies

```
Meta
Amazon
Google
Apple
Microsoft
Bloomberg
Adobe
```

LeetCode

https://leetcode.com/problems/employees-earning-more-than-their-managers/

---

## Problem Statement

Find employees whose salary is greater than their manager's salary.

Tables

Employee

| id | name | salary | managerId |
|---|---|---:|---:|

Return

| Employee |

---

## Interview Observation

Most candidates solve this using a self join.

A CTE makes the query significantly easier to read by separating employee data from manager data.

---

## Solution

```sql
-- Step 1: Create an employee lookup table
WITH EmployeeInfo AS (
    SELECT
        id,
        name,
        salary,
        managerId
    FROM Employee
)

-- Step 2: Compare employee salary with manager salary
SELECT
    e.name AS Employee
FROM EmployeeInfo e
JOIN EmployeeInfo m
ON e.managerId = m.id
WHERE e.salary > m.salary;
```

---

## Execution Flow

```
Employee

+----+-------+--------+-----------+
| id | name  | salary | managerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+

CTE
↓

EmployeeInfo

↓

Self Join

↓

Employee Salary > Manager Salary

↓

Answer
```

---

## Why This Works

Instead of repeatedly referencing Employee,

the CTE creates a reusable logical table.

```
Employee
      │
      ▼
 EmployeeInfo
      │
 ┌────┴────┐
 │         │
Employee Manager
```

This improves readability without changing logic.

---

## Edge Cases

### Employee has no manager

```
managerId IS NULL

Ignored automatically because INNER JOIN fails.
```

---

### Multiple employees reporting to same manager

Works correctly because join occurs independently.

---

### Equal salaries

Question asks strictly greater.

```
salary >
```

not

```
salary >=
```

---

## Complexity

| Operation | Complexity |
|-----------|------------|
| Build CTE | O(n) |
| Join | O(n) |
| Overall | O(n) |

Space

```
O(n)
```

(Logical only—many optimizers inline the CTE.)

---

## Interview Takeaways

✔ Great example of replacing repeated table references

✔ Makes self joins easier to understand

✔ Frequently appears as SQL warm-up question

---

# Problem 2 — Department Top Three Salaries (Hard)

**LeetCode #185**

Difficulty

**Hard**

Companies

```
Meta
Google
Amazon
Apple
Uber
LinkedIn
Airbnb
```

LeetCode

https://leetcode.com/problems/department-top-three-salaries/

---

## Problem Statement

Find employees whose salaries belong to the top three unique salaries within each department.

Tables

Employee

Department

Return

```
Department
Employee
Salary
```

---

## Interview Observation

Without CTEs, this query becomes deeply nested.

Using multiple CTEs makes every transformation obvious.

---

## Optimal Solution

```sql
-- Step 1: Join employees with departments
WITH EmployeeDepartment AS (

    SELECT
        e.id,
        e.name,
        e.salary,
        d.name AS department
    FROM Employee e
    JOIN Department d
    ON e.departmentId = d.id

),

-- Step 2: Rank unique salaries inside each department
RankedEmployees AS (

    SELECT
        department,
        name,
        salary,

        DENSE_RANK() OVER (

            PARTITION BY department
            ORDER BY salary DESC

        ) AS salary_rank

    FROM EmployeeDepartment

)

-- Step 3: Keep Top 3 salaries
SELECT
    department AS Department,
    name AS Employee,
    salary AS Salary

FROM RankedEmployees

WHERE salary_rank <= 3;
```

---

## Execution Pipeline

```
Employee
        \
         \
          JOIN
         /
Department

      │

EmployeeDepartment

      │

Window Function

(DENSE_RANK)

      │

Top 3

      │

Output
```

---

## Example

Employee

```
Engineering

Alice   120000
Bob     110000
Chris   110000
David   95000
Emma    90000
```

After ranking

```
120000 → Rank 1

110000 → Rank 2

110000 → Rank 2

95000 → Rank 3

90000 → Rank 4
```

Result

```
Alice

Bob

Chris

David
```

Notice

Both Bob and Chris remain because

```
DENSE_RANK()
```

does not skip duplicate salaries.

---

## Why CTE Helps

Without CTE

```
Nested Query

↓

Window Function

↓

Outer Query
```

With CTE

```
Join

↓

Rank

↓

Filter
```

Each transformation becomes an independent readable block.

---

## Common Mistake

Using

```sql
ROW_NUMBER()
```

instead of

```sql
DENSE_RANK()
```

Example

```
120000

110000

110000

95000
```

ROW_NUMBER

```
1

2

3

4
```

One employee with 110000 gets excluded incorrectly.

---

Correct

```
DENSE_RANK

1

2

2

3
```

---

## Edge Cases

### Duplicate salaries

Handled perfectly.

---

### Department with fewer than three employees

Return all.

---

### One employee only

Still Rank 1.

---

## Complexity

Window Function

```
O(n log n)
```

Sorting dominates.

Memory

```
O(n)
```

---

## Interview Insights

Interviewers are checking whether you know

- Window Functions

- DENSE_RANK()

- CTE decomposition

- Readable SQL

rather than one massive nested query.

---

## Pattern Learned So Far

| Problem | Pattern | CTE Purpose |
|----------|----------|-------------|
| Employees Earning More Than Managers | Self Join | Reuse table cleanly |
| Department Top Three Salaries | Window Ranking | Break ranking pipeline into readable stages |

---

**Continue with Part 2** for:

---

# Problem 3 — Human Traffic of Stadium (Hard)

**LeetCode #601**

Difficulty: **Hard**

Companies

```
Google
Meta
Amazon
Microsoft
ByteDance
Uber
```

LeetCode

https://leetcode.com/problems/human-traffic-of-stadium/

---

## Problem Statement

Table

```
Stadium
```

| id | visit_date | people |
|----|------------|--------|

Return all records that belong to **three or more consecutive IDs** where each day's attendance is **at least 100**.

Output should be ordered by `visit_date`.

---

## Interview Observation

This problem looks like a simple filtering problem, but the challenge is identifying **continuous sequences**.

A CTE pipeline makes the solution significantly easier to understand.

---

## Optimal Solution

```sql
-- Step 1: Keep only crowded days
WITH CrowdedDays AS (

    SELECT
        id,
        visit_date,
        people
    FROM Stadium
    WHERE people >= 100

),

-- Step 2: Create groups of consecutive ids
GroupedDays AS (

    SELECT
        id,
        visit_date,
        people,

        id - ROW_NUMBER() OVER (
            ORDER BY id
        ) AS grp

    FROM CrowdedDays

),

-- Step 3: Find groups having at least 3 rows
ValidGroups AS (

    SELECT grp

    FROM GroupedDays

    GROUP BY grp

    HAVING COUNT(*) >= 3

)

-- Step 4: Return qualifying rows
SELECT
    g.id,
    g.visit_date,
    g.people

FROM GroupedDays g
JOIN ValidGroups v
ON g.grp = v.grp

ORDER BY g.visit_date;
```

---

## Execution Flow

```
Original Table

↓

people >=100

↓

CrowdedDays

↓

ROW_NUMBER()

↓

id - row_number

↓

Same value

↓

Same Group

↓

Count >=3

↓

Answer
```

---

## Why `id - ROW_NUMBER()` Works

Example

```
id

5
6
7
8
```

ROW_NUMBER

```
1
2
3
4
```

Difference

```
5-1 = 4

6-2 = 4

7-3 = 4

8-4 = 4
```

Same difference

↓

Same consecutive sequence

---

If IDs are

```
5
6
9
10
```

Difference

```
4

4

6

6
```

Sequence automatically breaks.

---

## Visual Example

Filtered rows

```
+----+--------+
| id | people |
+----+--------+
| 5  | 120    |
| 6  | 160    |
| 7  | 190    |
| 8  | 130    |
|11  | 180    |
|12  | 170    |
+----+--------+
```

After grouping

```
+----+------------+
| id | group      |
+----+------------+
| 5  | 4          |
| 6  | 4          |
| 7  | 4          |
| 8  | 4          |
|11  | 6          |
|12  | 6          |
+----+------------+
```

Only group **4** has at least three rows.

---

## Edge Cases

### Attendance below 100

Removed before grouping.

---

### Missing IDs

Automatically start a new group.

---

### Four, five or ten consecutive rows

Entire group is returned.

---

## Complexity

Sorting

```
O(n log n)
```

Grouping

```
O(n)
```

Overall

```
O(n log n)
```

---

## Interview Takeaways

Recognize this pattern immediately:

```
Consecutive Rows

↓

ROW_NUMBER()

↓

id - row_number

↓

Grouping
```

This pattern appears repeatedly in FAANG SQL interviews.

---

# Problem 4 — Consecutive Numbers (Medium)

**LeetCode #180**

Difficulty: **Medium**

Companies

```
Meta
Amazon
Google
Microsoft
Oracle
Apple
```

LeetCode

https://leetcode.com/problems/consecutive-numbers/

---

## Problem Statement

Table

```
Logs
```

| id | num |

Return numbers that appear at least **three consecutive times**.

---

## Interview Observation

Many people solve this using three self joins.

The cleaner interview solution uses a CTE with window functions.

---

## Optimal Solution

```sql
WITH OrderedLogs AS (

    SELECT
        id,
        num,

        LAG(num,1) OVER(
            ORDER BY id
        ) AS prev1,

        LAG(num,2) OVER(
            ORDER BY id
        ) AS prev2

    FROM Logs

)

SELECT DISTINCT
    num AS ConsecutiveNums

FROM OrderedLogs

WHERE num = prev1
AND num = prev2;
```

---

## Execution Flow

```
Logs

↓

LAG()

↓

Previous 1

↓

Previous 2

↓

Compare

↓

Return Number
```

---

## Example

Original

```
+----+-----+
|id  |num  |
+----+-----+
|1   |1    |
|2   |1    |
|3   |1    |
|4   |2    |
|5   |2    |
+----+-----+
```

After LAG

```
+----+-----+-------+-------+
|id  |num  |prev1  |prev2  |
+----+-----+-------+-------+
|1   |1    |NULL   |NULL   |
|2   |1    |1      |NULL   |
|3   |1    |1      |1      |
|4   |2    |1      |1      |
|5   |2    |2      |1      |
+----+-----+-------+-------+
```

Only row 3 satisfies

```
num = prev1

AND

num = prev2
```

Answer

```
1
```

---

## Why This Works

Instead of joining the table three times,

the window function looks backward.

CTE stores those values once.

```
Logs

↓

Window Function

↓

Previous Values

↓

Comparison
```

Very readable.

---

## Alternative Interview Solution

Many interviewers also accept

```sql
SELECT DISTINCT
    l1.num

FROM Logs l1
JOIN Logs l2
ON l1.id=l2.id-1

JOIN Logs l3
ON l2.id=l3.id-1

WHERE
l1.num=l2.num
AND
l2.num=l3.num;
```

The CTE + window version scales better and demonstrates stronger SQL knowledge.

---

## Edge Cases

### Four identical values

```
1
1
1
1
```

Still returns only one row because of

```sql
DISTINCT
```

---

### Multiple valid numbers

Returns each once.

---

### NULL values

No issue because equality with NULL evaluates to unknown.

---

## Complexity

Window function

```
O(n)
```

Space

```
O(n)
```

---

## Interview Takeaways

Patterns tested

- Window Functions
- LAG()
- CTE readability
- Sequential comparison

---

# Progress So Far

| Problem | Difficulty | Primary Pattern | Main SQL Feature |
|----------|------------|-----------------|------------------|
| Employees Earning More Than Managers | Easy | Self Join | CTE |
| Department Top Three Salaries | Hard | Ranking | DENSE_RANK + CTE |
| Human Traffic of Stadium | Hard | Consecutive Groups | ROW_NUMBER + CTE |
| Consecutive Numbers | Medium | Sequential Comparison | LAG + CTE |

---

**Continue with Part 3** for:

- Problem 5 — Tree Node (Medium)
- Problem 6 — Exchange Seats (Medium)
- Comprehensive pattern mapping
- Interview-ready summary before advanced sections

---

# Problem 5 — Tree Node (Medium)

**LeetCode #608**

Difficulty: **Medium**

Companies

```
Google
Amazon
Meta
Microsoft
Oracle
Salesforce
```

LeetCode

https://leetcode.com/problems/tree-node/

---

## Problem Statement

Table

```
Tree
```

| id | p_id |
|----|------|

Classify every node into one of the following:

- Root
- Inner
- Leaf

Return

| id | type |

---

## Interview Observation

Most candidates immediately write a nested query.

A CTE makes the solution much easier by computing reusable information once.

---

## Optimal Solution

```sql
-- Step 1: Collect all parent nodes
WITH ParentNodes AS (

    SELECT DISTINCT
        p_id
    FROM Tree
    WHERE p_id IS NOT NULL

)

-- Step 2: Classify every node
SELECT
    t.id,

    CASE

        WHEN t.p_id IS NULL
            THEN 'Root'

        WHEN p.p_id IS NOT NULL
            THEN 'Inner'

        ELSE 'Leaf'

    END AS type

FROM Tree t

LEFT JOIN ParentNodes p
ON t.id = p.p_id;
```

---

## Execution Flow

```
Tree

↓

Find Every Parent

↓

ParentNodes

↓

LEFT JOIN

↓

CASE Statement

↓

Classification
```

---

## Example

Tree

```
+----+------+
| id | p_id |
+----+------+
|1   |NULL  |
|2   |1     |
|3   |1     |
|4   |2     |
|5   |2     |
+----+------+
```

ParentNodes

```
1

2
```

Classification

```
1 → Root

2 → Inner

3 → Leaf

4 → Leaf

5 → Leaf
```

---

## Why This Works

Instead of repeatedly checking whether a node appears as a parent,

the CTE computes that information once.

```
Tree

↓

Distinct Parents

↓

Join

↓

Classification
```

This avoids repeated scans of the same table.

---

## Edge Cases

### Single node tree

```
id

1

p_id

NULL
```

Answer

```
Root
```

---

### Root with one child

Child becomes

```
Leaf
```

---

### Internal node

A node having

- one parent
- at least one child

is classified as

```
Inner
```

---

## Complexity

Distinct parent extraction

```
O(n)
```

Join

```
O(n)
```

Overall

```
O(n)
```

---

## Interview Takeaways

Recognize this interview pattern:

```
Pre-compute lookup values

↓

Join once

↓

CASE classification
```

---

# Problem 6 — Exchange Seats (Medium)

**LeetCode #626**

Difficulty: **Medium**

Companies

```
Google
Meta
Amazon
Bloomberg
Apple
Microsoft
```

LeetCode

https://leetcode.com/problems/exchange-seats/

---

## Problem Statement

Table

```
Seat
```

| id | student |
|----|----------|

Swap every pair of adjacent students.

Example

```
1 ↔ 2

3 ↔ 4

5 stays if odd
```

---

## Interview Observation

Many candidates write multiple UNION queries.

A CTE makes the swapping logic easier to read and maintain.

---

## Optimal Solution

```sql
-- Step 1: Calculate new seat positions
WITH SwappedSeats AS (

    SELECT

        CASE

            WHEN id % 2 = 1
                 AND id != (SELECT MAX(id) FROM Seat)
            THEN id + 1

            WHEN id % 2 = 0
            THEN id - 1

            ELSE id

        END AS new_id,

        student

    FROM Seat

)

-- Step 2: Restore ordering
SELECT
    new_id AS id,
    student

FROM SwappedSeats

ORDER BY id;
```

---

## Execution Flow

```
Seat Table

↓

CASE

↓

New Seat Number

↓

Sort

↓

Answer
```

---

## Example

Original

```
+----+---------+
|id  |student  |
+----+---------+
|1   |A        |
|2   |B        |
|3   |C        |
|4   |D        |
|5   |E        |
+----+---------+
```

After CASE

```
A → 2

B → 1

C → 4

D → 3

E → 5
```

Output

```
+----+---------+
|1   |B        |
|2   |A        |
|3   |D        |
|4   |C        |
|5   |E        |
+----+---------+
```

---

## Why This Works

The CTE isolates seat reassignment from the final ordering.

```
Original IDs

↓

Compute New IDs

↓

Order

↓

Output
```

This separation improves readability and debugging.

---

## Edge Cases

### Even number of students

Every seat swaps.

---

### Odd number of students

Last student remains unchanged.

---

### Single student

Returned unchanged.

---

## Complexity

Single scan

```
O(n)
```

Sorting

```
O(n log n)
```

Overall

```
O(n log n)
```

---

## Interview Takeaways

Patterns tested

- CASE
- Conditional logic
- CTE decomposition
- Ordering after transformation

---

# Summary of All Six Problems

| # | Problem | Difficulty | Main Pattern | Key SQL Features |
|---|---------|------------|--------------|------------------|
| 181 | Employees Earning More Than Their Managers | Easy | Self Join | CTE + Self Join |
| 185 | Department Top Three Salaries | Hard | Ranking | CTE + DENSE_RANK |
| 601 | Human Traffic of Stadium | Hard | Consecutive Groups | ROW_NUMBER + CTE |
| 180 | Consecutive Numbers | Medium | Sequential Comparison | LAG + CTE |
| 608 | Tree Node | Medium | Classification | CASE + CTE |
| 626 | Exchange Seats | Medium | Data Transformation | CASE + CTE |

---

# CTE Patterns Learned

```
CTE

│

├── Self Join

├── Ranking

├── Classification

├── Consecutive Rows

├── Window Functions

├── Sequential Comparison

└── Data Transformation
```

---

# Interview Checklist

Before submitting a CTE-based solution, verify:

- Use descriptive CTE names (`RankedEmployees`, `GroupedDays`).
- Push filters into early CTEs to reduce rows.
- Avoid repeating the same subquery.
- Use window functions where appropriate.
- Keep each CTE focused on a single transformation.
- Order the final output only once, in the outer query unless intermediate ordering is required by a window function.

---

**Continue with Part 4** for:

- LeetCode Question Types & Patterns
- LLM-Proof Interview Questions
- Advanced CTE Techniques
- Performance Optimization
- CTE vs Subqueries vs Window Functions
- Common Pitfalls
- FAANG Interview Cheat Sheet

  ---

# LeetCode Question Patterns

The majority of CTE-based LeetCode SQL questions fall into a small number of recurring patterns. Interviewers are generally evaluating whether you can decompose a complex query into readable, reusable steps.

---

## Pattern 1 — Self Join

### Goal

Compare rows within the same table.

Typical examples

- Employees vs Managers
- Customers vs Referrers
- Employee Hierarchies

Representative Problem

| Problem | Difficulty |
|----------|------------|
| Employees Earning More Than Their Managers | Easy |

Pipeline

```
Table

↓

CTE

↓

Self Join

↓

Comparison

↓

Result
```

Typical SQL

```sql
WITH EmployeeInfo AS (...)

SELECT ...
FROM EmployeeInfo e
JOIN EmployeeInfo m
ON e.managerId = m.id;
```

Interview Signal

- Self Join
- Reusing computed datasets
- Cleaner query decomposition

---

## Pattern 2 — Ranking Within Groups

### Goal

Find Top K values inside each partition.

Representative Problems

| Problem | Difficulty |
|----------|------------|
| Department Top Three Salaries | Hard |

Pipeline

```
Join

↓

CTE

↓

Window Function

↓

Ranking

↓

Filter
```

Common Functions

```sql
ROW_NUMBER()

RANK()

DENSE_RANK()
```

Most Common Interview Mistake

Using

```sql
ROW_NUMBER()
```

instead of

```sql
DENSE_RANK()
```

when duplicate values should share the same rank.

---

## Pattern 3 — Consecutive Records

Representative Problems

| Problem | Difficulty |
|----------|------------|
| Human Traffic of Stadium | Hard |
| Consecutive Numbers | Medium |

Pipeline

```
Ordered Rows

↓

Window Function

↓

Grouping

↓

Aggregation

↓

Output
```

Common Techniques

```
ROW_NUMBER()

LAG()

LEAD()

id - row_number()
```

Recognize these immediately during interviews.

---

## Pattern 4 — Classification

Representative Problem

| Problem | Difficulty |
|----------|------------|
| Tree Node | Medium |

Pipeline

```
Precompute Lookup

↓

Join

↓

CASE

↓

Classification
```

Common SQL

```sql
CASE

WHEN ...

THEN ...

ELSE ...

END
```

---

## Pattern 5 — Row Transformation

Representative Problem

| Problem | Difficulty |
|----------|------------|
| Exchange Seats | Medium |

Pipeline

```
Rows

↓

Transformation

↓

CASE

↓

Ordering

↓

Output
```

Typical Operations

- Swap
- Rotate
- Reorder
- Normalize IDs

---

# Pattern Recognition Matrix

| Requirement | Preferred SQL Tool |
|-------------|-------------------|
| Compare rows | Self Join |
| Top K | DENSE_RANK |
| Consecutive rows | ROW_NUMBER |
| Previous row | LAG |
| Next row | LEAD |
| Running totals | SUM() OVER |
| Classification | CASE |
| Multiple reusable transformations | CTE |

---

# LLM-Proof Questions

These are not common LeetCode questions, but they frequently appear in senior backend, data engineering, and system design interviews. They test whether you understand **why** CTEs exist rather than just how to write them.

---

# Wild Card 1 — Recursive Employee Hierarchy

Difficulty

Hard

Asked By

```
Meta

Amazon

Microsoft

Oracle
```

---

## Problem

Given

```
Employee

id

manager_id

name
```

Return the entire reporting chain starting from the CEO.

---

## Solution

```sql
WITH RECURSIVE EmployeeHierarchy AS (

    -- Base case
    SELECT
        id,
        manager_id,
        name,
        1 AS level
    FROM Employee
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT
        e.id,
        e.manager_id,
        e.name,
        h.level + 1
    FROM Employee e
    JOIN EmployeeHierarchy h
    ON e.manager_id = h.id

)

SELECT *
FROM EmployeeHierarchy
ORDER BY level, id;
```

---

Execution

```
CEO

↓

Managers

↓

Employees

↓

Interns
```

Interview Purpose

Tests recursive thinking instead of simple joins.

Complexity

```
O(n)
```

assuming an indexed hierarchy.

---

# Wild Card 2 — Organizational Depth

Problem

Find the maximum reporting depth in a company.

Solution

```sql
WITH RECURSIVE OrgChart AS (

    SELECT
        id,
        manager_id,
        1 AS depth
    FROM Employee
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.id,
        e.manager_id,
        o.depth + 1
    FROM Employee e
    JOIN OrgChart o
    ON e.manager_id = o.id

)

SELECT MAX(depth) AS max_depth
FROM OrgChart;
```

Interview Goal

Tests recursive traversal plus aggregation.

Real-world Use

- Org charts
- Folder structures
- Category trees
- Permission inheritance

---

# Wild Card 3 — Bill of Materials Explosion

Asked By

```
Amazon

Tesla

NVIDIA

Apple
```

Problem

A product contains sub-components.

Each sub-component contains more components.

Return every component required to manufacture one final product.

Table

```
Component

parent_part

child_part
```

Solution

```sql
WITH RECURSIVE Parts AS (

    SELECT
        parent_part,
        child_part
    FROM Component

    WHERE parent_part = 'Laptop'

    UNION ALL

    SELECT
        c.parent_part,
        c.child_part
    FROM Component c

    JOIN Parts p
    ON c.parent_part = p.child_part

)

SELECT *
FROM Parts;
```

Execution

```
Laptop

↓

Motherboard

↓

CPU

↓

Cache

↓

Transistor
```

Real-world Usage

- Manufacturing
- ERP systems
- Supply chain management
- Product dependency graphs

---

# Advanced Tricks & Performance

---

## 1. Push Filters Into the Earliest CTE

Instead of

```sql
WITH Sales AS (

SELECT *
FROM Orders

)

SELECT *
FROM Sales

WHERE amount > 1000;
```

Prefer

```sql
WITH Sales AS (

SELECT *
FROM Orders

WHERE amount > 1000

)

SELECT *
FROM Sales;
```

Benefit

- Fewer rows
- Less memory
- Faster joins

---

## 2. Keep Each CTE Focused

Bad

```
CTE

↓

Join

↓

Aggregate

↓

Window

↓

CASE

↓

Filter
```

Good

```
CTE1 → Join

↓

CTE2 → Aggregate

↓

CTE3 → Rank

↓

Final Select
```

Readable SQL is easier to debug and optimize.

---

## 3. Avoid Repeating Expensive Logic

Bad

```sql
SELECT ...
FROM (

...

)

JOIN (

...

)
```

Same computation repeated multiple times.

Better

```sql
WITH Base AS (...)

SELECT ...

FROM Base
JOIN Base;
```

---

## 4. Name CTEs Clearly

Poor

```sql
WITH t1 AS (...)
```

Better

```sql
WITH RankedEmployees AS (...)
```

Readable names reduce cognitive load during interviews.

---

## 5. Watch Out for Materialization

Different databases optimize CTEs differently.

| Database | Typical Behavior |
|----------|------------------|
| PostgreSQL (older versions) | Often materialized |
| PostgreSQL (newer versions) | May inline CTEs |
| MySQL 8+ | Usually optimized |
| SQL Server | Optimizer dependent |
| SQLite | Usually inlined |

Implication

A CTE is not always "free." On large datasets, inspect execution plans.

---

# Common Pitfalls

### Using `ROW_NUMBER()` instead of `DENSE_RANK()`

Wrong for duplicate Top-K problems.

---

### Ordering inside a CTE unnecessarily

```sql
WITH X AS (

SELECT *

FROM Employee

ORDER BY salary
)
```

Unless a window function depends on that ordering, it has no effect on the final result.

---

### Overusing CTEs

Avoid splitting a simple query into many one-line CTEs.

Poor

```
CTE1

↓

CTE2

↓

CTE3

↓

CTE4

↓

CTE5
```

Use CTEs when each step represents a meaningful transformation.

---

### Forgetting Recursive Termination

Recursive CTEs require a stopping condition.

Without one:

```
Infinite recursion

↓

Query failure
```

---

# CTE vs Subquery vs Window Function

| Feature | CTE | Subquery | Window Function |
|---------|-----|----------|-----------------|
| Improves readability | Yes | Moderate | No |
| Reusable in query | Yes | No | N/A |
| Supports recursion | Yes | No | No |
| Best for multi-step transformations | Yes | No | No |
| Best for Top-K | With window | Possible | Yes |
| Best for running totals | Combined with window | Poor | Excellent |
| Best for hierarchical data | Recursive CTE | Impossible | Impossible |

---

# FAANG Interview Cheat Sheet

```
Need reusable logic?
        │
        ▼
      Use CTE

Need previous row?
        │
        ▼
       LAG()

Need next row?
        │
        ▼
      LEAD()

Need Top K?
        │
        ▼
   DENSE_RANK()

Need consecutive rows?
        │
        ▼
ROW_NUMBER()

Need hierarchy?
        │
        ▼
Recursive CTE

Need running total?
        │
        ▼
SUM() OVER()

Need classification?
        │
        ▼
CASE

Need multiple transformations?
        │
        ▼
CTE Pipeline
```

---

# Final Interview Notes

- Think of each CTE as one logical transformation.
- Filter as early as possible.
- Prefer window functions over self joins when solving sequential problems.
- Use recursive CTEs only for hierarchical or graph-like data.
- Name CTEs descriptively (`RankedEmployees`, `GroupedDays`, `EmployeeHierarchy`).
- Be prepared to explain *why* a CTE improves readability or maintainability, not just how it works.
- Always verify whether duplicate handling requires `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()`.

---

## Complete Coverage

This guide covered:

- 6 non-premium LeetCode problems
- Easy, Medium, and Hard difficulty levels
- CTE pipelines
- Window-function integration
- Recursive CTEs
- Hierarchical queries
- Consecutive-row techniques
- Ranking patterns
- Classification patterns
- Data transformation patterns
- Performance optimization
- Common pitfalls
- CTE vs Subqueries vs Window Functions
- Advanced interview variations
- FAANG-oriented problem-solving strategies

- Human Traffic of Stadium (Hard)
- Consecutive Numbers (Medium)
