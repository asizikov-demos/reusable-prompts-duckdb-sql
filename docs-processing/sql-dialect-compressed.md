# DuckDB SQL Dialect - Key Features

## Overview
- Based on PostgreSQL with some differences
- 1-based indexing except for JSON objects (0-based)

## Friendly SQL Features

### Key Clauses
- `CREATE OR REPLACE TABLE`: Avoids `DROP TABLE IF EXISTS`
- `CREATE TABLE ... AS SELECT` (CTAS): Create tables from query results
- `INSERT INTO ... BY NAME`: Use column names instead of positions
- `INSERT OR IGNORE INTO ...`: Skip rows causing unique/primary key conflicts
- `INSERT OR REPLACE INTO ...`: Replace rows causing conflicts
- `DESCRIBE`: Summarize schema
- `SUMMARIZE`: Return summary statistics
- `FROM`-first syntax with optional `SELECT`: `FROM tbl` selects all columns
- `GROUP BY ALL`: Infer group-by columns from SELECT clause
- `ORDER BY ALL`: Order by all columns
- `SELECT * EXCLUDE`: Exclude specific columns from `*`
- `SELECT * REPLACE`: Replace specific columns in `*`
- `UNION BY NAME`: Union by column names instead of positions
- Prefix aliases in SELECT/FROM: `x: 42` instead of `42 AS x`
- `PIVOT`/`UNPIVOT`: Convert between wide and long tables
- `SET VARIABLE`/`RESET VARIABLE`: SQL variables

### Query Features
- Column aliases in WHERE, GROUP BY, and HAVING
- `COLUMNS()` expression for multiple columns
- Reusable column aliases: `SELECT i + 1 AS j, j + 2 AS k FROM range(0, 3) t(i)`
- Advanced aggregation: `FILTER` clause, `GROUPING SETS`, `CUBE`, `ROLLUP`
- `count()` shorthand for `count(*)`
- Column aliases can't be used in JOIN's ON clause

### Data Types & Handling
- Case-insensitive identifiers (maintains case but matches case-insensitively)
- Underscores as digit separators in numeric literals
- `MAP` and `UNION` data types
- Auto-detecting CSV headers and schema
- Direct querying of CSV/Parquet files
- `FROM 'my.csv'`, `FROM 'my.parquet'` syntax
- Filename globbing: `FROM 'my-data/part-*.parquet'`

### Functions and Expressions
- Dot operator for chaining: `('hello').upper()`
- String formatters: `format()` with fmt syntax, `printf()`
- List comprehensions and slicing
- String slicing
- `STRUCT.*` notation
- Simple LIST and STRUCT creation
- Trailing commas allowed in lists and entity listings

### Join Types
- `ASOF` joins
- `LATERAL` joins
- `POSITIONAL` joins

### Top-N in Group
- Efficient functions: `max(arg, n)`, `min(arg, n)`, `arg_max(arg, val, n)`, etc.
- Returns "top" n rows in a group based on specific columns

## PostgreSQL Differences

### Behavior Differences
- IEEE-754 floating-point division by zero returns Infinity/NaN (PostgreSQL errors)
- Integer division: `1 / 2` returns 0.5 (use `//` for integer division)
- `UNION` of boolean and integer succeeds (PostgreSQL errors)
- Implicit casting on equality checks: `'1.1' = 1` is true
- Both `=` and `==` supported for equality
- `VACUUM` only rebuilds statistics (not garbage collection)
- `regexp_extract` returns empty strings for no match (not NULLs)
- Use `strptime` instead of PostgreSQL's `to_date`
- Type names in schema resolution are different
- Regular expression operators: `~` is full match (not partial match)

## Quirks
- Aggregate functions on empty groups: `sum`, `list`, `string_agg` return NULL
- `-2^2` equals 4.0 (unary minus has higher precedence)
- NaN comparisons violate IEEE-754 for total ordering
- `age(x)` is `current_date - x` not `current_timestamp - x`
- List/map_extract return NULL on missing keys, struct_extract errors
- Column deduplication in SELECT keeps first occurrence
- Case insensitivity affects column selection from Parquet files
- `USING SAMPLE` placed after WHERE/GROUP BY but applied before them
