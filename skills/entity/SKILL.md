---
name: entity
description: Interact with the Greenative Platform Entity API to transfer data between storages, load CSVs into databases, and export database tables to CSV.
---

User request: $ARGUMENTS

Transfer data between storage and database datasources on the Greenative Platform. All requests are POST to `$GN_ENDPOINT` with a **base64-encoded** JSON body.

## Authentication

Build the authorization header from the environment variables:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
```

Every request must include this header: `-H "Authorization: Basic $AUTH"`

The JSON body must include:
- `component`: `"entity"`

## Request Format

The JSON request body must be **base64-encoded** before sending. Use `printf` to build the JSON, pipe through `base64`, and pass via command substitution to `curl -d`:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"entity","action":"..."}' | base64)" \
  "$GN_ENDPOINT"
```

## Actions

All operations use the `transfer` action — behavior differs based on which parameters are provided.

### Transfer Single File Between Storages

Parameters:
- `from` (required) — Source storage datasource name
- `to` (required) — Destination storage datasource name
- `object` (required) — Specific object filename to transfer

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"entity","action":"transfer","from":"storage1","to":"storage2","object":"data.csv"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Transferred successfully", "status": "success"}
```

### Transfer Multiple Files Between Storages

Parameters:
- `from` (required) — Source storage datasource name
- `to` (required) — Destination storage datasource name
- `filter` (required) — String to match in object names (instead of `object`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"entity","action":"transfer","from":"storage1","to":"storage2","filter":"2024-04"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Transferred successfully", "status": "success"}
```

### Load CSV into Database

Parameters:
- `from` (required) — Storage datasource name (source)
- `to` (required) — Database datasource name (destination)
- `object` (required) — CSV filename in storage
- `table` (required) — Target database table name
- `worker` (required) — Number of parallel workers

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"entity","action":"transfer","from":"storage1","to":"db1","object":"data.csv","table":"my_table","worker":4}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Transferred successfully", "status": "success"}
```

### Export Database Table to CSV

Parameters:
- `from` (required) — Database datasource name (source)
- `to` (required) — Storage datasource name (destination)
- `table` (required) — Source table name (the CSV file will be named after the table)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"entity","action":"transfer","from":"db1","to":"storage1","table":"my_table"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Transferred successfully", "status": "success"}
```

## Notes

- All operations use the same `transfer` action — the combination of parameters determines the transfer type.
- Storage-to-storage: use `object` for a single file or `filter` for multiple files matching a pattern.
- CSV-to-DB: requires `object`, `table`, and `worker` parameters.
- DB-to-CSV: requires only `table` (the output CSV is named after the table).
- The `worker` parameter controls parallelism for CSV-to-DB loading.
- Use the `/datasource` skill to list available datasources and verify connections before transferring.
