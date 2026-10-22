# 03-constraints

# SQL Constraints in Oracle Database

## 1. What Are SQL Constraints?

SQL constraints are rules applied to table columns or tables to maintain **data accuracy, consistency, and integrity** in a database.

Constraints can be defined while creating a table using `CREATE TABLE` or added later using `ALTER TABLE`.

### Types of SQL Constraints

| Constraint | Purpose |
|---|---|
| `NOT NULL` | Prevents NULL values in a column. |
| `UNIQUE` | Prevents duplicate values in a column or column combination. |
| `PRIMARY KEY` | Uniquely identifies each row; combines UNIQUE and NOT NULL behavior. |
| `FOREIGN KEY` | Maintains referential integrity between related tables. |
| `CHECK` | Ensures values satisfy a specified condition. |
| `DEFAULT` | Supplies a default value when a column is omitted from an INSERT statement. |

> **Interview Tip:** Oracle supports all six concepts above. `DEFAULT` is a column default specification rather than a constraint enforced in the same way as the other five.

---

## 2. NOT NULL Constraint

### Definition

The `NOT NULL` constraint ensures that a column cannot store a NULL value.

### Create a Table with NOT NULL

```sql
CREATE TABLE employees (
    emp_id NUMBER NOT NULL,
    emp_name VARCHAR2(50)
);
```

### Add NOT NULL to an Existing Column

```sql
ALTER TABLE employees
MODIFY (emp_name NOT NULL);
```

This succeeds only if existing rows contain no NULL values in `emp_name`.

### Remove NOT NULL

```sql
ALTER TABLE employees
MODIFY (emp_name NULL);
```

### Important Points

- A NOT NULL column must have a value other than NULL.
- An empty string (`''`) is treated as NULL by Oracle for character data.
- NOT NULL does not prevent duplicate values.
- A primary key column is implicitly NOT NULL.

---

## 3. UNIQUE Constraint

### Definition

The `UNIQUE` constraint prevents duplicate values in a column or a combination of columns.

### Create a Table with UNIQUE

```sql
CREATE TABLE employees_unique (
    emp_id NUMBER,
    email VARCHAR2(100) UNIQUE
);
```

### Add UNIQUE to an Existing Table

```sql
ALTER TABLE employees_unique
ADD CONSTRAINT uq_emp_email UNIQUE (email);
```

### Drop UNIQUE Constraint

Use the actual constraint name:

```sql
ALTER TABLE employees_unique
DROP CONSTRAINT uq_emp_email;
```

### Find Constraint Names

```sql
SELECT constraint_name, constraint_type, status
FROM user_constraints
WHERE table_name = 'EMPLOYEES_UNIQUE';
```

Oracle normally stores unquoted table and constraint names in uppercase.

### Important Points

- Duplicate non-NULL values are not allowed.
- A nullable UNIQUE column can contain NULL values.
- Oracle permits multiple NULLs in a single-column UNIQUE constraint.
- A UNIQUE constraint can cover multiple columns.
- Existing duplicate values must be resolved before adding a UNIQUE constraint.

---

## 4. PRIMARY KEY Constraint

### Definition

A `PRIMARY KEY` uniquely identifies each row in a table.

It combines uniqueness and non-nullability.

### Create a Table with PRIMARY KEY

```sql
CREATE TABLE departments (
    dept_id NUMBER CONSTRAINT pk_departments PRIMARY KEY,
    dept_name VARCHAR2(50)
);
```

### Add a Primary Key to an Existing Table

```sql
ALTER TABLE departments
ADD CONSTRAINT pk_departments PRIMARY KEY (dept_id);
```

### Composite Primary Key

A composite primary key contains two or more columns. Their combined values must be unique.

```sql
CREATE TABLE employee_projects (
    emp_id NUMBER,
    project_id NUMBER,
    assigned_date DATE,
    CONSTRAINT pk_emp_projects
        PRIMARY KEY (emp_id, project_id)
);
```

Here, an employee can participate in multiple projects, but the same employee-project combination cannot appear twice.

### Drop a Primary Key

```sql
ALTER TABLE departments
DROP CONSTRAINT pk_departments;
```

Alternatively, Oracle supports:

```sql
ALTER TABLE departments
DROP PRIMARY KEY;
```

The operation can fail if other constraints depend on the primary key.

### Important Points

- A table can have only **one primary key constraint**.
- A primary key can contain multiple columns.
- Oracle supports a maximum of 32 columns in a primary key.
- Primary key values cannot be duplicated or NULL.
- A primary key is commonly referenced by foreign keys.

---

## 5. FOREIGN KEY Constraint

### Definition

A `FOREIGN KEY` establishes a relationship between tables and maintains referential integrity.

