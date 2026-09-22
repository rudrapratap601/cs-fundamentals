# 5. DBMS Interview Practice

[Index](README.md) · [Previous](04_indexes_storage_scaling.md)

Use the [Employee/Department schema](02_sql_and_query_patterns.md). Decide NULL, tie, and duplicate behavior before writing SQL.

## Rapid recall

| Question | Answer checkpoint |
|---|---|
| Candidate versus primary key? | Minimal superkey versus selected candidate |
| 3NF versus BCNF? | 3NF permits prime RHS attributes when determinant is not a superkey; BCNF does not |
| WHERE versus HAVING? | Filter rows versus groups |
| GROUP BY versus window? | Collapse rows versus retain rows with computed values |
| ACID? | Atomicity, consistency, isolation, durability |
| Serializable versus serial? | Equivalent to serial execution versus literally one after another |
| Index downside? | Storage and write/maintenance cost |
| MVCC versus locking? | Version-based visibility still requires concurrency coordination |

## SQL exercises with answers

### 1. Employees earning more than their manager

```sql
SELECT e.employee_id, e.name
FROM Employee e
JOIN Employee m ON m.employee_id = e.manager_id
WHERE e.salary > m.salary;
```

Employees without a manager are excluded by the inner join.

### 2. Highest-paid employees in each assigned department, including ties

```sql
WITH ranked AS (
    SELECT employee_id, name, department_id, salary,
           DENSE_RANK() OVER (
               PARTITION BY department_id ORDER BY salary DESC
           ) AS rnk
    FROM Employee
    WHERE department_id IS NOT NULL
)
SELECT employee_id, name, department_id, salary
FROM ranked
WHERE rnk = 1
ORDER BY department_id, employee_id;
```

### 3. Departments whose total payroll exceeds 200000

```sql
SELECT d.department_id, d.name, SUM(e.salary) AS payroll
FROM Department d
JOIN Employee e ON e.department_id = d.department_id
GROUP BY d.department_id, d.name
HAVING SUM(e.salary) > 200000;
```

Empty departments cannot qualify, so an inner join is appropriate.

### 4. Repeated employee names

```sql
SELECT name, COUNT(*) AS occurrences
FROM Employee
GROUP BY name
HAVING COUNT(*) > 1;
```

Repeated names are not necessarily duplicate people. A deduplication rule must use the domain's identity definition.

### 5. Three employees per assigned department, if available

```sql
WITH numbered AS (
    SELECT employee_id, name, department_id, salary,
           ROW_NUMBER() OVER (
               PARTITION BY department_id
               ORDER BY salary DESC, employee_id
           ) AS rn
    FROM Employee
    WHERE department_id IS NOT NULL
)
SELECT employee_id, name, department_id, salary
FROM numbered
WHERE rn <= 3
ORDER BY department_id, rn;
```

The primary-key tiebreaker makes selection deterministic. DENSE_RANK would instead select distinct salary levels and could return more than three people.

## Reasoning problems

**Normalize:** `Enrollment(student_id, course_id, student_name, course_name, grade)` with key `(student_id, course_id)`, student ID determining name and course ID determining course name.

**Answer:** Student(student_id, student_name), Course(course_id, course_name), Enrollment(student_id, course_id, grade), with appropriate primary/foreign keys. Names depend on parts of the composite key; grade depends on the whole enrollment identity under the assumptions.

**Concurrent stock purchase:** Two requests read stock=1 and both decide to buy.

**Answer:** An uncoordinated read/check/write can oversell. Use appropriate transactional concurrency control, for example a conditional decrement `UPDATE ... SET stock = stock - 1 WHERE ... AND stock > 0`, verify one affected row, and create the order within the same intended transaction. Handle aborts/retries and engine isolation semantics.

**Index choice:** Frequent requests filter by department and a salary range.

**Answer:** Consider `(department_id, salary)`, then inspect actual plans and workload costs. Salary-only queries may need a different access path.

**Rollback question:** Can TRUNCATE be rolled back?

**Answer:** It depends on the database and transaction context. Avoid a universal answer based on a single engine.

## Edge-case checklist

- Empty input and NULL values.
- Duplicate matches introduced by joins.
- Equal salaries and deterministic tiebreakers.
- Departments with zero employees.
- Concurrent writes and transaction retries.
- Engine-specific DDL, isolation, index, and execution-plan behavior.
