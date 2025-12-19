SQL Joins
1. What is a JOIN in SQL?
A JOIN in SQL is an operation that combines rows from two or more tables based on a related column, typically a primary key in one table and a foreign key in another. JOINs allow normalized data stored across multiple tables in PostgreSQL to be retrieved as a single logical dataset. They are essential for maintaining data integrity, avoiding duplication, and answering business questions that involve relationships between entities.
________________________________________
2. Why do we need JOINs?
JOINs are needed because relational databases store related data in separate tables. Instead of repeating information, relationships are created using keys. JOINs enable efficient querying of this related data, simplify application logic, and support reporting and analytics directly in SQL.
________________________________________
3. Types of JOINs in PostgresSQL
3.1  INNER JOIN:

An INNER JOIN returns only the rows where a matching record exists in both tables.

Explanation:
If a row in either table does not satisfy the join condition, it is excluded from the result set.


Example:
SELECT c.customer_name, o.order_id
FROM customers c
INNER JOIN orders o
ON c.customer_id = o.customer_id;
Use cases:
•	Fetching orders that belong to valid customers
•	Listing employees assigned to departments
•	Reporting completed or mandatory relationships
Best practices:
•	Join on indexed columns (primary and foreign keys)
•	Apply filters early to reduce dataset size
•	Use clear aliases for readability
Pitfalls:
•	Assuming all rows from the main table will be returned
•	Losing data when related records are missing

3.2  LEFT JOIN :

A LEFT JOIN (LEFT OUTER JOIN) returns all rows from the left table and matching rows from the right table. Unmatched rows return NULL values for right-table columns.

Explanation:
LEFT JOIN preserves all rows from the left table, regardless of whether a match exists.

Example:
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id;

Use cases:
•	Finding customers without orders
•	Auditing missing or optional data
•	Reporting complete master data

Best practices:
•	Use LEFT JOIN when related data is optional
•	Place filters on the right table inside the JOIN condition
•	Explicitly check for NULL values when identifying missing data

Pitfalls:
Incorrect:
WHERE o.order_id IS NOT NULL;
This converts the LEFT JOIN into an INNER JOIN and removes unmatched rows.
3.3 RIGHT JOIN 
A RIGHT JOIN (RIGHT OUTER JOIN) returns all rows from the right table and matching rows from the left table.

Explanation:
It is logically the reverse of a LEFT JOIN.

Example:
SELECT c.customer_name, o.order_id
FROM customers c
RIGHT JOIN orders o
ON c.customer_id = o.customer_id;

Use cases:
•	Rarely used; occasionally to emphasize the right table
•	Can always be rewritten as a LEFT JOIN

Best practices:
•	Prefer LEFT JOIN for clarity and consistency
•	Use RIGHT JOIN only when it improves readability

Common pitfalls:
•	Reduced readability in complex queries
•	Confusion during maintenance

3.4. FULL JOIN 
A FULL JOIN (FULL OUTER JOIN) returns all rows from both tables, including matched and unmatched rows from each side.
Explanation:
Rows without matches on either side contain NULL values for missing columns.
Example:
SELECT c.customer_name, o.order_id
FROM customers c
FULL JOIN orders o
ON c.customer_id = o.customer_id;
Use cases:
•	Data reconciliation between systems
•	Finding mismatched or missing records
•	Comparing datasets during migrations

Best practices:
•	Use for analysis and auditing purposes
•	Handle NULL values explicitly
•	Document query intent clearly

Pitfalls:
•	Confusing FULL JOIN with UNION
•	Misinterpreting NULL values
•	Unexpected row counts when multiple matches exist

3.5 CROSS JOIN 
A CROSS JOIN returns the Cartesian product of two tables, combining every row from the first table with every row from the second.
Explanation:
	No join condition is required; result size is rows(table1) × 		rows(table2)

Example:
SELECT d.date, p.product_name
FROM dates d
CROSS JOIN products p;

Use cases:
•	Generating combinations
•	Creating reporting dimensions
•	Test data generation


Best practices:
•	Use only with small or controlled datasets
•	Estimate result size before execution
•	Clearly document intent

Common pitfalls:
•	Unintentional massive result sets
•	Severe performance degradation

