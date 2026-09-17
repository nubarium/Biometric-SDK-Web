# Results and errors

[Leer en español](../es/docs/07-results-errors.md)

Handle a completed negative evaluation separately from a technical problem.

## Terminal callbacks

| Callback | Meaning | Does not mean |
|---|---|---|
| `onSuccess` | Approved evaluation | Every optional field is present |
| `onFail` | Negative evaluation | Camera or network failure |
| `onError` | Technical problem | Negative biometric evaluation |

```javascript
capture.onSuccess((payload) => {
  const evaluation = payload.result?.evaluation;
  const executionId = payload.componentExecutionId ?? payload.id;
  // Continue your application's flow with evaluation and executionId.
});

capture.onFail((payload) => {
  const reason = payload.reason ?? payload.result?.retro?.[0] ?? "unknown";
  // Choose a user-facing message for your current locale.
});

capture.onError((error) => {
  const code = error.code ?? "internal_error";
  // Use code for recovery; log requestId when available.
});
```

`onFail` supplies an evaluation with `result`, `reason`, and sometimes `reasonText`. It is not an error shaped like `{ code, msg }`; that belongs to `onError`. `reasonText` is descriptive text for integrators, while user-facing text should follow the experience locale.

The following examples show a failed evaluation and a technical error. IDs and optional fields vary by execution:

```json
{
  "id": "component-execution-id",
  "componentExecutionId": "component-execution-id",
  "reason": "face_evidence_incomplete",
  "result": {
    "evaluation": "fail",
    "score": 0,
    "retro": ["face_evidence_incomplete"],
    "addons": []
  }
}
```

```json
{
  "code": "access_denied",
  "msg": "This session cannot continue. Contact the service administrator.",
  "messageKey": "ERROR_PUBLIC_ACCESS_DENIED",
  "reason": "origin_not_registered",
  "reasonText": "Hostname not registered",
  "retriable": false,
  "status": 403
}
```

The first object belongs to `onFail`. Its score is illustrative and may differ or be `null`; do not use score to decide which callback fired. The second belongs to `onError` and has no `result.evaluation` because no biometric evaluation completed.

## User-selected exit

Enable component navigation and listen for `onExit` when the host needs to react to a user-selected exit:

```javascript
capture.onExit((event) => {
  // event.source identifies Back, the X, or a host-requested exit.
  // Your application decides whether to change route or offer another attempt.
});
capture.init({ rootElement: "face-component", navigation: { enabled: true } });
```

`onExit` is neither `onFail` nor `onError`. Calling `clear()` yourself, even after a terminal result, releases resources without emitting `onExit`. If the host receives Back from its router or device, pass it to `requestBack()` and use its decision; the SDK does not change browser history itself.

## Base fields

- `contractVersion`: result format version.
- `componentExecutionId`: complete attempt ID.
- `validationCode`: correlation code, when available.
- `result.evaluation`: functional outcome.
- `result.score`: score, when provided.
- `result.retro`: structured feedback.
- `result.addons`: optional task results.
- `reason`: public reason for `onFail`, when provided.
- `resources`: optional inline resources.

## Read defensively

```javascript
function normalizeCapture(payload = {}) {
  return {
    executionId: payload.componentExecutionId ?? payload.id ?? null,
    evaluation: payload.result?.evaluation ?? "unknown",
    score: Number.isFinite(payload.result?.score) ? payload.result.score : null,
    feedback: Array.isArray(payload.result?.retro) ? payload.result.retro : [],
    addons: Array.isArray(payload.result?.addons) ? payload.result.addons : [],
    resources: payload.resources ?? null
  };
}
```

Do not infer completion from `score`, or assume `resources` exists.

## Actionable errors

Use the public `code` from `onError` for logic and the localized `msg` when appropriate. Record `requestId` in telemetry when present. Internal transport codes may differ from callback codes.

```javascript
function recoveryMessage(error) {
  if (error.code === "access_denied" && error.reason === "origin_not_registered") {
    return "This domain is not enabled for capture yet.";
  }
  if (error.code === "authentication_failed") {
    return "The session expired. Start a new capture.";
  }
  if (error.code === "access_denied") {
    return "This session is not allowed to continue. Contact the administrator.";
  }
  return "Capture could not be completed. Try again or contact support.";
}
```

`access_denied` has several causes; inspect `reason` before claiming the origin is unregistered. `sdk_session_unavailable` belongs to the example backend and is handled while requesting a token, before creating the SDK.

> [!IMPORTANT] Do not log secrets, full tokens, images, or sensitive data in the browser.

## Retries

- Renew expired tokens.
- Call `clear()` before creating another instance.
- Avoid unbounded automatic retries.
- Preserve the original cause in telemetry.
- Require another user action before reopening the camera.

Next: [Frameworks →](08-frameworks.md)
