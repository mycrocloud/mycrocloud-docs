---
sidebar_position: 1
---

# Request & Response

Every MyCroCloud function receives a `request` object as its argument and must return a response object. This page documents both.

## Handler Signature

```js
function handler(request) {
  // process the request...

  return {
    statusCode: 200,
    body: 'Hello, world!'
  };
}
```

## Request Object

The `request` object contains the following properties:

| Property   | Type                       | Description                                             |
|------------|----------------------------|---------------------------------------------------------|
| `method`   | `string`                   | HTTP method in uppercase (e.g., `"GET"`, `"POST"`)      |
| `path`     | `string`                   | Request path (e.g., `"/api/users/123"`)                 |
| `params`   | `Record<string, string>`   | Route parameters extracted from the path                |
| `query`    | `Record<string, string>`   | Query string parameters                                 |
| `headers`  | `Record<string, string>`   | HTTP request headers                                    |
| `body`     | `any`                      | Parsed request body (`null` if no body was sent)        |

### `method`

The HTTP method of the incoming request, always uppercase.

```js
function handler(request) {
  if (request.method === 'GET') {
    return { statusCode: 200, body: 'Reading data' };
  }
  if (request.method === 'POST') {
    return { statusCode: 201, body: 'Created' };
  }
  return { statusCode: 405, body: 'Method not allowed' };
}
```

### `path`

The URL path of the incoming request, without the query string.

```js
function handler(request) {
  console.log(request.path); // "/api/users/123"
  return { statusCode: 200, body: 'OK' };
}
```

### `params`

An object containing route parameters. When you define a route like `/users/{id}`, the matched value is available as `request.params.id`.

```js
// Route: /users/{id}
function handler(request) {
  const userId = request.params.id;
  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ userId })
  };
}
```

### `query`

An object containing query string parameters as key-value pairs.

```js
// Request: /search?q=hello&page=2
function handler(request) {
  const searchTerm = request.query.q;     // "hello"
  const page = request.query.page;        // "2" (always a string)
  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ searchTerm, page: Number(page) })
  };
}
```

:::note
Query parameter values are always strings. Use `Number()`, `parseInt()`, or `parseFloat()` to convert numeric values.
:::

### `headers`

An object containing the HTTP request headers as key-value pairs.

```js
function handler(request) {
  const contentType = request.headers['Content-Type'];
  const auth = request.headers['Authorization'];

  if (!auth) {
    return { statusCode: 401, body: 'Unauthorized' };
  }

  return { statusCode: 200, body: 'OK' };
}
```

### `body`

The parsed request body. The body is automatically parsed as JSON. If the request has no body, `body` is `null`.

```js
// POST /users with body: {"name": "Alice", "email": "alice@example.com"}
function handler(request) {
  const { name, email } = request.body;
  console.log('Creating user:', name);

  return {
    statusCode: 201,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ name, email, created: true })
  };
}
```

:::tip
Since the body is parsed as JSON automatically, you don't need to call `JSON.parse()` yourself. Access fields directly with `request.body.fieldName`.
:::

## Response Object

Your handler must return an object with the following properties:

| Property     | Type                       | Required | Description                                     |
|--------------|----------------------------|----------|-------------------------------------------------|
| `statusCode` | `number`                   | Yes      | HTTP status code (e.g., `200`, `404`, `500`)    |
| `headers`    | `Record<string, string>`   | No       | Response headers as key-value pairs             |
| `body`       | `string`                   | Yes      | Response body as a string                       |

### `statusCode`

The HTTP status code to send back to the client.

```js
return { statusCode: 200, body: 'OK' };
return { statusCode: 404, body: 'Not found' };
return { statusCode: 500, body: 'Internal error' };
```

### `headers`

An optional object of response headers. Header values can be strings, numbers, or booleans. `null` values are ignored.

```js
return {
  statusCode: 200,
  headers: {
    'content-type': 'application/json',
    'cache-control': 'max-age=3600',
    'x-request-id': '12345'
  },
  body: JSON.stringify({ message: 'Hello' })
};
```

### `body`

The response body as a string. For JSON responses, use `JSON.stringify()`.

```js
// Plain text
return { statusCode: 200, body: 'Hello, world!' };

// JSON
return {
  statusCode: 200,
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ message: 'Hello' })
};

// HTML
return {
  statusCode: 200,
  headers: { 'content-type': 'text/html' },
  body: '<h1>Hello, world!</h1>'
};
```

## Full Example

```js
// Route: /api/greet/{name}
function handler(request) {
  console.log(request.method + ' ' + request.path);

  if (request.method !== 'POST') {
    return { statusCode: 405, body: 'Method not allowed' };
  }

  const name = request.params.name;
  const lang = request.query.lang || 'en';
  const { title } = request.body || {};

  const greetings = {
    en: 'Hello',
    es: 'Hola',
    fr: 'Bonjour'
  };

  const greeting = greetings[lang] || greetings.en;
  const fullName = title ? title + ' ' + name : name;

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ message: greeting + ', ' + fullName + '!' })
  };
}
```
