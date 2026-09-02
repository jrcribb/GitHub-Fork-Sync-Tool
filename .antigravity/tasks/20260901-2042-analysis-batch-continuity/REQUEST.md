# Tarea: continuidad del análisis por lotes

## Identidad y línea base
- ID: `20260901-2042-analysis-batch-continuity`.
- Repositorio: `D:\dev\GitHub Fork Sync Tool`.
- Rama base: `master`.
- Commit base: `47f0f6cc2146ba1e31f427d79068d6f61d318b27`.
- Estado previo: los archivos versionados están limpios; `.antigravity/tasks/` contiene únicamente documentos de coordinación sin seguimiento.
- Tarea precedente: `.antigravity/tasks/20260901-1655-review-improvements/`.

## Decisiones del usuario
- Implementar continuidad de análisis mediante una acción «Analizar siguiente lote» que preserve los resultados anteriores.
- Conservar la sincronización forzada automática tal como funciona actualmente. No modificar `syncRepo`, `forceSyncRepo`, la casilla de forzado ni la política ante conflictos.

## Objetivo
Permitir analizar sucesivamente todos los forks en grupos configurados por `Repos/lote`, sin volver a descargar la lista ni reiniciar el análisis desde el primer repositorio en cada lote. El usuario debe poder cargar o recargar la lista para iniciar una sesión nueva y avanzar después con un botón explícito.

## Alcance incluido
- Estado en memoria que indique el siguiente fork pendiente de análisis dentro de la lista cargada.
- Separar la carga/recarga de repositorios del procesamiento del siguiente lote cuando sea necesario para mantener el código claro.
- Botón «Analizar siguiente lote» en la zona de acciones.
- Estados, contadores y habilitación del botón coherentes antes, durante y después de cada lote.
- Actualizar `README.md` para describir el flujo real.
- Validación aislada con datos sintéticos y red totalmente simulada.

## Comportamiento requerido
1. `Cargar / Recargar Forks` debe iniciar una sesión nueva: recuperar la lista completa mediante la paginación existente, reiniciar el cursor, inicializar los estados y analizar el primer lote.
2. Si `Repos/lote` es `N > 0`, el primer lote comprende como máximo los primeros N forks; «Analizar siguiente lote» continúa desde el primer fork no analizado y procesa como máximo N adicionales.
3. Si `Repos/lote` es `0`, la carga analiza todos los forks y no queda un siguiente lote disponible.
4. Avanzar no debe volver a consultar `/user/repos`, vaciar `forks`, perder `cmp`, `parent`, `selected` ni las fechas guardadas, ni volver a analizar elementos ya completados.
5. Al finalizar cada lote, el panel debe informar al menos: total encontrado, analizados acumulados, procesados en el lote actual, pendientes de análisis y cantidad acumulada que requiere actualización.
6. El botón «Analizar siguiente lote» debe estar deshabilitado antes de una carga, mientras se carga o analiza, y cuando no quedan elementos pendientes. Debe habilitarse al terminar un lote si aún quedan forks pendientes.
7. Si el usuario cambia `Repos/lote` entre avances, aplicar el nuevo valor al siguiente lote sin retroceder el cursor. Documentar esta conducta.
8. Una recarga posterior debe descartar de forma intencional la sesión en memoria e iniciar nuevamente desde el primer fork de la lista recién recuperada.
9. Si el listado no contiene forks, mostrar un estado final claro y mantener deshabilitada la acción de avance.
10. Ante un fallo al recuperar la lista, no habilitar «Analizar siguiente lote». Ante un fallo aislado al enriquecer o comparar un fork, evitar que un reintento involuntario bloquee el avance del cursor; conservar el tratamiento visual actual salvo que sea imprescindible para la continuidad.

## Restricciones
- No cambiar la política de sincronización forzada automática ni otros flujos de sincronización.
- No implementar otros hallazgos de la revisión anterior: identidad por nombre, workflows, concurrencia global, logs, accesibilidad u otras mejoras quedan fuera de esta tarea.
- No introducir backend, frameworks, dependencias instaladas ni proceso de compilación.
- No usar tokens reales ni realizar llamadas reales a GitHub durante la validación.
- Preservar los documentos de coordinación y cualquier trabajo preexistente.
- No hacer commit, push, merge, cambio de rama, reset ni descarte de cambios.
- Al modificar `GitHub Fork Sync Tool.html`, cumplir `.agents/AGENTS.md`: actualizar las tres marcas de ID y fecha/hora en el formato indicado.

## Requisitos técnicos y de diseño
- Usar nombres que expresen el cursor y el total analizado; evitar inferir continuidad recorriendo estados visuales del DOM.
- Mantener un único origen de verdad en memoria para el progreso de análisis.
- Evitar duplicar la lógica de enriquecimiento, comparación, selección automática y panel de estado entre el primer lote y los siguientes.
- El botón debe llamar a una función dedicada y manejar promesas/errores sin permitir doble activación accidental de esa misma acción.
- No almacenar el cursor en `localStorage`: la continuidad solicitada corresponde a la sesión actual del documento.

## Criterios de aceptación
1. Con 125 forks sintéticos y `Repos/lote = 50`, la carga analiza 1–50; el primer avance analiza 51–100; el segundo analiza 101–125; no hay nuevas llamadas a `/user/repos` durante los avances.
2. Los resultados de 1–50 permanecen intactos al analizar 51–100 y los de 1–100 permanecen al analizar 101–125.
3. Después del último lote, el botón de avance queda deshabilitado y el panel informa 125 analizados, 0 pendientes.
4. Una recarga reinicia el cursor y vuelve a recuperar la lista; el primer lote de la nueva sesión comienza desde el inicio.
5. Con lote 0, todos los forks se analizan en la carga y el botón nunca queda habilitado al finalizar.
6. Cambiar el tamaño de 50 a 20 después del primer lote analiza exactamente los siguientes 20 forks.
7. Con 0 forks o error de listado, la UI no ofrece un avance inválido.
8. Las marcas de versión del HTML cumplen las reglas del repositorio y README describe «Cargar / Recargar» frente a «Analizar siguiente lote» sin conservar el flujo incorrecto anterior.
9. No hay cambios en funciones de sincronización forzada ni en comportamiento ajeno a esta tarea.

## Validación esperada
- Crear un arnés fuera del código del proyecto o usar otro método aislado reproducible con fetch estricto: toda ruta no simulada debe fallar localmente.
- Ejecutar y reportar los escenarios de los criterios 1–7, incluidos conteos de llamadas y rangos procesados.
- Revisar el diff para confirmar el criterio 9 y el alcance limitado.
- Comprobar sintaxis JavaScript del script extraído o mediante un método equivalente disponible.
- No marcar una prueba como aprobada si sólo fue razonada y no ejecutada.

## Resultados e informe
Crear `RESULT.md` en esta carpeta con:
- estado (`completed`, `partial` o `blocked`);
- resumen de implementación;
- archivos modificados y motivo;
- respuesta criterio por criterio con evidencia;
- comandos o pasos ejecutados y salida resumida;
- decisiones, supuestos y limitaciones;
- estado Git final: rama, HEAD, commits y cambios pendientes exactos.

Detenerse y reportar un bloqueo si la continuidad exige una decisión material no contemplada. No ampliar el alcance por cuenta propia.
