# FaceCapture

[Read in English](../../docs/03-face-capture.md)

`FaceCapture` controla la cámara, la guía de captura, los mensajes y la evaluación de prueba de vida facial. Puedes cargarlo con `core + face` o con el bundle opcional `nubsdk-all.min.js`; ambas formas exponen la misma clase. Consulta [Elige los archivos del SDK](02-quickstart.md#elige-los-archivos-del-sdk) antes de copiar el ejemplo.

## Configuración mínima

```javascript
const capture = new FaceCapture();

capture.setToken(token);
capture.onSuccess(handleSuccess);
capture.onFail(handleFail);
capture.onError(handleError);

capture.init({
  rootElement: "face-component",
  locale: "es",
  ui: { colorMode: "system" }
});

capture.load(() => capture.start());
```

Idioma y apariencia son opcionales. FaceCapture muestra por defecto su introducción `liveness` en `v2 stable`; usa `intro: false` sólo si deseas omitirla. Un perfil o configuración remota puede definir otra política.

## Resultado aprobado

La forma exacta depende de la configuración y de la versión. `resources` es opcional.

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

No asumas que `resources` siempre existe. Comprueba cada clave antes de decodificarla o subirla.

## Capacidades que pueden acompañarlo

| Clave | Propósito | Dato de entrada frecuente |
|---|---|---|
| `signature` | Captura manuscrita | declaración que se muestra al firmante |
| `speechValidation` | Captura y validación de frase | frase esperada |
| `presentId` | Presentación de identificación durante la toma | configuración permitida por la cuenta |
| `faceMatch` | Comparación con una referencia | referencia autorizada |

Cada capacidad conserva su propia evaluación. Consulta [Capacidades opcionales](05-addons.md) para configurar orden y obligatoriedad.

## Reinicio y limpieza

- Llama `clear()` antes de crear una instancia nueva.
- Evita reutilizar una instancia después de desmontar su contenedor.
- Solicita un token vigente para cada sesión según la política de tu backend.
- No interpretes un error de cámara como una evaluación negativa.

## Cuándo usar IdCapture

Usa `IdCapture` cuando el recorrido principal sea capturar uno o más lados de un documento. Si necesitas rostro y documento, mantén cada resultado identificable y explica al usuario el orden de pasos.

---

Siguiente: [integra IdCapture →](04-id-capture.md)
