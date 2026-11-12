# AGENTS.md: reglas del proyecto (esquema del LLM-Wiki)

Este archivo es el **esquema** del proyecto en el sentido de `docs/llm-wiki.md`. Define cómo debe trabajar cualquier agente (Claude Code, Codex, etc.) o persona en este repositorio. Léelo completo antes de cualquier tarea y, si una regla cambia, actualízalo en el mismo commit.

## 0. Propósito

Proyecto local de Business Intelligence sobre una red social **sintética** de 650 perfiles. Preguntas de negocio:

1. ¿Qué cuentas tienen mayor **potencial de influencia**?
2. ¿Qué perfiles son **seguidores de alto valor** por actividad y reciprocidad?
3. ¿Cómo se **actualizan** los resultados al recibir nuevos datos?

Detalle y criterios de éxito: `wiki/objectives.md`. Diagnóstico de los datos: `docs/data_dictionary.md`.

**Fase actual: 0, diagnóstico y diseño.** No se implementan pipeline, scores, rankings ni dashboard hasta que el usuario apruebe el plan (ver `wiki/log.md`). Avanzar de fase requiere aprobación explícita del usuario.

## 1. Estructura del repositorio

```
data/
  raw/          FUENTE DE VERDAD. Inmutable. Solo lectura.
  processed/    Datos limpios, tipados y enriquecidos con flags. Regenerables desde raw/.
  outputs/      Resultados versionados por corrida (métricas, scores, rankings).
docs/           Documentación técnica estable (diccionario de datos, guía llm-wiki).
wiki/           Base de conocimiento mantenida por el LLM (index, log, objetivos, hallazgos).
skills/         Procedimientos reutilizables (un directorio por skill con SKILL.md).
src/            Código Python del pipeline (paquete importable).
reports/        Informes generados (markdown/HTML) a partir de outputs/.
dashboards/     Dashboard local que lee solo de data/outputs/.
AGENTS.md       Este archivo.
```

Flujo permitido: `raw → processed → outputs → reports/dashboards → wiki`. **Ninguna capa escribe en una capa anterior.**

## 2. Preservación de los datos crudos (`data/raw/`)

1. **Prohibido** modificar, renombrar, mover, eliminar, sobrescribir o "corregir" cualquier archivo de `data/raw/`. Esto incluye abrirlo en modo escritura, reformatearlo o cambiarle la codificación.
2. Todo el código lee `data/raw/` en **solo lectura**. Las correcciones se expresan como *flags* o reglas en `processed/`, nunca editando el crudo.
3. Los **nuevos CSV** se agregan como archivos nuevos y nunca reemplazan a uno existente. Ubicación propuesta: `data/raw/entregas/<AAAA-MM-DD>_<descripcion>/`, sujeta a aprobación (ver plan). Los 5 archivos actuales en la raíz de `data/raw/` forman el **lote inicial** y se quedan donde están.
4. Antes de procesar, calcula el SHA-256 de cada archivo crudo y compáralo con el registro de ingestas. Si un archivo ya registrado cambió de hash, **detén el proceso e informa**; no lo proceses.
5. Si un crudo tiene errores, se documentan en `docs/data_dictionary.md` §5 y en la wiki; no se arreglan en origen.

## 3. Reproducibilidad

1. Entorno gestionado con **uv**. Python se ejecuta siempre como `uv run python ...` desde la raíz del proyecto. Las dependencias se agregan con `uv add` y `uv.lock` se versiona.
2. Todo resultado debe poder regenerarse desde `data/raw/` con un comando documentado (sin pasos manuales, sin notebooks como única fuente).
3. Usa rutas relativas a la raíz del proyecto, sin rutas absolutas de la máquina.
4. Fija las semillas de cualquier proceso aleatorio y regístralas en el manifiesto de la corrida.
5. Los parámetros (pesos, fecha de corte, ventanas, umbrales) viven en un archivo de configuración versionado (propuesto: `src/config.yaml` o `.toml`), nunca *hardcodeados* en el código.
6. Los tipos se declaran explícitamente al leer los CSV (IDs como `string`, fechas con formato `%Y-%m-%d`), sin depender de la inferencia.
7. El pipeline es **idempotente**: correrlo dos veces con los mismos crudos y la misma configuración produce resultados idénticos (mismos hashes en los outputs).
8. Toda transformación tiene validaciones automáticas (esquema, claves, FK, rangos, conteos). Una validación crítica fallida detiene la corrida.

## 4. Versionado de resultados (`data/outputs/`)

1. Cada corrida escribe en un directorio nuevo: `data/outputs/runs/<run_id>/`, con `run_id = AAAAMMDDTHHMMSS_<sha corto de git>`. **Nunca se sobrescribe una corrida anterior.**
2. Cada corrida incluye un `manifest.json` con: `run_id`, fecha, commit de git, si había cambios sin commitear, hashes SHA-256 de cada crudo usado, lotes incluidos, versión de Python y de los paquetes, parámetros completos, conteos de filas por tabla y resultado de las validaciones.
3. `data/outputs/latest` (archivo de texto con el `run_id`, o enlace simbólico) apunta a la última corrida válida. Dashboards y reportes leen desde ahí.
4. Las métricas por entidad se acumulan además en una tabla histórica (`data/outputs/historico/`) con columnas `run_id` y `fecha_corte`, para comparar rankings entre corridas.
5. Si cambia la **definición** de una métrica, se incrementa su versión (`influencia_v1` → `influencia_v2`) y se registra en `wiki/log.md` y en la página de la métrica.
6. Git: se versionan código, configuración, docs, wiki, skills y manifiestos. Los outputs pesados se versionan según la decisión pendiente D7 del plan (por defecto, solo `manifest.json` y los resúmenes pequeños). Haz commit solo cuando el usuario lo pida, con mensajes descriptivos en español.

