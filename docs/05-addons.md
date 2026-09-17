# Optional add-ons

[Leer en español](../es/docs/05-addons.md)

Add-ons run inside `FaceCapture` or `IdCapture` and retain their own results. Choose the SDK files first: some add-ons are not in the minimal component module. Do not combine `nubsdk-all.min.js` with duplicate modular files.

## Component compatibility

| Component | Documented add-ons |
|---|---|
| `FaceCapture` | `signature`, `speechValidation`, `presentId`, `faceMatch` |
| `IdCapture` | `documentOcr`, `faceMatch` |

Confirm that each key is available for your account, platform, and SDK version. For `presentId` on the current `v2 stable` channel, use the [all-in-one bundle](02-quickstart.md#choose-the-sdk-files), `nubsdk-all.min.js`. The minimal `nubsdk-face.min.js` module does not contain it. The planned `core + all-components` modular option is not yet published on stable.

## Set order and requirement

```javascript
faceCapture.init({
  rootElement: "face-component",
  addons: [
    { key: "speechValidation", after: "face", required: true },
    { key: "signature", after: "speechValidation", required: true }
  ]
});
```

| Field | Use |
|---|---|
| `key` | Identifies the add-on |
| `after` | Names the task it follows |
| `required` | If `true`, its rejection prevents `onSuccess` |

## Supply variable input

Use `setAddonInput()` before `load()` for data accepted by the add-on:

```javascript
faceCapture.setAddonInput("speechValidation", {
  phrase: "I authorize this operation"
});

faceCapture.setAddonInput("signature", {
  statement: "I confirm that I have read and accept this operation."
});
```

> [!WARNING] Browser-provided input does not replace server-defined endpoints, headers, credentials, or policy.

## Read the result

The main evaluation remains in `result`. Add-on results are in `result.addons`; inline add-on resources, when present, are in `resources.addons`.

```javascript
function handleSuccess(payload) {
  const addonResults = payload.result?.addons ?? [];
  for (const addon of addonResults) {
    console.log(addon.key, addon.evaluation, addon.addonExecutionId);
  }
}
```

`componentExecutionId` identifies the complete attempt. `addonExecutionId` identifies a child task; do not substitute one for the other. A required add-on that fails must prevent `onSuccess`; handle it through `onFail`, not as a technical exception.

Next: [Customization →](06-customization.md)
