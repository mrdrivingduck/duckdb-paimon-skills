# duckdb-paimon SQL Operations Reference

## Read Operations

```sql
SELECT * FROM my_catalog.db_name.table_name WHERE col1 > 100;
```

## Cross-Format Joins

```sql
SELECT o.order_id, o.amount, c.customer_name
FROM my_catalog.db.orders o
JOIN read_csv('customers.csv') c ON o.customer_id = c.id;

SELECT * FROM my_catalog.db.events e
JOIN read_parquet('dim_users.parquet') u ON e.user_id = u.id;
```

## Snapshot Inspection

```sql
SELECT snapshot_id, commit_kind, commit_time, total_record_count
FROM paimon_snapshots('/path/to/warehouse/db_name.db/table_name')
ORDER BY snapshot_id;

-- Also works with three-argument form and OSS paths
SELECT * FROM paimon_snapshots('/path/to/warehouse', 'db_name', 'table_name');
```

## Time Travel

```sql
-- By snapshot version
SELECT * FROM my_catalog.db_name.table_name AT (VERSION => 2);

-- By timestamp
SELECT * FROM my_catalog.db_name.table_name
    AT (TIMESTAMP => TIMESTAMP '2026-01-15 10:48:23.5');
```

## Write Operations

Write support is limited to **append-only tables** only.

### DDL

```sql
CREATE SCHEMA my_catalog.new_db;

CREATE TABLE my_catalog.new_db.orders AS
    SELECT 1 AS order_id, 99.9::DECIMAL(18,2) AS amount, 'Alice' AS customer;

DROP TABLE my_catalog.new_db.orders;
DROP SCHEMA my_catalog.new_db;
```

### DML

```sql
INSERT INTO my_catalog.new_db.orders
    SELECT 2, 49.5, 'Bob'
    UNION ALL
    SELECT 3, 150.0, 'Charlie';
```
