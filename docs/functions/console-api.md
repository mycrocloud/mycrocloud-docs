---
sidebar_position: 3
---

# Console API

MyCroCloud functions include a built-in `console` object for logging. Logs are captured during execution and displayed in the **Logs** page of your app dashboard.

## Basic Usage

```js
function handler(request) {
  console.log('Processing request:', request.method, request.path);

  const res = fetch('https://api.example.com/data');
  console.log('API responded with status:', res.status);

  if (!res.ok) {
    console.error('API request failed');
    return { statusCode: 502, body: 'Upstream error' };
  }

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify(res.json())
  };
}
```

## Methods

| Method           | Description                              |
|------------------|------------------------------------------|
| `console.log()`  | Logs a message at **Info** level         |
| `console.info()` | Logs a message at **Info** level (alias of `log`) |
| `console.warn()` | Logs a message at **Warning** level      |
| `console.error()`| Logs a message at **Error** level        |

All methods accept a single argument of any type (string, number, boolean, object, array).

## Supported Value Types

The console formats values based on their JavaScript type:

| Type      | Output example               |
|-----------|------------------------------|
| string    | `hello world`                |
| number    | `42`, `3.14`                 |
| boolean   | `true`, `false`              |
| null      | `null`                       |
| undefined | `undefined`                  |
| object    | `{ name: John, age: 30 }`   |
| array     | `[1, 2, 3]`                 |

## Examples

### Logging request details

```js
function handler(request) {
  console.log('Method: ' + request.method);
  console.log('Path: ' + request.path);
  console.log('Headers: ' + JSON.stringify(request.headers));

  return { statusCode: 200, body: 'OK' };
}
```

### Debugging with different log levels

```js
function handler(request) {
  console.info('Handler started');

  if (!request.body) {
    console.warn('Request has no body');
    return { statusCode: 400, body: 'Missing body' };
  }

  try {
    const data = JSON.parse(request.body);
    console.log('Parsed data: ' + JSON.stringify(data));
    return { statusCode: 200, body: 'OK' };
  } catch (e) {
    console.error('Failed to parse body: ' + e.message);
    return { statusCode: 400, body: 'Invalid JSON' };
  }
}
```

### Logging objects

```js
function handler(request) {
  const user = { name: 'Alice', role: 'admin' };
  console.log(user); // logs: { name: Alice, role: admin }

  const items = [1, 2, 3];
  console.log(items); // logs: [1, 2, 3]

  return { statusCode: 200, body: 'OK' };
}
```

## Limits

| Limit                  | Value        | Description                                          |
|------------------------|--------------|------------------------------------------------------|
| Max logs per second    | 10           | Logs beyond this rate within a 1-second window are silently dropped |
| Max message length     | 500 chars    | Messages longer than this are truncated with `... [truncated]` |
| Max total logs         | 100          | Oldest logs are discarded when this limit is exceeded |

:::tip
Logs are captured even when your function throws an error or times out. Use `console.error()` in error paths to help with debugging.
:::

## Viewing Logs

Function logs appear in the **Logs** page of your app dashboard. Each request that hits a function route shows its console output alongside the request details (method, path, status code, duration).
