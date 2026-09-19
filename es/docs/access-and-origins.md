# Tokens y orígenes autorizados

[Read in English](../../docs/access-and-origins.md)

Realiza estas llamadas a APIv2 desde tu backend o una herramienta administrativa. No las hagas desde el navegador ni incluyas credenciales permanentes en la configuración del SDK.

URL base: `https://apiv2.sdk.nubarium.com`.

## Generar un token

Envía las credenciales de la cuenta Nubarium mediante autenticación HTTP Basic:

```http
POST /identity/v2/tokens/generate
Authorization: Basic <base64(usuario:contraseña)>
Content-Type: application/json

{"accessProfile":"control"}
```

El body JSON selecciona un perfil de acceso:

| Perfil | Uso | Dónde permanece |
|---|---|---|
| `execution` | Ejecutar FaceCapture o IdCapture | Tu backend lo obtiene y entrega al navegador únicamente este JWT efímero |
| `api` | Operar procesos, ejecuciones, resultados, recursos y preservaciones | Tu backend |
| `control` | Administrar orígenes, configuraciones y almacenamiento | Tu backend o herramienta administrativa |
| `backend` | Combinar las capacidades de `api` y `control` cuando un mismo servicio necesita ambas | Sólo un backend de confianza |

Para una sesión del SDK, envía la misma solicitud con `{"accessProfile":"execution"}`. Una respuesta `200 OK` a la solicitud `control` anterior tiene esta forma (todos los valores son ilustrativos):

```json
{
  "tokenType": "Bearer",
  "accessToken": "<JWT-oculto>",
  "bearer_token": "<JWT-oculto>",
  "expiresIn": 900,
  "exp": 1787302800
}
```

`Bearer` es el esquema de autorización indicado por `tokenType`. `bearer_token` es un alias legado temporal del mismo valor de `accessToken`; las integraciones nuevas deben usar `accessToken`. `expiresIn` expresa la vigencia en segundos desde la emisión y `exp` el vencimiento absoluto en tiempo Unix (segundos). El JWT firmado contiene su `tenantId`, `jti`, perfil, scopes y el mismo `exp`; no uses campos decodificados sin verificar primero la firma, audiencia y vigencia.

La respuesta incluye el header `X-Nubarium-Request-Id` para correlación operativa. La emisión también puede responder `400` (perfil inválido), `401` (credenciales Basic incorrectas), `429` (límite de solicitudes) o `503` (fallo temporal). Los errores contienen `requestId` y `error: { code, message, retriable }`. No registres credenciales ni JWT en logs.

Tu aplicación puede exponer una ruta autenticada para entregar el token `execution` al navegador. `/api/my-middleware/session` en [Primera captura](02-quickstart.md#2-obten-la-sesion) representa una ruta de **tu aplicación**, no de Nubarium. Su respuesta al navegador nunca debe incluir el token `control` ni las credenciales permanentes.

## Registrar el origen de la aplicación

Usa un token `control` de la misma cuenta que emitirá los tokens `execution` para el SDK. La cuenta debe tener permiso para administrar orígenes (`origins:write`).

```http
POST /control/v2/origins
Authorization: Bearer <accessToken-de-control>
Content-Type: application/json

{"origin":"https://app.example.com"}
```

`origin` incluye exactamente protocolo, hostname y puerto cuando exista. No incluye ruta, comodines ni el nombre de una página. Si la aplicación abre `https://app.example.com/capture`, registra `https://app.example.com`. Los subdominios y otros puertos son orígenes distintos. APIv2 determina cuenta y ambiente desde la solicitud autenticada; no los agregues al body.

La respuesta `200 OK` contiene el registro del origen y el resultado de publicación. Ejemplo con valores ilustrativos:

```json
{
  "contractVersion": "nubarium.runtime-origin/2.0",
  "status": "OK",
  "requestId": "origin-request-example",
  "data": {
    "origin": {
      "originId": "origin-example",
      "origin": "https://app.example.com",
      "environment": "production",
      "status": "active",
      "verification": { "state": "verified" },
      "createdAt": "2026-09-18T10:00:00.000Z",
      "updatedAt": "2026-09-18T10:00:00.000Z",
      "disabledAt": null
    },
    "publication": {
      "generation": "generation-example",
      "generatedAt": "2026-09-18T10:00:00.000Z",
      "originCount": 1,
      "bindingCount": 1,
      "changed": true
    }
  }
}
```

`data.publication` puede ser `null` cuando no haya información de publicación. Comprueba que el origen devuelto coincida con el solicitado y tenga estado `active` antes de ejecutar el SDK. Puedes consultar los orígenes de la cuenta con `GET /control/v2/origins` usando un token del servidor con `origins:read`. El contrato de registro también define respuestas de error `400`, `401`, `403`, `404`, `409`, `429` y `503`. Un `403` puede indicar que faltan permisos `origins:write`; un token `execution` no sirve para registrar orígenes.

Este registro autoriza el origen web para el runtime del SDK. No es una opción de CORS en el navegador. Si tu cuenta admite HTTP `localhost` durante desarrollo, registra por separado ese origen exacto; los orígenes de producción usan HTTPS.

Siguiente: [revisa los demás requisitos](01-prerequisites.md) o [ejecuta una captura](02-quickstart.md).