The foreign key value must match a referenced primary key or eligible UNIQUE key value, unless the foreign key is NULL.

### Step 1: Create the Parent Table

```sql
CREATE TABLE parent_dept (
    dept_no NUMBER CONSTRAINT pk_parent_dept PRIMARY KEY,
    dept_name VARCHAR2(50)
);
```

### Step 2: Create the Child Table

```sql
CREATE TABLE oneview (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(50),
    deptno NUMBER,
    CONSTRAINT fk_oneview_dept
        FOREIGN KEY (deptno)
        REFERENCES parent_dept (dept_no)
);
```

### Insert Valid Data

```sql
INSERT INTO parent_dept VALUES (10, 'IT');

INSERT INTO oneview VALUES (1, 'Nitin', 10);
```

### Insert Invalid Data

```sql
INSERT INTO oneview VALUES (2, 'Rahul', 99);
```

This fails because department 99 does not exist in the parent table.

### Drop a Foreign Key

```sql
ALTER TABLE oneview
DROP CONSTRAINT fk_oneview_dept;
```

### ON DELETE Options

**CASCADE:** Deletes dependent child rows when the referenced parent row is deleted.

```sql
FOREIGN KEY (deptno)
REFERENCES parent_dept (dept_no)
ON DELETE CASCADE
```

**SET NULL:** Sets the child foreign key to NULL when the referenced parent row is deleted. The child column must permit NULL.

```sql
FOREIGN KEY (deptno)
REFERENCES parent_dept (dept_no)
ON DELETE SET NULL
```

Without either option, Oracle normally prevents deletion of a referenced parent row while matching child rows exist.

### Important Points

- A table can have multiple foreign keys.
- A foreign key can reference a primary key or an eligible UNIQUE key.
- Foreign key columns can contain NULL unless separately restricted.
- A foreign key does not automatically create an index on the child table.

---

## 6. CHECK Constraint

### Definition

The `CHECK` constraint ensures that inserted or updated values satisfy a specified condition.

### Create a Table with CHECK

```sql
CREATE TABLE employee_salary (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(50),
    salary NUMBER,
    CONSTRAINT chk_salary CHECK (salary > 0)
);
```

### Add CHECK to an Existing Table

```sql
ALTER TABLE employee_salary
ADD CONSTRAINT chk_salary_range
CHECK (salary BETWEEN 10000 AND 200000);
```

### Drop CHECK Constraint

```sql
ALTER TABLE employee_salary
DROP CONSTRAINT chk_salary_range;
```

### Examples

```sql
CHECK (age >= 18)

CHECK (salary > 0)

CHECK (status IN ('ACTIVE', 'INACTIVE'))

CHECK (end_date >= start_date)
```

### Important Points

- A CHECK condition must not evaluate to FALSE for the row.
- A condition evaluating to UNKNOWN because of NULL can pass. Use NOT NULL separately if NULL is prohibited.
- CHECK constraints cannot generally enforce conditions that depend on other rows or tables.
- Existing rows must satisfy a newly added CHECK constraint unless it is added using `NOVALIDATE`.

---

## 7. DEFAULT Value

### Definition

A default supplies a value when an INSERT statement omits the corresponding column or specifies the `DEFAULT` keyword.

### Create a Table with DEFAULT

```sql
CREATE TABLE employee_status (
    emp_id NUMBER PRIMARY KEY,
    emp_name VARCHAR2(50),
    status VARCHAR2(20) DEFAULT 'ACTIVE',
    created_date DATE DEFAULT SYSDATE
);
```

### Insert Without Specifying the Defaulted Column

```sql
INSERT INTO employee_status (emp_id, emp_name)
VALUES (101, 'Nitin');
```

Oracle supplies `'ACTIVE'` for `status` and the current database date and time for `created_date`.

### Specify DEFAULT Explicitly

```sql
INSERT INTO employee_status (emp_id, emp_name, status)
VALUES (102, 'Rahul', DEFAULT);
```

### Important Points

- A default is applied when the column is omitted or `DEFAULT` is specified.
- Explicitly inserting NULL does not normally invoke the default.
- DEFAULT does not prevent NULL values.
- To prohibit NULL, use NOT NULL as well.

---

## 8. Naming Constraints

Giving constraints meaningful names makes database maintenance easier.

```sql
CREATE TABLE employees_demo (
    emp_id NUMBER,
    email VARCHAR2(100),
    salary NUMBER,

    CONSTRAINT pk_employees_demo PRIMARY KEY (emp_id),
    CONSTRAINT uq_employees_email UNIQUE (email),
    CONSTRAINT chk_employees_salary CHECK (salary > 0)
);
```

Naming conventions:

| Prefix | Meaning |
|---|---|
| `pk_` | Primary key |
| `fk_` | Foreign key |
| `uq_` | Unique |
| `chk_` | Check |

