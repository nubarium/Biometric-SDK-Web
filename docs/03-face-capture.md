# FaceCapture

[Leer en español](../es/docs/03-face-capture.md)

`FaceCapture` handles camera access, capture guidance, messages, and face-liveness evaluation. Load `core + face` or the optional `nubsdk-all.min.js` bundle; both expose the same class. See [Choose the SDK files](02-quickstart.md#choose-the-sdk-files).

## Minimal setup

```javascript
const capture = new FaceCapture();

capture.setToken(token);
capture.onSuccess(handleSuccess);
capture.onFail(handleFail);
capture.onError(handleError);

capture.init({ rootElement: "face-component" });
capture.load(() => capture.start());
```

The `liveness` intro is shown by default on `v2 stable`. The `intro` option is not required; `intro: false` skips it. Language and appearance options are also optional. A profile or remote configuration can set a different policy.

## Approved result

The exact shape depends on configuration and version. `resources` is optional.

```json
{
  "contractVersion": "nubarium.component-result/2.0",
  "id": "component-execution-id",
  "componentExecutionId": "component-execution-id",
  "validationCode": "validation-code",
  "result": {
    "evaluation": "pass",
    "score": 99.58,
    "retro": [],
    "addons": []
  },
  "resources": {
    "face": "BASE64",
    "frame": "BASE64"
  }
}
```

Check each resource key before decoding or uploading it; the `resources` object may be absent.

## Available add-ons

| Key | Purpose | Typical input |
|---|---|---|
| `signature` | Handwritten signature | Statement shown to the signer |
| `speechValidation` | Spoken phrase capture and validation | Expected phrase |
| `presentId` | ID presentation during capture | Account-enabled configuration |
| `faceMatch` | Comparison with a reference | Authorized reference |

Each add-on has its own evaluation. See [Optional add-ons](05-addons.md) for ordering and required tasks.

## Restart and cleanup

- Call `clear()` before creating another instance.
- Do not reuse an instance after unmounting its container.
- Request a valid token for each session according to your backend policy.
- Do not treat a camera error as a negative evaluation.

Use [IdCapture](04-id-capture.md) when the main task is to capture one or more sides of an ID. If your flow needs both face and ID, keep their results identifiable and tell the user the order of tasks.

Next: [IdCapture →](04-id-capture.md)