4. ERROR: Syntax error at or near "JOIN" (SQLSTATE 42601)
Explanation:
This error indicates invalid SQL syntax where a JOIN keyword is used incorrectly. It typically occurs when the JOIN clause is placed in the wrong order, missing required keywords (such as ON or USING), or combined improperly with other clauses.
________________________________________
Use Cases:
•	Writing multi-table queries with incorrect clause ordering
•	Forgetting ON conditions in JOIN statements
•	Mixing old-style comma joins with explicit JOIN syntax
•	Dynamically generating SQL that produces malformed joins
________________________________________
Examples:
Incorrect JOIN syntax:
SELECT *
FROM employees
JOIN departments;
(Missing ON or USING clause)

Correct JOIN syntax:
SELECT *
FROM employees e
JOIN departments d
  ON e.dept_id = d.id;

Incorrect clause order:
SELECT *
JOIN departments d
FROM employees e;

Correct clause order:
SELECT *
FROM employees e
JOIN departments d
  ON e.dept_id = d.id;
________________________________________
Best Practices:
•	Follow correct SQL clause order: SELECT → FROM → JOIN → WHERE
•	Always include ON or USING with JOIN statements
•	Use explicit JOIN syntax instead of comma-separated tables
•	Format SQL clearly to make syntax errors easier to detect
•	Validate generated SQL before execution
________________________________________
Pitfalls:
•	Omitting the ON condition in a JOIN
•	Placing JOIN before the FROM clause
•	Mixing implicit and explicit join styles
•	Leaving trailing commas before JOIN
•	Misplaced parentheses in complex joins

5. ERROR: column reference "column_name" is ambiguous 
    (SQLSTATE 42702)

Explanation:
This error occurs when a query references a column name that exists in more than one table involved in the query, and the database cannot determine which one to use. It commonly appears in SELECT, WHERE, JOIN, or ORDER BY clauses when column names are not properly qualified.
________________________________________
Use Cases :
•	Joining tables that share common column names such as id, name, or created_at
•	Writing queries without table aliases in multi-table joins
•	Adding new joins to existing queries that introduce duplicate column names
•	Using ORDER BY with unqualified column names in joined queries
________________________________________
Examples:
Ambiguous column reference:
SELECT id
FROM employees e
JOIN departments d
  ON e.department_id = d.id;
(Both tables contain an id column)

Resolved by qualifying column:
SELECT e.id
FROM employees e
JOIN departments d
  ON e.department_id = d.id;

Ambiguous column in WHERE clause:
SELECT *
FROM orders o
JOIN customers c
  ON o.customer_id = c.id
WHERE id = 10;

Corrected WHERE clause:
SELECT *
FROM orders o
JOIN customers c
 ON o.customer_id = c.id
WHERE o.id = 10;
________________________________________
Best Practices :
•	Always qualify column names when using multiple tables
•	Use clear and consistent table aliases
•	Explicitly reference columns in SELECT and ORDER BY clauses
•	Review queries after adding new joins to avoid ambiguity
•	Prefer descriptive column names when designing schemas

 Pitfalls:
•	Assuming the database will infer the correct table
•	Forgetting to update SELECT or WHERE clauses after adding joins
•	Using SELECT * in complex joins
•	Reusing generic column names across many tables
•	Relying on implicit column resolution in multi-table queries

6. ERROR: missing FROM-clause entry for table "table_name"  
     (SQLSTATE 42P01)
Explanation:
This error occurs when a query references a table or table alias that is not listed in the FROM clause or joined properly. The database cannot find the specified table in the query scope.
________________________________________
Use Cases :
•	Referencing a table in SELECT, WHERE, or JOIN clauses without including it in FROM
•	Using an incorrect or undefined table alias
•	Removing a join but leaving column references to that table
•	Writing subqueries that reference tables not available in their scope
Examples :
Missing table in FROM clause:
SELECT o.id, c.name
FROM orders o
WHERE c.id = o.customer_id;
(Table customers is referenced but not included)

Corrected query with JOIN:
SELECT o.id, c.name
FROM orders o
JOIN customers c
  ON c.id = o.customer_id;

Incorrect alias usage:
SELECT d.name
FROM departments dept;
(Alias d is not defined)

Corrected alias usage:
SELECT dept.name
FROM departments dept;

Best Practices:
•	Ensure all referenced tables and aliases appear in the FROM clause
•	Use consistent and meaningful table aliases
•	Review queries after removing or modifying joins
•	Keep joins explicit and readable
•	Validate query scope when using subqueries
________________________________________
Pitfalls:
•	Typographical errors in table or alias names
•	Forgetting to add a required JOIN
•	Using aliases inconsistently across the query
•	Copying query fragments without updating table references
•	Misunderstanding table visibility in subqueries

