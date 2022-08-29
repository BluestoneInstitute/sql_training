# SQL Module 1: Introduction to SQL

Structured Query Language or “SQL” is a scripting language used to query databases. Invented in the 1970s by IBM, it is now ubiquitous with versions available in statistical software like SAS (PROC SQL) and R (sqldf). Other software like Stata (odbc) and Python (pyodbc) allow you to connect to a SQL server and run queries on the data stored there. These are not the only use cases for SQL; there are many, many more. 

### Videos
* [What is SQL? SQL for Beginners 2022 – Learn SQL Step by Step](https://www.youtube.com/watch?v=dFywO9lQ6MQ) (59 seconds)

### Basics SQL Syntax
SQL is a pretty intuitive language. You basically type what you're thinking. There are 3 main keywords included as part of a SQL query:

* SELECT - list the column(s) you want to retrieve
* FROM - used to specify which table to select or delete data from.
* WHERE -  command filters a result set to include only records that fulfill a specified condition.

##### Notes: [W3 Schools SQL Keywords Reference](https://www.w3schools.com/sql/sql_ref_keywords.asp)

The order in which the commands are written matters. For example SELECT always must precede FROM and WHERE must always follow FROM. The SQL query will not run if the keywords are entered out of order. A common SQL command will have the following order of keywords:

```sql
SELECT fieldname1, fieldname2, ..., fieldname100
FROM tablename
WHERE fieldname100 >= 10
--value can be text ‘value’ or value depending on the type of data contained in fieldname100.
```
* _Optional_: [Learn SQL: SQL Order of Operations](https://learnsql.com/blog/sql-order-of-operations/)

> **BEST PRACTICE:**
> Add comments to SQL queries by adding '--' before a statement.

### Other Useful Keywords

The SQL language has a multitude of other keywords. Some of the most commonly used include:

* DISTINCT - selects only distinct (different) values
* GROUP BY - used to group the result set (used with aggregate functions: COUNT, MAX, MIN, SUM, AVG).
* ORDER BY - sorts the result set in ascending order by default. To sort the records in descending order, use the DESC keyword
* UNION - combines the result set of two or more SELECT statements (only distinct values)
* UNION ALL - combines the result set of two or more SELECT statements (allows duplicate values).
* INNER JOIN - returns rows that have matching values in both tables
* OUTER JOIN - returns all rows when there is a match in either left table or right table
* LEFT JOIN -  returns all rows from the left table, and the matching rows from the right table. The result is NULL from the right side, if there is no match.
* RIGHT JOIN - returns all rows from the right table, and the matching records from the left table. The result is NULL from the left side, when there is no match.
* LIKE - keyword is used in a WHERE clause to search for a specified pattern in a column
* CASE - creates different outputs based on conditions

##### Notes: [W3 Schools SQL Keywords Reference](https://www.w3schools.com/sql/sql_ref_keywords.asp)

### _NULL_ Values
SQL databases include have a special class of values called "NULL". In SQL, NULL is a special _marker_ used to indicate that a data _value_ does not exist in the database. Because NULL is a _marker_ and not a _value_ it is possible for an analysis to yield different results when NULL values are included versus excluded. Moreover, data analysts familiar with SAS or Stata may find NULLs counterintuitive.  This is because neither SAS nor Stata use NULL values and instead treat missing data as "." (for numeric values) or "" (for character values).

The existence of the NULL marker is somewhat [controversial](https://en.wikipedia.org/wiki/Null_(SQL)#Controversy). And "misunderstanding how NULL works is the cause of a great number of errors in SQL code."

> **BEST PRACTICE NOTE:**
> Always be alert to the potential for NULL values in a SQL database.

## SQL Resources

[SQL Cheat Sheet](https://www.sqltutorial.org/wp-content/uploads/2016/04/SQL-cheat-sheet.pdf)