## 5. Mantenimiento de la wiki (`wiki/`)

La wiki es la memoria acumulativa del proyecto. El LLM la escribe y la mantiene; el usuario la lee y la dirige.

### 5.1 Páginas base obligatorias
- `wiki/index.md`: catálogo de **todas** las páginas, cada una con enlace y resumen de una línea, agrupadas por categoría. Se actualiza en cada operación que cree o elimine páginas.
- `wiki/log.md`: registro cronológico **solo de anexión**. Nunca se editan ni borran entradas pasadas; las correcciones se agregan como entradas nuevas.
- `wiki/objectives.md`: preguntas de negocio, alcance, criterios de éxito y decisiones aprobadas.

### 5.2 Formato
- Idioma: **español**. Los nombres de archivo van en minúsculas, con guiones y sin tildes (`metrica-influencia.md`).
- Cada página empieza con frontmatter YAML:
  ```yaml
  ---
  titulo: ...
  tipo: fuente | entidad | metrica | hallazgo | decision | consulta | proceso
  estado: borrador | vigente | obsoleto
  actualizado: AAAA-MM-DD
  fuentes: [data/raw/usuarios.csv, ...]
  run_id: ...        # si cita resultados
  ---
  ```
- Los enlaces internos son relativos en Markdown (`[texto](metricas/influencia.md)`), compatibles con Obsidian.
- Cada cifra citada indica su origen: archivo crudo, o `run_id` + tabla de outputs. **Sin cifras sin fuente.**
- Marca explícitamente lo **[O] observado**, los **[S] supuestos** y la **[F] información faltante**.

### 5.3 Entradas del log
Cada entrada empieza con `## [AAAA-MM-DD] <operación> | <título>`, donde la operación es `setup`, `ingest`, `run`, `query`, `lint`, `decision` o `fix`. Luego vienen viñetas con lo hecho, los archivos tocados y los pendientes. Así funciona `grep "^## \[" wiki/log.md | tail -5`.

### 5.4 Operaciones
- **Ingest** (nuevo CSV): validar → registrar el lote → procesar → crear o actualizar la página de la fuente → actualizar las páginas de métricas y hallazgos afectadas → anotar las contradicciones con resultados previos → actualizar `index.md` → agregar la entrada a `log.md`.
- **Query** (pregunta del usuario): leer primero `index.md`, responder citando páginas y `run_id`, y archivar la respuesta como página nueva si aporta valor duradero.
- **Lint** (periódico o cuando el usuario lo pida): buscar páginas huérfanas, enlaces rotos, cifras sin fuente, afirmaciones obsoletas frente a la última corrida y entradas del índice sin página (y al revés). Registrarlo en el log.
- **Decisión**: toda decisión del usuario se registra en el log y se refleja en `objectives.md` o en una página `decisiones/`.

## 6. Skills (`skills/`)

- Una skill por directorio: `skills/<nombre>/SKILL.md`, con propósito, entradas, salidas, pasos, validaciones y comando de ejecución. Los scripts de apoyo van en `src/` y no se duplican dentro de la skill.
- Las skills se invocan con `uv run python -m src.<modulo>` o un comando equivalente documentado.
- Cuando una skill cambia de comportamiento, se registra en `wiki/log.md`.

## 7. Calidad de datos

- Los problemas Q1–Q11 de `docs/data_dictionary.md` §5 se tratan con columnas `flag_*` en `processed/`; no se eliminan filas en silencio.
- Cualquier exclusión de filas se cuenta y se registra en el manifiesto de la corrida.
- No usar `latitud`/`longitud` (Q5) ni `fecha_registro` (Q1–Q4) para métricas sin una decisión explícita registrada.
- `verificado` es ficticio: no entra en los scores; solo se usa como variable de contraste.

## 8. Privacidad y ética

Los datos son sintéticos, pero `nombre`, `usuario` y `organizacion` se tratan como PII: los reportes y dashboards muestran `usuario_id` o `usuario` y no publican los nombres completos sin necesidad. Ningún dato sale del equipo local (sin APIs ni servicios externos) sin autorización explícita.

## 9. Comportamiento esperado del agente

- Antes de actuar: leer `AGENTS.md`, `wiki/index.md` y las últimas entradas de `wiki/log.md`.
- Ante ambigüedades de negocio (pesos, umbrales, definiciones), **preguntar** en lugar de suponer, y registrar la respuesta.
- No borrar trabajo previo de `wiki/`, `data/outputs/` ni `reports/`; marcarlo como `obsoleto` cuando corresponda.
- Al terminar una tarea, actualizar la wiki (index y log) en la misma sesión.
