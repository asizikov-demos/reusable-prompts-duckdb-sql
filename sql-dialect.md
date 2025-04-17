# DuckDB SQL Syntax reference

## DuckDB's SQL Dialect {#sql:dialect}

### Overview 

DuckDB's SQL dialect is based on PostgreSQL.
DuckDB tries to closely match PostgreSQL's semantics, however, some use cases require slightly different behavior.
For example, interchangeability with data frame libraries necessitates [order preservation of inserts](#docs:stable:sql:dialect:order_preservation) to be supported by default.
These differences are documented in the pages below.

### Indexing 
DuckDB uses 1-based indexing except for [JSON objects](#docs:stable:data:json:overview), which use 0-based indexing.

### Examples

The index origin is 1 for strings, lists, etc.

``` sql
SELECT list[1] AS element
FROM (SELECT ['first', 'second', 'third'] AS list);
```

``` text
┌─────────┐
│ element │
│ varchar │
├─────────┤
│ first   │
└─────────┘
```

The index origin is 0 for JSON objects.

``` sql
SELECT json[1] AS element
FROM (SELECT '["first", "second", "third"]'::JSON AS json);
```

``` text
┌──────────┐
│ element  │
│   json   │
├──────────┤
│ "second" │
└──────────┘
```

### Friendly SQL

DuckDB offers several advanced SQL features and syntactic sugar to make SQL queries more concise. We refer to these colloquially as “friendly SQL”.

> Several of these features are also supported in other systems while some are (currently) exclusive to DuckDB.

### Clauses 

-   Creating tables and inserting data:
    -   [`CREATE OR REPLACE TABLE`](#docs:stable:sql:statements:create_table::create-or-replace): avoid `DROP TABLE IF EXISTS` statements in scripts.
    -   [`CREATE TABLE ... AS SELECT` (CTAS)](#docs:stable:sql:statements:create_table::create-table--as-select-ctas): create a new table from the output of a table without manually defining a schema.
    -   [`INSERT INTO ... BY NAME`](#docs:stable:sql:statements:insert::insert-into--by-name): this variant of the `INSERT` statement allows using column names instead of positions.
    -   [`INSERT OR IGNORE INTO ...`](#docs:stable:sql:statements:insert::insert-or-ignore-into): insert the rows that do not result in a conflict due to `UNIQUE` or `PRIMARY KEY` constraints.
    -   [`INSERT OR REPLACE INTO ...`](#docs:stable:sql:statements:insert::insert-or-replace-into): insert the rows that do not result in a conflict due to `UNIQUE` or `PRIMARY KEY` constraints. For those that result in a conflict, replace the columns of the existing row to the new values of the to-be-inserted row.
-   Describing tables and computing statistics:
    -   [`DESCRIBE`](#docs:stable:guides:meta:describe): provides a succinct summary of the schema of a table or query.
    -   [`SUMMARIZE`](#docs:stable:guides:meta:summarize): returns summary statistics for a table or query.
-   Making SQL clauses more compact and readable:
    -   [`FROM`-first syntax with an optional `SELECT` clause](#docs:stable:sql:query_syntax:from::from-first-syntax): DuckDB allows queries in the form of `FROM tbl` which selects all columns (performing a `SELECT *` statement).
    -   [`GROUP BY ALL`](#docs:stable:sql:query_syntax:groupby::group-by-all): omit the group-by columns by inferring them from the list of attributes in the `SELECT` clause.
    -   [`ORDER BY ALL`](#docs:stable:sql:query_syntax:orderby::order-by-all): shorthand to order on all columns (e.g., to ensure deterministic results).
    -   [`SELECT * EXCLUDE`](#docs:stable:sql:expressions:star::exclude-clause): the `EXCLUDE` option allows excluding specific columns from the `*` expression.
    -   [`SELECT * REPLACE`](#docs:stable:sql:expressions:star::replace-clause): the `REPLACE` option allows replacing specific columns with different expressions in a `*` expression.
    -   [`UNION BY NAME`](#docs:stable:sql:query_syntax:setops::union-all-by-name): perform the `UNION` operation along the names of columns (instead of relying on positions).
    -   [Prefix aliases in the `SELECT` and `FROM` clauses](#docs:stable:sql:query_syntax:select): write `x: 42` instead of `42 AS x` for improved readability.
-   Transforming tables:
    -   [`PIVOT`](#docs:stable:sql:statements:pivot) to turn long tables to wide tables.
    -   [`UNPIVOT`](#docs:stable:sql:statements:unpivot) to turn wide tables to long tables.
-   Defining SQL-level variables:
    -   [`SET VARIABLE`](#docs:stable:sql:statements:set::set-variable)
    -   [`RESET VARIABLE`](#docs:stable:sql:statements:set::reset-variable)

### Query Features 

-   [Column aliases in `WHERE`, `GROUP BY`, and `HAVING`](https://duckdb.org/2022/05/04/friendlier-sql#column-aliases-in-where--group-by--having). (Note that column aliases cannot be used in the `ON` clause of [`JOIN` clauses](#docs:stable:sql:query_syntax:from::joins).)
-   [`COLUMNS()` expression](#docs:stable:sql:expressions:star::columns-expression) can be used to execute the same expression on multiple columns:
    -   [with regular expressions](https://duckdb.org/2023/08/23/even-friendlier-sql#columns-with-regular-expressions)
    -   [with `EXCLUDE` and `REPLACE`](https://duckdb.org/2023/08/23/even-friendlier-sql#columns-with-exclude-and-replace)
    -   [with lambda functions](https://duckdb.org/2023/08/23/even-friendlier-sql#columns-with-lambda-functions)
-   Reusable column aliases (also known as “lateral column aliases”), e.g.: `SELECT i + 1 AS j, j + 2 AS k FROM range(0, 3) t(i)`
-   Advanced aggregation features for analytical (OLAP) queries:
    -   [`FILTER` clause](#docs:stable:sql:query_syntax:filter)
    -   [`GROUPING SETS`, `GROUP BY CUBE`, `GROUP BY ROLLUP` clauses](#docs:stable:sql:query_syntax:grouping_sets)
-   [`count()` shorthand](#docs:stable:sql:functions:aggregates) for `count(*)`

### Literals and Identifiers 

-   [Case-insensitivity while maintaining case of entities in the catalog](#docs:stable:sql:dialect:keywords_and_identifiers::case-sensitivity-of-identifiers)
-   [Deduplicating identifiers](#docs:stable:sql:dialect:keywords_and_identifiers::deduplicating-identifiers)
-   [Underscores as digit separators in numeric literals](#docs:stable:sql:dialect:keywords_and_identifiers::numeric-literals)

### Data Types

-   [`MAP` data type](#docs:stable:sql:data_types:map)
-   [`UNION` data type](#docs:stable:sql:data_types:union)

### Data Import 

-   [Auto-detecting the headers and schema of CSV files](#docs:stable:data:csv:auto_detection)
-   Directly querying [CSV files](#docs:stable:data:csv:overview) and [Parquet files](#docs:stable:data:parquet:overview)
-   Loading from files using the syntax `FROM 'my.csv'`, `FROM 'my.csv.gz'`, `FROM 'my.parquet'`, etc.
-   [Filename expansion (globbing)](#docs:stable:sql:functions:pattern_matching::globbing), e.g.: `FROM 'my-data/part-*.parquet'`

### Functions and Expressions 

-   [Dot operator for function chaining](#docs:stable:sql:functions:overview::function-chaining-via-the-dot-operator): `SELECT ('hello').upper()`
-   String formatters:
    the [`format()` function with the `fmt` syntax](#docs:stable:sql:functions:char::fmt-syntax) and
    the [`printf() function`](#docs:stable:sql:functions:char::printf-syntax)
-   [List comprehensions](https://duckdb.org/2023/08/23/even-friendlier-sql#list-comprehensions)
-   [List slicing](https://duckdb.org/2022/05/04/friendlier-sql#string-slicing)
-   [String slicing](https://duckdb.org/2022/05/04/friendlier-sql#string-slicing)
-   [`STRUCT.*` notation](https://duckdb.org/2022/05/04/friendlier-sql#struct-dot-notation)
-   [Simple `LIST` and `STRUCT` creation](https://duckdb.org/2022/05/04/friendlier-sql#simple-list-and-struct-creation)

### Join Types 

-   [`ASOF` joins](#docs:stable:sql:query_syntax:from::as-of-joins)
-   [`LATERAL` joins](#docs:stable:sql:query_syntax:from::lateral-joins)
-   [`POSITIONAL` joins](#docs:stable:sql:query_syntax:from::positional-joins)

### Trailing Commas 

DuckDB allows [trailing commas](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Trailing_commas),
both when listing entities (e.g., column and table names) and when constructing [`LIST` items](#docs:stable:sql:data_types:list::creating-lists).
For example, the following query works:

``` sql
SELECT
    42 AS x,
    ['a', 'b', 'c',] AS y,
    'hello world' AS z,
;
```

### "Top-N in Group" Queries 

Computing the "top-N rows in a group" ordered by some criteria is a common task in SQL that unfortunately often requires a complex query involving window functions and/or subqueries.

To aid in this, DuckDB provides the aggregate functions [`max(arg, n)`](#docs:stable:sql:functions:aggregates::maxarg-n), [`min(arg, n)`](#docs:stable:sql:functions:aggregates::minarg-n), [`arg_max(arg, val, n)`](#docs:stable:sql:functions:aggregates::arg_maxarg-val-n), [`arg_min(arg, val, n)`](#docs:stable:sql:functions:aggregates::arg_minarg-val-n), [`max_by(arg, val, n)`](#docs:stable:sql:functions:aggregates::max_byarg-val-n) and [`min_by(arg, val, n)`](#docs:stable:sql:functions:aggregates::min_byarg-val-n) to efficiently return the "top" `n` rows in a group based on a specific column in either ascending or descending order.

For example, let's use the following table:

``` sql
SELECT * FROM t1;
```

``` text
┌─────────┬───────┐
│   grp   │  val  │
│ varchar │ int32 │
├─────────┼───────┤
│ a       │     2 │
│ a       │     1 │
│ b       │     5 │
│ b       │     4 │
│ a       │     3 │
│ b       │     6 │
└─────────┴───────┘
```

We want to get a list of the top-3 `val` values in each group `grp`. The conventional way to do this is to use a window function in a subquery:

``` sql
SELECT array_agg(rs.val), rs.grp
FROM
    (SELECT val, grp, row_number() OVER (PARTITION BY grp ORDER BY val DESC) AS rid
    FROM t1 ORDER BY val DESC) AS rs
WHERE rid < 4
GROUP BY rs.grp;
```

``` text
┌───────────────────┬─────────┐
│ array_agg(rs.val) │   grp   │
│      int32[]      │ varchar │
├───────────────────┼─────────┤
│ [3, 2, 1]         │ a       │
│ [6, 5, 4]         │ b       │
└───────────────────┴─────────┘
```

But in DuckDB, we can do this much more concisely (and efficiently!):

``` sql
SELECT max(val, 3) FROM t1 GROUP BY grp;
```

``` text
┌─────────────┐
│ max(val, 3) │
│   int32[]   │
├─────────────┤
│ [3, 2, 1]   │
│ [6, 5, 4]   │
└─────────────┘
```


### Keywords and Identifiers 

### Identifiers 

Similarly to other SQL dialects and programming languages, identifiers in DuckDB's SQL are subject to several rules.

-   Unquoted identifiers need to conform to a number of rules:
    -   They must not be a reserved keyword (see [`duckdb_keywords()`](#docs:stable:sql:meta:duckdb_table_functions::duckdb_keywords)), e.g., `SELECT 123 AS SELECT` will fail.
    -   They must not start with a number or special character, e.g., `SELECT 123 AS 1col` is invalid.
    -   They cannot contain whitespaces (including tabs and newline characters).
-   Identifiers can be quoted using double-quote characters (`"`). Quoted identifiers can use any keyword, whitespace or special character, e.g., `"SELECT"` and `" § 🦆 ¶ "` are valid identifiers.
-   Double quotes can be escaped by repeating the quote character, e.g., to create an identifier named `IDENTIFIER "X"`, use `"IDENTIFIER ""X"""`.

#### Deduplicating Identifiers 

In some cases, duplicate identifiers can occur, e.g., column names may conflict when unnesting a nested data structure.
In these cases, DuckDB automatically deduplicates column names by renaming them according to the following rules:

-   For a column named `⟨name⟩`{:.language-sql .highlight}, the first instance is not renamed.
-   Subsequent instances are renamed to `⟨name⟩_⟨count⟩`{:.language-sql .highlight}, where `⟨count⟩`{:.language-sql .highlight} starts at 1.

For example:

``` sql
SELECT *
FROM (SELECT UNNEST({'a': 42, 'b': {'a': 88, 'b': 99}}, recursive := true));
```

|   a | a_1 |   b |
|----:|----:|----:|
|  42 |  88 |  99 |

### Database Names 

Database names are subject to the rules for [identifiers](#::identifiers).

Additionally, it is best practice to avoid DuckDB's two internal [database schema names](#docs:stable:sql:meta:duckdb_table_functions::duckdb_databases), `system` and `temp`.
By default, persistent databases are named after their filename without the extension.
Therefore, the filenames `system.db` and `temp.db` (as well as `system.duckdb` and `temp.duckdb`) result in the database names `system` and `temp`, respectively.
If you need to attach to a database that has one of these names, use an alias, e.g.:

``` sql
ATTACH 'temp.db' AS temp2;
USE temp2;
```

### Rules for Case-Sensitivity 

#### Keywords and Function Names 

SQL keywords and function names are case-insensitive in DuckDB.

For example, the following two queries are equivalent:

``` matlab
select COS(Pi()) as CosineOfPi;
SELECT cos(pi()) AS CosineOfPi;
```

| CosineOfPi |
|-----------:|
|       -1.0 |

#### Case-Sensitivity of Identifiers 

Identifiers in DuckDB are always case-insensitive, similarly to PostgreSQL.
However, unlike PostgreSQL (and some other major SQL implementations), DuckDB also treats quoted identifiers as case-insensitive.

Despite treating identifiers in a case-insensitive manner, each character's case (uppercase/lowercase) is maintained as originally specified by the user even if a query uses different cases when referring to the identifier.
For example:

``` sql
CREATE TABLE tbl AS SELECT cos(pi()) AS CosineOfPi;
SELECT cosineofpi FROM tbl;
```

| CosineOfPi |
|-----------:|
|       -1.0 |

To change this behavior, set the `preserve_identifier_case` [configuration option](#docs:stable:configuration:overview::configuration-reference) to `false`.

#### Case-Sensitivity of Keys in Nested Data Structures

The keys of `MAP`s are case-sensitive:

``` sql
SELECT MAP(['key1'], [1]) = MAP(['KEY1'], [1]) AS equal;
```

``` text
false
```

The keys of `UNION`s and `STRUCT`s are case-insensitive:

``` sql
SELECT {'key1': 1} = {'KEY1': 1} AS equal;
```

``` text
true
```

``` sql
SELECT union_value(key1 := 1) = union_value(KEY1 := 1) as equal;
```

``` text
true
```

##### Handling Conflicts 

In case of a conflict, when the same identifier is spelt with different cases, one will be selected randomly. For example:

``` sql
CREATE TABLE t1 (idfield INTEGER, x INTEGER);
CREATE TABLE t2 (IdField INTEGER, y INTEGER);
INSERT INTO t1 VALUES (1, 123);
INSERT INTO t2 VALUES (1, 456);
SELECT * FROM t1 NATURAL JOIN t2;
```

| idfield |   x |   y |
|--------:|----:|----:|
|       1 | 123 | 456 |

##### Disabling Preserving Cases 

With the `preserve_identifier_case` [configuration option](#docs:stable:configuration:overview::configuration-reference) set to `false`, all identifiers are turned into lowercase:

``` sql
SET preserve_identifier_case = false;
CREATE TABLE tbl AS SELECT cos(pi()) AS CosineOfPi;
SELECT CosineOfPi FROM tbl;
```

| cosineofpi |
|-----------:|
|       -1.0 |

### Order Preservation 

For many operations, DuckDB preserves the order of rows, similarly to data frame libraries such as Pandas.

### Example

Take the following table for example:

``` sql
CREATE TABLE tbl AS
    SELECT *
    FROM (VALUES (1, 'a'), (2, 'b'), (3, 'c')) t(x, y);

SELECT *
FROM tbl;
```

|   x | y   |
|----:|-----|
|   1 | a   |
|   2 | b   |
|   3 | c   |

Let's take the following query that returns the rows where `x` is an odd number:

``` sql
SELECT *
FROM tbl
WHERE x % 2 == 1;
```

|   x | y   |
|----:|-----|
|   1 | a   |
|   3 | c   |

Because the row `(1, 'a')` occurs before `(3, 'c')` in the original table, it is guaranteed to come before that row in this table too.

### Clauses 

The following clauses guarantee that the original row order is preserved:

-   `COPY` (see [Insertion Order](#::insertion-order))
-   `FROM` with a single table
-   `LIMIT`
-   `OFFSET`
-   `SELECT`
-   `UNION ALL`
-   `WHERE`
-   Window functions with an empty `OVER` clause
-   Common table expressions and table subqueries as long as they only contains the aforementioned components

> **Tip.** `row_number() OVER ()` allows turning the original row order into an explicit column that can be referenced in the operations that don't preserve row order by default. On materialized tables, the `rowid` pseudo-column can be used to the same effect.

The following operations **do not** guarantee that the row order is preserved:

-   `FROM` with multiple tables and/or subqueries
-   `JOIN`
-   `UNION`
-   `USING SAMPLE`
-   `GROUP BY` (in particular, the output order is undefined and the order in which rows are fed into [order-sensitive aggregate functions](https://duckdb.org/docs/sql/functions/aggregates.html#order-by-clause-in-aggregate-functions) is undefined unless explicitly specified in the aggregate function)
-   `ORDER BY` (specifically, `ORDER BY` may not use a [stable algorithm](https://en.m.wikipedia.org/wiki/Stable_algorithm))
-   Scalar subqueries

### Insertion Order 

By default, the following components preserve insertion order:

-   [CSV reader](#docs:stable:data:csv:overview::order-preservation) (`read_csv` function)
-   [JSON reader](#docs:stable:data:json:overview::order-preservation) (`read_json` function)
-   [Parquet reader](#docs:stable:data:parquet:overview::order-preservation) (`read_parquet` function)

Preservation of insertion order is controlled by the `preserve_insertion_order` [configuration option](#docs:stable:configuration:overview).
This setting is `true` by default, indicating that the order should be preserved.
To change this setting, use:

``` sql
SET preserve_insertion_order = false;
```

### PostgreSQL Compatibility 

DuckDB's SQL dialect closely follows the conventions of the PostgreSQL dialect.
The few exceptions to this are listed on this page.

### Floating-Point Arithmetic 

DuckDB and PostgreSQL handle floating-point arithmetic differently for division by zero. DuckDB conforms to the [IEEE Standard for Floating-Point Arithmetic (IEEE 754)](https://en.wikipedia.org/wiki/IEEE_754) for both division by zero and operations involving infinity values. PostgreSQL returns an error for division by zero but aligns with IEEE 754 for handling infinity values. To show the differences, run the following SQL queries:

``` sql
SELECT 1.0 / 0.0 AS x;
SELECT 0.0 / 0.0 AS x;
SELECT -1.0 / 0.0 AS x;
SELECT 'Infinity'::FLOAT / 'Infinity'::FLOAT AS x;
SELECT 1.0 / 'Infinity'::FLOAT AS x;
SELECT 'Infinity'::FLOAT - 'Infinity'::FLOAT AS x;
SELECT 'Infinity'::FLOAT - 1.0 AS x;
```

| Expression              | PostgreSQL |    DuckDB |  IEEE 754 |
|:------------------------|-----------:|----------:|----------:|
| 1.0 / 0.0               |      error |  Infinity |  Infinity |
| 0.0 / 0.0               |      error |       NaN |       NaN |
| -1.0 / 0.0              |      error | -Infinity | -Infinity |
| 'Infinity' / 'Infinity' |        NaN |       NaN |       NaN |
| 1.0 / 'Infinity'        |        0.0 |       0.0 |       0.0 |
| 'Infinity' - 'Infinity' |        NaN |       NaN |       NaN |
| 'Infinity' - 1.0        |   Infinity |  Infinity |  Infinity |

### Division on Integers 

When computing division on integers, PostgreSQL performs integer division, while DuckDB performs float division:

``` sql
SELECT 1 / 2 AS x;
```

PostgreSQL returns `0`, while DuckDB returns `0.5`.

To perform integer division in DuckDB, use the `//` operator:

``` sql
SELECT 1 // 2 AS x;
```

This returns `0`.

### `UNION` of Boolean and Integer Values 

The following query fails in PostgreSQL but successfully completes in DuckDB:

``` sql
SELECT true AS x
UNION
SELECT 2;
```

PostgreSQL returns an error:

``` console
ERROR:  UNION types boolean and integer cannot be matched
```

DuckDB performs an enforced cast, therefore, it completes the query and returns the following:

|   x |
|----:|
|   1 |
|   2 |

### Implicit Casting on Equality Checks 

DuckDB performs implicit casting on equality checks, e.g., converting strings to numeric and boolean values.
Therefore, there are several instances, where PostgreSQL throws an error while DuckDB successfully computes the result:

| Expression    | PostgreSQL | DuckDB |
|:--------------|------------|--------|
| '1.1' = 1     | error      | true   |
| '1.1' = 1.1   | true       | true   |
| 1 = 1.1       | false      | false  |
| true = 'true' | true       | true   |
| true = 1      | error      | true   |
| 'true' = 1    | error      | error  |

### Case Sensitivity for Quoted Identifiers 

PostgreSQL is case-insensitive. The way PostgreSQL achieves case insensitivity is by lowercasing unquoted identifiers within SQL, whereas quoting preserves case, e.g., the following command creates a table named `mytable` but tries to query for `MyTaBLe` because quotes preserve the case.

``` sql
CREATE TABLE MyTaBLe (x INTEGER);
SELECT * FROM "MyTaBLe";
```

``` console
ERROR:  relation "MyTaBLe" does not exist
```

PostgreSQL does not only treat quoted identifiers as case-sensitive, PostgreSQL treats all identifiers as case-sensitive, e.g., this also does not work:

``` sql
CREATE TABLE "PreservedCase" (x INTEGER);
SELECT * FROM PreservedCase;
```

``` console
ERROR:  relation "preservedcase" does not exist
```

Therefore, case-insensitivity in PostgreSQL only works if you never use quoted identifiers with different cases.

For DuckDB, this behavior was problematic when interfacing with other tools (e.g., Parquet, Pandas) that are case-sensitive by default – since all identifiers would be lowercased all the time.
Therefore, DuckDB achieves case insensitivity by making identifiers fully case insensitive throughout the system but [*preserving their case*](#docs:stable:sql:dialect:keywords_and_identifiers::rules-for-case-sensitivity).

In DuckDB, the scripts above complete successfully:

``` sql
CREATE TABLE MyTaBLe (x INTEGER);
SELECT * FROM "MyTaBLe";
CREATE TABLE "PreservedCase" (x INTEGER);
SELECT * FROM PreservedCase;
SELECT table_name FROM duckdb_tables();
```

| table_name    |
|---------------|
| MyTaBLe       |
| PreservedCase |

PostgreSQL's behavior of lowercasing identifiers is accessible using the [`preserve_identifier_case` option](#docs:stable:configuration:overview::local-configuration-options):

``` sql
SET preserve_identifier_case = false;
CREATE TABLE MyTaBLe (x INTEGER);
SELECT table_name FROM duckdb_tables();
```

| table_name |
|------------|
| mytable    |

However, the case insensitive matching in the system for identifiers cannot be turned off.

### Using Double Equality Sign for Comparison 

DuckDB supports both `=` and `==` for quality comparison, while PostgreSQL only supports `=`.

``` sql
SELECT 1 == 1 AS t;
```

DuckDB returns `true`, while PostgreSQL returns:

``` console
postgres=# SELECT 1 == 1 AS t;
ERROR:  operator does not exist: integer == integer
LINE 1: SELECT 1 == 1 AS t;
```

Note that the use of `==` is not encouraged due to its limited portability.

### Vacuuming tables 

In PostgreSQL, the `VACUUM` statement garbage collects tables and analyzes tables.
In DuckDB, the [`VACUUM` statement](#docs:stable:sql:statements:vacuum) is only used to rebuild statistics.
For instruction on reclaiming space, refer to the [“Reclaiming space” page](#docs:stable:operations_manual:footprint_of_duckdb:reclaiming_space).

### Functions

#### `regexp_extract` Function 

Unlike PostgreSQL's `regexp_substr` function, DuckDB's `regexp_extract` returns empty strings instead of `NULL`s when there is no match.

#### `to_date` Function 

DuckDB does not support the [`to_date` PostgreSQL date formatting function](https://www.postgresql.org/docs/17/functions-formatting.html).
Instead, please use the [`strptime` function](#docs:stable:sql:functions:dateformat::strptime-examples).

### Resolution of Type Names in the Schema 

For [`CREATE TABLE` statements](#docs:stable:sql:statements:create_table), DuckDB attempts to resolve type names in the schema where a table is created. For example:

``` sql
CREATE SCHEMA myschema;
CREATE TYPE myschema.mytype AS ENUM ('as', 'df');
CREATE TABLE myschema.mytable (v mytype);
```

PostgreSQL returns an error on the last statement:

``` console
ERROR:  type "mytype" does not exist
LINE 1: CREATE TABLE myschema.mytable (v mytype);
```

DuckDB runs the statement and creates the table successfully, confirmed by the following query:

``` sql
DESCRIBE myschema.mytable;
```

| column_name | column_type      | null | key  | default | extra |
|-------------|------------------|------|------|---------|-------|
| v           | ENUM('as', 'df') | YES  | NULL | NULL    | NULL  |

### Exploiting Functional Dependencies for `GROUP BY` 

PostgreSQL can exploit functional dependencies, such as `i -> j` in the following query:

``` sql
CREATE TABLE tbl (i INTEGER, j INTEGER, PRIMARY KEY (i));
SELECT j
FROM tbl
GROUP BY i;
```

PostgreSQL runs the query.

DuckDB fails:

``` console
Binder Error:
column "j" must appear in the GROUP BY clause or must be part of an aggregate function.
Either add it to the GROUP BY list, or use "ANY_VALUE(j)" if the exact value of "j" is not important.
```

To work around this, add the other attributes or use the [`GROUP BY ALL` clause](https://duckdb.org/docs/sql/query_syntax/groupby#group-by-all).

### Behavior of Regular Expression Match Operators 

PostgreSQL supports the [POSIX regular expression matching operators](#docs:stable:sql:functions:pattern_matching) `~` (case-sensitive partial regex matching) and `~*` (case-insensitive partial regex matching) as well as their negated variants, `!~` and `!~*`, respectively.

In DuckDB, `~` is equivalent to [`regexp_full_match`](#docs:stable:sql:functions:char::regexp_full_matchstring-regex) and `!~` is equivalent to `NOT regexp_full_match`.
The operators `~*` and `!~*` are not supported.

The table below shows that the correspondence between these functions in PostgreSQL and DuckDB is almost non-existent.
We recommend avoiding the POSIX regular expression matching operators in DuckDB.

| Expression          | PostgreSQL | DuckDB |
|:--------------------|------------|--------|
| `'aaa' ~ '(a|b)'`   | true       | false  |
| `'AAA' ~* '(a|b)'`  | true       | error  |
| `'aaa' !~ '(a|b)'`  | false      | true   |
| `'AAA' !~* '(a|b)'` | false      | error  |

### SQL Quirks 

Like all programming languages and libraries, DuckDB has its share of idiosyncrasies and inconsistencies.
Some are vestiges of our feathered friend's evolution; others are inevitable because we strive to adhere to the [SQL Standard](https://blog.ansi.org/sql-standard-iso-iec-9075-2023-ansi-x3-135/) and specifically to PostgreSQL's dialect (see the [“PostgreSQL Compatibility”](#docs:stable:sql:dialect:postgresql_compatibility) page for exceptions).
The rest may simply come down to different preferences, or we may even agree on what *should* be done but just haven’t gotten around to it yet.

Acknowledging these quirks is the best we can do, which is why we have compiled below a list of examples.

### Aggregating Empty Groups 

On empty groups, the aggregate functions `sum`, `list`, and `string_agg` all return `NULL` instead of `0`, `[]` and `''`, respectively. This is dictated by the SQL Standard and obeyed by all SQL implementations we know. This behavior is inherited by the list aggregate [`list_sum`](#docs:stable:sql:functions:list::list_-rewrite-functions), but not by the DuckDB original [`list_dot_product`](#docs:stable:sql:functions:list::list_dot_productlist1-list2) which returns `0` on empty lists.

### Indexing 

To comply with standard SQL, one-based indexing is used almost everywhere, e.g., array and string indexing and slicing, and window functions (`row_number`, `rank`, `dense_rank`). However, similarly to PostgreSQL, [JSON features use a zero-based indexing](#docs:stable:data:json:overview::indexing).

### Expressions 

#### Results That May Surprise You 

| Expression | Result | Note |
|------------------|-------|------------------------------------------------|
| `-2^2` | `4.0` | PostgreSQL compatibility means the unary minus has higher precedence than the exponentiation operator. Use additional parentheses, e.g., `-(2^2)` or the [`pow` function](#docs:stable:sql:functions:numeric::powx-y), e.g. `-pow(2, 2)`, to avoid mistakes. |
| `'t' = true` | `true` | Compatible with PostgreSQL. |
| `1 = '1'` | `true` | Compatible with PostgreSQL. |
| `1 = ' 1'` | `true` | Compatible with PostgreSQL. |
| `1 = '01'` | `true` | Compatible with PostgreSQL. |
| `1 = ' 01 '` | `true` | Compatible with PostgreSQL. |
| `1 = true` | `true` | Not compatible with PostgreSQL. |
| `1 = '1.1'` | `true` | Not compatible with PostgreSQL. |
| `1 IN (0, NULL)` | `NULL` | Makes sense if you think of the `NULL`s in the input and output as `UNKNOWN`. |
| `1 in [0, NULL]` | `false` |  |
| `concat('abc', NULL)` | `abc` | Compatible with PostgreSQL. `list_concat` behaves similarly. |
| `'abc' || NULL` | `NULL` |  |

#### `NaN` Values 

`'NaN'::FLOAT = 'NaN'::FLOAT` and `'NaN'::FLOAT > 3` violate IEEE-754 but mean floating point data types have a total order, like all other data types (beware the consequences for `greatest` / `least`).

#### `age` Function 

`age(x)` is `current_date - x` instead of `current_timestamp - x`. Another quirk inherited from PostgreSQL.

#### Extract Functions 

`list_extract` / `map_extract` return `NULL` on non-existing keys. `struct_extract` throws an error because keys of structs are like columns.

### Clauses 

#### Automatic Column Deduplication in `SELECT` 

Column names are deduplicated with the first occurrence shadowing the others:

``` sql
CREATE TABLE tbl AS SELECT 1 AS a;
SELECT a FROM (SELECT *, 2 AS a FROM tbl);
```

|   a |
|----:|
|   1 |

#### Case Insensitivity for `SELECT`ing Columns 

Due to case-insensitivity, it's not possible to use `SELECT a FROM 'file.parquet'` when a column called `A` appears before the desired column `a` in `file.parquet`.

#### `USING SAMPLE`

The `USING SAMPLE` clause is syntactically placed after the `WHERE` and `GROUP BY` clauses (same as the `LIMIT` clause) but is semantically applied before both (unlike the `LIMIT` clause).
