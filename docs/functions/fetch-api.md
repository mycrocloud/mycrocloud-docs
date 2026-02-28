---
sidebar_position: 3
---

# Fetch API

MyCroCloud functions include a built-in `fetch()` function for making HTTP requests to external APIs. It follows the standard [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) with some differences noted below.

## Basic Usage

```js
function handler(request) {
  const res = fetch('https://api.example.com/users/1');
  const user = res.json();

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify(user)
  };
}
```

## Syntax

```js
fetch(url)
fetch(url, options)
```

### Parameters

#### `url` (string, required)

The URL to fetch. Must use `http://` or `https://` scheme.

#### `options` (object, optional)

| Property  | Type   | Default      | Description                     |
|-----------|--------|--------------|---------------------------------|
| `method`  | string | `"GET"`      | HTTP method (GET, POST, PUT, DELETE, PATCH, etc.) |
| `headers` | object | `{}`         | Request headers as key-value pairs |
| `body`    | string | `undefined`  | Request body (typically used with POST/PUT) |

### Return Value

Returns a **Response** object with the following properties and methods:

| Property/Method | Type     | Description                                  |
|-----------------|----------|----------------------------------------------|
| `status`        | number   | HTTP status code (e.g., `200`, `404`)        |
| `statusText`    | string   | HTTP status text (e.g., `"OK"`, `"Not Found"`) |
| `ok`            | boolean  | `true` if status is 200–299                  |
| `headers`       | object   | Response headers as key-value pairs (lowercase keys) |
| `text()`        | function | Returns the response body as a string        |
| `json()`        | function | Parses the response body as JSON and returns the result |

## Examples

### GET request

```js
function handler(request) {
  const res = fetch('https://jsonplaceholder.typicode.com/posts/1');

  return {
    statusCode: res.status,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify(res.json())
  };
}
```

### POST with JSON body

```js
function handler(request) {
  const res = fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer ' + env.API_TOKEN
    },
    body: JSON.stringify({
      name: request.body.name,
      email: request.body.email
    })
  });

  if (!res.ok) {
    return {
      statusCode: 502,
      body: 'Upstream API error: ' + res.status + ' ' + res.statusText
    };
  }

  return {
    statusCode: 201,
    headers: { 'content-type': 'application/json' },
    body: res.text()
  };
}
```

### Reading response headers

```js
function handler(request) {
  const res = fetch('https://api.example.com/data');

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({
      contentType: res.headers['content-type'],
      data: res.json()
    })
  };
}
```

### Multiple requests

```js
function handler(request) {
  const users = fetch('https://api.example.com/users').json();
  const posts = fetch('https://api.example.com/posts').json();

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ users, posts })
  };
}
```

## Differences from the Web Fetch API

| Feature              | Web Fetch API         | MyCroCloud `fetch()`         |
|----------------------|-----------------------|------------------------------|
| Return type          | `Promise<Response>`   | `Response` (synchronous)     |
| `.text()` / `.json()`| Returns a `Promise`   | Returns the value directly   |
| Streaming body       | Supported             | Not supported                |
| `Request` object     | Supported as input    | Not supported (use URL string) |
| `Headers` class      | Supported             | Plain object instead         |
| `AbortController`    | Supported             | Not supported                |
| `mode`, `credentials`, `cache` | Supported   | Not supported                |

Since MyCroCloud functions run in a synchronous JavaScript engine, `fetch()` and its response methods (`text()`, `json()`) return values directly instead of Promises. There is no need for `await` or `.then()`.

## Limits

| Limit                    | Value        | Description                                          |
|--------------------------|--------------|------------------------------------------------------|
| Max requests per execution | 50         | Total number of `fetch()` calls allowed per function invocation |
| Request body size        | 1 MB         | Maximum size of the outgoing request body            |
| Response body size       | 5 MB         | Maximum size of a single response body               |
| Total bandwidth          | 10 MB        | Total bytes received across all fetch calls          |
| Per-request timeout      | 5 seconds    | Maximum time for a single fetch request              |
| Max redirects            | 5            | Maximum number of HTTP redirects followed            |
| Max recursion depth      | 3            | Maximum depth when functions call other MyCroCloud functions |

Exceeding these limits will throw an error that terminates your function execution.

## Security Restrictions

For security, the following restrictions are enforced:

- **HTTPS/HTTP only** — Only `http://` and `https://` URLs are allowed. Schemes like `file://`, `ftp://`, and `data://` are blocked.
- **No private network access** — Requests to private/internal IP addresses are blocked, including `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, and cloud metadata endpoints (`169.254.169.254`).
- **Restricted headers** — The `Host`, `Cookie`, `Set-Cookie`, and `User-Agent` headers cannot be set on outgoing requests.
- **Fixed User-Agent** — All requests are sent with `User-Agent: MycroCloud-FunctionInvoker/1.0`, which cannot be overridden.
- **Recursion depth limit** — When a function uses `fetch()` to call another MyCroCloud function (which may in turn call another), the call chain is limited to 3 levels deep. This prevents infinite loops such as function A calling function B which calls function A back. Exceeding this limit returns HTTP `508 Loop Detected`.