7. ERROR: relation "table_name" does not exist (SQLSTATE 
     42P01)

Explanation:
This error occurs when a query references a table or view that the database cannot find. The specified relation does not exist in the current schema or is named incorrectly.
________________________________________
Use Cases:
•	Querying a table that has not been created
•	Misspelling table or view names
•	Referencing tables in the wrong schema

•	Using incorrect case-sensitive identifiers
•	Running queries in a different database or schema context

Examples:
Non-existent table reference:
SELECT *
FROM employee;
(Table name is incorrect)

Corrected table reference:
SELECT *
FROM employees;
Missing schema qualification:
SELECT *
FROM orders;

Corrected with schema:
SELECT *
FROM sales.orders;

Best Practices:
•	Verify table and view names before writing queries
•	Maintain up-to-date schema documentation
•	Follow consistent naming conventions
•	Confirm database and schema context before execution

Pitfalls:
•	Typographical errors in table names
•	Assuming default schema contains the table
•	Confusing table names with similar views
•	Ignoring case sensitivity when identifiers are quoted
•	Running scripts in the wrong environment

________________________________________

8. ERROR: column "column_name" does not exist 
    (SQLSTATE  42703)
Explanation:
This error occurs when a query references a column name that the database cannot find in the specified table or result set. It commonly happens in SELECT clauses or JOIN conditions due to:
•	Misspelled column names
•	Using columns from the wrong table alias
•	Referencing columns not included in subqueries or views
•	Case-sensitivity issues (in systems that enforce it)
________________________________________
Use Cases:
•	Joining multiple tables using incorrect or missing column references
•	Selecting columns that exist in a different table than assumed
•	Refactoring schemas where column names changed but queries were not updated
•	Working with aliases in complex joins and forgetting to qualify columns
________________________________________
Examples :
Incorrect JOIN (causes error):
SELECT e.id, d.name
FROM employees e
JOIN departments d
ON e.dept_id = d.department_id;
(If department_id does not exist in departments)

Corrected JOIN:
SELECT e.id, d.name FROM employees e
JOIN departments d ON e.dept_id = d.id;

Incorrect SELECT with alias:
SELECT name FROM employees e;
(If name exists only in another table)


Corrected SELECT:
SELECT e.full_name
FROM employees e;
Best Practices:
•	Always verify column names against the table schema before writing queries
•	Use table aliases consistently and qualify column names (alias.column)
•	Keep schema documentation updated and accessible
•	Validate queries incrementally when building complex joins
•	Use descriptive, consistent naming conventions for columns
________________________________________

 Pitfalls:
•	Assuming column names are the same across related tables
•	Forgetting to update queries after schema changes
•	Mixing up table aliases in JOIN conditions
•	Relying on SELECT * and later referencing non-existent columns
•	Ignoring case sensitivity rules in column definitions
 ________________________________________
9. ERROR: FULL JOIN is only supported with merge-joinable join conditions (SQLSTATE 0A000)
Explanation:
This error occurs when a FULL OUTER JOIN uses join conditions that are not based on equality (=). Some databases require FULL JOIN conditions to be merge-joinable, meaning the join must be defined using equality between columns from both tables.

Use Cases

•	Attempting a FULL OUTER JOIN with range conditions (<, >, <=, >=)
•	Using expressions or functions in FULL JOIN conditions
•	Migrating queries from databases with different FULL JOIN support
•	Writing analytical queries that compare partially matching datasets
________________________________________
Examples:
Unsupported FULL JOIN condition:

SELECT *
FROM orders o
FULL JOIN invoices i
  ON o.order_date > i.invoice_date;


Supported FULL JOIN using equality:
SELECT * FROM orders o
FULL JOIN invoices i ON o.order_id = i.order_id;


Restructured query using UNION:


SELECT *FROM orders o LEFT JOIN invoices i
         ON o.order_date > i.invoice_date
UNION
SELECT *FROM orders o RIGHT JOIN invoices i
ON o.order_date > i.invoice_date;
________________________________________

   Best Practices:

