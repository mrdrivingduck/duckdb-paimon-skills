# duckdb-paimon-analyze — AI Agent Workflow Guide

> This document instructs AI agents on how to set up and use DuckDB + duckdb-paimon to analyze Apache Paimon data lake tables. Follow the phases in order; skip completed phases on subsequent activations within the same conversation.

## Before Use: Check Skill Updates

When this skill is activated **for the first time in a conversation**, check for newer commits on the remote:

```bash
git -C <skill_directory> fetch origin main
git -C <skill_directory> log --oneline HEAD..origin/main
```

If newer commits exist, summarize them and ask the user whether to update. If the check fails (network, credentials), continue with the local version.

## Phase 1: Environment Check

Follow `references/setup-guide.md` to detect the platform, find the latest duckdb-paimon release, ensure the matching DuckDB version is installed, and locate any existing extension installation.

Outcomes:
- **Extension already installed** — Skip to Phase 3.
- **Extension not installed** — Proceed to Phase 2.

## Phase 2: Extension Download

Follow `references/setup-guide.md` Steps 3–4 to download and install the extension. Save the absolute path to `paimon.duckdb_extension` for Phase 3.

## Phase 3: Extension Loading

Start DuckDB with unsigned extension support and load the binary:

```bash
duckdb -unsigned
```

```sql
LOAD '/absolute/path/to/paimon.duckdb_extension';
```

## Phase 4: Catalog Attachment

Ask the user for their Paimon warehouse path, then attach it:

```sql
ATTACH '/path/to/warehouse' AS paimon_cat (TYPE paimon, READ_ONLY);
```

Use `READ_ONLY` by default to prevent accidental writes. Drop `READ_ONLY` only when the user explicitly intends to write data.

If the warehouse path starts with `oss://`, the user also needs to provide OSS credentials (AccessKey ID, AccessKey Secret, Endpoint). Ask the user to put them in a file (to keep credentials out of the conversation) and provide the file path. See `references/oss-access.md` for credential file format and attachment syntax.

### Verify

```sql
SHOW ALL TABLES;
```

This should list the databases and tables in the warehouse. If empty, verify the warehouse path is correct and contains Paimon metadata (snapshot/manifest directories).

## Phase 5: Schema Exploration

Help the user understand what data is available:

```sql
SHOW ALL TABLES;
DESCRIBE paimon_cat.db_name.table_name;
SELECT * FROM paimon_cat.db_name.table_name LIMIT 5;
```

Present the schema information clearly before generating analysis queries. Understanding column names, types, and sample values is essential for producing correct SQL.

## Phase 6: Query & Analysis

Generate SQL queries based on the user's analysis requirements. See `references/sql-operations.md` for the complete syntax reference including time travel, snapshot inspection, write operations, and cross-format joins.

### SQL Visibility Protocol

For every SQL statement that is generated and executed on the user's behalf:

1. Show the exact SQL to the user in a fenced `sql` code block before executing it.
2. Execute only SQL that has already been shown, except for trivial session setup commands already documented in earlier phases.
3. Enable DuckDB CLI timing with `.timer on` before running analysis SQL when the CLI supports it.
4. When presenting results, include an "Executed SQL" section that lists the statements used to produce those results and their DuckDB-reported execution time when available. If several exploratory statements were run, include all of them in execution order.
5. If a statement must be changed after an error, show the revised SQL before running it.

Do not summarize results from hidden ad hoc SQL. The user must be able to see which SQL produced each answer and whether DuckDB reported how long each statement took to run.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Catalog Error: ... not allowed for unsigned extensions` | DuckDB not started with `-unsigned` | Exit the session, restart with `duckdb -unsigned`, then load the extension again |
| `Extension ... version mismatch` | DuckDB version != extension build version | Re-run Phase 1 to verify version alignment |
| `Failed to load ... libpaimon.dylib` | Companion shared libraries missing | Re-extract the release tarball; don't move `paimon.duckdb_extension` out of its directory |
| `SHOW ALL TABLES` returns empty | Wrong warehouse path, or not a Paimon warehouse | Verify path contains `snapshot/` and `manifest/` subdirectories |
| `OSS access denied` | Wrong credentials or permissions | See `references/oss-access.md` troubleshooting section |
| `Table scan returns 0 rows` | Querying wrong snapshot or empty table | Use `paimon_snapshots()` to verify table has data |
