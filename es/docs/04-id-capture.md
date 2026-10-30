# IdCapture

[Read in English](../../docs/04-id-capture.md)

`IdCapture` captura los lados requeridos por una familia documental y entrega un único resultado terminal. Puedes cargarlo con `core + id` o con el bundle opcional `nubsdk-all.min.js`; ambas formas exponen la misma clase. Consulta [Elige los archivos del SDK](02-quickstart.md#elige-los-archivos-del-sdk) para escoger una sola distribución.

## Configuración mínima

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

`MEX_IdCard` requiere frente y reverso. Una familia no soportada debe fallar antes de abrir la cámara.

En `v2 stable`, IdCapture muestra por defecto su introducción `id-capture` cuando omites `intro`. Puedes desactivarla con `intro: false`; una configuración explícita o un perfil puede cambiar esta política.

## Familia documental

La familia determina lados, guías y reglas disponibles. No inventes una clave a partir del nombre comercial del documento: usa una familia confirmada para tu cuenta y versión.

Puedes declarar la familia antes de `init()`:

```javascript
idCapture.setDocumentFamily("MEX_IdCard");
```

O dentro de la configuración cuando esa forma esté habilitada:

```javascript
idCapture.init({
  rootElement: "id-component",
  document: { documentFamily: "MEX_IdCard" }
});
```

## OCR como capacidad opcional

```javascript
idCapture.init({
  rootElement: "id-component",
  document: { documentFamily: "MEX_IdCard" },
  addons: [
    {
      key: "documentOcr",
      after: "back",
      required: true
    }
  ]
});
```

El resultado OCR aparece en `result.addons`. Las imágenes continúan perteneciendo al resultado base del componente.

## Resultado base

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

Un arreglo `addons` vacío significa que no se ejecutaron tareas opcionales. `resources` puede omitirse cuando la respuesta no incluye bytes inline.

## Captura de pasaporte

Para capturar un pasaporte mexicano, selecciona `MEX_Passport`.
Se captura únicamente la página con fotografía, datos personales y las
dos líneas inferiores de lectura automática. No se solicita reverso.

Este ejemplo utiliza el canal `v2/latest`. Carga sus archivos CSS, Core
e ID desde el mismo canal:

```html
<link rel="stylesheet"
      href="https://cdn.nubarium.com/nubSdk/v2/latest/assets/css/nubSdk-biometrics.css">

<div id="passport-component"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/latest/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/latest/js/nubsdk-id.min.js"></script>
```

Reutiliza el token de ejecución obtenido por tu backend y tus manejadores
de resultado:

```javascript
const passport = new IdCapture();

passport.setToken(token);
passport.setDocumentFamily("MEX_Passport");

passport.onSuccess(handleDocument);
passport.onFail(handleRejectedDocument);
passport.onError(handleTechnicalError);

passport.init({
  rootElement: "passport-component"
});

passport.load(() => passport.start());
```

### Recibir también los datos OCR

Para incluir los datos leídos del pasaporte, sustituye únicamente
el bloque `init()` anterior por:

```javascript
passport.init({
  rootElement: "passport-component",
  addons: [{ key: "documentOcr", required: true }]
});
```

El resultado OCR se entrega en `payload.result.addons`.
No configures `after: "back"`: el pasaporte sólo tiene una captura.
La introducción del componente se muestra por defecto.

Al abandonar la pantalla, ejecuta `passport.clear()` para liberar
la cámara y los recursos.

## Buenas prácticas de UX

- Explica cuántos lados se capturarán antes de solicitar cámara.
- Nombra frente y reverso de forma consistente.
- Conserva el progreso al pasar de un lado al siguiente.
- Distingue documento rechazado de cámara no disponible.
- No retires el contenedor hasta recibir un callback terminal o ejecutar `clear()`.

---

Siguiente: [agrega capacidades opcionales →](05-addons.md)