•	Use equality (=) conditions in FULL OUTER JOIN clauses
•	Precompute expressions in subqueries before joining
•	Consider LEFT JOIN / RIGHT JOIN with UNION for non-equality logic
•	Keep join conditions simple and index-friendly
•	Test join behavior with representative data
________________________________________
Pitfalls:
•	Using range or inequality conditions in FULL JOIN
•	Applying functions directly in join conditions
•	Assuming all databases support arbitrary FULL JOIN conditions
•	Overcomplicating join logic instead of restructuring queries
•	Ignoring database-specific join limitations


10. ERROR: invalid reference to FROM-clause entry for  
      table "table_name" (SQLSTATE 42P01)
Explanation:
This error occurs when a query references a table name instead of its defined alias, or references a table in a scope where it is not valid. Once a table is given an alias in the FROM clause, the original table name can no longer be used elsewhere in the query.
________________________________________
Use Cases:
•	Using the base table name instead of its alias in JOIN conditions
•	Referencing tables outside their valid query scope
•	Writing correlated subqueries with incorrect table references
•	Modifying queries to add aliases without updating all references
________________________________________
Examples:
Invalid reference using table name instead of alias:
SELECT *
FROM employees e
JOIN departments d
  ON employees.dept_id = d.id;

        Correct reference using alias:
SELECT * FROM employees e
JOIN departments d
  ON e.dept_id = d.id;
        Invalid reference in subquery:
SELECT *FROM orders o
WHERE EXISTS (
  SELECT 1 FROM customers
  WHERE customers.id = o.customer_id
);
(If customers is aliased in the subquery but referenced incorrectly)
Corrected subquery:
SELECT * FROM orders o
WHERE EXISTS (
  SELECT 1 FROM customers c
  WHERE c.id = o.customer_id
);
________________________________________
Best Practices
•	Always use table aliases consistently after they are defined
•	Choose clear, short aliases and apply them throughout the query
•	Review all join and filter conditions after introducing aliases
•	Keep query scopes clear when using subqueries
•	Format SQL to make alias usage obvious
________________________________________
Pitfalls:
•	Mixing table names and aliases in the same query
•	Forgetting to update references after adding aliases
•	Reusing the same alias for multiple tables
•	Copying query fragments without adjusting aliases
________________________________________

11. ERROR: could not determine which collation to use for  
      string comparison (SQLSTATE 42P22)
Explanation:
This error occurs when a query compares text columns that use different or unspecified collations, and the database cannot automatically choose which collation rules to apply. It commonly appears in JOIN conditions or WHERE clauses involving string comparisons.
________________________________________
Use Cases:
•	Joining tables with text columns defined using different collations
•	Comparing text columns from different schemas or databases
•	Migrating data between systems with varying collation settings
•	Using expressions or literals with text columns in join conditions
________________________________________
Examples:
Join causing collation conflict:
SELECT * FROM users u
JOIN accounts a ON u.username = a.username;
(Columns use different collations)

Resolved with explicit collation:
SELECT *
FROM users u
JOIN accounts a
ON u.username COLLATE "en_US" = a.username COLLATE "en_US";

Collation specified in WHERE clause:
SELECT * FROM products p
WHERE p.name COLLATE "en_US" = 'Laptop' COLLATE "en_US";
________________________________________
Best Practices:
•	Use consistent collations for related text columns
•	Explicitly specify collation in joins when mixing sources
•	Standardize collation settings at database or schema level
•	Document collation choices in schema design
•	Review collation behavior during migrations

 Pitfalls:
•	Assuming all text columns share the same collation
•	Ignoring collation differences across schemas
•	Comparing text columns to literals without collation
•	Overlooking collation issues in join conditions
•	Relying on default collation inference

12. ERROR: operator does not exist: type1 = type2 
      (SQLSTATE 42883)
Explanation:
This error occurs when a query attempts to compare two columns or values of incompatible data types using an operator (commonly =). The database cannot find a valid operator that works for both types, which often happens in JOIN conditions.
________________________________________
Use Cases:
•	Joining tables where key columns have different data types
•	Comparing numeric columns to text columns
•	Migrating schemas where column types changed inconsistently
•	Using literals that do not match column data types in joins
________________________________________
Examples:
Incompatible types in JOIN condition:
SELECT * FROM orders o
JOIN customers c
  ON o.customer_id = c.id;
(If orders.customer_id is TEXT and customers.id is INTEGER)

