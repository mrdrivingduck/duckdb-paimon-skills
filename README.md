# duckdb-paimon-analyze

Make duckdb-paimon agentic. (🤖 + 🦆)

A reusable skill for analyzing Apache Paimon warehouses with DuckDB and the duckdb-paimon extension.

## What It Does

- Checks the local DuckDB and duckdb-paimon environment.
- Loads the matching duckdb-paimon extension.
- Attaches local or OSS-backed Paimon warehouses.
- Explores tables, schemas, snapshots, and sample rows.
- Helps produce analysis SQL with read-only access by default.

## When To Use It

Use this skill when an agent needs to inspect, query, or analyze data stored in an Apache Paimon warehouse through DuckDB.

It is intended for workflows such as schema discovery, snapshot inspection, time travel queries, warehouse troubleshooting, and joining Paimon tables with other formats supported by DuckDB.

## Documentation

See the repository documentation files for setup, workflow, SQL operations, and OSS access details.

## Safety

The workflow attaches warehouses as read-only by default. Write access should only be enabled when the user explicitly intends to modify data.

Do not paste OSS credentials into conversation history. Store credentials in a local file and provide only the file path to the agent.
