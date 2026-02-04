---
sidebar_position: 2
---

# Configuration Reference

Complete reference for the routing configuration schema.

## Schema

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "name": "string (optional)",
      "priority": "number (optional)",
      "match": {
        "type": "Prefix | Exact | Regex",
        "path": "string"
      },
      "target": {
        "type": "Api | Static",
        "stripPrefix": "boolean (optional)",
        "rewrite": "string (optional)",
        "fallback": "string (optional)"
      }
    }
  ]
}
```

## Root Properties

### schemaVersion

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `schemaVersion` | string | Yes | Schema version. Currently `"1.0"` |

### routes

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `routes` | array | Yes | List of route definitions. Must have at least one route |

## Route Object

### name

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `name` | string | No | - | Human-readable name for the route. Used for logging and debugging |

### priority

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `priority` | number | No | `MAX_INT` | Route evaluation order. Lower numbers are evaluated first |

:::note
Routes without a priority are evaluated last. When multiple routes have the same priority, the order is undefined.
:::

### match

The match object defines when this route should be used.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `match.type` | string | Yes | Match strategy: `Prefix`, `Exact`, or `Regex` |
| `match.path` | string | Yes | Path pattern to match against |

#### Match Types

**Prefix**
```json
{ "type": "Prefix", "path": "/api" }
```
Matches any path starting with `/api`: `/api`, `/api/users`, `/api/orders/123`

**Exact**
```json
{ "type": "Exact", "path": "/health" }
```
Only matches the exact path `/health`. Does not match `/health/` or `/healthcheck`.

**Regex**
```json
{ "type": "Regex", "path": "^/v[0-9]+/.*" }
```
Matches using .NET regular expressions (case-insensitive).

### target

The target object defines how to handle matched requests.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `target.type` | string | Yes | Target type: `Api` or `Static` |
| `target.stripPrefix` | boolean | No | Remove matched prefix from path |
| `target.rewrite` | string | No | Override the resolved file path (Static only) |
| `target.fallback` | string | No | Fallback file if not found (Static only) |

#### Target Type: Api

Routes requests to API route definitions.

```json
{
  "type": "Api",
  "stripPrefix": true
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `stripPrefix` | boolean | `false` | If true, removes the matched prefix before looking up the route in the database |

**Example with stripPrefix:**

Request: `GET /api/users`
Match path: `/api`
With `stripPrefix: true`: Looks up route `/users`
With `stripPrefix: false`: Looks up route `/api/users`

#### Target Type: Static

Serves static files from the app's build artifacts.

```json
{
  "type": "Static",
  "fallback": "/index.html",
  "stripPrefix": true,
  "rewrite": "/main.js"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `fallback` | string | - | File to serve when requested file is not found |
| `stripPrefix` | boolean | `false` | Remove matched prefix when resolving file path |
| `rewrite` | string | - | Override the file path entirely |

:::tip SPA Support
Use `fallback: "/index.html"` to enable client-side routing for Single Page Applications. When a file is not found, the fallback file is served instead of a 404.
:::

## Validation Rules

1. `schemaVersion` must be `"1.0"`
2. `routes` array must not be empty
3. Each route must have valid `match` and `target` objects
4. `match.type` must be one of: `Prefix`, `Exact`, `Regex`
5. `target.type` must be one of: `Api`, `Static`
6. Static targets only support `GET` requests

## Full Example

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "name": "Health Check",
      "priority": 0,
      "match": { "type": "Exact", "path": "/health" },
      "target": { "type": "Api" }
    },
    {
      "name": "API v2",
      "priority": 1,
      "match": { "type": "Prefix", "path": "/api/v2" },
      "target": { "type": "Api", "stripPrefix": false }
    },
    {
      "name": "API v1",
      "priority": 2,
      "match": { "type": "Prefix", "path": "/api" },
      "target": { "type": "Api", "stripPrefix": true }
    },
    {
      "name": "Static Assets",
      "priority": 10,
      "match": { "type": "Regex", "path": "\\.(js|css|png|jpg|svg|ico)$" },
      "target": { "type": "Static" }
    },
    {
      "name": "SPA Fallback",
      "priority": 100,
      "match": { "type": "Prefix", "path": "/" },
      "target": { "type": "Static", "fallback": "/index.html" }
    }
  ]
}
```

This configuration:
1. Routes exact `/health` requests to an API health check endpoint
2. Routes `/api/v2/*` requests to API routes (keeping full path)
3. Routes other `/api/*` requests to API routes (stripping `/api` prefix)
4. Serves static files for common asset extensions
5. Falls back to SPA routing for all other requests