Resolved with explicit casting:
SELECT * FROM orders o JOIN customers c ON o.customer_id::INTEGER = c.id;

Casting the other side:
SELECT *FROM orders o
JOIN customers c ON o.customer_id = c.id::TEXT;

Best Practices:
•	Align data types for related columns at schema design time
•	Use explicit casts in join conditions when types differ
•	Prefer casting smaller or indexed columns cautiously
•	Document expected data types for join keys
•	Validate data before applying type casts

Pitfalls:
•	Assuming implicit type conversion will occur
•	Casting columns in ways that prevent index usage
•	Overlooking hidden type differences (e.g., UUID vs TEXT)
•	Using incorrect cast syntax for the database system
•	Allowing inconsistent data types across related tables
________________________________________
13. ERROR: JOIN/ON clause refers to "table_name"
      which is not part of JOIN (SQLSTATE 42P01)
Explanation:
This error occurs when a JOIN ... ON condition references a table or alias that is not included in the current join operation. Each ON clause may only reference the two relations being joined at that point in the query.

Use Cases:
•	Referencing a previously joined table in a later JOIN condition incorrectly
•	Using the wrong table alias in an ON clause
•	Reordering joins without updating join conditions
•	Writing complex multi-join queries without clear alias tracking
________________________________________
Examples:
Invalid JOIN condition:
SELECT * FROM orders o JOIN customers c
ON o.customer_id = c.id  JOIN payments p ON p.customer_id = o.customer_id AND p.status = s.status;
(Alias s is not part of this JOIN)

Corrected JOIN condition:
SELECT * FROM orders o JOIN customers c
  ON o.customer_id = c.id  JOIN payments p
  ON p.customer_id = o.customer_id;

Correct approach using proper join order:
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN statuses s ON s.id = o.status_id
JOIN payments p ON p.customer_id = o.customer_id 
AND p.status = s.status;
Best Practices:
•	Reference only the two tables involved in each JOIN ... ON clause
•	Use clear and consistent table aliases
•	Order joins logically based on dependencies
•	Review join conditions after adding or reordering joins
•	Keep join logic simple and readable

Pitfalls:
•	Using aliases that are not yet introduced
•	Assuming all joined tables are visible in every ON clause
•	Copying join conditions without adjusting table references
•	Losing track of aliases in large queries
________________________________________

14. ERROR: subquery in FROM cannot refer to other 
      relations of same query level (SQLSTATE 42P01)
      Explanation:
This error occurs when a subquery in the FROM clause (a derived table) references columns from other tables at the same query level. In SQL, derived tables cannot access columns outside their own scope; such references must be moved to a WHERE clause or rewritten as a correlated subquery.

Use Cases:
•	Using a derived table in the FROM clause that incorrectly references outer query tables
•	Attempting to simplify joins with subqueries that depend on other tables at the same level
•	Migrating queries from databases that allow non-standard cross-level references
•	Writing complex aggregation queries with subquery joins

Examples:
Invalid subquery in FROM:

       SELECT o.id, sub.total
                FROM orders o,
               (SELECT SUM(amount) AS total FROM payments
               WHERE payments.order_id = o.id) sub;
(Cannot reference o.id inside the subquery in FROM)

Corrected using correlated subquery in SELECT:
SELECT o.id,
       (SELECT SUM(amount) FROM payments p
        WHERE p.order_id = o.id) AS total FROM orders o;

Alternative with JOIN:
SELECT o.id, SUM(p.amount) AS total
FROM orders o LEFT JOIN payments p
  ON p.order_id = o.id
GROUP BY o.id;
Best Practices:
•	Avoid referencing outer query columns inside FROM subqueries
•	Use correlated subqueries in SELECT or WHERE instead
•	Consider rewriting subqueries as joins with aggregation
•	Keep derived tables self-contained
•	Test query logic incrementally when refactoring

Pitfalls :
•	Mixing subquery scopes and outer query references
•	Assuming all databases allow cross-level references
•	Forgetting to alias derived tables properly
•	Ignoring performance implications of correlated subqueries vs joins
________________________________________
15. ERROR: cannot use aggregate function in JOIN 
      condition (SQLSTATE 42803)

Explanation:
This error occurs when an aggregate function (e.g., SUM, COUNT, AVG, MAX, MIN) is used directly in a JOIN ... ON condition. SQL does not allow aggregates in join conditions because aggregation requires grouping of rows, which happens after the join phase.

