# Customization

[Leer en español](../es/docs/06-customization.md)

Language, intro, and appearance can be set without changing the component lifecycle. Omit options you do not need.

## Language

```javascript
capture.init({ rootElement: "face-component", locale: "en" });
```

Use a locale included in the distribution or explicitly registered by your application. If you add a custom locale, provide every required key and a safe fallback.

## Intro

FaceCapture and IdCapture show their own introductions by default on `v2 stable`: `liveness` and `id-capture`, respectively. **The `intro` option is optional.** Leave it out to use the component default.

Only set `intro: false` when you want to skip the intro:

```javascript
capture.init({ rootElement: "face-component", intro: false });
```

A profile or remote configuration can override this behavior. Specify another intro key only if you have registered or chosen a different introduction.

## Appearance

```javascript
capture.init({
  rootElement: "face-component",
  ui: { colorMode: "system" }
});
```

`colorMode: "system"` follows the device preference when supported by the loaded version. Check contrast, focus, and messages in every mode you publish.

## Developer Hub preview

The [visual configurator](https://developer.nubarium.com/builder.html?section=theme) previews palette, surfaces, mask, typography, and intros. Its [FaceCapture](https://developer.nubarium.com/builder.html?section=face) and [IdCapture](https://developer.nubarium.com/builder.html?section=id) sections use the stable v2 SDK templates and dimensions for intro and capture guides. The preview does not open the camera or evaluate images; voice and signature are illustrative.

Check exported code and run a real capture on supported devices before publishing a configuration.

## Versioned remote configuration

You can set options in code or resolve a published `configurationId`:

```javascript
capture.init({
  rootElement: "face-component",
  remoteConfiguration: {
    configurationId: "cfg_face_capture_v3",
    authoritative: true
  }
});
```

The ID above is illustrative; replace it with a published value authorized for your account. You do not need a remote configuration to get started. Use one when you want a reusable, versioned configuration outside your application bundle.

## Configuration sources

Decide which values the client may change and which remain authoritative in the published configuration. Do not let a local option override a requirement of your integration.

| Source | Use |
|---|---|
| Defaults | Try the minimum lifecycle |
| `init()` options | Set screen-specific options |
| Published configuration | Reuse an identifiable version |
| `setAddonInput()` | Supply permitted variable task input |

## Visual checks

- [ ] Text fits on mobile and desktop.
- [ ] Keyboard focus is visible.
- [ ] Controls have at least a 44 px interactive area.
- [ ] Text contrast meets AA.
- [ ] The UI remains understandable with reduced motion.
- [ ] Messages explain the next action or recovery.

Next: [Results and errors →](07-results-errors.md)
