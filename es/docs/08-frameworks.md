# JavaScript, React, Vue y Next.js

[Read in English](../../docs/08-frameworks.md)

Los cuatro entornos usan la misma distribución del Web SDK. Primero elige archivos modulares o el bundle opcional `nubsdk-all.min.js`; después adapta sólo el montaje del contenedor, la carga de scripts y la llamada a `clear()`. No necesitas un bundle distinto por framework.

Antes de usar estos ejemplos, [prepara el origen y el token](01-prerequisites.md). El navegador recibe sólo un JWT efímero desde tu backend; nunca usuario o contraseña de Nubarium.

| Entorno | Montaje | Limpieza |
|---|---|---|
| JavaScript puro | Después de crear el elemento HTML | Al salir de la página o retirar el elemento |
| React | `useEffect` después del render | Función de limpieza del efecto |
| Vue | `onMounted` | `onBeforeUnmount` |
| Next.js (App Router) | Componente cliente, después de cargar Core y Face | Limpieza del componente React |

## JavaScript puro

Coloca el contenedor y carga Core **antes** de Face. Obtén el token desde una ruta propia cuando el usuario pulse iniciar:

```html
<button id="start-capture" type="button">Iniciar prueba</button>
<div id="face-root"></div>

<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-core.min.js"></script>
<script src="https://cdn.nubarium.com/nubSdk/v2/stable/js/nubsdk-face.min.js"></script>
<script src="/capture.js"></script>
```

En `/capture.js` usa el código de [Primera captura](02-quickstart.md) y cambia `rootElement: "face-component"` por `rootElement: "face-root"`. Ese ejemplo incluye el bloqueo de clics duplicados, la obtención del token y la limpieza. También puedes conservar el ID `face-component` en ambos lugares.

## React

Carga los dos scripts del CDN una sola vez antes de montar este componente. Conserva los callbacks estables con `useCallback` y entrega un token válido desde tu backend.

```jsx
import { useEffect } from "react";

export function FaceCaptureView({ token, onSuccess, onFail, onError }) {
  useEffect(() => {
    if (!token || typeof window.FaceCapture !== "function") return;

    let disposed = false;
    const capture = new window.FaceCapture();
    capture.setToken(token);
    capture.onSuccess(onSuccess);
    capture.onFail(onFail);
    capture.onError(onError);
    capture.init({ rootElement: "react-face-root" });
    capture.load(() => {
      if (!disposed) capture.start();
    });

    return () => {
      disposed = true;
      capture.clear();
    };
  }, [token, onSuccess, onFail, onError]);

  return <div id="react-face-root" />;
}
```

### Reglas para React

- Carga los scripts del SDK una sola vez en el documento.
- Evita recrear callbacks en cada render o estabilízalos con `useCallback`.
- No cambies la `key` del contenedor durante una captura.
- En SSR, ejecuta la instancia únicamente en cliente.
- En modo estricto de desarrollo, React puede montar y limpiar el efecto más de una vez; `disposed` evita que una carga tardía reabra la instancia anterior.
- Si montas varias capturas a la vez, usa un identificador de contenedor distinto para cada instancia.

## Vue

```vue
<script setup>
import { onBeforeUnmount, onMounted } from "vue";

const props = defineProps({
  token: { type: String, required: true },
  onSuccess: { type: Function, required: true },
  onFail: { type: Function, required: true },
  onError: { type: Function, required: true }
});
let capture = null;
let disposed = false;

onMounted(() => {
  capture = new window.FaceCapture();
  capture.setToken(props.token);
  capture.onSuccess(props.onSuccess);
  capture.onFail(props.onFail);
  capture.onError(props.onError);
  capture.init({ rootElement: "vue-face-root" });
  capture.load(() => {
    if (!disposed) capture.start();
  });
});

onBeforeUnmount(() => {
  disposed = true;
  capture?.clear();
  capture = null;
});
</script>

<template>
  <div id="vue-face-root" />
</template>
```

### Reglas para Vue

- Monta el componente sólo cuando tengas el token y se hayan cargado los scripts del CDN.
- No uses `v-if` para retirar el contenedor mientras la cámara esté activa.
- Libera la instancia en `onBeforeUnmount`.
- En una vista conservada con `KeepAlive`, define cuándo limpiar al desactivarla; `onBeforeUnmount` puede no ejecutarse al cambiar de ruta.
- En Nuxt, monta el SDK dentro de un componente exclusivo del cliente.
- Mantén el token como dato efímero y las credenciales permanentes en backend.

## Next.js (App Router)

El SDK usa cámara y `window`, así que el montaje va en un **Client Component**. Este ejemplo carga Core y después Face mediante `next/script`; cuando ambos están listos reutiliza `FaceCaptureView` del ejemplo React. `token` debe venir de una ruta propia del backend.

```jsx
"use client";

import { useState } from "react";
import Script from "next/script";
import { FaceCaptureView } from "./FaceCaptureView";

const cdn = "https://cdn.nubarium.com/nubSdk/v2/stable/js";

export function NextFaceCapture({ token, onSuccess, onFail, onError }) {
  const [coreReady, setCoreReady] = useState(false);
  const [faceReady, setFaceReady] = useState(false);

  return (
    <>
      <Script src={`${cdn}/nubsdk-core.min.js`} onReady={() => setCoreReady(true)} />
      {coreReady && (
        <Script src={`${cdn}/nubsdk-face.min.js`} onReady={() => setFaceReady(true)} />
      )}
      {faceReady && token ? (
        <FaceCaptureView
          token={token}
          onSuccess={onSuccess}
          onFail={onFail}
          onError={onError}
        />
      ) : (
        <p>Preparando la captura…</p>
      )}
    </>
  );
}
```

No instancies el SDK en un Server Component ni pongas credenciales en variables `NEXT_PUBLIC_*`. Define los callbacks en un componente cliente; no pases funciones desde un Server Component como props. Si usas el Pages Router, aplica el mismo principio: scripts cargados en orden, contenedor montado y `clear()` en la limpieza.

## Patrón compartido

1. El host crea un contenedor visible.
2. Tu backend entrega un token efímero.
3. Core y el archivo Face o ID se cargan en ese orden; si eliges `nubsdk-all.min.js`, carga sólo ese archivo.
4. La instancia registra token, callbacks y configuración; `load()` prepara y luego llama `start()`.
5. La limpieza ejecuta `clear()` antes de retirar el contenedor.

Para ID, si usas módulos cambia el archivo Face por `nubsdk-id.min.js`; luego usa `IdCapture` y configura la familia documental. Si elegiste `nubsdk-all.min.js`, conserva ese único archivo y cambia sólo la clase y la configuración. Consulta [IdCapture](04-id-capture.md).

Si habilitas `navigation: { enabled: true }`, conecta el evento Atrás de tu router o contenedor a `capture.requestBack()`. El componente decide si vuelve a la intro, ofrece recapturar o solicita salir según la tarea. Tu host es responsable de cambiar de ruta tras `onExit`; evita que el mismo gesto retroceda también por el router. `clear()` durante el desmontaje no emite `onExit`. Consulta [Resultados y errores](07-results-errors.md#salida-elegida-por-el-usuario).

---

Siguiente: [diagnostica problemas frecuentes →](09-troubleshooting.md)
