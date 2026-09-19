# Prerequisites

[Leer en español](../es/docs/01-prerequisites.md)

Set up the allowed origin, SDK files, container, and short-lived token before loading a component. The [quickstart](02-quickstart.md) shows the full capture sequence.

## HTTPS and exact origin

Serve the application over HTTPS. HTTP `localhost` can be used for development if your account permits it. Register the exact origin where the SDK runs: scheme, hostname, and port must match.

| Application URL | Origin to register |
|---|---|
| `https://onboarding.example.com/start` | `https://onboarding.example.com` |
| `https://example.com:8443/capture` | `https://example.com:8443` |
| `http://localhost:3000` for development | `http://localhost:3000`, if permitted |

An origin does not automatically include its subdomains or another port.

Register the origin from your backend or an administrative tool using a `control` token. The request body and endpoint are in [Tokens and allowed origins](access-and-origins.md#register-an-origin).

## Token from your backend

Your server requests a token with the `execution` access profile and gives only its `accessToken` to the browser. Your Nubarium credentials and `control` tokens remain on the server. See [Tokens and allowed origins](access-and-origins.md#generate-a-token) for the API request and body; the [quickstart](02-quickstart.md#2-request-a-session) shows how browser code obtains the session from your own backend.

> [!WARNING] Do not put permanent credentials or higher-privilege tokens in `localStorage`, globals, HTML, or client bundles.

## Script order and container

Choose one distribution; do not mix them on one page:

- **Modular files:** load `nubsdk-core.min.js` before `nubsdk-face.min.js` or `nubsdk-id.min.js`.
- **Optional bundle:** load only `nubsdk-all.min.js`; it already includes Core, Face, and ID.

The [quickstart](02-quickstart.md#choose-the-sdk-files) lists the files and availability of the planned `all-components` module.

The container must exist and stay mounted throughout capture. For FaceCapture with modular files:

```html
<main>
  <div id="capture-root" aria-live="polite"></div>
</main>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
```

The selected distribution resolves its standard styles, templates, intros, and models. Supply custom asset paths only when your configuration requires them.

## Camera and page lifecycle

- Request camera access after a clear user action.
- Keep the container mounted while the instance is active.
- Call `clear()` before removing the container or changing routes.
- Do not start two captures in the same element.
- Give the user a way to recover after denying permission.

## Checklist

- [ ] The page is served over HTTPS.
- [ ] The exact origin is registered.
- [ ] Your backend returns a short-lived JWT.
- [ ] Core precedes Face or ID, or you load only `nubsdk-all.min.js`.
- [ ] The container exists before `init()`.
- [ ] The route calls `clear()` before unmounting.

Next: [run your first capture →](02-quickstart.md)