Use Cases:
•	Joining tables while trying to filter by a total or average value
•	Using SUM, COUNT, etc., directly in a JOIN rather than in a subquery
•	Writing queries that mix row-level and aggregated data incorrectly
•	Attempting to simplify queries without proper grouping or subquery usage

Examples:
Invalid JOIN using aggregate:
SELECT *
FROM orders o
JOIN payments p
  ON SUM(p.amount) > 100 AND p.order_id = o.id;
(Cannot use SUM(p.amount) in ON condition)

Corrected using subquery:
SELECT *FROM orders o
JOIN (
    SELECT order_id, SUM(amount) AS total_amount
    FROM payments
    GROUP BY order_id
) p_total
  ON p_total.order_id = o.id
WHERE p_total.total_amount > 100;

     Best Practices:
•	Perform aggregation in subqueries or with GROUP BY + HAVING
•	Keep join conditions limited to row-level comparisons
•	Validate join logic after moving aggregation to subqueries
•	Test queries on small datasets to verify aggregation behavior

    Pitfalls:
•	Placing aggregate functions directly in ON or WHERE clauses
•	Confusing row-level and aggregated data scopes
•	Forgetting to include GROUP BY when aggregating
•	Assuming all databases allow aggregates in join conditions
________________________________________
16. ERROR: window function call requires an OVER clause 
      (SQLSTATE 42P20)
Explanation:
This error occurs when a window function (e.g., ROW_NUMBER(), RANK(), SUM() OVER, AVG() OVER) is used without an OVER clause. The OVER clause defines the partitioning and ordering of rows for the window function. Window functions cannot be used in a JOIN condition or elsewhere without specifying OVER.
________________________________________
Use Cases:
•	Using ROW_NUMBER(), RANK(), SUM(), AVG() or other window functions without OVER
•	Attempting to filter or join on a window function result directly
•	Writing analytical queries that require row numbering or running totals
•	Combining window functions with joins or aggregations incorrectly
________________________________________
Examples:
Invalid use without OVER clause:
SELECT *
FROM orders o
JOIN payments p
  ON ROW_NUMBER() = 1;

Correct use with OVER clause:
SELECT *
FROM (SELECT *,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY       payment_date) AS rn FROM payments) p
JOIN orders o ON p.order_id = o.id WHERE p.rn = 1;

Window function with partition and order:
SELECT order_id,
SUM(amount) OVER (PARTITION BY customer_id 
ORDER BY payment_date) AS running_total
FROM payments;
________________________________________

Best Practices:
•	Always include an OVER clause with window functions
•	Use subqueries to compute window functions before joining
•	Partition and order data explicitly for clarity and correctness
•	Avoid using window functions directly in join conditions

Pitfalls:
•	Forgetting to include OVER with a window function
•	Using window functions in places that expect scalar values
•	Ignoring partitioning and ordering implications
•	Attempting to use window functions in JOIN ON conditions directly

17. ERROR: table name "table_name" specified more than 
      once (SQLSTATE 42712)
Explanation:
This error occurs when the same table is included multiple times in a query without using distinct aliases. SQL requires each reference to a table in a query to have a unique identifier if it appears more than once.
________________________________________
Use Cases:
•	Self-joins where a table is joined to itself
•	Joining a table multiple times for different purposes (e.g., comparing previous and current records)
•	Writing complex queries without properly aliasing repeated table references
Examples:
Invalid query with repeated table name:
SELECT * FROM employees
JOIN employees ON employees.manager_id = employees.id;

Corrected using aliases:
SELECT e.id, e.name, m.name AS manager_name
FROM employees e
JOIN employees m
  ON e.manager_id = m.id;

Multiple joins with the same table:
SELECT o.id, c1.name AS billing_name, c2.name AS shipping_name
FROM orders o
JOIN customers c1 ON o.billing_customer_id = c1.id
JOIN customers c2 ON o.shipping_customer_id = c2.id;
________________________________________
Best Practices:
•	Always assign unique aliases when referencing the same table multiple times
•	Use descriptive alias names to indicate the role of each table reference
•	Apply consistent aliasing across the query for clarity
•	Keep query structure readable when self-joining tables

Pitfalls:
•	Forgetting to alias one or more repeated table references
•	Using the same alias for multiple tables
•	Confusing column references when aliases are not used
•	Copying queries with repeated tables without updating aliases
________________________________________
18. ERROR: cross-database references are not implemented 
      (SQLSTATE 0A000)
