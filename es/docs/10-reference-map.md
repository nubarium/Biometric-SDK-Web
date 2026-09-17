# Mapa de referencia

[Read in English](../../docs/10-reference-map.md)

Resumen de métodos, callbacks y términos usados en esta edición.

Para elegir qué scripts cargar, consulta [Core, Face, ID y bundle completo](02-quickstart.md#elige-los-archivos-del-sdk).

## Métodos del ciclo principal

| Método | Momento | Responsabilidad |
|---|---|---|
| `setToken(token)` | Antes de `load()` | Entrega el JWT efímero a la instancia |
| `onSuccess(callback)` | Antes de iniciar | Registra resultado aprobado |
| `onFail(callback)` | Antes de iniciar | Registra evaluación negativa |
| `onError(callback)` | Antes de iniciar | Registra problema técnico |
| `onExit(callback)` | Antes de iniciar | Informa una salida elegida por el usuario |
| `init(options)` | Después de configurar callbacks | Define contenedor y opciones |
| `load(callback)` | Después de `init()` | Prepara recursos, sesión y permisos |
| `start()` | Dentro del callback de `load()` | Inicia la experiencia |
| `clear()` | Al terminar o desmontar | Libera cámara, listeners y UI |
| `requestBack()` | Cuando el host recibe Atrás | Pide al SDK resolver el paso interno; devuelve una decisión |
| `requestExit()` | Cuando el host ofrece Salir | Solicita la salida administrada del componente |

## Métodos por componente

| Método | Componente | Uso |
|---|---|---|
| `setDocumentFamily("MEX_IdCard")` | `IdCapture` | Selecciona la identificación mexicana del ejemplo (frente y reverso) |
| `setAddonInput(key, value)` | Ambos, según capacidad | Entrega datos variables antes de `load()` |

## Opciones de `init()` mostradas

| Opción | Ejemplo | Propósito |
|---|---|---|
| `rootElement` | `"face-component"` | ID del contenedor existente |
| `locale` | `"es"` | Idioma de la experiencia |
| `intro` | `false` | Desactiva la introducción que FaceCapture e IdCapture muestran por defecto en `v2 stable` |
| `intro.key` | Opcional | Selecciona una introducción distinta de la propia del componente |
| `navigation.enabled` | `true` | Habilita la navegación y la salida administrada del componente |
| `ui.colorMode` | `"system"` | Define el modo de color |
| `document.documentFamily` | `"MEX_IdCard"` | Define la misma familia documental desde `init()` |
| `addons` | `[{ key, after, required }]` | Declara tareas opcionales |
| `remoteConfiguration.configurationId` | `"cfg_face_capture_v3"` | Resuelve una configuración publicada |

## Glosario

**Componente**  
Unidad ejecutable principal, como `FaceCapture` o `IdCapture`.

**Capacidad opcional**  
Tarea hija ejecutada dentro de un componente, con configuración y resultado propios.

**Evaluación**  
Resultado funcional de la ejecución. No es equivalente a un error técnico.

**Origen**  
Combinación exacta de protocolo, hostname y puerto desde la que corre la aplicación.

**Token de ejecución**  
JWT de corta vida que el backend entrega al navegador para iniciar una sesión del SDK.

**Recurso inline**  
Contenido incluido directamente en la respuesta, por ejemplo una imagen codificada. Su presencia es opcional.

**Familia documental**  
Clave que define lados, guías y reglas disponibles para un tipo de documento.

## Recorrido recomendado

1. Confirma [Requisitos](01-prerequisites.md).
2. Ejecuta [Primera captura](02-quickstart.md).
3. Elige [FaceCapture](03-face-capture.md) o [IdCapture](04-id-capture.md).
4. Agrega [Capacidades opcionales](05-addons.md) sólo cuando el ciclo base funcione.
5. Aplica [Personalización](06-customization.md).
6. Integra [Resultados y errores](07-results-errors.md).
7. Ajusta el ciclo para [JavaScript, React, Vue o Next.js](08-frameworks.md) si corresponde.
8. Conserva [Solución de problemas](09-troubleshooting.md) como guía de diagnóstico.

---

Volver al [README del SDK](../README.md).
