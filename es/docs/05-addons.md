# Capacidades opcionales

[Read in English](../../docs/05-addons.md)

Las capacidades opcionales se ejecutan dentro de `FaceCapture` o `IdCapture` y conservan su propio resultado. Revisa primero los archivos que cargarás: el bundle opcional `nubsdk-all.min.js` incluye los componentes principales; si cargas módulos, algunos add-ons requieren otro archivo. No combines el bundle con módulos duplicados.

## Compatibilidad por componente

| Componente | Capacidades documentadas |
|---|---|
| `FaceCapture` | `signature`, `speechValidation`, `presentId`, `faceMatch` |
| `IdCapture` | `documentOcr`, `faceMatch` |

Confirma la disponibilidad de cada clave para tu cuenta, plataforma y versión antes de depender de ella.

Para `presentId` en el canal `v2 stable` actual, usa el [bundle completo](02-quickstart.md#elige-los-archivos-del-sdk): `nubsdk-all.min.js`. El módulo `nubsdk-face.min.js` por sí solo no incluye `presentId`. En el siguiente corte podrás cargar `nubsdk-core.min.js` y después `nubsdk-all-components.min.js` como alternativa modular.

## Declara orden y obligatoriedad

```javascript
faceCapture.init({
  rootElement: "face-component",
  addons: [
    {
      key: "speechValidation",
      after: "face",
      required: true
    },
    {
      key: "signature",
      after: "speechValidation",
      required: true
    }
  ]
});
```

| Campo | Uso |
|---|---|
| `key` | Identifica la capacidad |
| `after` | Indica después de qué tarea se ejecuta |
| `required` | Si es `true`, su rechazo impide `onSuccess` |

## Entrega datos variables

`setAddonInput()` entrega información admitida por la capacidad antes de `load()`.

```javascript
faceCapture.setAddonInput("speechValidation", {
  phrase: "Autorizo esta operación"
});

faceCapture.setAddonInput("signature", {
  statement: "Confirmo que leí y acepto esta operación."
});
```

> [!WARNING] Los datos enviados desde el navegador no deben sustituir endpoints, encabezados, credenciales o políticas definidas fuera del cliente.

## Lee el resultado

El resultado principal permanece en `result`. Las tareas opcionales aparecen en `result.addons`; sus recursos inline, cuando existen, aparecen en `resources.addons`.

```javascript
function handleSuccess(payload) {
  const addonResults = payload.result?.addons ?? [];

  for (const addon of addonResults) {
    console.log(addon.key, addon.evaluation, addon.addonExecutionId);
  }
}
```

No reemplaces `componentExecutionId` con el identificador de una tarea. El primero identifica el intento completo; `addonExecutionId` identifica la tarea hija.

## Fallo de una tarea requerida

Cuando una tarea marcada como `required` no aprueba, el componente no debe emitir `onSuccess`. Atiende `onFail` y conserva suficiente contexto para explicar qué parte terminó con evaluación negativa sin presentar un error técnico.

---

Siguiente: [personaliza la experiencia →](06-customization.md)
