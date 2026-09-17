# Requisitos de integración

[Read in English](../../docs/01-prerequisites.md)

Esta página reúne lo necesario antes de cargar un componente: origen permitido, distribución JavaScript, contenedor y token efímero. La secuencia completa de una captura está en [Primera captura](02-quickstart.md).

## HTTPS y origen exacto

Sirve la aplicación mediante HTTPS. `localhost` puede usarse por HTTP durante desarrollo si tu cuenta lo permite. Registra el origen exacto desde el que se ejecutará el SDK: protocolo, hostname y puerto deben coincidir.

| Aplicación | Origen que debes registrar |
|---|---|
| `https://onboarding.example.com/start` | `https://onboarding.example.com` |
| `https://example.com:8443/capture` | `https://example.com:8443` |
| `http://localhost:3000` para desarrollo | `http://localhost:3000` si tu cuenta lo permite |

Un origen no cubre automáticamente sus subdominios ni otro puerto.

## Token emitido desde tu backend

Tu servidor solicita un token con perfil `execution`. La respuesta de APIv2 contiene `accessToken`; tu aplicación entrega ese valor al navegador como `token`. Las credenciales permanentes permanecen fuera del navegador.

Ejemplo de ruta en una aplicación Express existente. `/api/my-middleware/session` es un nombre ilustrativo: reemplázalo por la ruta de tu backend. `requireAuthenticatedUser` representa el middleware de sesión de tu aplicación:

```javascript
app.post("/api/my-middleware/session", requireAuthenticatedUser, async (_req, res) => {
  const username = process.env.NUBARIUM_USERNAME;
  const password = process.env.NUBARIUM_PASSWORD;
  if (!username || !password) {
    return res.status(500).json({ code: "sdk_credentials_missing" });
  }

  try {
    const basicCredentials = Buffer.from(`${username}:${password}`).toString("base64");
    const response = await fetch("https://apiv2.sdk.nubarium.com/identity/v2/tokens/generate", {
      method: "POST",
      headers: {
        Authorization: `Basic ${basicCredentials}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ accessProfile: "execution" })
    });
    const identity = await response.json();
    if (!response.ok || identity.status !== "OK" || typeof identity.accessToken !== "string") {
      return res.status(502).json({ code: "sdk_session_unavailable" });
    }

    return res.set("Cache-Control", "no-store").json({ token: identity.accessToken });
  } catch (_error) {
    return res.status(502).json({ code: "sdk_session_unavailable" });
  }
});
```

El navegador llama esa ruta de tu aplicación:

```javascript
const response = await fetch("/api/my-middleware/session", {
  method: "POST",
  credentials: "same-origin"
});

if (!response.ok) throw new Error("sdk_session_unavailable");
const { token } = await response.json();
if (typeof token !== "string" || !token) throw new Error("sdk_session_unavailable");
```

> [!WARNING] No guardes credenciales permanentes ni tokens de mayor alcance en `localStorage`, variables globales, HTML o bundles del cliente.

## Distribución y orden de scripts

Elige una de estas formas. No las mezcles en la misma página:

- **Archivos modulares:** `nubsdk-core.min.js` antes de `nubsdk-face.min.js` o `nubsdk-id.min.js`.
- **Bundle opcional:** `nubsdk-all.min.js`, que ya contiene Core, Face e ID.

La lista de archivos, el próximo módulo `all-components` y el paquete Web están en [Elige los archivos del SDK](02-quickstart.md#elige-los-archivos-del-sdk).

El contenedor debe existir y permanecer montado durante la captura. Este ejemplo usa los archivos modulares de FaceCapture:

```html
<main>
  <div id="capture-root" aria-live="polite"></div>
</main>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
```

La distribución seleccionada resuelve sus estilos, plantillas, introducciones y modelos para una integración estándar. Declara rutas propias sólo cuando tu configuración realmente lo requiera.

## Permisos y ciclo de página

- Solicita cámara como consecuencia de una acción clara del usuario.
- Conserva el contenedor mientras la instancia esté activa.
- Llama `clear()` antes de retirar el contenedor o cambiar de ruta.
- Evita iniciar dos capturas sobre el mismo elemento.
- Presenta una recuperación comprensible cuando el usuario rechaza el permiso.

## Lista de comprobación

- [ ] La página se sirve con HTTPS.
- [ ] El origen exacto está registrado.
- [ ] El backend entrega un JWT de corta vida.
- [ ] Core aparece antes de Face o ID, o se carga solamente `nubsdk-all.min.js`.
- [ ] El contenedor existe antes de `init()`.
- [ ] La ruta libera la instancia con `clear()`.

---

Siguiente: [ejecuta tu primera captura →](02-quickstart.md)
