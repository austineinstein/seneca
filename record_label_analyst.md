# Record Label Access Token Documentation

This repository supports record label users requesting a usable access token through the `https://www.fancradle.com/api/get-token` endpoint.

## Token endpoint

Use the following endpoint to request an access token:

- `GET https://www.fancradle.com/api/get-token`

This repository also includes a local proxy implementation in `server/server.js` that forwards requests to:

- `https://fancradle.com/get-token`

## Purpose

- Provides a usable access token for record label users.
- The token is used for authenticated access to the upstream token service.
- The response should be JSON containing the usable token data.

## Example request

```bash
curl -X GET "https://www.fancradle.com/api/get-token"
```

### Example response

A successful response typically returns JSON similar to:

```json
{
  "accessToken": "...",
  "expiresIn": 3600,
  "tokenType": "Bearer"
}
```

> The exact response shape may differ depending on the token provider.

## Proxy implementation note

This repository exposes `server/server.js` as a simple proxy handler:

- It accepts `GET` requests.
- It forwards the request to `https://fancradle.com/api/get-token`.
- It returns the upstream JSON response back to the client.

When deployed to a platform that maps the repo's server function to `/api/get-token`, record label clients can use the same route without calling the upstream host directly.

## Record label user flow

1. Record label requests a token from `https://www.fancradle.com/api/get-token`.
2. The proxy or upstream server returns a usable access token in JSON form.
3. The record label uses that token for further authenticated API calls.

## Important

- The request method is `GET`.
- The endpoint should return a usable access token without requiring a request body.
- If the repo is deployed with server support, this file forwards requests to the upstream provider.
