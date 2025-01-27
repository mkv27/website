# Bearer Auth Middleware

The Bearer Auth Middleware provides authentication by verifying an API token in the Request header.
The HTTP clients accessing the endpoint will add the `Authorization` header with `Bearer {token}` as the header value.

Using `curl` from the terminal, it would look like this:

```
curl -H 'Authorization: Bearer honoiscool' http://localhost:8787/auth/page
```

## Import

::: code-group

```ts [npm]
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'
```

```ts [Deno]
import { Hono } from 'https://deno.land/x/hono/mod.ts'
import { bearerAuth } from 'https://deno.land/x/hono/middleware.ts'
```

:::

## Usage

```ts
const app = new Hono()

const token = 'honoiscool'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/page', (c) => {
  return c.json({ message: 'You are authorized' })
})
```

To restrict to a specific route + method:

```ts
const app = new Hono()

const token = 'honoiscool'

app.get('/api/page', (c) => {
  return c.json({ message: 'Read posts' })
})

app.post('/api/page', bearerAuth({ token }), (c) => {
  return c.json({ message: 'Created post!' }, 201)
})
```

To implement multiple tokens (E.g., any valid token can read but create/update/delete are restricted to a privileged token):

```ts
const app = new Hono()

const readToken = 'read'
const privilegedToken = 'read+write'
const privilegedMethods = ['POST', 'PUT', 'PATCH', 'DELETE']

app.on('GET', '/api/page/*', async (c, next) => {
  // List of valid tokens
  const bearer = bearerAuth({ token: [readToken, privilegeToken] });
  return bearer(c, next);
})
app.on(privilegedMethods, '/api/page/*', async (c, next) => {
  // Single valid privileged token
  const bearer = bearerAuth({ token: privilegedToken });
  return bearer(c, next);
})

// Define handlers for GET, POST, etc.
```

## Options

- `token`: string | string[] - _required_
  - The string to validate the incoming bearer token against
- `realm`: string
  - The domain name of the realm, as part of the returned WWW-Authenticate challenge header. Default is `""`
  - _See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate#directives_
- `prefix`: string
  - The prefix for the Authorization header value. Default is `"Bearer"`
- `hashFunction`: Function
  - A function to handle hashing for safe comparison of authentication tokens
