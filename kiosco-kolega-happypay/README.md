# Kiosco Kolega — HappyPay Ecuador

Aplicación de kiosco (un solo archivo HTML + React, sin build step) para el
asistente virtual "Kolega" de HappyPay Ecuador. Pensada para correr en un
tótem táctil con micrófono, en Chrome a pantalla completa.

## Qué incluye

- **Avatar SVG animado** de cuerpo medio (paleta verde/azul), con parpadeo
  cada 4-6 s, inclinación de cabeza al escuchar, mirada hacia arriba +
  puntos animados al "pensar", y 5 visemas de boca (cerrada, semiabierta,
  abierta, "o", "e") sincronizados de forma aproximada con los eventos
  `boundary` de `SpeechSynthesis` mientras habla.
- **Voz**: entrada por `SpeechRecognition` (es-EC) en modo continuo mientras
  el botón "Toca para hablar" está activo; salida por `SpeechSynthesis`
  buscando una voz es-US/es-MX/es-419 (seleccionable en el panel admin).
  Subtítulos grandes con todo lo que dice el avatar.
- **Cerebro**: cada turno se envía a la API de Anthropic
  (`claude-sonnet-5`) con tool use. La herramienta `evaluar_credito` calcula
  la cuota por amortización francesa con la tasa cargada en políticas y
  valida monto, plazo, ingreso mínimo y ratio cuota/ingreso.
- **Pantalla principal**: avatar centrado, subtítulos, botón de voz, 4
  accesos rápidos, teclado numérico táctil (monto e ingreso nunca se dicen
  en voz alta, siempre se digitan) y chips de plazo. Tarjeta de
  pre-evaluación con el sello "PRE-EVALUACIÓN REFERENCIAL — sujeta a
  verificación con un ejecutivo". Timeout de 60 s sin interacción que
  reinicia la conversación.
- **Panel de administración** (engranaje arriba a la derecha, PIN por
  defecto `254800`): pestañas de Políticas de crédito (JSON), Convenios y
  pagos (texto libre) y FAQ (lista editable), más selección de voz y campo
  de API key de respaldo. Exportar/Importar JSON para respaldar o replicar
  configuración entre tótems. **No usa `localStorage` ni `sessionStorage`**:
  todo vive en estado de React durante la sesión; el respaldo es el
  exportar/importar JSON.

## Cómo se conecta con Claude (importante)

El archivo es 100% estático (HTML + React vía CDN), así que necesita algún
lugar donde vivir la API key de Anthropic sin quedar expuesta en el código:

1. **Recomendado — despliegue en Vercel con proxy propio**: este folder
   incluye `api/chat.js`, una función serverless que reenvía la petición a
   `https://api.anthropic.com/v1/messages` usando la variable de entorno
   `ANTHROPIC_API_KEY` (nunca viaja al navegador). El `index.html` llama
   primero a `/api/chat` (ruta relativa), así que basta con:
   - Desplegar esta carpeta como proyecto de Vercel (root directory =
     `kiosco-kolega-happypay`).
   - Configurar `ANTHROPIC_API_KEY` en las variables de entorno del
     proyecto de Vercel.
2. **Fallback — modo directo desde el navegador**: si no hay backend
   disponible (por ejemplo, se abre el `index.html` directo desde el
   disco), el panel admin tiene un campo de API key que se guarda solo en
   memoria (se pierde al recargar) y la app llama a Anthropic directo desde
   Chrome con el header `anthropic-dangerous-direct-browser-access`. Esto
   expone la key en las herramientas de desarrollador del navegador — úsalo
   solo para demos o pruebas locales, nunca en un tótem de producción
   expuesto al público.

## Cómo probarlo

- **Local rápido**: abrir `index.html` directo en Chrome y cargar una API
  key en el panel admin (modo fallback).
- **Kiosco real**: desplegar en Vercel con `ANTHROPIC_API_KEY` configurada,
  abrir la URL en Chrome fullscreen (`chrome --kiosk https://tu-deploy...`)
  en el tótem con micrófono habilitado (Chrome pedirá permiso de
  micrófono la primera vez).

## Variante: avatar humano realista (opcional, no implementada aquí)

Si en el futuro se quiere reemplazar el avatar SVG por un rostro humano en
video (HeyGen `@heygen/streaming-avatar` o D-ID Agents), la integración
consiste en enviar el texto de la respuesta de Claude a esos SDKs para el
lip-sync realista, dejando el API key de HeyGen/D-ID como campo
configurable en el panel admin y manteniendo el avatar SVG como fallback si
no hay key configurada. Esas plataformas parten en planes de ~USD
100-500/mes por stream concurrente; para una demo el avatar SVG es
suficiente y gratis.

## Para producción con varios tótems

Esta versión guarda la configuración en memoria + exportar/importar JSON,
pensada para un tótem o para replicar manualmente. Si se necesitan muchos
tótems sincronizados, la configuración (políticas, convenios, FAQ, PIN)
debería migrar a una base central (por ejemplo Supabase) en lugar de
configurarse tótem por tótem.
