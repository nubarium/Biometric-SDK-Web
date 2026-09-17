# Quickstart

[Leer en español](../es/docs/02-quickstart.md)

This page runs a minimal `FaceCapture` flow. `IdCapture` uses the same lifecycle.

## Choose the SDK files

These files are from the **same Web SDK v2**. Use modular files **or** the optional all-in-one bundle.

| Use case | Scripts, in order | Available class |
|---|---|---|
| Face liveness | `nubsdk-core.min.js` → `nubsdk-face.min.js` | `FaceCapture` |
| ID capture | `nubsdk-core.min.js` → `nubsdk-id.min.js` | `IdCapture` |
| Face and ID | `nubsdk-core.min.js` → `nubsdk-face.min.js` → `nubsdk-id.min.js` | Both |
| Face and ID in one file | `nubsdk-all.min.js` | Both, plus modules in the bundle |

Core alone does not provide FaceCapture or IdCapture. Load component modules after Core from the same channel and version. The `all` bundle already includes Core: do not load modular files alongside it. A modular `nubsdk-all-components.min.js` is planned but **not published on `v2 stable`**; do not use its stable URL yet.

For **ID only**:

```html
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-id.min.js"></script>
```

Select the Mexican ID family before `init()`:

```javascript
const idCapture = new IdCapture();
idCapture.setDocumentFamily("MEX_IdCard");
```

This family requires front and back. Continue with the token, callbacks, and lifecycle below; [IdCapture](04-id-capture.md#minimal-setup) shows the full order.

To use the **all-in-one bundle** instead of the two Face scripts below, load only:

```html
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-all.min.js"></script>
```

For `presentId` on the current stable channel, use `nubsdk-all.min.js`; the minimal Face module does not include this add-on. See [Optional add-ons](05-addons.md).

> [!NOTE] React, Vue, and Next.js use the same CDN files. An optional Web package, `@nubarium/biometrics`, exposes `/face` and `/id` imports. The planned `/all-components` import is not needed for CDN integration.

## 1. Add the HTML

```html
<button id="start-capture" type="button">Start capture</button>
<p id="capture-status" role="status"></p>
<div id="face-component"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
<script type="module" src="/capture.js"></script>
```

## 2. Request a session

`/api/my-middleware/session` is a route in **your backend**, not a Nubarium API. Replace it with your own route.

```javascript
async function getSessionToken() {
  const response = await fetch("/api/my-middleware/session", {
    method: "POST",
    credentials: "same-origin"
  });

  if (!response.ok) throw new Error("sdk_session_unavailable");
  const session = await response.json();
  if (typeof session.token !== "string" || !session.token) {
    throw new Error("sdk_session_unavailable");
  }
  return session.token;
}
```

## 3. Handle terminal callbacks

```javascript
let capture;
const button = document.querySelector("#start-capture");
const status = document.querySelector("#capture-status");

function finish(message) {
  status.textContent = message;
  button.disabled = false;
}

function handleSuccess(_payload) {
  finish("Capture completed.");
}

function handleFail(_payload) {
  finish("The evaluation was not approved. You can try again.");
}

function handleError(_error) {
  finish("Capture could not be completed. Try again.");
}
```

## 4. Start after a user action

```javascript
button.addEventListener("click", async () => {
  if (button.disabled) return;
  button.disabled = true;
  status.textContent = "Preparing capture…";

  try {
    const token = await getSessionToken();
    capture?.clear();
    const nextCapture = new FaceCapture();
    capture = nextCapture;
    nextCapture.setToken(token);
    nextCapture.onSuccess(handleSuccess);
    nextCapture.onFail(handleFail);
    nextCapture.onError(handleError);
    nextCapture.init({ rootElement: "face-component" });
    nextCapture.load(() => {
      if (capture === nextCapture) {
        nextCapture.start().catch(() => {
          if (capture === nextCapture && button.disabled) {
            finish("Capture could not be opened. Try again.");
          }
        });
      }
    });
  } catch (_error) {
    finish("The session could not be started. Try again.");
  }
});
```

There is no `intro` option in the normal setup. FaceCapture shows the `liveness` intro by default; IdCapture shows `id-capture`. Use `intro: false` only if you want to skip the intro. An explicit configuration or profile can change that behavior.

## 5. Clean up

```javascript
window.addEventListener("pagehide", () => {
  const current = capture;
  capture = null;
  current?.clear();
});
```

In a single-page application, also call `clear()` when the page component unmounts.

## Which callback to use

| Callback | Meaning | Typical action |
|---|---|---|
| `onSuccess` | Approved evaluation | Continue your application flow |
| `onFail` | Negative evaluation | Explain the result or offer a retry |
| `onError` | Technical problem | Log `code`, show recovery, and decide whether to retry |

Use structured result fields for program logic. Descriptive text may change as localization improves.

Next: [FaceCapture →](03-face-capture.md)
