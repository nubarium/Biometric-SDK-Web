# Reference

[Leer en español](../es/docs/10-reference-map.md)

Methods, callbacks, and terms used in this guide. For script selection, see [SDK files](02-quickstart.md#choose-the-sdk-files).

## Main lifecycle methods

| Method | When | Purpose |
|---|---|---|
| `setToken(token)` | Before `load()` | Passes the short-lived JWT to the instance |
| `onSuccess(callback)` | Before starting | Registers an approved result callback |
| `onFail(callback)` | Before starting | Registers a negative evaluation callback |
| `onError(callback)` | Before starting | Registers a technical error callback |
| `onExit(callback)` | Before starting | Reports a user-selected exit |
| `init(options)` | After callbacks | Sets container and options |
| `load(callback)` | After `init()` | Prepares resources, session, and permissions |
| `start()` | From the `load()` callback | Starts capture |
| `clear()` | On completion or unmount | Releases camera, listeners, and UI |
| `requestBack()` | When the host receives Back | Asks the SDK to resolve internal navigation; returns a decision |
| `requestExit()` | When the host offers Exit | Requests component-managed exit |

## Component methods

| Method | Component | Use |
|---|---|---|
| `setDocumentFamily("MEX_IdCard")` | `IdCapture` | Selects the example Mexican ID family (front and back) |
| `setAddonInput(key, value)` | Both, where supported | Supplies variable data before `load()` |

## `init()` options used here

| Option | Example | Purpose |
|---|---|---|
| `rootElement` | `"face-component"` | ID of an existing container |
| `locale` | `"en"` | Experience language |
| `intro` | `false` | Optional opt-out of the default FaceCapture or IdCapture intro |
| `intro.key` | Optional | Selects a different registered intro |
| `navigation.enabled` | `true` | Enables managed component navigation and exit |
| `ui.colorMode` | `"system"` | Chooses color mode |
| `document.documentFamily` | `"MEX_IdCard"` | Sets document family in `init()` |
| `addons` | `[{ key, after, required }]` | Declares optional tasks |
| `remoteConfiguration.configurationId` | `"cfg_face_capture_v3"` | Resolves a published configuration |

## Terms

**Component**  
A main executable unit, such as `FaceCapture` or `IdCapture`.

**Add-on**  
A child task within a component, with its own configuration and result.

**Evaluation**  
The functional outcome, distinct from a technical error.

**Origin**  
The exact scheme, hostname, and port from which the application runs.

**Execution token**  
A short-lived JWT issued by the backend for an SDK session.

**Inline resource**  
Content included directly in a result, such as an encoded image. It is optional.

**Document family**  
A key that defines the available sides, guides, and rules for a document type.

## Suggested reading order

1. [Prerequisites](01-prerequisites.md)
2. [Quickstart](02-quickstart.md)
3. [FaceCapture](03-face-capture.md) or [IdCapture](04-id-capture.md)
4. [Optional add-ons](05-addons.md)
5. [Customization](06-customization.md)
6. [Results and errors](07-results-errors.md)
7. [Frameworks](08-frameworks.md)
8. [Troubleshooting](09-troubleshooting.md)

Back to the [SDK README](../README.md).
