# 2. SQL and Query Patterns

[Index](README.md) · [Previous](01_modeling_and_normalization.md) · [Next](03_transactions_and_concurrency.md)

## Shared example schema

The examples use common SQL syntax. Identity generation, pagination, date functions, and some DDL vary by engine.

```sql
CREATE TABLE Department (
    department_id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE Employee (
    employee_id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES Department(department_id),
    manager_id INTEGER REFERENCES Employee(employee_id),
    salary DECIMAL(12, 2) NOT NULL CHECK (salary >= 0)
);
```

An employee may have no department or manager. Salary is non-NULL to simplify ranking examples.

## Statement categories and logical order

- DDL: `CREATE`, `ALTER`, `DROP` define structures.
- DML: `INSERT`, `UPDATE`, `DELETE` change rows; SELECT is sometimes classified separately as DQL.
- Transaction control: `COMMIT`, `ROLLBACK`, transaction-start syntax.
- Access control: `GRANT`, `REVOKE` where supported.

For an ordinary grouped query, a useful conceptual order is:

```text
FROM/JOIN → WHERE → GROUP BY → HAVING → window evaluation
          → SELECT → DISTINCT → ORDER BY → row limiting
```

This is a reasoning aid, not a literal physical execution plan or a complete grammar for every dialect. The optimizer may reorder work while preserving semantics. SELECT aliases generally cannot be used in WHERE; alias availability in other clauses varies.

## Filtering and NULL — core

SQL uses three-valued logic: TRUE, FALSE, UNKNOWN. Comparisons such as `salary = NULL` produce UNKNOWN. Use `IS NULL` / `IS NOT NULL`. WHERE keeps TRUE rows, not UNKNOWN rows.

```sql
SELECT employee_id, name
FROM Employee
WHERE department_id IS NULL;
```

`COUNT(*)` counts rows; `COUNT(column)` counts non-NULL values. Most common aggregates such as SUM and AVG ignore NULL inputs. On no input rows, COUNT returns 0 while SUM/AVG normally return NULL.

**Trap:** `NOT IN` with a NULL in its subquery can yield UNKNOWN for unmatched values. Use a correlated `NOT EXISTS` for many anti-join tasks.

## Joins

| Join | Result |
|---|---|
| INNER | Matching combinations only |
| LEFT | All left rows plus matching right rows; NULL-extended when unmatched |
| RIGHT | Symmetric preservation of right side |
| FULL OUTER | Preserve unmatched rows from both sides; not supported everywhere |
| CROSS | Every pair of rows |
| SELF | Join a table to itself using aliases; not a separate SQL join keyword |

```sql
-- Include departments with zero employees.
SELECT d.department_id, d.name, COUNT(e.employee_id) AS employee_count
FROM Department d
LEFT JOIN Employee e ON e.department_id = d.department_id
GROUP BY d.department_id, d.name;
```

Use `COUNT(e.employee_id)`, not `COUNT(*)`, to return zero for a department's NULL-extended row.

```sql
-- Keep every department; match only employees earning at least 60000.
SELECT d.name, e.name AS employee_name
FROM Department d
LEFT JOIN Employee e
  ON e.department_id = d.department_id AND e.salary >= 60000;
```

Moving `e.salary >= 60000` to WHERE removes unmatched rows because their salary is NULL, changing the outer join's effect.

## Grouping and HAVING

```sql
SELECT department_id, AVG(salary) AS average_salary
FROM Employee
WHERE salary >= 30000
GROUP BY department_id
HAVING COUNT(*) >= 3
ORDER BY average_salary DESC;
```

WHERE filters individual rows before aggregation; HAVING filters groups after aggregation. This averages only employees surviving WHERE, not every employee in qualifying departments.

## Subqueries, EXISTS, and CTEs

```sql
-- Employees above their department's average.
-- Employees with NULL department_id are excluded by this equality condition.
SELECT e.employee_id, e.name, e.salary
FROM Employee e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM Employee e2
    WHERE e2.department_id = e.department_id
);

-- Departments with no employees; safe even if Employee.department_id has NULLs.
SELECT d.department_id, d.name
FROM Department d
WHERE NOT EXISTS (
    SELECT 1 FROM Employee e
    WHERE e.department_id = d.department_id
);
```

A correlated subquery references the outer query. It is not necessarily executed naively once per row; optimizers may transform it. A CTE (`WITH`) names a query expression for readability/composition; materialization and optimization behavior depend on the engine and query.

## Window functions — core interview patterns

Window functions compute across related rows while retaining individual rows; GROUP BY reduces rows into groups.

| Function | For salaries 100, 100, 90 |
|---|---|
| `ROW_NUMBER()` | 1, 2, 3; tied order needs a tiebreaker for determinism |
| `RANK()` | 1, 1, 3 |
| `DENSE_RANK()` | 1, 1, 2 |

```sql
-- Employees with the top two DISTINCT salaries per assigned department.
-- Includes all ties, so a department may return more than two people.
WITH ranked AS (
    SELECT employee_id, name, department_id, salary,
           DENSE_RANK() OVER (
               PARTITION BY department_id ORDER BY salary DESC
           ) AS salary_rank
    FROM Employee
    WHERE department_id IS NOT NULL
)
SELECT employee_id, name, department_id, salary
FROM ranked
WHERE salary_rank <= 2
ORDER BY department_id, salary DESC, employee_id;
```

For exactly two people per department when available, use `ROW_NUMBER()` ordered by `salary DESC, employee_id`. Do not put employee_id inside DENSE_RANK's ORDER BY if equal salaries should tie.

```sql
-- Second-highest DISTINCT salary; returns one row with NULL if absent.
SELECT MAX(salary) AS second_highest_salary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

## Other interview distinctions

- `UNION` removes duplicate result rows; `UNION ALL` preserves them and avoids that deduplication requirement.
- `DELETE` removes selected rows; `TRUNCATE` empties a table with engine-specific locking, identity, trigger, and transaction behavior; `DROP` removes the object. Do not claim TRUNCATE can never roll back in every engine.
- A regular view stores a query definition, not necessarily its results. A materialized view stores results and requires an appropriate refresh/maintenance strategy.
- Without `ORDER BY`, result order is not guaranteed. Use a unique tiebreaker when deterministic pagination matters.
- Use parameterized queries for values. Concatenating untrusted text into SQL can change query structure; identifiers usually need separate validation/allowlisting.

## Interview answers

**WHERE versus HAVING?** Row filtering before grouping versus group filtering afterward.

**JOIN versus EXISTS?** JOIN returns matching combinations and can multiply rows. EXISTS tests whether a match exists without returning every matching combination.

**Why can a join inflate totals?** A one-to-many join repeats parent rows. Aggregate at the intended grain before joining, or otherwise account for multiplicity.
