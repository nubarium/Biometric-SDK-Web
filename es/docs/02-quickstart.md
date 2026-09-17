# Primera captura

[Read in English](../../docs/02-quickstart.md)

Esta página muestra el patrón mínimo de `FaceCapture`, que también utiliza `IdCapture`.

## Elige los archivos del SDK

Los archivos siguientes pertenecen al **mismo Web SDK v2**; no son versiones diferentes. Carga una combinación modular **o** el bundle completo:

| Quieres usar | Scripts, en este orden | Clase disponible |
|---|---|---|
| Sólo prueba de vida | `nubsdk-core.min.js` → `nubsdk-face.min.js` | `FaceCapture` |
| Sólo captura de ID | `nubsdk-core.min.js` → `nubsdk-id.min.js` | `IdCapture` |
| Face e ID en la misma aplicación | `nubsdk-core.min.js` → `nubsdk-face.min.js` → `nubsdk-id.min.js` | Ambas |
| Todos los componentes por módulos (siguiente corte) | `nubsdk-core.min.js` → `nubsdk-all-components.min.js` | `FaceCapture`, `IdCapture`, `FaceWithIdCapture` y `VideoRecorder` |
| Face e ID desde un solo archivo | `nubsdk-all.min.js` | Ambas y los módulos incluidos en el bundle |

`Core` aporta la base común, pero **por sí solo no habilita FaceCapture ni IdCapture**. Los módulos Face e ID se cargan después de Core y deben venir del mismo canal o versión. El bundle `all` ya incluye Core, Face e ID: **no lo combines** con los archivos modulares en la misma página. Elige los módulos si sólo necesitas una parte; usa `all` si prefieres una sola etiqueta y utilizarás varias capacidades.

Por ejemplo, para **sólo ID**:

```html
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-id.min.js"></script>
```

Para capturar la identificación mexicana del ejemplo, selecciona la familia antes de `init()`:

```javascript
const idCapture = new IdCapture();
idCapture.setDocumentFamily("MEX_IdCard");
```

Esa familia requiere frente y reverso. Continúa con `setToken()`, los callbacks y el ciclo `init → load → start` descrito más abajo; el [ejemplo completo de IdCapture](04-id-capture.md#configuracion-minima) muestra el orden.

Si eliges el **bundle completo**, sustituye las dos etiquetas del ejemplo facial de abajo por una sola:

```html
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-all.min.js"></script>
```

Para `presentId` en el **canal estable actual**, usa el bundle completo `nubsdk-all.min.js`; el módulo facial mínimo no contiene ese add-on. La nueva pareja `core` → `all-components` ofrecerá la alternativa modular **cuando se publique en `v2 stable`**. No uses su URL estable antes de ese corte. Consulta [Capacidades opcionales](05-addons.md).

> [!NOTE] React, Vue y Next.js usan los mismos archivos CDN; el framework sólo cambia cómo montas y limpias la instancia. Existe además un paquete Web opcional `@nubarium/biometrics` con rutas de importación `/face` y `/id`; la próxima versión añadirá `/all-components`. No es necesario instalarlo si cargas los scripts del CDN.

## 1. Prepara el HTML

```html
<button id="start-capture" type="button">Iniciar captura</button>
<p id="capture-status" role="status"></p>
<div id="face-component"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
<script type="module" src="/capture.js"></script>
```

## 2. Obtén la sesión

`/api/my-middleware/session` representa una ruta de tu propio backend, no una API de Nubarium. Sustitúyela por la ruta que implementes.

```javascript
async function getSessionToken() {
  const response = await fetch("/api/my-middleware/session", {
    method: "POST",
    credentials: "same-origin"
  });

  if (!response.ok) throw new Error("sdk_session_unavailable");
  const session = await response.json();
  if (typeof session.token !== "string" || !session.token) {
    throw new Error("sdk_session_unavailable");
  }
  return session.token;
}
```

## 3. Configura los callbacks

```javascript
let capture;
const button = document.querySelector("#start-capture");
const status = document.querySelector("#capture-status");

function finish(message) {
  status.textContent = message;
  button.disabled = false;
}

function handleSuccess(_payload) {
  finish("Captura completada.");
}

function handleFail(_payload) {
  finish("La evaluación no fue aprobada. Puedes volver a intentarlo.");
}

function handleError(_error) {
  finish("No se pudo completar la captura. Intenta de nuevo.");
}
```

## 4. Inicia desde una acción del usuario

```javascript
button.addEventListener("click", async () => {
  if (button.disabled) return;
  button.disabled = true;
  status.textContent = "Preparando captura…";

  try {
    const token = await getSessionToken();
    capture?.clear();
    const nextCapture = new FaceCapture();
    capture = nextCapture;
    nextCapture.setToken(token);
    nextCapture.onSuccess(handleSuccess);
    nextCapture.onFail(handleFail);
    nextCapture.onError(handleError);
    nextCapture.init({
      rootElement: "face-component",
      locale: "es",
      ui: { colorMode: "system" }
    });
    nextCapture.load(() => {
      if (capture === nextCapture) {
        nextCapture.start().catch(() => {
          if (capture === nextCapture && button.disabled) {
            finish("No se pudo abrir la captura. Intenta de nuevo.");
          }
        });
      }
    });
  } catch (_error) {
    finish("No se pudo iniciar la sesión. Intenta de nuevo.");
  }
});
```

En `v2 stable`, FaceCapture muestra su introducción `liveness` al omitir `intro`. IdCapture muestra la introducción `id-capture`. Puedes desactivar la de cualquiera de los dos componentes con `intro: false`; una configuración explícita o un perfil puede cambiar esta política.

## 5. Libera la instancia

```javascript
window.addEventListener("pagehide", () => {
  const current = capture;
  capture = null;
  current?.clear();
});
```

En una SPA también debes ejecutar `clear()` al desmontar el componente de página.

## Qué callback atender

| Callback | Significado | Acción típica |
|---|---|---|
| `onSuccess` | El componente terminó con evaluación aprobada | Usa el resultado para continuar |
| `onFail` | El componente terminó con evaluación negativa | Explica el resultado o permite reintentar |
| `onError` | La ejecución encontró un problema técnico | Registra `code`, muestra recuperación y decide si reintentar |

> [!NOTE] Usa los campos estructurados del resultado para la lógica. Los textos descriptivos pueden cambiar para mejorar claridad o localización.

---

Siguiente: [profundiza en FaceCapture →](03-face-capture.md)
