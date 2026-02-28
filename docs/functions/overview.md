---
sidebar_position: 1
---

# Overview

MyCroCloud functions let you write JavaScript handlers that run on the server in response to HTTP requests. Each function is a single `handler` function that receives a [request](./request) object and returns a response.

```js
function handler(request) {
  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ message: 'Hello, world!' })
  };
}
```

## How It Works

When a request matches a function route, MyCroCloud:

1. Spins up an isolated Docker container
2. Passes the HTTP request data to your handler
3. Executes your JavaScript code
4. Returns the response to the caller
5. Destroys the container

Each invocation runs in a fresh, isolated environment — there is no shared state between requests.

## Runtime

Functions run on [Jint](https://github.com/sebastienros/jint), a JavaScript engine for .NET. Jint supports most of the ECMAScript 2022 specification, including:

- `let`, `const`, arrow functions, template literals
- Destructuring, spread/rest operators
- `Array` and `Object` methods (`map`, `filter`, `reduce`, `entries`, `keys`, `values`, etc.)
- `JSON.parse()` and `JSON.stringify()`
- `Map`, `Set`, `WeakMap`, `WeakSet`
- `Symbol`
- `for...of` loops, iterators
- Classes
- Optional chaining (`?.`) and nullish coalescing (`??`)
- Regular expressions

## What's Not Supported

Since MyCroCloud functions use a synchronous JavaScript engine, several features commonly available in Node.js or browsers are **not supported**:

| Feature | Why |
|---|---|
| `Promise`, `async`/`await` | The runtime is synchronous. `fetch()` is provided as a synchronous alternative. |
| `setTimeout`, `setInterval` | No event loop — timers are not available. |
| `import` / `require` | No module system. All code must be in a single handler file. |
| Node.js APIs (`fs`, `path`, `http`, `crypto`, etc.) | Functions run in Jint, not Node.js. |
| Web APIs (`DOM`, `window`, `document`, `localStorage`) | No browser environment. |
| `Buffer` | Not available. Use strings for data handling. |
| `TextEncoder` / `TextDecoder` | Not available. |
| `Date` timezone configuration | `Date` is available, but timezone behavior depends on the server. |

:::tip
The built-in [`fetch()`](./fetch-api) function works **synchronously** — no `await` needed. Call it like a regular function and use the result immediately.

```js
const data = fetch('https://api.example.com/data').json();
```
:::

## Available APIs

MyCroCloud provides the following built-in APIs:

| API | Description |
|---|---|
| [`fetch()`](./fetch-api) | Make HTTP requests to external services (synchronous) |
| [`console`](./console-api) | Log messages for debugging |

## Resource Limits

Each function invocation runs in an isolated container with the following limits:

| Resource | Limit |
|---|---|
| Execution timeout | 3 seconds (code execution) |
| Container timeout | 10 seconds (configurable, includes startup) |
| Memory | 64 MB |
| CPU | 0.25 cores |
| Processes/threads | 100 PIDs |

## Environment Variables

You can define environment variables in your app settings. They are available as a global `env` object:

```js
function handler(request) {
  const apiKey = env.API_KEY;
  const res = fetch('https://api.example.com/data', {
    headers: { 'Authorization': 'Bearer ' + apiKey }
  });

  return {
    statusCode: 200,
    headers: { 'content-type': 'application/json' },
    body: res.text()
  };
}
```
