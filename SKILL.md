---
name: duckdb-paimon-analyze
description: Analyze Apache Paimon data lake tables using DuckDB and the duckdb-paimon extension. Covers installation verification, extension download, loading, warehouse attachment, schema browsing, SQL query generation, time travel, snapshots, OSS access, paimon_scan, and cross-format joins.
metadata:
  short-description: Analyze Paimon data lake tables with DuckDB
---

# duckdb-paimon-analyze

Use this skill when the user mentions Paimon, data lake, duckdb-paimon, paimon_scan, paimon_snapshots, warehouse path, or asks to query, browse, or analyze tables stored in an Apache Paimon warehouse using DuckDB.

## Supported Platforms

`osx-arm64`, `linux_amd64`, `linux_arm64`

Version matching is dynamic — the setup guide queries GitHub Releases API to find the correct extension build for the local DuckDB version and platform.

## Resources

Read `AGENTS.md` for the agent workflow covering environment check, extension download, loading, catalog attachment, schema exploration, and query generation.

When running analysis queries, follow the SQL visibility protocol in `AGENTS.md`: show each generated SQL statement before execution and include the executed SQL with the results.

Reference docs (load on demand):

- `references/setup-guide.md` -- Platform detection, DuckDB installation, extension download, and extraction steps.
- `references/sql-operations.md` -- SQL reference: paimon_snapshots, time travel, DDL/DML, cross-format joins.
- `references/oss-access.md` -- Alibaba Cloud OSS credential setup and remote warehouse attachment.
