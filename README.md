# Nubarium Web SDK

[Leer en español](es/README.md)

This guide covers Web SDK v2: which files to load, how to obtain a short-lived token, how to run `FaceCapture` or `IdCapture`, and how to handle the result. The component lifecycle is `init → load → start`.

> [!IMPORTANT] Keep your Nubarium username, password, and permanent keys out of browser code. Your backend obtains the short-lived JWT and sends only that token to the browser.

## Choose the files

Use one distribution from the same channel and version.

| You need | Load |
|---|---|
| Face liveness | `nubsdk-core.min.js` + `nubsdk-face.min.js` |
| ID capture | `nubsdk-core.min.js` + `nubsdk-id.min.js` |
| Face and ID | `nubsdk-core.min.js` + `nubsdk-face.min.js` + `nubsdk-id.min.js` |
| Face and ID in one file | `nubsdk-all.min.js` (optional bundle) |

Do not combine `nubsdk-all.min.js` with modular files. `nubsdk-all-components.min.js` is a planned modular option, but is **not available on `v2 stable`**. For `presentId` on the current stable channel, use `nubsdk-all.min.js`. See [Choose the SDK files](docs/02-quickstart.md#choose-the-sdk-files) for file order and available exports.

## Find a topic

| Task | Guide |
|---|---|
| Prepare HTTPS, origin, and token | [Prerequisites](docs/01-prerequisites.md) |
| Generate tokens and register an origin | [Tokens and allowed origins](docs/access-and-origins.md) |
| Run the first capture | [Quickstart](docs/02-quickstart.md) |
| Integrate a component | [FaceCapture](docs/03-face-capture.md) · [IdCapture](docs/04-id-capture.md) |
| Add signature, voice, OCR, or comparison | [Optional add-ons](docs/05-addons.md) |
| Set language, intro, or appearance | [Customization](docs/06-customization.md) |
| Handle callbacks and result data | [Results and errors](docs/07-results-errors.md) |
| Mount in JavaScript, React, Vue, or Next.js | [Frameworks](docs/08-frameworks.md) |
| Diagnose a problem | [Troubleshooting](docs/09-troubleshooting.md) · [Reference](docs/10-reference-map.md) |

These pages describe the `v2 stable` channel. Confirm access for your account before shipping.

## Minimum requirements

- An HTTPS page and its exact origin registered for your account.
- A visible, mounted container for the component.
- Core before the component module, or the optional `nubsdk-all.min.js` bundle alone.
- A short-lived JWT obtained by your backend.

See [Prerequisites](docs/01-prerequisites.md) for the checklist and [Tokens and allowed origins](docs/access-and-origins.md) for the API requests.

## Minimal FaceCapture example

`token` is the short-lived JWT from your backend. For a button, session request, and error handling, use the [quickstart](docs/02-quickstart.md).

```html
<div id="face-component"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
```

```javascript
const capture = new FaceCapture();

capture.setToken(token);
capture.onSuccess((payload) => console.info("Capture approved", payload.componentExecutionId ?? payload.id));
capture.onFail((payload) => console.info("Capture declined", payload.reason));
capture.onError((error) => console.error("Capture error", error.code, error.requestId));

capture.init({ rootElement: "face-component" });
capture.load(() => capture.start());
```

The intro is shown by default for both FaceCapture and IdCapture. You do not need an `intro` option in the normal setup. To skip it, pass `intro: false`; see [Customization](docs/06-customization.md#intro).

## Lifecycle

1. `setToken(token)` passes the JWT to the instance.
2. Register callbacks before loading.
3. `init(options)` sets the container and options.
4. `load(callback)` prepares resources and permissions.
5. `start()` opens the capture flow.
6. `clear()` releases camera, listeners, and UI before unmounting.

`onFail` is a completed negative evaluation; `onError` is a technical problem. With navigation enabled, `onExit` reports a user-selected exit. Calling `clear()` yourself only cleans up the instance.

## Guides

- [Prerequisites](docs/01-prerequisites.md)
- [Tokens and allowed origins](docs/access-and-origins.md)
- [Quickstart](docs/02-quickstart.md)
- [FaceCapture](docs/03-face-capture.md)
- [IdCapture](docs/04-id-capture.md)
- [Optional add-ons](docs/05-addons.md)
- [Customization](docs/06-customization.md)
- [Results and errors](docs/07-results-errors.md)
- [Frameworks](docs/08-frameworks.md)
- [Troubleshooting](docs/09-troubleshooting.md)
- [Reference](docs/10-reference-map.md)
