# ⏱️ Auditor Time Tracker

Aplicación web de página única (vanilla HTML/CSS/JS, sin frameworks ni dependencias de backend) diseñada para que equipos de auditoría registren y controlen el tiempo dedicado a cada proyecto durante la jornada laboral.

Todo se ejecuta en el navegador y persiste en `localStorage`, lo que permite su uso sin conexión, sin servidores y sin configuración.

---

## ¿Qué problema resuelve?

En equipos de auditoría es común trabajar en múltiples proyectos simultáneamente, sufrir bloqueos (stoppers) que pueden ser internos o externos, asistir a reuniones no planificadas y necesitar reportar con precisión cuánto tiempo efectivo se dedicó a cada tarea. Esta herramienta centraliza todo eso en una sola pantalla.

---

## Características principales

### Panel del Auditor ("Mi Panel")

- **Registro de proyectos** con inicio en un clic. Cada proyecto transita entre los estados: *en ejecución → stopper → reanudado → en revisión / pausado*.
- **Stoppers con clasificación**: al detener un proyecto se selecciona si el bloqueo es **#Interno** (del equipo/organización) o **#Externo** (dependencia de terceros), con campo de razón obligatorio.
- **Reuniones**: registro de reuniones de Teams con hora de inicio/fin y motivo. La reunión en curso se persiste y sobrevive recargas de página.
- **Estadísticas en vivo**: tiempo efectivo, tiempo en stopper (desglosado interno/externo), tiempo en reuniones y proyectos activos, actualizados cada 30 segundos.
- **Timeline del día**: vista cronológica con franjas cada 30 minutos (padre) y eventos a la hora exacta en que ocurrieron (hijo). Muestra rangos horarios en stoppers y reuniones.
- **Exports**: descarga del reporte diario en `.txt` (mismo formato que el timeline) y `.csv` (una fila por proyecto con tiempos efectivos y stoppers desglosados por tipo).

### Panel Admin

- Vista en tiempo real de todos los auditores registrados con badges de estado (ejecutando, stopper, en revisión) y tiempo efectivo/stopper del día.
- Alta de nuevos auditores.

### Panel de Reportes

- Filtros por auditor y fecha.
- **Gráfico donut SVG** con porcentaje de tiempo efectivo, stopper, reuniones y sin actividad.
- **Barras por proyecto** con desglose efectivo/stopper y detalle interno/externo.
- Listado de reuniones con horarios.
- Export CSV consolidado (todos los auditores o uno específico, por fecha o histórico completo).

---

## Modelo de datos

```
state.auditors[id]
  ├── name
  └── days["2026-06-09"]
        ├── projects[pid]
        │     ├── status: running | stopped | review | paused
        │     ├── startedAt, stoppedAt, endedAt (ISO)
        │     ├── stopReason, stopTipo (interno | externo)
        │     ├── pauseReason
        │     ├── events[]          ← historial de transiciones
        │     └── stopperHistory[]  ← stoppers cerrados con rango y tipo
        ├── meetings[]              ← reuniones finalizadas
        └── snapshots[]             ← log de acciones (audit trail)
```

Todo se serializa en `localStorage` bajo la clave `aud_tracker_v2`. La sesión activa (auditor seleccionado y reunión en curso) se guarda en `aud_tracker_session`.

---

## Lógica de cálculo de tiempos

- **Tiempo efectivo**: suma de los intervalos en que el proyecto estuvo en estado `running`, excluyendo la hora de almuerzo (13:00–14:00).
- **Tiempo de stopper**: suma de los intervalos en estado `stopped`, separados en interno y externo según el tipo registrado al momento de detener el proyecto. Se calcula tanto a nivel de proyecto individual como agregado por día.
- **Reuniones**: duración bruta inicio–fin (el almuerzo **no** se excluye, ya que las reuniones pueden ocurrir en ese horario). Las reuniones en curso se cuentan en vivo.
- **Porcentajes del donut**: calculados sobre la jornada laboral configurada (`WORK_START` a `WORK_END` menos la hora de almuerzo).

---

## Configuración

Las constantes de jornada laboral se encuentran al inicio del `<script>`:

```js
const WORK_START  = 8 * 60;   // 08:00 — inicio de jornada
const WORK_END    = 17 * 60;  // 17:00 — fin de jornada (cambiar a 24*60 para pruebas fuera de horario)
const LUNCH_START = 13 * 60;  // 13:00 — inicio de almuerzo
const LUNCH_END   = 14 * 60;  // 14:00 — fin de almuerzo
const SNAP        = 30;       // intervalo del timeline en minutos
```

---

## Uso

1. Abrir `auditor_tracker.html` en cualquier navegador moderno.
2. Ir a **Admin** → crear uno o más auditores.
3. En **Mi Panel**, seleccionar un auditor e iniciar proyectos.
4. Usar los botones de cada tarjeta para registrar stoppers, reanudar, pausar o enviar a revisión.
5. Al final del día, descargar el reporte en `.txt` o `.csv`.

No requiere instalación, servidor, ni conexión a internet (salvo para cargar los iconos de Tabler desde CDN en la primera visita).

---

## Stack técnico

- **HTML/CSS/JS** vanilla — archivo único, sin build, sin dependencias npm.
- **Tabler Icons** vía CDN para la iconografía.
- **localStorage** para persistencia.
- **SVG inline** para el gráfico donut.
- Soporte automático de **modo oscuro** (`prefers-color-scheme: dark`).

---

## Estructura del archivo

| Sección | Descripción |
|---|---|
| `<style>` | Variables CSS (light/dark), componentes (badges, cards, timeline, modal) |
| HTML | Tres paneles (auditor, admin, reportes) + contenedor de modales |
| `save / load` | Persistencia en localStorage con migración de claves de fecha |
| `calcTime` | Cálculo agregado del día (efectivo, stopper int/ext, reuniones) |
| `calcProjEffective / calcProjStopper` | Cálculo por proyecto individual |
| `collectDayEvents` | Centraliza la construcción de eventos (alimenta timeline y TXT) |
| `renderTimeline` | Timeline visual (padre 30 min / hijo hora exacta) |
| `generateReport` | Donut + barras + detalle para la pestaña de reportes |
| `buildLogLines` | Genera el reporte en texto plano |
| `exportLogCsv / exportReportCsv` | Exports CSV con valores por proyecto |

---

## Licencia

Uso interno.
