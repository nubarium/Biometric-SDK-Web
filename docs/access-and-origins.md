# Tokens and allowed origins

[Leer en español](../es/docs/access-and-origins.md)

Make these APIv2 calls from your backend or an administrative tool. Do not call them from browser code or include permanent credentials in the SDK configuration.

Base URL: `https://apiv2.sdk.nubarium.com`.

## Generate a token

Send Nubarium account credentials with HTTP Basic authentication:

```http
POST /identity/v2/tokens/generate
Authorization: Basic <base64(username:password)>
Content-Type: application/json

{"accessProfile":"control"}
```

The JSON body must select one access profile:

| Profile | Use | Where it stays |
|---|---|---|
| `execution` | Run FaceCapture or IdCapture | Your backend obtains it and passes only this short-lived JWT to the browser |
| `api` | Operate processes, executions, results, resources, and preservation | Your backend |
| `control` | Manage origins, configurations, and storage | Your backend or administrative tool |
| `backend` | Combine `api` and `control` capabilities when one service needs both | A trusted backend only |

For an SDK session, send the same request with `{"accessProfile":"execution"}`. A `200 OK` response for the `control` request above has this shape (all values below are illustrative):

```json
{
  "tokenType": "Bearer",
  "accessToken": "<redacted-JWT>",
  "expiresIn": 900,
  "exp": 1787302800
}
```

`Bearer` is the authorization scheme identified by `tokenType`. Use `accessToken` as the credential. `expiresIn` is the lifetime in seconds from issuance and `exp` is the absolute Unix expiration time in seconds. The signed JWT contains its `tenantId`, `jti`, profile, scopes, and the same `exp`; do not trust decoded claims without first verifying its signature, audience, and validity.

The response includes an `X-Nubarium-Request-Id` header for operational correlation. Token issuance can also return `400` (invalid profile), `401` (invalid Basic credentials), `429` (rate limit), or `503` (temporary service failure). Errors contain `requestId` and `error: { code, message, retriable }`. Do not log credentials or JWTs.

Your application can expose its own authenticated session route to deliver the `execution` token to the browser. `/api/my-middleware/session` in the [quickstart](02-quickstart.md#2-request-a-session) is an example route in **your application**, not a Nubarium endpoint. Never send the `control` token or permanent credentials to that route's browser response.

## Register an origin

Use a `control` token belonging to the same account that will issue the SDK's `execution` tokens. The account must have permission to manage origins (`origins:write`).

```http
POST /control/v2/origins
Authorization: Bearer <control-accessToken>
Content-Type: application/json

{"origin":"https://app.example.com"}
```

`origin` is the exact scheme, hostname, and port, if present. It does not include a path, wildcard, or trailing page name. For `https://app.example.com/capture`, register `https://app.example.com`. Subdomains and other ports are separate origins. APIv2 determines the account and environment from the authenticated request; do not add them to the body.

A successful `200 OK` response contains the origin record and a publication result. Example (illustrative values):

```json
{
  "contractVersion": "nubarium.runtime-origin/2.0",
  "status": "OK",
  "requestId": "origin-request-example",
  "data": {
    "origin": {
      "originId": "origin-example",
      "origin": "https://app.example.com",
      "environment": "production",
      "status": "active",
      "verification": { "state": "verified" },
      "createdAt": "2026-09-18T10:00:00.000Z",
      "updatedAt": "2026-09-18T10:00:00.000Z",
      "disabledAt": null
    },
    "publication": {
      "generation": "generation-example",
      "generatedAt": "2026-09-18T10:00:00.000Z",
      "originCount": 1,
      "bindingCount": 1,
      "changed": true
    }
  }
}
```

`data.publication` can be `null` when publication details are unavailable. Check that the returned origin matches the one requested and is `active` before running the SDK. You can list the account's origins with `GET /control/v2/origins` using a server-side token with `origins:read`. The registration contract also defines `400`, `401`, `403`, `404`, `409`, `429`, and `503` error responses. A `403` can mean the account or token lacks `origins:write`; an `execution` token cannot be used to register origins.

This registration authorizes the web origin for the SDK runtime. It is not a browser-side CORS setting. If your account allows HTTP `localhost` during development, register that exact development origin separately; production origins use HTTPS.

Next: [check the other prerequisites](01-prerequisites.md) or [run a capture](02-quickstart.md).
