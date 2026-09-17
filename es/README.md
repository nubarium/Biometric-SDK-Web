# Nubarium Web SDK

[Read in English](../README.md)

Esta documentación describe cómo integrar el Web SDK v2 en una aplicación web: cargar el archivo adecuado, obtener un token efímero, crear `FaceCapture` o `IdCapture` y ejecutar el ciclo `init → load → start`.

> [!IMPORTANT] Nunca incluyas usuario, contraseña ni llaves permanentes de Nubarium en HTML o JavaScript. Tu backend obtiene el JWT de corta vida y entrega al navegador únicamente ese token.

## Antes de empezar: elige la distribución

El SDK ofrece archivos modulares y un bundle opcional. Carga sólo los archivos que necesites y mantenlos en el mismo canal y versión.

| Necesitas | Carga |
|---|---|
| Prueba de vida | `nubsdk-core.min.js` + `nubsdk-face.min.js` |
| Captura de ID | `nubsdk-core.min.js` + `nubsdk-id.min.js` |
| Face e ID | `nubsdk-core.min.js` + `nubsdk-face.min.js` + `nubsdk-id.min.js` |
| Todos los componentes, por módulos (siguiente corte) | `nubsdk-core.min.js` + `nubsdk-all-components.min.js` |
| Face e ID desde un solo archivo | `nubsdk-all.min.js` (bundle opcional) |

No combines `nubsdk-all.min.js` con los archivos modulares. El archivo `nubsdk-all-components.min.js` reúne Face, ID y sus complementos sin duplicar Core, pero **todavía no está publicado en `v2 stable`**. Para usar `presentId` en el canal estable actual, carga `nubsdk-all.min.js`. Los nombres, el orden y las diferencias entre CDN y paquete Web están explicados en [Primera captura → Elige los archivos del SDK](docs/02-quickstart.md#elige-los-archivos-del-sdk).

## Qué contiene esta documentación

Esta documentación cubre la integración directa del Web SDK desde una aplicación web:

- preparación del origen, HTTPS y token de ejecución;
- `FaceCapture` para prueba de vida facial;
- `IdCapture` para captura de ID;
- capacidades opcionales de firma, voz, OCR y comparación;
- personalización de idioma, introducción y apariencia;
- interpretación de `onSuccess`, `onFail` y `onError`;
- salida elegida por el usuario mediante `onExit` y navegación opcional;
- montaje seguro en JavaScript puro, React, Vue y Next.js.

Esta edición documenta el canal `v2 stable`. Valida su disponibilidad para tu cuenta antes de publicar una integración.

La introducción aparece por defecto en FaceCapture e IdCapture. No hace falta configurar `intro` para mostrarla; usa `intro: false` sólo si quieres omitirla.

## Busca lo que necesitas

| Si necesitas… | Empieza en… | Después revisa… |
|---|---|---|
| Probar una primera captura | [Primera captura](docs/02-quickstart.md) | [Resultados y errores](docs/07-results-errors.md) |
| Elegir Core, Face, ID o el bundle completo | [Archivos del SDK](docs/02-quickstart.md#elige-los-archivos-del-sdk) | [Capacidades opcionales](docs/05-addons.md) |
| Integrar prueba de vida facial | [FaceCapture](docs/03-face-capture.md) | [Capacidades opcionales](docs/05-addons.md) |
| Capturar una identificación | [IdCapture](docs/04-id-capture.md) | [Resultados y errores](docs/07-results-errors.md) |
| Adaptar idioma y apariencia | [Personalización](docs/06-customization.md) | [Mapa de referencia](docs/10-reference-map.md) |
| Montar el SDK en tu aplicación | [JavaScript, React, Vue y Next.js](docs/08-frameworks.md) | [Solución de problemas](docs/09-troubleshooting.md) |
| Diagnosticar un fallo | [Solución de problemas](docs/09-troubleshooting.md) | [Mapa de referencia](docs/10-reference-map.md) |

## Requisitos mínimos

Antes de instanciar un componente necesitas:

- una página servida mediante HTTPS;
- el origen exacto registrado para la cuenta;
- un contenedor visible y vacío para el componente;
- Core cargado antes del archivo modular del componente, o el bundle opcional `nubsdk-all.min.js`;
- un JWT de corta vida obtenido por tu backend.

Consulta [Requisitos](docs/01-prerequisites.md) para ver el flujo completo.

## Ejemplo mínimo de FaceCapture

`token` es el JWT efímero que entrega tu backend según [Requisitos](docs/01-prerequisites.md). Este fragmento muestra la instancia; el ejemplo con botón y control de errores está en [Primera captura](docs/02-quickstart.md).

```html
<div id="face-component"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
```

```javascript
const capture = new FaceCapture();

capture.setToken(token);
capture.onSuccess((payload) => console.info("Captura aprobada", payload.componentExecutionId ?? payload.id));
capture.onFail((payload) => console.info("Captura no aprobada", payload.reason));
capture.onError((error) => console.error("Error de captura", error.code, error.requestId));

capture.init({ rootElement: "face-component" });
capture.load(() => capture.start());
```

## Ciclo de vida

1. `setToken(token)` entrega el JWT a la instancia.
2. Los callbacks se registran antes de cargar.
3. `init(options)` define contenedor y configuración.
4. `load(callback)` prepara recursos y permisos.
5. `start()` abre la experiencia de captura.
6. `clear()` libera cámara, listeners y UI al desmontar.

`onFail` y `onError` no son equivalentes: el primero comunica una evaluación funcional negativa; el segundo comunica un problema técnico.
Si habilitas navegación, `onExit` comunica una salida elegida por el usuario. Una llamada de tu aplicación a `clear()` sólo limpia la instancia.

## Mapa del repositorio

- [Requisitos](docs/01-prerequisites.md)
- [Primera captura](docs/02-quickstart.md)
- [FaceCapture](docs/03-face-capture.md)
- [IdCapture](docs/04-id-capture.md)
- [Capacidades opcionales](docs/05-addons.md)
- [Personalización](docs/06-customization.md)
- [Resultados y errores](docs/07-results-errors.md)
- [JavaScript, React, Vue y Next.js](docs/08-frameworks.md)
- [Solución de problemas](docs/09-troubleshooting.md)
- [Mapa de referencia](docs/10-reference-map.md)

---

Siguiente: [prepara acceso, origen y contenedor →](docs/01-prerequisites.md)
