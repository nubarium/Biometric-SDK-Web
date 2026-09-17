# Solución de problemas

[Read in English](../../docs/09-troubleshooting.md)

Diagnostica por etapa: carga, acceso, contenedor, permisos, ejecución y resultado.

## El constructor no existe

Síntoma: `FaceCapture is not defined` o `IdCapture is not defined`.

Comprueba:

- Core cargó antes del archivo Face o ID, o cargaste sólo `nubsdk-all.min.js`.
- La URL del script responde sin redirección inesperada.
- La política CSP permite el origen del CDN.
- Tu código se ejecuta después de cargar ambos scripts.
- En SSR, la instancia se crea sólo dentro del navegador.

```javascript
if (typeof window.FaceCapture !== "function") {
  throw new Error("sdk_face_bundle_unavailable");
}
```

## El origen no está habilitado

Síntoma: `onError` entrega `code: "access_denied"` y `reason: "origin_not_registered"`. `runtime_origin_not_allowed` es una clasificación interna que no debes esperar en el callback público.

Compara el origen reportado por el navegador con el registrado:

```javascript
console.log(window.location.origin);
```

Protocolo, hostname y puerto deben coincidir. Registra el valor exacto y vuelve a ejecutar `load()` con una sesión vigente.

## La cámara no abre

Comprueba en este orden:

1. La página usa HTTPS.
2. El navegador permite cámara para ese sitio.
3. Otra aplicación no mantiene el dispositivo ocupado.
4. El contenedor sigue visible y montado.
5. No existe otra instancia activa sobre el mismo contenedor.

Solicita permiso después de una acción explícita. Si el usuario lo rechazó, explica cómo recuperarlo desde la configuración del navegador.

## La captura aparece dos veces

Suele indicar que el ciclo de montaje creó dos instancias.

```javascript
capture?.clear();
capture = new FaceCapture();
```

En React estabiliza dependencias del efecto. En cualquier SPA ejecuta `clear()` antes de cambiar de ruta o reconstruir el componente.

## Recibo `onFail` y lo trato como excepción

`onFail` comunica una evaluación negativa terminada. Reserva `onError` para problemas técnicos. Separa mensajes, telemetría y reglas de reintento para ambos caminos.

## Faltan recursos en la respuesta

`resources` es opcional. Tu código debe tolerar su ausencia:

```javascript
const faceImage = payload.resources?.face ?? null;
if (faceImage) useFaceImage(faceImage);
```

No derives la evaluación de la presencia de una imagen. Usa `result.evaluation` y el callback recibido.

## Una tarea opcional no aparece

Revisa:

- que la clave sea compatible con el componente;
- que la versión y la cuenta tengan acceso;
- que `after` apunte a una tarea válida;
- que `setAddonInput()` se ejecute antes de `load()`;
- que estés leyendo `result.addons`, no sólo `result`.

## Información útil para soporte

Registra sin incluir secretos ni contenido biométrico:

- versión del SDK y navegador;
- `window.location.origin`;
- nombre del componente y familia documental;
- etapa alcanzada: `init`, `load` o `start`;
- `code`, `componentExecutionId`, `validationCode` o `requestId` cuando existan;
- hora y zona horaria del evento.

---

Siguiente: [consulta el mapa de referencia →](10-reference-map.md)
