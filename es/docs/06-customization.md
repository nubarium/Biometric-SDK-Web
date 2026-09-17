# Personalización

[Read in English](../../docs/06-customization.md)

Adapta idioma, introducción y apariencia sin modificar el ciclo base del componente.

## Idioma

```javascript
capture.init({
  rootElement: "face-component",
  locale: "es"
});
```

Usa una clave incluida en la distribución o registrada explícitamente por tu aplicación. Si agregas una variante propia, conserva todas las claves requeridas y define una alternativa segura.

## Introducción

```javascript
capture.init({
  rootElement: "face-component",
  intro: false
});
```

La introducción prepara al usuario antes de abrir la cámara. En `v2 stable` está activa por defecto para FaceCapture e IdCapture: el SDK elige `liveness` o `id-capture`, respectivamente. El ejemplo muestra cómo desactivarla expresamente. Un perfil o configuración remota puede cambiar esta política; indica otra clave sólo si registraste o elegiste una introducción distinta.

## Apariencia

```javascript
capture.init({
  rootElement: "face-component",
  ui: {
    colorMode: "system"
  }
});
```

`colorMode: "system"` sigue la preferencia del dispositivo cuando la versión cargada lo admite. Verifica contraste, foco y mensajes en los modos que publiques.

## Vista previa en Developer Hub

El [configurador visual](https://developer.nubarium.com/builder.html?section=theme) permite revisar paleta, superficies, máscara, tipografía e introducciones. En las secciones de [FaceCapture](https://developer.nubarium.com/builder.html?section=face) e [IdCapture](https://developer.nubarium.com/builder.html?section=id), la introducción y las guías de captura usan las plantillas y dimensiones del SDK estable v2. La vista previa no abre la cámara ni evalúa imágenes; voz y firma siguen siendo representaciones orientativas.

Antes de publicar una configuración, comprueba el código exportado y ejecuta una prueba real en los dispositivos que soportará tu integración.

## Configuración remota versionada

Puedes conservar opciones en el código o resolver una configuración publicada con `configurationId`.

```javascript
capture.init({
  rootElement: "face-component",
  remoteConfiguration: {
    configurationId: "cfg_face_capture_v3",
    authoritative: true
  }
});
```

`cfg_face_capture_v3` es un valor ilustrativo: sustitúyelo por un identificador publicado y autorizado para tu cuenta. No necesitas `configurationId` para comenzar. Úsalo cuando requieras reutilizar y versionar una configuración sin incluirla dentro del bundle de la aplicación.

## Prioridad de configuración

Antes de combinar fuentes, define qué valores puede cambiar el cliente y cuáles deben permanecer en la configuración publicada. Evita que un ajuste local invalide un requisito que tu integración considera obligatorio.

| Fuente | Úsala para |
|---|---|
| Valores predeterminados | Probar el ciclo mínimo |
| Configuración en `init()` | Opciones específicas de una pantalla |
| Configuración publicada | Reutilizar una versión identificable |
| `setAddonInput()` | Datos variables admitidos por una tarea |

## Comprobación visual

- [ ] Los textos caben en móvil y escritorio.
- [ ] El foco es visible con teclado.
- [ ] Los controles mantienen al menos 44 px de área interactiva.
- [ ] El contraste de texto cumple AA.
- [ ] La interfaz sigue siendo comprensible con movimiento reducido.
- [ ] Los mensajes indican acción y recuperación.

---

Siguiente: [interpreta resultados y errores →](07-results-errors.md)
