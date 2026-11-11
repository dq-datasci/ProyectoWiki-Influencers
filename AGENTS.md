# AGENTS.md — Reglas del proyecto BI de influencia en redes sociales

Este archivo es el **esquema** del proyecto (patrón descrito en `docs/llm-wiki.md`). Todo agente
(Claude Code, Codex u otro) debe leerlo antes de actuar y seguirlo al pie de la letra. Los cambios a
este archivo se proponen al usuario y solo se aplican con su aprobación.

## 0. Propósito

Responder, con datos observables y de forma reproducible:

1. ¿Qué cuentas tienen mayor potencial de influencia?
2. ¿Qué perfiles son seguidores de alto valor por actividad y reciprocidad?
3. ¿Cómo se actualizan los resultados al recibir nuevos datos?

Objetivos detallados y criterios de éxito: `wiki/objectives.md`.

## 1. Estado actual del proyecto

- Fase: **F0 — diagnóstico y diseño**. El plan de acción espera aprobación del usuario.
- Hasta que el usuario apruebe el plan **no** se implementan pipeline, scores, rankings ni dashboards.
- La fase vigente y las decisiones tomadas se registran en `wiki/log.md` y `wiki/decisions/`.

## 2. Estructura de carpetas

| Ruta | Contenido | Quién escribe | ¿Versionado en git? |
|---|---|---|---|
| `data/raw/` | CSV fuente, **inmutables** (fuente de verdad) | Solo el usuario (o el agente al **añadir** un lote nuevo aprobado) | Sí |
| `data/processed/` | Datos tipados, limpios y derivados; regenerables desde `raw` | Código en `src/` | Pendiente de decisión (por defecto no) |
| `data/outputs/` | Resultados por corrida (`runs/<run_id>/`): scores, rankings, metadatos | Código en `src/` | Sí (son pequeños y auditables) |
| `src/` | Código Python del pipeline | Agente | Sí |
| `skills/` | Procedimientos reutilizables para agentes (`skills/<nombre>/SKILL.md`) | Agente | Sí |
| `reports/` | Informes generados por corrida | Código / agente | Sí |
| `dashboards/` | Tableros o vistas | Código / agente | Sí |
| `wiki/` | Base de conocimiento mantenida por el agente | Agente | Sí |
| `docs/` | Documentación de referencia (`data_dictionary.md`, `llm-wiki.md`) | Agente / usuario | Sí |

## 3. Reglas de preservación de datos crudos (obligatorias)

1. **Nunca** modificar, renombrar, mover, eliminar, reordenar, re-codificar ni sobrescribir archivos de
   `data/raw/`. Esto incluye "arreglos" de formato, normalización de saltos de línea y conversiones.
2. Leer `data/raw/` solo en modo lectura. Ningún script puede abrir un archivo de `raw` en modo escritura.
   El código debe escribir **exclusivamente** en `data/processed/`, `data/outputs/`, `reports/` y `dashboards/`.
3. Toda limpieza, corrección o imputación ocurre en `data/processed/` y queda documentada (regla y conteo
   de filas afectadas).
4. Antes de cada corrida, verificar las huellas SHA-256 registradas (`docs/data_dictionary.md` §1 y el
   manifiesto de lotes cuando exista). Si una huella no coincide: **detenerse** y avisar al usuario.
5. Los nuevos datos se **añaden** como un lote nuevo (ver §6); nunca reemplazan un archivo existente.
6. Los problemas de calidad se marcan con banderas (`flag_*`) en `processed`; no se descartan filas en
   silencio. Cualquier exclusión debe estar justificada en `wiki/quality/` y contada en el reporte de la corrida.
7. No publicar datos personales (`nombre`, `organizacion`, `latitud`, `longitud`) en salidas ni reportes;
   identificar perfiles por `usuario_id` y `usuario`.

## 4. Reproducibilidad

1. Entorno gestionado **solo** con `uv` (Python 3.12). Ejecutar con `uv run python ...`; añadir
   dependencias con `uv add <paquete>`. Prohibido `pip` y el Python del sistema. `uv.lock` se versiona.
2. Toda transformación vive en código dentro de `src/` (nada de pasos manuales ni ediciones a mano en
   CSV). Los notebooks, si se usan, son exploratorios y no producen resultados oficiales.
3. Cada corrida es determinista: parámetros explícitos (en particular `fecha_corte`), sin dependencia
   del reloj del sistema, semillas fijas si hay aleatoriedad, orden estable en desempates (por `usuario_id`).
4. Parámetros de negocio (pesos, ventanas, umbrales) en un archivo de configuración versionado, nunca
   embebidos en el código.
