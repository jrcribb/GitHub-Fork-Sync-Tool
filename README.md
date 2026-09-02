# 🔄 GitHub Fork Sync Tool

Una herramienta de escritorio **sin backend**, ejecutable directamente en el navegador, que permite gestionar, evaluar y sincronizar masivamente todos tus forks de GitHub con sus repositorios upstream originales sin necesidad de descargarlos a tu equipo. O sea, la sincronización se hace directamente, mediante la API de GitHub, en la nube.

> Diseñada para usuarios que mantienen cientos o miles de forks y necesitan controlar el estado de sincronización de forma eficiente, sin saturar la API de GitHub.

---

## 🎯 Propósito

Cuando tienes una gran cantidad de forks en GitHub (cientos o miles), mantenerlos actualizados es una tarea tediosa e impráctica desde la interfaz web de GitHub. Esta herramienta resuelve ese problema centralizando toda la gestión en una sola página HTML que se comunica directamente con la API REST de GitHub usando tu Personal Access Token.

---

## ✨ Funcionalidades principales

### 🔍 Análisis de forks

- Carga automáticamente **todos tus forks** mediante paginación de la API.
- **Continuidad de análisis por lotes**: permite analizar secuencialmente todos los repositorios por grupos mediante el botón **"Analizar siguiente lote"**, acumulando el progreso y preservando los resultados previos sin reconsultar la API de repositorios.
- Para cada fork detecta su repositorio **upstream (parent)** y compara el estado de la rama principal.
- Informa si el fork está **actualizado**, **atrasado N commits**, o si **no tiene upstream**.
- Muestra el **About / descripción** del repositorio para brindar contexto.
- Los repositorios aún no evaluados se marcan como **"No evaluado"** en lugar de mostrar un estado incorrecto.

### 🔄 Sincronización

- **Sincronización estándar**: usa `POST /repos/{owner}/{repo}/merge-upstream` de la API de GitHub (fast-forward limpio).
- **Sincronización forzada** (⚡): hace un hard-reset de la rama del fork al commit SHA del upstream, sobrescribiendo cualquier divergencia o conflicto.
- Si la sincronización estándar falla por conflictos, reintenta automáticamente con la sincronización forzada.

### ⚡ Procesamiento por lotes

- Configura la cantidad de repositorios por lote, la pausa entre lotes (en segundos) y la cantidad de lotes a ejecutar.
- Permite procesar repositorios en etapas para **evitar saturar el rate limit** de la API de GitHub.
- Botón dedicado **"⚡ Procesar por Lotes"** para ejecutar la sincronización automática secuencial.
- Countdown en tiempo real durante la pausa entre lotes.

### 🚫 Desactivación de Workflows y Actions

- Si la casilla **"Mantener workflows activados"** está desmarcada, al sincronizar se desactivan automáticamente:
  - Los permisos de GitHub Actions a nivel de repositorio.
  - Cada workflow activo individualmente.
- También disponible como acción independiente: **"Desactivar Workflows (Seleccionados)"**.

### 🗂️ Filtrado y ordenamiento

| Opción | Descripción |
| --- | --- |
| **Mostrar: Sólo pendientes** | Muestra únicamente forks atrasados respecto al upstream |
| **Mostrar: Todos** | Muestra todos los forks cargados |
| **Ordenar: Última sync (asc)** | Primero los nunca sincronizados (orden predeterminado) |
| **Ordenar: Estado** | Pendientes primero |
| **Ordenar: Nombre / Nombre completo** | Orden alfabético |

> El reordenamiento se aplica solo al presionar el botón **"Reordenar"**.

### 📦 Historial de sincronizaciones (localStorage)

- Cada vez que un repositorio se sincroniza exitosamente, se guarda la fecha y hora en el **almacenamiento local del navegador** (`localStorage`).
- Cada tarjeta muestra la **fecha de última sincronización**.
- El historial persiste entre sesiones del navegador.

### 📊 Panel de estado dual (50% / 50%)

Durante cualquier operación, la sección superior muestra en tiempo real:

| Panel Izquierdo | Panel Derecho |
| --- | --- |
| Estado general del proceso | Repositorio que se está procesando en este momento |
| Lotes: procesados / total | Enlace al repositorio |
| Repositorios: procesados / total | Estado actual (badge), descripción, upstream, última sync |
| Countdown de pausa entre lotes | — |

### 🃏 Tarjetas por repositorio

Cada repositorio se muestra como una tarjeta con:

- ✅ Casilla de selección individual
- 🔗 Enlace directo al repositorio en GitHub
- 📝 Descripción (About) del repositorio
- 🏷️ Badge de estado: `No evaluado` / `Sin upstream` / `Actualizado` / `⚠️ Atrás N commits` / `Error API`
- 📅 Fecha de última sincronización
- 📋 Log en vivo del resultado de la operación
- 🔘 Botón **Sync** individual (deshabilitado si no fue evaluado, sin upstream, o ya actualizado)

