# Reglas para GitHub Fork Sync Tool

## Actualización de ID de Versión y Título
Cada vez que se modifique o actualice el código en `GitHub Fork Sync Tool.html`, se debe actualizar siempre la marca de ID y fecha/hora (con formato `YYYYMMDD-HHMM.1` derivado de la hora local actual) en las siguientes 3 ubicaciones:

1. El comentario de encabezado HTML: `<!-- ID: YYYYMMDD-HHMM.1 -->`
2. La etiqueta de título HTML: `<title>GitHub Fork Sync Tool YYYYMMDD-HHMM.1</title>`
3. El comentario dentro de la etiqueta de script JS: `/* ID: YYYYMMDD-HHMM.2 */`
