---
name: storage
description: Interact with the Greenative Platform Storage API to upload, download, list, and delete objects in storage datasources.
---

User request: $ARGUMENTS

Manage objects in Greenative Platform storage datasources. All requests are POST to `$GN_ENDPOINT` with a **base64-encoded** JSON body.

## Authentication

Build the authorization header from the environment variables:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
```

Every request must include this header: `-H "Authorization: Basic $AUTH"`

The JSON body must include:
- `component`: `"storage"`

## Request Format

The JSON request body must be **base64-encoded** before sending. Use `printf` to build the JSON, pipe through `base64`, and pass via command substitution to `curl -d`:

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"storage","action":"..."}' | base64)" \
  "$GN_ENDPOINT"
```

## Actions

### Upload Object (`putref`)

Upload is a **two-step** process: first get a presigned upload URL, then PUT the file to that URL.

**Step 1 — Get presigned upload URL:**

Parameters:
- `datasource` (required) — Storage datasource name
- `object` (required) — Object filename

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"storage","action":"putref","datasource":"ds1","object":"data.csv"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "https://<upload url>", "status": "success"}
```

**Step 2 — Upload the file using the presigned URL:**

```bash
curl -X PUT --upload-file ./data.csv "https://<upload url>"
```

### Download Object (`getref`)

Download is a **two-step** process: first get a presigned download URL, then GET the file from that URL.

**Step 1 — Get presigned download URL:**

Parameters:
- `datasource` (required) — Storage datasource name
- `object` (required) — Object filename

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"storage","action":"getref","datasource":"ds1","object":"data.csv"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "https://<download url>", "status": "success"}
```

**Step 2 — Download the file using the presigned URL:**

```bash
curl -X GET -o ./data.csv "https://<download url>"
```

### List Objects (`list`)

Parameters:
- `datasource` (required) — Storage datasource name

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"storage","action":"list","datasource":"ds1"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{
  "data": [
    {
      "name": "data/file1.txt",
      "registeredAt": "2024-04-24T09:42:29.628Z",
      "size": 20017,
      "updatedAt": "2024-04-24T09:42:29.628Z"
    }
  ],
  "status": "success"
}
```

Response fields per object:
- `name` — Object name/path
- `registeredAt` — Creation timestamp (ISO 8601)
- `size` — File size in bytes
- `updatedAt` — Last update timestamp (ISO 8601)

### Delete Object (`delete`)

Parameters:
- `datasource` (required) — Storage datasource name
- `object` (required) — Object name to delete

```bash
AUTH=$(printf "%s:%s" "$GN_KEY" "$GN_SECRET" | base64 | tr -d '\n')
curl -s -X POST -H 'content-type: application/json' -H "Authorization: Basic $AUTH" \
  -d "$(printf '{"component":"storage","action":"delete","datasource":"ds1","object":"file1.txt"}' | base64)" \
  "$GN_ENDPOINT"
```

Response:
```json
{"data": "Deleted successfully", "status": "success"}
```

## Notes

- Upload and download are two-step operations using presigned URLs.
- The filename parameter is `object` for all actions.
- Always list objects first if you are unsure of exact object names.