---

## 🚀 Modo de uso

### 1. Requisitos previos: Personal Access Token de GitHub

Necesitas un **Fine-grained Personal Access Token** o un **Classic Token** con los siguientes permisos:

| Permiso | Necesario para |
| --- | --- |
| `repo` (o `Contents: Read & Write`) | Sincronizar ramas |
| `workflow` | Desactivar workflows/actions |
| `Actions: Read & Write` | Desactivar Actions a nivel de repositorio |

> 📌 Genera tu token en: **GitHub → Settings → Developer settings → Personal access tokens**

---

### 2. Abrir la herramienta

No requiere instalación ni servidor. Simplemente abre el archivo en tu navegador:

```
GitHub Fork Sync Tool.html
```

> Compatible con cualquier navegador moderno (Chrome, Edge, Firefox).

---

### 3. Flujo de trabajo recomendado

```
1. Pegar el token en el campo "Github API key"
2. Configurar parámetros de lote (Repos/lote, Pausa, Cant. lotes)
3. Presionar "Cargar / Recargar Forks"
   → Recupera la lista completa de forks e inicia una nueva sesión analizando el primer lote
4. Para continuar analizando los siguientes repositorios sin reiniciar la sesión ni volver a consultar la lista, presionar "Analizar siguiente lote"
5. Revisar la lista: los forks pendientes quedan seleccionados automáticamente
6. Ajustar la selección si es necesario
7. Elegir modo de acción:
   - "Sincronizar Seleccionados" → procesa todos de una vez
   - "⚡ Procesar por Lotes" → procesa en etapas automáticamente
8. Repetir el análisis del siguiente lote y sincronización hasta completar todos los forks
```

---

### 4. Parámetros de lotes

| Campo | Descripción | Valor 0 |
| --- | --- | --- |
| **Repos/lote** | Cuántos repositorios se evalúan/sincronizan por lote | Sin límite |
| **Pausa (seg)** | Segundos de espera entre un lote y el siguiente | Sin pausa |
| **Cant. lotes** | Cuántos lotes consecutivos ejecutar automáticamente | Todos los lotes |

> **Ejemplo para 900+ repos**: `Repos/lote = 50`, `Pausa = 15`, `Cant. lotes = 3` → procesa 150 repos en 3 rondas con 15 segundos de respiro entre ellas.

---

### 5. Opciones de sincronización

| Opción | Efecto |
| --- | --- |
| **Mantener workflows activados** ✅ | No toca los workflows al sincronizar |
| **Mantener workflows activados** ❌ | Desactiva Actions y todos los workflows activos en cada repo sincronizado |
| **⚡ Forzar sync** ✅ | Hace hard-reset al commit de upstream (sobrescribe divergencias) |
| **⚡ Forzar sync** ❌ | Usa merge-upstream estándar (falla si hay conflictos y reintenta con force) |

---

## 🔒 Seguridad

- El token **nunca sale de tu navegador**. Todas las llamadas a la API de GitHub se hacen directamente desde el frontend.
- No hay backend, no hay base de datos, no hay servidor.
- El historial de sincronizaciones se guarda únicamente en el `localStorage` de tu navegador bajo la clave `gfst_sync_history_v1`.

---

## 📁 Estructura del proyecto

```
GitHub Fork Sync Tool/
├── GitHub Fork Sync Tool.html   # Aplicación completa (HTML + CSS + JS)
└── .agents/
    └── AGENTS.md                # Reglas del agente de IA (versionado de código)
```

---

## 🛠️ Tecnologías utilizadas

- **HTML5 / CSS3 / JavaScript (Vanilla)** — sin frameworks ni dependencias externas
- **GitHub REST API v3** — para listar repos, comparar ramas, sincronizar, gestionar workflows
- **localStorage** — para persistir el historial de sincronizaciones entre sesiones

---

## 📋 Estados de los repositorios

| Badge | Significado |
| --- | --- |
| `No evaluado` | El fork no ha sido analizado en la sesión actual |
| `Sin upstream` | El repositorio fue evaluado y no tiene un parent (upstream) |
| `✅ Actualizado` | La rama del fork está al día con el upstream |
| `⚠️ Atrás N commits` | El fork está N commits por detrás del upstream |
| `Error API` | Ocurrió un error al consultar la API de GitHub |

---

## ⚠️ Limitaciones conocidas

- La **API de GitHub tiene rate limits**: 5000 requests/hora para tokens autenticados. Con 900+ repos y evaluación de 2 llamadas por repo, conviene procesar en lotes de 50–100 con pausa de 10–15 segundos.
- La desactivación de workflows requiere el permiso `workflow` en el token. Sin él, la operación fallará con un error visible en la tarjeta del repo.
- Los forks privados también se listan si el token tiene acceso a repositorios privados.