For NOT NULL, Oracle's `NOT NULL` declaration is used on the column; its associated constraint name can also be specified in the column definition.

**Why name constraints?**

Oracle may generate names such as `SYS_C007003` when you do not provide one. Meaningful names make it easier to identify, modify, and drop constraints.

---

## 9. View Constraints in Oracle

### View Constraints for a Specific Table

```sql
SELECT constraint_name,
       constraint_type,
       status,
       validated
FROM user_constraints
WHERE table_name = 'EMPLOYEES_DEMO';
```

### View Constraint Columns

```sql
SELECT constraint_name,
       column_name,
       position
FROM user_cons_columns
WHERE table_name = 'EMPLOYEES_DEMO'
ORDER BY constraint_name, position;
```

### Constraint Types

| Code | Meaning |
|---|---|
| `P` | Primary key |
| `U` | Unique |
| `R` | Referential integrity / foreign key |
| `C` | Check constraint, including NOT NULL |

**Note:** DEFAULT values are not represented as ordinary constraints in `USER_CONSTRAINTS`.

---

## 10. Enable, Disable, and Validate Constraints

### Disable a Constraint

```sql
ALTER TABLE employees_demo
DISABLE CONSTRAINT chk_employees_salary;
```

### Enable a Constraint

```sql
ALTER TABLE employees_demo
ENABLE CONSTRAINT chk_employees_salary;
```

### Enable Without Validating All Existing Rows

```sql
ALTER TABLE employees_demo
ENABLE NOVALIDATE CONSTRAINT chk_employees_salary;
```

`ENABLE NOVALIDATE` enforces the constraint for new or modified data without checking all pre-existing rows. Existing data may therefore violate the constraint.

### Validate Existing Data

```sql
ALTER TABLE employees_demo
ENABLE VALIDATE CONSTRAINT chk_employees_salary;
```

Validation fails if existing data violates the constraint.

---

## 11. Common Interview Questions and Answers

### Q1. What are SQL constraints?

SQL constraints are rules applied to columns or tables to maintain data integrity and prevent invalid data.

### Q2. What is the difference between PRIMARY KEY and UNIQUE?

- A table can have only one primary key constraint, but multiple UNIQUE constraints.
- A primary key does not permit NULL values.
- A UNIQUE constraint permits NULL values in Oracle.
- Both enforce uniqueness according to their respective rules.

### Q3. Can a table have multiple primary keys?

No. A table can have only one primary key constraint, although that key may contain multiple columns.

### Q4. Can a foreign key contain NULL?

Yes, unless the foreign key column is also defined as NOT NULL.

### Q5. Can a UNIQUE constraint contain multiple NULL values in Oracle?

Yes. Oracle permits multiple NULL values in a single-column UNIQUE constraint.

### Q6. What happens when you insert a duplicate primary key?

Oracle raises a constraint violation, commonly `ORA-00001: unique constraint violated`.

### Q7. What happens when a foreign key references a nonexistent parent row?

Oracle rejects the insert or update, commonly with `ORA-02291: integrity constraint violated - parent key not found`.

### Q8. How do you remove a constraint?

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

For a primary key, `DROP PRIMARY KEY` is also supported.

### Q9. What is the difference between a constraint and a DEFAULT value?

A constraint enforces a data rule. DEFAULT supplies a value when the column is omitted or DEFAULT is explicitly requested in an INSERT statement.

### Q10. Can we add NOT NULL to an existing table?

Yes, using `ALTER TABLE ... MODIFY`. Existing rows must not contain NULL in that column.

### Q11. What is a composite primary key?

A primary key consisting of two or more columns, where their combined values uniquely identify each row.

### Q12. How can you find the system-generated constraint name?

Query `USER_CONSTRAINTS` using the table name, and use the returned name in `ALTER TABLE ... DROP CONSTRAINT`.

---

## 12. Quick Revision

| Constraint | Duplicate Values | NULL Allowed? | Main Purpose |
|---|---|---|---|
| NOT NULL | Yes | No | Require a value |
| UNIQUE | No duplicate non-NULL keys | Yes | Enforce uniqueness |
| PRIMARY KEY | No | No | Identify a row |
| FOREIGN KEY | Yes | Yes, unless separately restricted | Maintain table relationships |
| CHECK | Depends on condition | Potentially | Enforce a condition |
| DEFAULT | Depends on other rules | Yes, unless separately restricted | Supply an omitted value |

> **Final Interview Tip:** Remember the core difference: PRIMARY KEY identifies a row, UNIQUE prevents duplicate keys, FOREIGN KEY connects tables, CHECK validates a condition, NOT NULL requires a value, and DEFAULT supplies an omitted value.

