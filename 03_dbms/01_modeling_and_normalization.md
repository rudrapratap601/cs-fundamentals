# 1. Relational Modeling and Normalization

[Index](README.md) · [Next](02_sql_and_query_patterns.md)

## DBMS and relational concepts

A DBMS manages data storage, retrieval, integrity, concurrency, and recovery. A relational DBMS represents data using relations; SQL tables approximate these with practical features such as NULL and duplicate rows.

- **Schema:** structure and constraints; **instance:** the data at a particular time.
- **Relation/table:** collection of tuples/rows described by attributes/columns.
- **Degree:** number of attributes; **cardinality:** number of rows in this context.
- **Physical data independence:** change storage details without changing the logical schema/interface.
- **Logical data independence:** change the logical organization while preserving relevant external views; often harder in practice.

Relational algebra provides conceptual operations: selection filters rows, projection selects attributes, join combines related tuples, union combines compatible relations, and difference removes members. SQL often uses bag/multiset semantics, so projection does not automatically remove duplicates.

## Keys and integrity — core

| Term | Meaning |
|---|---|
| Superkey | Attribute set that uniquely identifies a row under the schema's rules |
| Candidate key | Minimal superkey; no attribute can be removed while retaining uniqueness |
| Primary key | Chosen candidate key; unique and non-NULL |
| Alternate key | Candidate key not chosen as primary |
| Composite key | Key containing multiple attributes |
| Foreign key | References an eligible unique/primary key in a related or the same table |
| Natural key | Domain-derived identifier, such as a business code |
| Surrogate key | Artificial identifier, such as a generated ID |

Minimal means no proper subset is a key, not “fewest columns among all possible keys.” Foreign keys may allow NULL unless prohibited; engine-specific rules govern composite NULL cases and eligible referenced constraints.

Constraints include `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, and `CHECK`. A `CHECK (salary > 0)` alone does not generally reject NULL; add `NOT NULL` if required. Multiple-NULL behavior under unique constraints varies by database.

## ER modeling

An entity has attributes; relationships connect entities. State both **cardinality** and **optionality**.

- One-to-many: Department → Employees, using `Employee.department_id` as a foreign key.
- Many-to-many: Student ↔ Course, using `Enrollment(student_id, course_id, ...)` with an appropriate composite key or equivalent uniqueness constraint.
- One-to-one: User ↔ Profile, using a unique foreign key when that matches the requirements.
- Weak entity: identification depends on an owner, such as an order line identified by `(order_id, line_no)`.

Do not store comma-separated related IDs in one column when they represent independently queryable relationships.

## Functional dependencies — core

`X → Y` means any two valid rows agreeing on X must agree on Y. Dependencies come from domain rules, not accidental uniqueness in today's sample.

The closure `X+` is all attributes functionally determined by X. X is a superkey if its closure includes all attributes. **Prime attributes** belong to at least one candidate key.

Example: In `Employee(employee_id, name, department_id, department_name)`, assume:

```text
employee_id → name, department_id
department_id → department_name
Therefore employee_id → department_name (transitivity).
```

Repeated department names create update anomalies: changing the name requires many rows; deleting the last employee can lose department information; adding a department may require an unrelated employee row.

## Normal forms

| Form | Interview definition |
|---|---|
| 1NF | Attributes contain single values in their intended domains; no repeating groups |
| 2NF | 1NF and no non-prime attribute depends on a proper subset of any candidate key |
| 3NF | For every nontrivial FD `X → A`, X is a superkey or A is prime |
| BCNF | For every nontrivial FD `X → Y`, X is a superkey |

“No transitive dependencies” is a useful 3NF intuition for simple cases, but the formal definition handles multiple candidate keys correctly. A table with only single-attribute candidate keys cannot have partial-key dependencies and is in 2NF if it is in 1NF.

### Worked normalization

Assume each product appears at most once per order. Start with:

```text
OrderLine(order_id, product_id, order_date, customer_id,
          customer_name, product_name, quantity)
Key: (order_id, product_id)
order_id → order_date, customer_id
customer_id → customer_name
product_id → product_name
(order_id, product_id) → quantity
```

1. **1NF:** one product per row; do not store a list of products in one field.
2. **2NF:** separate order details and product details because they depend on parts of the composite key.
3. **3NF:** move customer names out of Orders because `order_id → customer_id → customer_name`.

Result:

```text
Customer(customer_id PK, customer_name)
Orders(order_id PK, order_date, customer_id FK)
Product(product_id PK, product_name)
OrderLine(order_id FK, product_id FK, quantity,
          PK(order_id, product_id))
```

If products can appear multiple times in an order, use a line identifier and revisit the dependencies. Historical sale price belongs on the order line if it must preserve the price at purchase, not only on the current Product record.

### 3NF but not BCNF — follow-up

For `Teaching(student, course, instructor)`, assume `(student, course) → instructor` and `instructor → course` (each instructor teaches one course). Candidate keys are `(student, course)` and `(student, instructor)`.

`instructor → course` violates BCNF because instructor is not a superkey. It meets 3NF because course is prime. Decomposing into `(instructor, course)` and `(student, instructor)` is lossless under the given dependency, but enforcing `(student, course) → instructor` now requires cross-table reasoning. BCNF decomposition can sacrifice dependency preservation.

## Decomposition properties

- **Lossless join:** joining decomposed relations reconstructs the original valid relation without spurious rows. For a binary decomposition under FDs, a useful criterion is that shared attributes determine all of at least one component.
- **Dependency preservation:** original dependencies can be enforced on individual components without joining them.
- **Denormalization:** intentional duplication to improve particular reads; introduces consistency and write-maintenance costs.

**Follow-up:** 4NF addresses nontrivial multivalued dependencies. If a person's independent skills and languages are stored together, all skill/language combinations can be redundant; separate PersonSkill and PersonLanguage relations.

## Interview answers

**Does normalization always improve speed?** No. It primarily reduces redundancy and anomalies. More joins can cost time; selective denormalization should follow workload evidence.

**Can a primary key be composite?** Yes. Choose according to identity and access requirements, not a rule that every table must use one generated column.

**Is a foreign key automatically an index?** Do not assume so. Engines differ; integrity and access-path support are distinct concerns.
