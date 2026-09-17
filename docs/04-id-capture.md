# IdCapture

[Leer en español](../es/docs/04-id-capture.md)

`IdCapture` captures the sides required by a document family and returns one terminal result. Load `core + id` or the optional `nubsdk-all.min.js` bundle; both expose the same class. See [Choose the SDK files](02-quickstart.md#choose-the-sdk-files).

## Minimal setup

```javascript
const idCapture = new IdCapture();

idCapture.setToken(token);
idCapture.setDocumentFamily("MEX_IdCard");
idCapture.onSuccess(handleDocument);
idCapture.onFail(handleRejectedDocument);
idCapture.onError(handleTechnicalError);

idCapture.init({ rootElement: "id-component" });
idCapture.load(() => idCapture.start());
```

`MEX_IdCard` requires front and back. An unsupported family should fail before the camera opens. The `id-capture` intro is shown by default on `v2 stable`; no `intro` setting is needed. Pass `intro: false` to skip it. A profile or explicit configuration can set a different policy.

## Document family

The family determines sides, guides, and available rules. Use a family confirmed for your account and version; do not derive a key from a document's commercial name.

Set the family before `init()`:

```javascript
idCapture.setDocumentFamily("MEX_IdCard");
```

Or inside `init()` where that form is enabled:

```javascript
idCapture.init({
  rootElement: "id-component",
  document: { documentFamily: "MEX_IdCard" }
});
```

## Optional OCR task

```javascript
idCapture.init({
  rootElement: "id-component",
  document: { documentFamily: "MEX_IdCard" },
  addons: [{ key: "documentOcr", after: "back", required: true }]
});
```

The OCR result is in `result.addons`. Images still belong to the component's base result.

## Base result

```json
{
  "contractVersion": "nubarium.component-result/2.0",
  "componentExecutionId": "component-execution-id",
  "validationCode": "validation-code",
  "result": {
    "evaluation": "pass",
    "score": 100,
    "addons": []
  },
  "resources": {
    "front": "BASE64",
    "back": "BASE64"
  }
}
```

An empty `addons` array means no optional tasks ran. `resources` may be absent when the response does not include inline bytes.

## User flow

- Explain how many sides will be captured before requesting camera access.
- Label front and back consistently.
- Preserve progress between sides.
- Distinguish a rejected ID from an unavailable camera.
- Keep the container mounted until a terminal callback or `clear()`.

Next: [Optional add-ons →](05-addons.md)
