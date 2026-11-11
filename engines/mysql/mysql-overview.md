# MySQL overview

## Summary

MySQL is the most widely adopted open source relational database, created in 1995, owned by Oracle since 2010, and the "M" of the LAMP stack that built much of the web. It is used for OLTP in web application servers, cloud applications and enterprise systems. Its default storage engine, InnoDB, provides ACID transactions, row-level locking, MVCC and a clustered primary key index. Replication through the binary log (binlog) is simple and widely used, and Group Replication and InnoDB Cluster add consensus-based high availability. MariaDB is a community fork of MySQL created in 2009 by MySQL's original author, now a separate product with its own features. Both are GPLv2 licensed.

## Prerequisites

- [Relational model](../../database-design-and-models/relational/relational-model.md) and [transactions and ACID](../../transactions-and-concurrency/transactions-and-acid.md).

## Core concepts

### Profile

| Property | MySQL |
| --- | --- |
| Model | Relational |
| License (checked 2026-11) | GPLv2 (Community Edition); commercial Enterprise Edition from Oracle |
| Storage engine | InnoDB (default): ACID, row locks, MVCC, clustered index on the primary key |
| Default isolation | `REPEATABLE READ` |
| Replication | Asynchronous binlog replication (row-based by default), semi-synchronous, Group Replication (InnoDB Cluster) |
| Release model | LTS releases (8.4 LTS in 2024) and innovation releases (9.x) |
| Version used here | 8.4.13 (Docker image `mysql:8.4`) |

### InnoDB essentials

- **Clustered index.** Rows are stored in the leaf pages of the primary key B-tree. Secondary indexes store the primary key, so lookups through a secondary index do a second lookup in the primary key tree. A short, monotonically increasing primary key keeps inserts cheap; a random UUID primary key fragments pages.
- **Redo and undo logs.** The redo log provides durability (`innodb_flush_log_at_trx_commit = 1` flushes it at every commit); undo logs hold old versions for MVCC and rollback.
- **Locking.** Row locks plus *gap* and *next-key* locks under `REPEATABLE READ` prevent phantoms for locking reads, at the cost of more lock conflicts and deadlocks (see [deadlocks](../../transactions-and-concurrency/deadlocks.md)).

The diagram shows a lookup through a secondary index.

```mermaid
flowchart LR
  q["WHERE customer_id = 1"] --> sec["Secondary index idx_customer<br/>(customer_id -> order_id)"]
  sec -->|"order_id 1, 2"| pk["Clustered index on order_id<br/>(leaf pages hold full rows)"]
  pk --> rows[Rows returned]
```

*Analogy.* InnoDB stores a table like a phone book printed in primary key order: the full entries are the pages themselves. A secondary index is a separate card index that says "for customer 1, see entries 1 and 2"; you then open the phone book at those entries. The analogy breaks for covering indexes: if the card already holds all the data you need, you never open the phone book.

### MariaDB

MariaDB is a fork of MySQL started in 2009 by Michael "Monty" Widenius, the original author of MySQL, after Oracle announced the acquisition of Sun Microsystems (which owned MySQL). It is a relational database for transaction processing workloads, GPLv2 licensed, and was a drop-in replacement for MySQL 5.x. Since then the two have diverged: MariaDB has its own optimizer features, Galera-based clustering, system-versioned (temporal) tables, the ColumnStore engine and different JSON and replication (GTID) implementations, so neither is a drop-in replacement for current versions of the other. Checked against mariadb.org and dev.mysql.com, 2026-11.

### Tooling

| Tool | Purpose |
| --- | --- |
| `mysql` client, MySQL Shell | Command line clients; MySQL Shell also manages InnoDB Cluster. |
| MySQL Workbench | Graphical design, administration and query tool. |
| `EXPLAIN`, `EXPLAIN ANALYZE` | Execution plans (8.0.18 and later for `ANALYZE`); `FORMAT=TREE` shows the iterator tree. |
| `pt-query-digest` (Percona Toolkit) | Aggregates the slow query log into the most expensive query patterns. |
| Performance Schema, sys schema | Built-in instrumentation for statements, waits and locks. |
| Slow query log | Logs statements above `long_query_time`. |

