# JavaScript, React, Vue, and Next.js

[Leer en español](../es/docs/08-frameworks.md)

All four environments use the same Web SDK distribution. Choose modular files or the optional `nubsdk-all.min.js` bundle first. The framework only changes script loading, container mounting, and when to call `clear()`.

Before using these examples, [set up the origin and token](01-prerequisites.md). The browser receives only a short-lived JWT from your backend, never permanent credentials.

| Environment | Mount | Cleanup |
|---|---|---|
| Plain JavaScript | After creating the HTML element | Before leaving the page or removing the element |
| React | `useEffect` after render | Effect cleanup function |
| Vue | `onMounted` | `onBeforeUnmount` |
| Next.js App Router | Client Component after Core and Face load | React component cleanup |

## Plain JavaScript

Load Core **before** Face. Obtain the token from your backend after a user action:

```html
<button id="start-capture" type="button">Start capture</button>
<div id="face-root"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
<script src="/capture.js"></script>
```

In `/capture.js`, use the [quickstart](02-quickstart.md) code and change `rootElement: "face-component"` to `rootElement: "face-root"`. That example handles duplicate clicks, token retrieval, and cleanup. You can also use `face-component` as the ID in both places.

## React

Load the two CDN scripts once before mounting this component. Keep callbacks stable with `useCallback` and obtain a valid token from your backend.

```jsx
import { useEffect } from "react";

export function FaceCaptureView({ token, onSuccess, onFail, onError }) {
  useEffect(() => {
    if (!token || typeof window.FaceCapture !== "function") return;

    let disposed = false;
    const capture = new window.FaceCapture();
    capture.setToken(token);
    capture.onSuccess(onSuccess);
    capture.onFail(onFail);
    capture.onError(onError);
    capture.init({ rootElement: "react-face-root" });
    capture.load(() => {
      if (!disposed) capture.start();
    });

    return () => {
      disposed = true;
      capture.clear();
    };
  }, [token, onSuccess, onFail, onError]);

  return <div id="react-face-root" />;
}
```

### React checks

- Load SDK scripts only once per document.
- Stabilize callbacks with `useCallback` or avoid recreating them every render.
- Do not change the container's `key` during capture.
- Create SDK instances only on the client in SSR apps.
- React Strict Mode may mount and clean up an effect more than once in development; `disposed` prevents a late load from reopening an old instance.
- Use a distinct container ID for each simultaneous capture.

## Vue

```vue
<script setup>
import { onBeforeUnmount, onMounted } from "vue";

const props = defineProps({
  token: { type: String, required: true },
  onSuccess: { type: Function, required: true },
  onFail: { type: Function, required: true },
  onError: { type: Function, required: true }
});
let capture = null;
let disposed = false;

onMounted(() => {
  capture = new window.FaceCapture();
  capture.setToken(props.token);
  capture.onSuccess(props.onSuccess);
  capture.onFail(props.onFail);
  capture.onError(props.onError);
  capture.init({ rootElement: "vue-face-root" });
  capture.load(() => {
    if (!disposed) capture.start();
  });
});

onBeforeUnmount(() => {
  disposed = true;
  capture?.clear();
  capture = null;
});
</script>

<template>
  <div id="vue-face-root" />
</template>
```

### Vue checks

- Mount only after the token and CDN scripts are ready.
- Do not remove an active camera container with `v-if`.
- Call `clear()` in `onBeforeUnmount`.
- With `KeepAlive`, define cleanup on deactivation; route changes may not unmount the component.
- In Nuxt, mount the SDK in a client-only component.
- Keep permanent credentials on your backend.

## Next.js App Router

Camera access and `window` require a **Client Component**. Load Core before Face with `next/script`; once both are ready, use `FaceCaptureView` from the React example. Obtain `token` from your backend route.

```jsx
"use client";

import { useState } from "react";
import Script from "next/script";
import { FaceCaptureView } from "./FaceCaptureView";

const cdn = "https://cdn.nubarium.com/nubSdk/v2/stable/js";

export function NextFaceCapture({ token, onSuccess, onFail, onError }) {
  const [coreReady, setCoreReady] = useState(false);
  const [faceReady, setFaceReady] = useState(false);

  return (
    <>
      <Script src={`${cdn}/nubsdk-core.min.js`} onReady={() => setCoreReady(true)} />
      {coreReady && (
        <Script src={`${cdn}/nubsdk-face.min.js`} onReady={() => setFaceReady(true)} />
      )}
      {faceReady && token ? (
        <FaceCaptureView
          token={token}
          onSuccess={onSuccess}
          onFail={onFail}
          onError={onError}
        />
      ) : (
        <p>Preparing capture…</p>
      )}
    </>
  );
}
```

Do not create the SDK in a Server Component or put credentials in `NEXT_PUBLIC_*` variables. Define callbacks in a Client Component; functions cannot be passed from a Server Component as props. The Pages Router follows the same principles: ordered scripts, mounted container, and `clear()` during cleanup.

## Shared lifecycle

1. The host mounts a visible container.
2. Your backend supplies a short-lived token.
3. Load Core before Face or ID, or load only `nubsdk-all.min.js`.
4. Register token and callbacks, call `init()`, then `load()` and `start()`.
5. Call `clear()` before removing the container.

For ID with modular files, replace Face with `nubsdk-id.min.js`, use `IdCapture`, and select a document family. With the bundle, keep the one script and change only the class and configuration. See [IdCapture](04-id-capture.md).

If you enable `navigation: { enabled: true }`, pass your router or container's Back event to `capture.requestBack()`. The component decides whether to return to its intro, offer recapture, or request exit according to the current task. Your host changes route after `onExit`; do not also let the same gesture navigate the router. Calling `clear()` during unmount does not emit `onExit`. See [User-selected exit](07-results-errors.md#user-selected-exit).

Next: [Troubleshooting →](09-troubleshooting.md)
