---
name: function
description: Interact with the Greenative Platform Function API to execute functions, monitor executions, and terminate running functions.
---

User request: $ARGUMENTS

Execute and manage functions on the Greenative Platform. All requests are POST to `$GN_ENDPOINT` with a **base64-encoded** JSON body.

## Authentication

Build the authorization header from the environment variables:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
```

Every request must include this header: `-H "Authorization: Basic $AUTH"`

The JSON body must include:
- `component`: `"function"`

## Request Format

The JSON request body must be **base64-encoded** before sending. Use `printf` to build the JSON, pipe through `base64`, and pass via command substitution to `curl -d`:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"..."}' | base64)" \
  "$GN_ENDPOINT"
```

## Actions

### Execute Function (`request`)

Parameters:
- `name` (required) — Function name
- `datasource` (optional) — Avgidea Storage datasource name (required for File/Line function types, omit for Standalone functions)
- `object` (optional) — Object/file name to process (required for File/Line function types)

Standalone function (no datasource needed):
```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"request","name":"myfunc"}' | base64)" \
  "$GN_ENDPOINT"
```

File/Line function (with datasource and object):
```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"request","name":"linetest","datasource":"ds1","object":"data1.csv"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": "Kxf9K3L9T83Ke0pwMIOn",
  "status": "success"
}
```

The `data` field contains the **execution ID** — use it to track or terminate the execution.

### Terminate Execution (`terminate`)

Parameters:
- `id` (required) — Execution ID (returned from `request`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"terminate","id":"Kxf9K3L9T83Ke0pwMIOn"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": "Termination request sent",
  "status": "success"
}
```

### List Executions (`list`)

Parameters:
- `status` (optional) — Filter by execution status. Valid values: `starting`, `running`, `success`, `failed`, `timeout`, `terminated`. Omit for all executions.

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"list","status":"success"}' | base64)" \
  "$GN_ENDPOINT"
```

List all executions (no filter):
```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"function","action":"list"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": [
    {
      "name": "linetest",
      "registeredAt": "2024-04-16T02:23:26.512039Z",
      "id": "Kxf9K3L9T83Ke0pwMIOn",
      "status": "success",
      "updatedAt": "2024-04-16T02:23:29.258Z",
      "logs": [
        {
          "registeredAt": "2024-04-16T02:23:28.514009Z",
          "text": "Start processing Kxf9K3L9T83Ke0pwMIOn"
        }
      ]
    }
  ],
  "status": "success"
}
```

Response fields per execution:
- `name` — Function name
- `registeredAt` — Execution start timestamp (ISO 8601)
- `id` — Execution ID
- `status` — Current status (`starting`, `running`, `success`, `failed`, `timeout`, `terminated`)
- `updatedAt` — Last update timestamp (ISO 8601)
- `logs` — Array of log entries, each with:
  - `registeredAt` — Log timestamp
  - `text` — Log message

## Notes

- Three function types exist: **Standalone** (no datasource/object), **File**, and **Line** (both require datasource and object).
- The `request` action returns an execution ID — save it to track progress or terminate.
- Use `list` with `status` filter to monitor executions (e.g., check for `running` or `failed`).
- Terminated functions transition to `"terminated"` status.
