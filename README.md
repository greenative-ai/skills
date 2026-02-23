# Greenative Skills

Claude Code skills for the [Greenative Platform](https://doc.avgidea.io) (a.k.a Avgidea Data Platform). These skills let you manage datasources, transfer data, run functions, persist state, and handle storage objects — all from Claude Code or Claude Desktop.

## Skills Overview

| Skill | Command | Description |
|-------|---------|-------------|
| Datasource | `/datasource` | List datasources, verify connections, and execute SQL |
| Entity | `/entity` | Transfer data between storages and databases |
| Function | `/function` | Execute platform functions and monitor their status |
| KVS | `/kvs` | Read and write entries in the native key-value store |
| Storage | `/storage` | Upload, download, list, and delete storage objects |

## Skill Details

### /datasource

Manage [datasources](https://doc.avgidea.io/data-exchange) (cloud storage, databases, etc.) and execute SQL statements against database datasources.

**Capabilities:**
- **list** — Discover available datasources, optionally filtered by name or type
- **connect** — Verify that a datasource is reachable
- **execute** — Run DDL/DML statements (`statement` field) or SELECT queries (`query` field), with optional bind variables (`:1`, `:2` style) and session parameters

**Key details:**
- SELECT uses the `query` parameter; INSERT/UPDATE/DELETE/DDL uses `statement` — they are different fields
- Bind variable placeholders are `:1`, `:2`, etc. (not `?`)
- Session parameters (e.g., `NLS_DATE_FORMAT`) can be set per request

### /entity

Transfer data between storages, load CSV files into databases, and export database tables to CSV.

**Capabilities:**
- **transfer** (single file) — Copy one object between storage datasources
- **transfer** (bulk) — Copy multiple objects matching a `filter` pattern
- **transfer** (CSV → DB) — Load a CSV from storage into a database table with parallel workers
- **transfer** (DB → CSV) — Export a database table to CSV in storage

**Key details:**
- All operations use the same `transfer` action — the parameter combination determines behavior
- CSV-to-DB loading supports a `worker` parameter for parallelism
- Use `/datasource` first to discover and verify available datasources

### /function

Execute registered functions on the platform, monitor their execution, and retrieve logs.

**Capabilities:**
- **request** — Execute a function (standalone, file-based, or line-based)
- **list** — List executions, optionally filtered by status (`starting`, `running`, `success`, `failed`, `timeout`, `terminated`)
- **terminate** — Stop a running execution by its ID

**Key details:**
- `request` returns an execution ID for tracking and termination
- Three function types: **Standalone** (no datasource needed), **File**, and **Line** (both require datasource and object)
- Each execution produces timestamped log entries

### /kvs

Persist application state, metadata, and configurations in a native key-value store.

**Capabilities:**
- **stores** — List available KVS stores
- **create** — Insert a new entry (fails if the key already exists)
- **put** — Create or overwrite an entry (upsert)
- **get** — Retrieve an entry by key
- **delete** — Remove an entry
- **keys** — List all keys in a store, with optional filtering

**Key details:**
- Values are JSON objects
- `create` enforces insert-only semantics; `put` is upsert
- Multi-store support; defaults to the `"default"` store

### /storage

Upload, download, list, and delete objects in storage datasources.

**Capabilities:**
- **putref** — Upload an object (two-step: get presigned URL, then PUT the file)
- **getref** — Download an object (two-step: get presigned URL, then GET the file)
- **list** — List all objects in a storage datasource with metadata (name, size, timestamps)
- **delete** — Remove an object

**Key details:**
- Upload and download use presigned URLs — each is a two-step operation
- Use `list` to discover exact object names before downloading or deleting

## Setup

### Account and Keys

1. Register an account on the Greenative Platform
   - https://adp.avgidea.io (use the appropriate URL for dedicated instances)
   - [Account registration guide](https://doc.avgidea.io/account-registration)

2. Issue a key file
   - [API keys guide](https://doc.avgidea.io/application-integration/external-application-access)

3. Set environment variables
   
   Claude Code
   ```
   GN_KEY=<access key>
   GN_SECRET=<secret key>
   GN_ENDPOINT=<Greenative API Endpoint>
   ```

   Claude Desktop
   ```
   # ~/.claude/settings.json
   {
      "env": {
       "GN_KEY": "<access key>",
       "GN_SECRET": "<secret key>",
       "GN_ENDPOINT": "<Greenative API Endpoint>"
      },
      ...
   }
   ```


### Claude Code

Register this repository as a Claude Code plugin marketplace:
```
/plugin marketplace add greenative-ai/skills
```

Then install the skills and select the appropriate scope:
```
/plugin install greenative-skills
```

greenative-skills will be available after restarting Claude Code

## License

This project is licensed under the [MIT License](LICENSE).
