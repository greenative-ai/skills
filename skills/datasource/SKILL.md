---
name: datasource
description: Interact with the Greenative Platform Datasource API to list datasources, verify connections, and execute SQL statements.
---

User request: $ARGUMENTS

Manage datasources and execute SQL on the Greenative Platform. All requests are POST to `$GN_ENDPOINT` with a **base64-encoded** JSON body.

## Authentication

Every request must include:
- `accesskey`: `$GN_KEY`
- `secret`: `$GN_SECRET`
- `component`: `"datasource"`

## Request Format

The JSON request body must be **base64-encoded** before sending. Use `printf` to build the JSON, pipe through `base64`, and pass via command substitution to `curl -d`:

```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"..."}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

## Actions

### List Datasources (`list`)

Parameters (all optional):
- `datasource` — Filter by datasource name
- `type` — Filter by datasource type (e.g., `"oc"` for Oracle, `"as"` for Avgidea Storage)

```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"list"}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

With filters:
```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"list","type":"oc"}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": [
    {
      "name": "ds1",
      "type": "oc",
      "description": "Oracle production",
      "value": "connection-string",
      "registeredAt": "2024-04-24T09:42:29.628Z"
    }
  ],
  "status": "success"
}
```

Response fields per datasource:
- `name` — Datasource name
- `type` — Datasource type
- `description` — Description
- `value` — Connection string/value
- `registeredAt` — Registration timestamp

### Verify Connection (`connect`)

Parameters:
- `datasource` (required) — Datasource name to test

```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"connect","datasource":"ds1"}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": {
    "host": "hostname",
    "port": 1521,
    "service": "service_name",
    "type": "oc",
    "uname": "username",
    "value": "connection-string"
  },
  "status": "success"
}
```

### Execute DDL/DML Statements (`execute` with `statement`)

Use the `statement` field for DDL (CREATE, ALTER, DROP) and DML (INSERT, UPDATE, DELETE) operations.

Parameters:
- `datasource` (required) — Target database datasource name
- `statement` (required) — SQL statement to execute
- `args` (optional) — Array of bind variable values (placeholders use `:1`, `:2`, etc.)
- `session` (optional) — Database session settings object (e.g., `{"NLS_DATE_FORMAT": "YYYY-MM-DD"}`)

```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"execute","datasource":"ds1","statement":"CREATE TABLE test (id NUMBER, name VARCHAR2(100))"}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

With bind variables:
```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"execute","datasource":"ds1","statement":"INSERT INTO test (id, name) VALUES (:1, :2)","args":[1,"Alice"]}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

With session parameters:
```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"execute","datasource":"ds1","statement":"INSERT INTO test (id, dt) VALUES (:1, :2)","args":[1,"2024-01-15"],"session":{"NLS_DATE_FORMAT":"YYYY-MM-DD"}}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Executed successfully", "status": "success"}
```

### Execute SELECT Queries (`execute` with `query`)

Use the `query` field for SELECT operations. This is a **different field** from `statement`.

Parameters:
- `datasource` (required) — Target database datasource name
- `query` (required) — SELECT statement
- `args` (optional) — Array of bind variable values
- `session` (optional) — Database session settings object

```bash
curl -s -X POST -H 'content-type: application/json' \
  -d "$(printf '{"accesskey":"%s","secret":"%s","component":"datasource","action":"execute","datasource":"ds1","query":"SELECT id, name FROM test WHERE id = :1","args":[1]}' "$GN_KEY" "$GN_SECRET" | base64)" \
  "$GN_ENDPOINT"
```

Response (two-dimensional array — first row is headers):
```json
{
  "data": [
    ["ID", "NAME"],
    [1, "Alice"],
    [2, "Bob"]
  ],
  "status": "success"
}
```

## Notes

- The `execute` action uses `statement` for DDL/DML and `query` for SELECT — these are **different field names**.
- Bind variable placeholders use `:1`, `:2` style (not `?`).
- The `session` parameter sets database session variables (e.g., Oracle `NLS_DATE_FORMAT`).
- Always use `list` first to discover available datasources and their types.
- Use `connect` to verify a datasource is reachable before executing statements.