See [SQL tuning](../../query-processing-and-optimization/sql-tuning.md) for the general method.

## Worked example

Tested on MySQL 8.4.13. Default settings, an index lookup plan, and MySQL's upsert syntax.

```sql
CREATE DATABASE shop; USE shop;
CREATE TABLE orders (
  order_id    INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  total       DECIMAL(10, 2) NOT NULL,
  KEY idx_customer (customer_id)
) ENGINE=InnoDB;
INSERT INTO orders (customer_id, total) VALUES (1, 59.90), (1, 12.50), (2, 99.00);
SELECT @@transaction_isolation, @@innodb_flush_log_at_trx_commit, @@binlog_format;
EXPLAIN FORMAT=TREE SELECT * FROM orders WHERE customer_id = 1;
INSERT INTO orders (order_id, customer_id, total) VALUES (1, 3, 10.00)
  ON DUPLICATE KEY UPDATE total = VALUES(total);
SELECT * FROM orders ORDER BY order_id;
```

Output:

```text
@@transaction_isolation  @@innodb_flush_log_at_trx_commit  @@binlog_format
REPEATABLE-READ          1                                 ROW

-> Index lookup on orders using idx_customer (customer_id=1)  (cost=0.7 rows=2)

order_id  customer_id  total
1         1            10.00
2         1            12.50
3         2            99.00
```

The defaults confirm `REPEATABLE READ`, a redo log flush at every commit (full durability) and row-based binlog replication. The upsert updated only `total` of order 1 and left `customer_id` unchanged, because `ON DUPLICATE KEY UPDATE` lists the columns to change. (`VALUES()` in that clause is deprecated in 8.x in favor of a row alias, `AS new ... total = new.total`.)

## Trade offs and when to use it

- **Use** for web and SaaS OLTP, especially with existing MySQL expertise, tooling (Vitess for sharding, ProxySQL, Percona tools) or managed offerings (Amazon RDS and Aurora MySQL, Cloud SQL, Azure Database for MySQL).
- **Weaknesses** compared with PostgreSQL: fewer advanced SQL and indexing features, fewer extension points, weaker standard conformance in places.
- **Alternatives.** PostgreSQL; MariaDB where its features (Galera, temporal tables) fit; TiDB or PlanetScale (Vitess) for MySQL-compatible horizontal scale.

## Common mistakes

- **Random UUID primary keys in InnoDB.** Page splits and large secondary indexes. Fix: auto-increment or time-ordered IDs (UUIDv7).
- **Assuming MySQL and MariaDB are interchangeable.** Fix: test on the exact product and version.
- **`utf8` instead of `utf8mb4`.** MySQL's old `utf8` stores at most three bytes per character and cannot store emoji. Fix: `utf8mb4` (the default since 8.0).
- **Large transactions on replicas with single-threaded apply.** Lag spikes. Fix: parallel replication and smaller transactions.

## Practice

1. Why do InnoDB secondary index lookups often need a second B-tree lookup?
2. What is MariaDB, and why is it no longer a drop-in replacement?
3. Which isolation level is MySQL's default, and what extra locks does it use?

Hints:

1. Secondary index leaves store the primary key, and the row lives in the clustered primary key index.
2. A 2009 fork of MySQL by its original author; the products diverged in features and internals.
3. `REPEATABLE READ`, with gap and next-key locks for locking reads.

## Further reading

- Oracle, "MySQL 8.4 Reference Manual". https://dev.mysql.com/doc/refman/8.4/en/ - The reference.
- Oracle, "The InnoDB Storage Engine", MySQL 8.4 Reference Manual, chapter 17. https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html - Clustered indexes, locking, logs.
- MariaDB Foundation, "MariaDB Server". https://mariadb.org/ - The fork's documentation and history.
- Percona, "pt-query-digest", Percona Toolkit documentation. https://docs.percona.com/percona-toolkit/pt-query-digest.html - Slow log analysis.