Explanation:
This error occurs when a query attempts to join tables across different databases in PostgreSQL. PostgreSQL does not support cross-database queries directly; all tables referenced in a JOIN must exist within the same database.
________________________________________
Use Cases:
•	Attempting to join tables located in separate PostgreSQL databases
•	Trying to integrate multiple data sources without using foreign data wrappers
•	Combining legacy and new databases in a single query
________________________________________
Examples:
Invalid cross-database join:
SELECT * FROM db1.orders o
JOIN db2.customers c
 ON o.customer_id = c.id;
Correct approach using foreign data wrapper (FDW):

CREATE EXTENSION IF NOT EXISTS postgres_fdw;
CREATE SERVER foreign_server FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'localhost', dbname 'db2', port '5432');

CREATE USER MAPPING FOR current_user
SERVER foreign_server
OPTIONS (user 'username', password 'password');

IMPORT FOREIGN SCHEMA public
FROM SERVER foreign_server
INTO public;

Then join using the foreign table
SELECT * FROM orders o
JOIN customers_foreign c
  ON o.customer_id = c.id;
________________________________________
Best Practices
•	Keep all tables involved in a join within the same database whenever possible
•	Document database locations and dependencies for queries
•	Consider ETL processes to consolidate data before joining
Pitfalls:
•	Assuming cross-database joins are allowed like in other RDBMS
•	Forgetting to set up FDWs for external databases
•	Overcomplicating queries instead of consolidating data
•	Using inconsistent schemas or table names across databases
________________________________________
19. ERROR: could not find common type for columns 
      (SQLSTATE 42804)

Explanation:
This error occurs when a query attempts to join or compare columns that do not have compatible data types, and the database cannot implicitly convert them to a common type. The error commonly appears in JOIN conditions or UNION queries.
________________________________________
Use Cases:
•	Joining tables where key columns have different but incompatible types
•	Comparing numeric columns to text columns
•	Joining columns with different domain types (e.g., INTEGER vs BIGINT)
________________________________________
Examples:
Invalid JOIN with incompatible types:


SELECT *FROM orders o
JOIN customers c
 ON o.customer_id = c.customer_code;
(If orders.customer_id is INTEGER andcustomers.customer_code is TEXT)

Resolved with explicit cast:
SELECT *
FROM orders o
JOIN customers c
ON o.customer_id::TEXT = c.customer_code;

Alternative by casting the other column:
SELECT * FROM orders o JOIN customers c
ON o.customer_id = c.customer_code::INTEGER;
________________________________________
Best Practices:
•	Ensure columns used in JOIN conditions have compatible types at schema design time
•	Use explicit type casting when necessary
•	Document data types for key columns used in joins
•	Test queries for type compatibility on representative data



Pitfalls:
•	Assuming implicit type conversion will occur automatically
•	Casting columns in ways that prevent index usage
•	Overlooking subtle differences, e.g., CHAR vs VARCHAR
•	Using different numeric types without considering precision or range
________________________________________
20. ERROR: JOIN condition cannot be empty (SQLSTATE 
      42601)
Explanation:
This error occurs when a JOIN is specified without a proper ON or USING clause. SQL requires that joins explicitly define the condition for combining rows from the participating tables, except for CROSS JOIN.
________________________________________
Use Cases:
•	Writing INNER JOIN, LEFT JOIN, RIGHT JOIN, or FULL JOIN without specifying ON or USING
•	Omitting join conditions while attempting to match related tables
•	Confusing CROSS JOIN with other types of joins
________________________________________
Examples:
Invalid JOIN without condition:
SELECT * FROM orders o
JOIN customers c;

Corrected JOIN using ON clause:
SELECT * FROM orders o
JOIN customers c
ON o.customer_id = c.id;

CROSS JOIN without condition (valid):
SELECT * FROM orders o
CROSS JOIN customers c;
________________________________________
Best Practices:
•	Always specify ON or USING for non-CROSS joins
•	Use clear and consistent table aliases in join conditions
•	Validate join logic to ensure it matches intended relationships
________________________________________
Pitfalls:
•	Forgetting to add the join condition when copying query templates
•	Confusing CROSS JOIN with INNER JOIN or LEFT JOIN
•	Assuming SQL will automatically infer the join condition






