---
sidebar_position: 1
---

# Request Handling Overview

This guide explains how MyCroCloud processes incoming HTTP requests and routes them to the appropriate target.

## Request Flow

When a request arrives at MyCroCloud, it goes through the following pipeline:

```
Request → App Resolution → CORS → Routing → Route Resolution → Authentication → Authorization → Validation → Response
```

### 1. App Resolution

MyCroCloud first identifies which app should handle the request based on the incoming host header or app identifier.

### 2. CORS Handling

Cross-Origin Resource Sharing (CORS) headers are processed if configured for the app.

### 3. Routing

The routing middleware matches the request path against the app's routing configuration to determine where the request should be directed.

### 4. Route Resolution

For API routes, the system resolves the specific route definition from the database.

### 5. Authentication & Authorization

If the route requires authentication, the request is validated against the configured authentication scheme.

### 6. Response Generation

Based on the route type, the response is generated:
- **Static**: Returns a static response or file
- **Function**: Executes a serverless function

## Routing Configuration

MyCroCloud uses a declarative routing configuration that determines how requests are handled. If no custom configuration is provided, a default configuration is applied.

### Default Configuration

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "priority": 1,
      "match": { "type": "Prefix", "path": "/api" },
      "target": { "type": "Api", "stripPrefix": true }
    },
    {
      "priority": 2,
      "match": { "type": "Prefix", "path": "/" },
      "target": { "type": "Static", "fallback": "/index.html" }
    }
  ]
}
```

This default configuration:
1. Routes all `/api/*` requests to API routes (stripping the `/api` prefix)
2. Serves static files for all other requests, falling back to `index.html` for SPA support

## Route Matching

Routes are evaluated in order of priority (lower number = higher precedence). The first matching route handles the request.

### Match Types

| Type | Description | Example |
|------|-------------|---------|
| `Prefix` | Matches if the path starts with the specified value | `/api` matches `/api/users`, `/api/orders` |
| `Exact` | Matches only if the path exactly equals the value | `/health` only matches `/health` |
| `Regex` | Matches using a regular expression | `^/v[0-9]+/.*` matches `/v1/users`, `/v2/orders` |

## Target Types

### API Target

Routes requests to API route definitions stored in the database.

```json
{
  "type": "Api",
  "stripPrefix": true
}
```

**Options:**
- `stripPrefix`: If `true`, removes the matched prefix from the path before route resolution

### Static Target

Serves static files from the app's latest build artifacts.

```json
{
  "type": "Static",
  "fallback": "/index.html",
  "rewrite": "/assets/main.js"
}
```

**Options:**
- `fallback`: File to serve when the requested file is not found (useful for SPAs)
- `rewrite`: Override the file path to serve
- `stripPrefix`: Remove the matched prefix when looking up files

## Example Configurations

### SPA with API Backend

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "name": "API Routes",
      "priority": 1,
      "match": { "type": "Prefix", "path": "/api" },
      "target": { "type": "Api", "stripPrefix": true }
    },
    {
      "name": "Static Assets",
      "priority": 2,
      "match": { "type": "Prefix", "path": "/" },
      "target": { "type": "Static", "fallback": "/index.html" }
    }
  ]
}
```

### API Only (No Prefix Stripping)

If your API routes are stored with the full path (e.g., `/api/users`), disable prefix stripping:

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "priority": 1,
      "match": { "type": "Prefix", "path": "/" },
      "target": { "type": "Api", "stripPrefix": false }
    }
  ]
}
```

### Multiple API Versions

```json
{
  "schemaVersion": "1.0",
  "routes": [
    {
      "name": "API v2",
      "priority": 1,
      "match": { "type": "Prefix", "path": "/v2" },
      "target": { "type": "Api", "stripPrefix": false }
    },
    {
      "name": "API v1 (Legacy)",
      "priority": 2,
      "match": { "type": "Prefix", "path": "/v1" },
      "target": { "type": "Api", "stripPrefix": false }
    },
    {
      "name": "Default to v2",
      "priority": 3,
      "match": { "type": "Prefix", "path": "/api" },
      "target": { "type": "Api", "stripPrefix": true }
    }
  ]
}
```

## Understanding stripPrefix

The `stripPrefix` option is important for matching routes in the database. Here's how it works:

| Request Path | Match Path | stripPrefix | Path Sent to Route Resolution |
|-------------|------------|-------------|-------------------------------|
| `/api/users` | `/api` | `true` | `/users` |
| `/api/users` | `/api` | `false` | `/api/users` |
| `/v1/orders` | `/v1` | `true` | `/orders` |
| `/v1/orders` | `/v1` | `false` | `/v1/orders` |

:::tip
Make sure your route definitions in the database match the path that will be used for resolution. If you use `stripPrefix: true` with match path `/api`, your routes should be defined as `/users`, not `/api/users`.
:::

## Configuring Routing

You can configure routing through:

1. **Dashboard**: Navigate to App Settings > General > Routing Configuration
2. **API**: `POST /api/apps/{appId}/routing-config`

The configuration editor in the dashboard provides a JSON editor with syntax highlighting for easy editing.
