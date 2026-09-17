# Troubleshooting

[Leer en español](../es/docs/09-troubleshooting.md)

Diagnose the stage that failed: file loading, access, container, permissions, execution, or result.

## The constructor is missing

Symptom: `FaceCapture is not defined` or `IdCapture is not defined`.

- Load Core before Face or ID, or load only `nubsdk-all.min.js`.
- Check that script URLs respond without unexpected redirects.
- Allow the CDN origin in your Content Security Policy.
- Run your code only after scripts have loaded.
- Create instances in the browser, not during SSR.

```javascript
if (typeof window.FaceCapture !== "function") {
  throw new Error("sdk_face_bundle_unavailable");
}
```

## The origin is not allowed

Symptom: `onError` returns `code: "access_denied"` and `reason: "origin_not_registered"`. `runtime_origin_not_allowed` is an internal classification, not the public callback value.

```javascript
console.log(window.location.origin);
```

Compare this exact scheme, hostname, and port with the registered origin. Register it, then retry `load()` with a valid session.

## The camera does not open

Check HTTPS, browser permission, whether another app occupies the camera, whether the container is still mounted and visible, and whether another instance uses that container. Request permission after an explicit user action. If permission was denied, explain how to change it in browser settings.

## Capture opens twice

The page likely created two instances. Clean up before constructing another one:

```javascript
capture?.clear();
capture = new FaceCapture();
```

Stabilize React effect dependencies. In any SPA, call `clear()` before route changes or reconstruction.

## `onFail` is treated as an exception

`onFail` is a completed negative evaluation. Reserve `onError` for technical problems. Keep messages, telemetry, and retry rules distinct.

## The response has no resources

`resources` is optional:

```javascript
const faceImage = payload.resources?.face ?? null;
if (faceImage) useFaceImage(faceImage);
```

Use `result.evaluation` and the callback received to determine the outcome, not image presence.

## An add-on is missing

Check component compatibility, account and version access, a valid `after` task, whether `setAddonInput()` ran before `load()`, and whether you read `result.addons` rather than only `result`.

## Data for support

Record the SDK and browser versions, `window.location.origin`, component and document family, last lifecycle stage (`init`, `load`, or `start`), and available `code`, `componentExecutionId`, `validationCode`, or `requestId`. Include event time and time zone. Do not include secrets or biometric content.

Next: [Reference →](10-reference-map.md)
