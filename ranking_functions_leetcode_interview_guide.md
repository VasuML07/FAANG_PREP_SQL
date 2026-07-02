# SQL Ranking Functions (ROW_NUMBER, RANK, DENSE_RANK, NTILE) — FAANG Interview Guide

A practical interview guide focused on solving LeetCode SQL ranking-function problems.

---

# Table of Contents

- [1. Department Top Three Salaries](#1-department-top-three-salaries)
- [2. Rank Scores](#2-rank-scores)
- [3. Consecutive Numbers](#3-consecutive-numbers)
- [4. Human Traffic of Stadium](#4-human-traffic-of-stadium)
- [5. Students Report By Geography](#5-students-report-by-geography)
- [6. Ranking Function Problem Types](#6-ranking-function-problem-types)
- [7. Ranking Function Techniques](#7-ranking-function-techniques)
- [8. LLM-Proof Ranking Function Questions](#8-llm-proof-ranking-function-questions)
- [9. Interview Cheat Sheet](#9-interview-cheat-sheet)

---

# 1. Department Top Three Salaries

**LeetCode:** 185

**Difficulty:** Hard

**Asked By:** Google, Amazon, Meta, Microsoft, Apple

## Problem

Return the employees having the top 3 highest salaries in each department.

If multiple employees have the same salary, they should all be returned.

---

## SQL Solution

```sql
SELECT
    d.name AS Department,
    t.name AS Employee,
    t.salary AS Salary
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER(
            PARTITION BY departmentId
            ORDER BY salary DESC
        ) AS salary_rank
    FROM Employee e
) t
JOIN Department d
ON t.departmentId = d.id
WHERE salary_rank <= 3;
```

---

## Why DENSE_RANK?

Suppose salaries are:

|Salary|
|------:|
|10000|
|9000|
|9000|
|8500|

Ranking becomes

|Salary|ROW_NUMBER|RANK|DENSE_RANK|
|------:|---------:|---:|----------:|
|10000|1|1|1|
|9000|2|2|2|
|9000|3|2|2|
|8500|4|4|3|

If we used

```sql
ROW_NUMBER <=3
```

the employee with 8500 disappears.

If we used

```sql
RANK <=3
```

8500 also disappears because its rank becomes 4.

Only

```sql
DENSE_RANK<=3
```

returns every salary within the top three distinct salary levels.

---

## Interview Trick

Whenever the statement says

> Top K distinct values

think

```sql
DENSE_RANK()
```

instead of ROW_NUMBER.

---

# 2. Rank Scores

**LeetCode:** 178

**Difficulty:** Medium

**Asked By:** Google, Amazon, Bloomberg

## Problem

Rank every score.

Equal scores receive the same rank.

There should be no gaps.

---

## SQL Solution

```sql
SELECT
    score,
    DENSE_RANK() OVER(
        ORDER BY score DESC
    ) AS `rank`
FROM Scores
ORDER BY score DESC;
```

---

## Why DENSE_RANK?

Example

|Score|
|----:|
|100|
|95|
|95|
|90|

Output

|Score|Rank|
|----:|---:|
|100|1|
|95|2|
|95|2|
|90|3|

Notice there is **no missing rank**.

---

## Common Mistake

Using

```sql
RANK()
```

would produce

```
1
2
2
4
```

which violates the problem statement.

---

## Pattern

Whenever the question says

> consecutive ranking

or

> without gaps

immediately consider

```sql
DENSE_RANK()
```

---

# 3. Consecutive Numbers

**LeetCode:** 180

**Difficulty:** Medium

**Asked By:** Amazon, Microsoft

## Problem

Find numbers appearing at least three consecutive times.

---

## SQL Solution

```sql
WITH numbered AS (
    SELECT
        *,
        ROW_NUMBER() OVER(ORDER BY id) AS rn
    FROM Logs
),
groups AS (
    SELECT
        *,
        id - rn AS grp
    FROM numbered
)
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT
        grp,
        num,
        COUNT(*) AS cnt
    FROM groups
    GROUP BY grp, num
) t
WHERE cnt >= 3;
```

---

## Technique

Instead of comparing

```
LAG()
LAG()
LEAD()
```

we build groups using

```
id - ROW_NUMBER()
```

Rows inside the same consecutive sequence share the same difference.

Example

|id|num|ROW_NUMBER|id-rn|
|--:|--:|---------:|----:|
|1|5|1|0|
|2|5|2|0|
|3|5|3|0|
|4|8|4|0|
|5|8|5|0|

The grouping key identifies continuous ranges.

---

## Interview Insight

ROW_NUMBER is frequently used to detect

- consecutive records
- gaps
- islands

This is one of the most common interview patterns.

---

# 4. Human Traffic of Stadium

**LeetCode:** 601

**Difficulty:** Hard

**Asked By:** Google, Amazon, Uber

## Problem

Return records belonging to three or more consecutive days where people ≥100.

---

## SQL Solution

```sql
WITH filtered AS (
    SELECT
        *,
        ROW_NUMBER() OVER(ORDER BY id) AS rn
    FROM Stadium
    WHERE people >= 100
),
grp AS (
    SELECT *,
           id-rn AS grp
    FROM filtered
)
SELECT
    id,
    visit_date,
    people
FROM grp
WHERE grp IN (
    SELECT grp
    FROM grp
    GROUP BY grp
    HAVING COUNT(*) >=3
)
ORDER BY visit_date;
```

---

## Core Technique

Again,

```
id - ROW_NUMBER()
```

creates consecutive groups.

The interviewer is checking whether you recognize this pattern.

---

## Common Mistakes

❌ Using self joins

❌ Nested subqueries

❌ Comparing every adjacent row

The ROW_NUMBER grouping solution is shorter and scales much better.

---

# 5. Students Report By Geography

**LeetCode:** 618

**Difficulty:** Hard

**Asked By:** Google

## Problem

Pivot student names into columns grouped by continent.

---

## SQL Solution

```sql
WITH ranked AS (
SELECT
    name,
    continent,
    ROW_NUMBER() OVER(
        PARTITION BY continent
        ORDER BY name
    ) AS rn
FROM Student
)

SELECT
    MAX(CASE WHEN continent='America' THEN name END) AS America,
    MAX(CASE WHEN continent='Asia' THEN name END) AS Asia,
    MAX(CASE WHEN continent='Europe' THEN name END) AS Europe
FROM ranked
GROUP BY rn
ORDER BY rn;
```

---

## Ranking Trick

Each continent gets independent numbering.

Example

America

|Name|ROW_NUMBER|
|----|---------:|
|Amy|1|
|John|2|
|Mark|3|

Asia

|Name|ROW_NUMBER|
|----|---------:|
|Li|1|
|Wei|2|
|Zhang|3|

Grouping by ROW_NUMBER aligns rows horizontally.

---

## Pattern

Whenever you need

> pivot by row position

ROW_NUMBER is often the missing ingredient.

---

# 6. Ranking Function Problem Types

|Problem Type|Preferred Function|Typical Example|
|------------|-----------------|---------------|
|Top K per group|DENSE_RANK|Department Top Three Salaries|
|Leaderboard|RANK / DENSE_RANK|Rank Scores|
|Pagination|ROW_NUMBER|Page 2 of results|
|Deduplication|ROW_NUMBER|Keep latest record|
|Consecutive sequences|ROW_NUMBER|Human Traffic|
|Gap & Island|ROW_NUMBER|Consecutive Numbers|
|Quartiles|NTILE|Sales segmentation|
|Percentile grouping|NTILE|Customer buckets|
|Pivot alignment|ROW_NUMBER|Students Report|

---

# 7. Ranking Function Techniques

## ROW_NUMBER

Best for

- Unique numbering
- Remove duplicates
- Pagination
- Consecutive groups

---

## RANK

Best when

- Ties exist
- Rank gaps are acceptable

Example

```
1
2
2
4
```

---

## DENSE_RANK

Best when

- Ties exist
- No gaps allowed

Example

```
1
2
2
3
```

---

## NTILE

Useful for

```sql
NTILE(4)
```

Produces

```
Quartile 1
Quartile 2
Quartile 3
Quartile 4
```

Common interview applications

- Sales quartiles
- Customer segmentation
- Revenue buckets

---

## Important Algorithmic Tricks

### Partition first

```sql
PARTITION BY department
```

creates independent rankings.

---

### Order correctly

```sql
ORDER BY salary DESC
```

Ranking changes entirely if ASC is used.

---

### Ranking after filtering

Bad

```sql
Filter
↓

Rank
```

Correct

```text
Filter rows
↓

Apply window function
↓

Filter on rank
```

---

### Duplicate Removal

```sql
ROW_NUMBER()
OVER(
PARTITION BY email
ORDER BY created_at DESC
)
```

Keep only

```
row_number =1
```

---

### Performance

Create indexes on

- PARTITION BY columns
- ORDER BY columns

Large window sorts become much faster.

---

# 8. LLM-Proof Ranking Function Questions

These questions require genuine understanding rather than memorized templates.

---

## 1. Mixed Ranking + Aggregation

Example

> Return top two products per category but only categories having average sales >5000.

Requires

- GROUP BY
- HAVING
- DENSE_RANK

---

## 2. Ranking After Complex Filtering

Example

Rank only active users after excluding suspended accounts.

Filtering order matters.

---

## 3. Multiple Window Functions Together

Example

```sql
ROW_NUMBER()

DENSE_RANK()

LAG()
```

Many candidates misuse partitions.

---

## 4. Nested Window Problems

Ranking inside one CTE followed by another ranking.

Very common at Google.

---

## 5. Gap & Island Variants

These require recognizing

```
id - ROW_NUMBER()
```

instead of comparing adjacent rows manually.

---

## Why These Are Difficult

They test

- execution order
- window evaluation
- partition boundaries
- tie handling
- logical query processing

rather than SQL syntax alone.

---

# 9. Interview Cheat Sheet

|Situation|Function|
|---------|--------|
|Need unique numbering|ROW_NUMBER|
|Need leaderboard with gaps|RANK|
|Need leaderboard without gaps|DENSE_RANK|
|Need top K distinct values|DENSE_RANK|
|Need remove duplicates|ROW_NUMBER|
|Need pagination|ROW_NUMBER|
|Need quartiles|NTILE|
|Need consecutive sequence detection|ROW_NUMBER|
|Need pivot alignment|ROW_NUMBER|

---

## Final Decision Tree

```
Unique numbering?
        │
        ▼
ROW_NUMBER()

──────────────────────────

Need ties?

        │
       Yes
        │
        ▼

Need gaps?

   Yes ─────► RANK()

   No ─────► DENSE_RANK()

──────────────────────────

Need equal-sized buckets?

        │
        ▼

NTILE()
```

---

## Interview Takeaways

- `ROW_NUMBER()` solves positioning problems.
- `RANK()` models competitive rankings with skipped positions.
- `DENSE_RANK()` handles distinct rankings without gaps and is the preferred choice for Top-K salary questions.
- `NTILE()` divides ordered data into approximately equal buckets for segmentation.
- The `id - ROW_NUMBER()` pattern is one of the highest-value interview tricks for detecting consecutive sequences (Gap & Island problems).
- Always verify whether ties should be preserved before choosing between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`.
