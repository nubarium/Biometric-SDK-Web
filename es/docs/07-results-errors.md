# Resultados y errores

[Read in English](../../docs/07-results-errors.md)

Trata evaluación funcional y problema técnico como rutas distintas desde el primer ejemplo.

## Los tres callbacks terminales

| Callback | Qué comunica | No significa |
|---|---|---|
| `onSuccess` | El componente terminó con evaluación aprobada | Que todos los campos opcionales existan |
| `onFail` | El componente terminó con evaluación negativa | Que la cámara o la red hayan fallado |
| `onError` | La ejecución encontró un problema técnico | Que la evaluación biométrica sea negativa |

```javascript
capture.onSuccess((payload) => {
  const evaluation = payload.result?.evaluation;
  const executionId = payload.componentExecutionId ?? payload.id;
  // Continúa el flujo de tu aplicación con evaluation y executionId.
});

capture.onFail((payload) => {
  const reason = payload.reason ?? payload.result?.retro?.[0] ?? "unknown";
  // Usa reason para elegir un mensaje propio en el idioma del usuario.
});

capture.onError((error) => {
  const code = error.code ?? "internal_error";
  // Usa code para recuperación y registra requestId si está disponible.
});
```

`onFail` entrega una evaluación con `result`, `reason` y, cuando aplica, `reasonText`. No lo proceses como un error `{ code, msg }`; esa forma corresponde a `onError`. `reasonText` es descriptivo para integradores, mientras que el texto para el usuario debe seguir el idioma de la experiencia.

Estas son formas ilustrativas de los dos resultados no aprobados. Los identificadores y campos opcionales dependen de la ejecución:

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
  "msg": "Esta sesión no puede continuar. Contacta al administrador del servicio.",
  "messageKey": "ERROR_PUBLIC_ACCESS_DENIED",
  "reason": "origin_not_registered",
  "reasonText": "Hostname not registered",
  "retriable": false,
  "status": 403
}
```

El primer objeto corresponde a `onFail` y conserva la evaluación funcional. Su `score` es ilustrativo: puede variar o ser `null`; no lo uses para decidir qué callback ocurrió. El segundo corresponde a `onError`: no tiene `result.evaluation` porque no representa una evaluación biométrica terminada.

## Salida elegida por el usuario

Puedes habilitar la navegación del componente y escuchar `onExit`:

```javascript
capture.onExit((event) => {
  // event.source indica si eligió Atrás, la X o una salida solicitada por el host.
  // Tu aplicación decide si cambia de ruta o permite comenzar de nuevo.
});
capture.init({ rootElement: "face-component", navigation: { enabled: true } });
```

`onExit` representa una cancelación elegida por el usuario; no es `onFail` ni `onError`. Llamar `clear()` desde tu aplicación, incluso tras un resultado terminal, libera recursos sin emitir `onExit`. Si el host recibe Atrás del router o del dispositivo, entrégalo a `requestBack()` y usa su decisión; el SDK no modifica el historial del navegador por su cuenta.

## Campos base

El contrato puede incluir:

- `contractVersion`: versión del formato de respuesta;
- `componentExecutionId`: identificador del intento;
- `validationCode`: código de correlación cuando está disponible;
- `result.evaluation`: evaluación funcional;
- `result.score`: puntuación cuando el componente la entrega;
- `result.retro`: retroalimentación estructurada;
- `result.addons`: resultados de tareas opcionales;
- `reason`: motivo público de un `onFail`, si existe;
- `resources`: recursos inline opcionales.

## Lee con tolerancia

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

No uses la presencia de `score` como única señal de terminación y no asumas que `resources` existe.

## Errores accionables

Usa el `code` público de `onError` para la lógica y `msg` como mensaje localizado cuando corresponda. Conserva `requestId` en telemetría si está presente. Los códigos internos del transporte pueden diferir de los que recibe el callback.

```javascript
function recoveryMessage(error) {
  if (error.code === "access_denied" && error.reason === "origin_not_registered") {
    return "Este dominio todavía no está habilitado para la captura.";
  }
  if (error.code === "authentication_failed") {
    return "La sesión venció. Inicia una nueva captura.";
  }
  if (error.code === "access_denied") {
    return "Esta sesión no tiene permiso para continuar. Contacta al administrador.";
  }
  return "No pudimos completar la captura. Intenta de nuevo o contacta a soporte.";
}
```

`access_denied` puede tener distintas causas; consulta `reason` antes de indicar que falta registrar el origen. `sdk_session_unavailable` pertenece al ejemplo de tu backend y se atiende al solicitar el token, antes de crear el SDK.

> [!IMPORTANT] No muestres secretos, tokens completos, imágenes o datos sensibles dentro de logs del navegador.

## Reintentos

- Renueva el token cuando haya expirado.
- Ejecuta `clear()` antes de crear una instancia nueva.
- Evita ciclos automáticos sin límite.
- Conserva la causa original en telemetría.
- Pide una nueva acción del usuario antes de reabrir cámara.

---

Siguiente: [monta el SDK en React o Vue →](08-frameworks.md)
