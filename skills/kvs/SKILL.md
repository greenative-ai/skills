---
name: kvs
description: Interact with the Greenative Platform KVS (Key-Value Store) API to manage stores and key-value entries.
---

User request: $ARGUMENTS

Manage key-value stores and entries on the Greenative Platform. All requests are POST to `$GN_ENDPOINT` with a **base64-encoded** JSON body.

## Authentication

Build the authorization header from the environment variables:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
```

Every request must include this header: `-H "Authorization: Basic $AUTH"`

The JSON body must include:
- `component`: `"kvs"`

## Request Format

The JSON request body must be **base64-encoded** before sending. Use `printf` to build the JSON, pipe through `base64`, and pass via command substitution to `curl -d`:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"..."}' | base64)" \
  "$GN_ENDPOINT"
```

## Actions

### List Stores (`stores`)

No additional parameters required.

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"stores"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": [
    {"id": "default", "registeredAt": "2024-04-24T09:42:29.628Z"}
  ],
  "status": "success"
}
```

### Create Entry (`create`)

Creates a new entry. **Fails if the key already exists** — use `put` to overwrite.

Parameters:
- `key` (required) — Entry key
- `value` (required) — JSON object value
- `store` (optional) — Store name (defaults to `"default"`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"create","key":"user1","value":{"name":"Alice","age":30}}' | base64)" \
  "$GN_ENDPOINT"
```

With explicit store:
```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"create","key":"user1","value":{"name":"Alice"},"store":"mystore"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "user1", "status": "success"}
```

### Put Entry (`put`)

Creates or **overwrites** an entry. Use this when you want to update an existing key.

Parameters:
- `key` (required) — Entry key
- `value` (required) — JSON object value
- `store` (optional) — Store name (defaults to `"default"`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"put","key":"user1","value":{"name":"Alice","age":31}}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "user1", "status": "success"}
```

### Get Entry (`get`)

Parameters:
- `key` (required) — Entry key
- `store` (optional) — Store name (defaults to `"default"`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"get","key":"user1"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": {"name": "Alice", "age": 31}, "status": "success"}
```

### Delete Entry (`delete`)

Parameters:
- `key` (required) — Entry key
- `store` (optional) — Store name (defaults to `"default"`)

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"delete","key":"user1"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Entry deleted", "status": "success"}
```

### List Keys (`keys`)

Parameters:
- `store` (optional) — Store name (defaults to `"default"`)
- `filter` (optional) — String to filter keys by

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"keys"}' | base64)" \
  "$GN_ENDPOINT"
```

With filter:
```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"kvs","action":"keys","filter":"user","store":"mystore"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": [
    {"id": "user1", "registeredAt": "2024-04-24T09:42:29.628Z", "updatedAt": "2024-04-24T10:15:00.000Z"}
  ],
  "status": "success"
}
```

## Notes

- `create` **fails** if the key already exists; `put` **overwrites** if the key exists. Use `create` for insert-only semantics, `put` for upsert.
- The `store` parameter defaults to `"default"` if omitted.
- Floating-point numbers representable as integers (e.g., `1.0`, `2.0`) auto-convert to integer type.
- Each operation consumes one KVS quota unit.
- Use `stores` to discover available stores, then `keys` to browse entries within a store.