5. Cada corrida escribe `metadata.json` con: `run_id`, `fecha_corte`, lotes y huellas de entrada,
   versión de configuración, commit de git (o "sin commit"), versión de Python y de librerías, conteos
   de filas por etapa y resultado de las validaciones.
6. Una corrida con validaciones bloqueantes fallidas no se marca como vigente.

## 5. Versionado de resultados

1. Los resultados nunca se sobrescriben: cada corrida escribe en `data/outputs/runs/<run_id>/`, con
   `run_id = <fecha_corte>_<AAAAMMDDTHHMMSS de ejecución>` (formato definitivo pendiente de aprobación).
2. La corrida vigente se indica con un puntero (`data/outputs/latest.json`), no copiando archivos.
3. Cada corrida nueva incluye una comparación con la vigente anterior (altas/bajas en top-N, cambios de
   posición, correlación de rangos).
4. Cambios de definición de métricas suben la versión de la configuración (`metricas_vX`) y se
   documentan en `wiki/decisions/` y `wiki/log.md`.
5. Commits de git: solo cuando el usuario lo pida; mensajes descriptivos que mencionen `run_id` o lote.

## 6. Incorporación de nuevos datos (resumen operativo)

1. Los CSV nuevos se colocan en una carpeta de lote nueva dentro de `data/raw/` (convención definitiva
   pendiente de aprobación; propuesta: `data/raw/lotes/<AAAAMMDD>_<nn>/`). Los archivos del lote inicial
   permanecen en `data/raw/` y se registran como lote `L0000`.
2. Se registra el lote (huella, filas, esquema, tipo de carga) antes de procesarlo; un archivo con
   huella ya registrada no se reprocesa.
3. Se valida el contrato de esquema; si falla, el lote queda en cuarentena (`data/processed/_cuarentena/`)
   y se informa al usuario.
4. `processed` se reconstruye desde todos los lotes registrados; el histórico de corridas se conserva.
5. Se actualiza la wiki (página del lote, calidad, log) y se documentan cambios en rankings.

## 7. Mantenimiento de la wiki

La wiki (`wiki/`) es la memoria persistente del proyecto. El agente la escribe y la mantiene; el
usuario la lee (p. ej. en Obsidian).

Estructura:

- `wiki/index.md` — catálogo de todas las páginas, con enlace y resumen de una línea. Se actualiza en
  **cada** operación que cree o renombre páginas.
- `wiki/log.md` — registro cronológico **solo de anexión**. Nunca editar ni borrar entradas previas;
  las correcciones se registran como entradas nuevas.
- `wiki/objectives.md` — preguntas de negocio, alcance y criterios de éxito.
- `wiki/sources/` — una página por archivo/lote fuente.
- `wiki/metrics/` — una página por métrica: definición, fórmula, insumos, si es calculable, limitaciones.
- `wiki/quality/` — problemas de calidad (Q01…) con estado (abierto/mitigado/cerrado).
- `wiki/decisions/` — registros de decisión (D01…): contexto, opciones, decisión, fecha, quién aprobó.
- `wiki/runs/` — resumen de cada corrida y su comparación con la anterior.
- `wiki/entities/` — páginas de cuentas destacadas (solo tras resultados aprobados).

Convenciones:

1. Markdown con frontmatter YAML (`titulo`, `tipo`, `actualizado`, `fuentes`, `estado`).
2. Enlaces internos estilo Obsidian `[[ruta/pagina]]` o enlaces markdown relativos; sin páginas huérfanas.
3. Cada afirmación cuantitativa cita su origen (archivo/lote, `run_id` o sección del diccionario).
4. Distinguir explícitamente **observado**, **supuesto** y **decisión**.
5. Formato de entrada de log: `## [AAAA-MM-DD] <operación> | <título>` donde operación ∈
   {`setup`, `ingesta`, `diagnostico`, `decision`, `corrida`, `consulta`, `lint`}.
6. Operaciones: **ingesta** (nuevo lote → páginas de fuente, calidad, índice, log), **consulta**
   (respuestas valiosas se archivan como página nueva), **lint** (contradicciones, páginas obsoletas,
   huérfanas, enlaces rotos, métricas sin página).
7. Si un dato nuevo contradice una afirmación de la wiki, actualizar la página y anotar la contradicción
   en el log; no borrar el historial.

## 8. Calidad del trabajo del agente

- Basarse en datos observables; marcar todo supuesto como tal.
- No inventar métricas que los datos no soportan; si algo no es calculable, decirlo.
- Validar conteos clave tras cada etapa (filas de entrada = filas registradas + filas marcadas).
- Responder al usuario en español.
- Ante ambigüedad de negocio, preguntar antes de implementar.
