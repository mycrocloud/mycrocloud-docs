---
sidebar_position: 3
---

# Troubleshooting

Common routing issues and how to resolve them.

## 404 Not Found for API Routes

### Symptom
API requests return 404 even though routes are defined in the database.

### Cause
Mismatch between the routing configuration and how routes are stored.

### Solution

Check if your routing config uses `stripPrefix` and ensure your route paths match.

**Scenario 1: Using `stripPrefix: true`**

```json
{
  "match": { "type": "Prefix", "path": "/api" },
  "target": { "type": "Api", "stripPrefix": true }
}
```

With this config, request `/api/users` becomes `/users` for route lookup.
Your routes should be stored as: `/users`, `/orders`, etc.

**Scenario 2: Using `stripPrefix: false`**

```json
{
  "match": { "type": "Prefix", "path": "/api" },
  "target": { "type": "Api", "stripPrefix": false }
}
```

With this config, request `/api/users` stays as `/api/users` for route lookup.
Your routes should be stored as: `/api/users`, `/api/orders`, etc.

:::tip
If you're unsure, set `stripPrefix: false` and store routes with their full paths.
:::

## Static Files Not Loading

### Symptom
Static assets return 404 or the wrong content.

### Possible Causes

1. **No build deployed**: Check that a build has been deployed to the app
2. **Wrong file path**: The file path in the build doesn't match the request
3. **Priority conflict**: A higher-priority route is matching first

### Solution

1. Verify a build exists and is set as the latest build
2. Check that file paths in your build match the request paths
3. Review route priorities to ensure static routes have appropriate priority

## SPA Routes Not Working

### Symptom
Refreshing the page on a client-side route returns 404.

### Cause
Missing fallback configuration for the static target.

### Solution

Add a fallback to your static route:

```json
{
  "match": { "type": "Prefix", "path": "/" },
  "target": { "type": "Static", "fallback": "/index.html" }
}
```

## Wrong Route Matched

### Symptom
Requests are being handled by unexpected routes.

### Cause
Route priority order issue.

### Solution

1. Add explicit `priority` values to all routes
2. More specific routes should have lower priority numbers (higher precedence)
3. Catch-all routes should have higher priority numbers (lower precedence)

**Example:**
```json
{
  "routes": [
    {
      "priority": 1,
      "match": { "type": "Exact", "path": "/api/health" },
      "target": { "type": "Api" }
    },
    {
      "priority": 2,
      "match": { "type": "Prefix", "path": "/api" },
      "target": { "type": "Api", "stripPrefix": true }
    },
    {
      "priority": 100,
      "match": { "type": "Prefix", "path": "/" },
      "target": { "type": "Static", "fallback": "/index.html" }
    }
  ]
}
```

## Debugging Tips

### Enable Debug Logging

Contact support to enable debug logging for your app. Debug logs show:
- Which routing config is being used (custom or default)
- Which route matched the request
- Path transformations (strip prefix, rewrite)
- File lookups for static targets

### Common Checks

1. **Verify the routing config is saved**: Fetch your current config via API
2. **Check route paths case sensitivity**: Matching is case-insensitive
3. **Test with simple config**: Start with a minimal config and add complexity
4. **Check build artifacts**: Ensure your static files are in the build

## Getting Help

If you're still having issues:

1. Check the request logs in your app dashboard
2. Verify your routing configuration is valid JSON
3. Test with the default configuration to isolate the issue
