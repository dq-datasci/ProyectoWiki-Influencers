---
titulo: Plan de acción presentado por el agente (respuesta textual)
fuente: respuesta final del agente IA independiente, 2026-11-11
nota: copia literal, sin edición
---

# Plan de acción: BI de influencia en redes sociales (fase F0, diagnóstico y diseño)

## Qué hice

Leí `docs/llm-wiki.md` y los 5 CSV de `data/raw/`. Los perfilé con pandas, sin escribir nada en ellos: las huellas SHA-256 de antes y de después coinciden. Todavía no hay pipeline, scores ni rankings. No hice commits.

Archivos creados:
- `/home/dq-datasci/Projects/ProyectoWiki/docs/data_dictionary.md`: inventario, columnas, tipos, ejemplos, faltantes, claves, relaciones, cobertura temporal, 17 problemas de calidad (Q01–Q17), información que falta, supuestos y las huellas SHA-256 del lote inicial.
- `/home/dq-datasci/Projects/ProyectoWiki/AGENTS.md`: reglas para preservar los crudos, reproducibilidad con `uv`, versionado por corrida, incorporación de lotes y mantenimiento de la wiki.
- `/home/dq-datasci/Projects/ProyectoWiki/wiki/index.md`, `wiki/log.md` y `wiki/objectives.md`.
- Carpetas con `.gitkeep`: `wiki/{sources,metrics,quality,decisions,runs,entities}/`, `skills/`, `data/processed/`, `data/outputs/`, `src/`, `reports/`, `dashboards/`.

---

## 1. Resumen y modelo conceptual de los datos

**Inventario (lote L0000)**

| Archivo | Filas | Una fila es | Clave |
|---|---:|---|---|
| `usuarios.csv` | 650 | un perfil (11 atributos) | `usuario_id` |
| `relaciones.csv` | 8 133 | una arista dirigida "A sigue a B" | (`seguidor_id`, `seguido_id`) |
| `publicaciones.csv` | 5 200 | una publicación (formato, tema, calidad, impresiones) | `publicacion_id` |
| `interacciones.csv` | 35 810 | un evento usuario→publicación (Me_gusta / Comentario / Compartido / Guardado) | `interaccion_id` |
| `diccionario_datos.csv` | 7 | metadato; describe solo 7 de las 26 columnas | — |

**Modelo conceptual.** Hay una sola entidad de actor, el **Perfil**, que cumple tres papeles:
- **Nodo seguido**, por su audiencia.
- **Autor**, por su contenido y alcance.
- **Seguidor e interactor**, por su actividad.

Los perfiles se conectan por dos redes:
- **Red de seguimiento** (explícita): 8 133 aristas.
- **Red de interacción** (derivada de interacción ⋈ publicación): 33 557 pares dirigidos "A interactúa con contenido de B".

**Estado de los datos**

En lo estructural están limpios:
- No hay celdas vacías ni filas duplicadas.
- Las claves primarias son únicas e integridad referencial es del 100 %.
- Los IDs son contiguos.
- No hay auto-seguimientos ni auto-interacciones.
- Ninguna interacción es anterior a su publicación.

En lo semántico y temporal hay problemas:

- **Q01, actividad antes del registro.** El 45 % de los seguimientos, el 21 % de las publicaciones y el 21 % de las interacciones ocurren antes de la `fecha_registro` del perfil implicado.
- **Q02, ventanas que no coinciden.** El contenido cubre de 2024-01 a 2025-10; el grafo, de 2022-01 a 2026-08.
- **Q03, interacciones casi separadas del grafo.** Solo el 2,3 % de las interacciones viene de seguidores del autor.
- **Q04, poca reciprocidad de seguimiento.** Hay 106 pares mutuos y el 72,6 % de los perfiles no tiene ninguno.
- **Q05, coordenadas sin sentido.** Latitud y longitud aleatorias, incoherentes con el país.
- **Q07, `verificado` es ficticio.** Aun así correlaciona 0,65 con el número de seguidores.
- **Q15, publicaciones recientes incompletas.** Las interacciones solo llegan entre 0 y 14 días después de publicar, así que las publicaciones más recientes aparecen con menos interacción de la real.

Todo indica que los datos son **sintéticos**. Los resultados servirán para demostrar el método, no para describir un mercado real.

## 2. Métricas preliminares

Sí = se puede calcular hoy. Parcial = se puede calcular con limitaciones. No = faltan datos.

**Influencia (P1), por perfil**

| Métrica | Definición | ¿Calculable? |
|---|---|---|
| I1 Seguidores | Grado de entrada a la fecha de corte | Sí (no hay bajas: Q11) |
| I2 Centralidad | PageRank del grafo de seguimiento (seguidores "importantes" pesan más) | Sí (requiere `networkx` o una implementación propia) |
| I3 Alcance | Impresiones totales y mediana por publicación | Sí (impresiones simuladas y sin definición operativa: Q08) |
| I4 Engagement ponderado | Σ interacciones recibidas × peso por tipo | Sí (los pesos son una decisión pendiente) |
| I5 Tasa de engagement | Interacciones / impresiones por publicación (mediana 1,6 %) | Sí, excluyendo publicaciones censuradas (Q15) |
| I6 Audiencia activa única | Usuarios distintos que interactúan con el autor | Sí |
| I7 Amplificación | Compartidos recibidos por publicación | Sí (sin alcance secundario) |
| I8 Calidad media | Media de `calidad_contenido` | Parcial: variable simulada de origen desconocido (Q09); queda como componente opcional |
| I9 Crecimiento | Nuevos seguidores por periodo | Parcial: solo altas, sin bajas |
| I10 Influencia por nicho | Ranking dentro de cada `nicho` | Sí |
| — | Conversión, sentimiento, historial oficial de seguidores | No |

**Actividad, por perfil**

| Métrica | Definición | ¿Calculable? |
|---|---|---|
| A1 Volumen | Interacciones realizadas, total y por tipo | Sí |
| A2 Recencia y constancia | Días desde la última interacción; meses activos | Sí (granularidad diaria) |
| A3 Profundidad | % de interacciones de mayor esfuerzo (comentario, compartido, guardado) | Sí |
| A4 Diversidad | Autores distintos con los que interactúa | Sí |
| A5 Producción | Publicaciones por mes | Sí |
| A6 Actividad dirigida | Interacciones con cuentas que sigue | Sí, pero escasa (Q03) |
| — | Tiempo de sesión, visualizaciones | No |

**Reciprocidad**

| Métrica | Definición | ¿Calculable? |
|---|---|---|
| R1 Mutuo | Indicador por arista de seguimiento mutuo | Sí (106 pares) |
| R2 Tasa de reciprocidad | Mutuos / seguidos y mutuos / seguidores | Sí, con baja varianza (Q04) |
| R3 Reciprocidad de interacción | A interactúa con B y B con A | Sí; señal más rica: 3 140 pares dirigidos recíprocos |
| R4 Follow-back | B sigue a A después de A, y con qué latencia | Parcial: depende de la fiabilidad de las fechas |

**Valor de seguidor (P2)**

| Métrica | Definición | ¿Calculable? |
|---|---|---|
| V1 Valor global del seguidor | Combinación de actividad (A1–A4) + reciprocidad (R2–R3) + influencia propia (I1/I2) + amplificación (compartidos realizados) | Sí |
| V2 Valor del seguidor para una cuenta | Interacción del seguidor con esa cuenta + mutualidad + influencia propia | Parcial: solo el 9,5 % de las aristas tiene interacción |
| V3 Alcance de segundo grado | Seguidores del seguidor | Sí |
| — | Valor monetario, segmentación geográfica | No (coordenadas inválidas) |

Pesos que propongo para el engagement, a aprobar: Me_gusta 1, Guardado 2, Comentario 3, Compartido 4.

Agregación: normalizar con percentiles (las distribuciones son muy asimétricas; por ejemplo, las impresiones van de 20 a 7 336) y sumar con pesos. Cada componente se publica por separado.

`verificado` queda fuera de los scores y se usa solo como contraste de validación.

## 3. Arquitectura propuesta de datos y resultados

```
data/raw/                      Inmutable. L0000 = archivos actuales (no se mueven)
  lotes/<AAAAMMDD>_<nn>/       Lotes nuevos (propuesta)
data/processed/                Regenerable
  _control/manifiesto_lotes.csv  lote, archivo, sha256, filas, esquema, tipo_carga, fecha_recepcion
  _cuarentena/                 Lotes que no cumplen el contrato
  staging/                     Tipado, con linaje (_lote_id, _archivo, _sha256) y banderas flag_*
  core/                        dim_usuario (SCD2), fact_seguimiento, fact_publicacion, fact_interaccion
  features/<fecha_corte>/      Métricas por perfil, por arista y por publicación
data/outputs/
  runs/<run_id>/               scores_influencia, valor_seguidor, rankings (global y por nicho),
                               validaciones.json, comparacion_vs_anterior.csv, metadata.json
  latest.json                  Puntero a la corrida vigente
src/                           ingest, contracts/validate, transform, features, scores, compare, report, pipeline (CLI)
config/  (propuesta)           metricas_v1.yaml con pesos, ventanas, umbrales y fecha_corte
tests/   (propuesta)           pytest con fixtures sintéticos (nunca dentro de raw)
reports/<run_id>/              Informe en markdown/HTML por corrida
dashboards/                    Tablero que lee data/outputs/latest
wiki/                          sources, metrics, quality, decisions, runs, entities + index/log/objectives
```

Principios:
- **Reconstrucción completa y determinista** de `processed` desde todos los lotes registrados. Con unas 50 000 filas tarda segundos, y es más simple y auditable que hacer upserts incrementales.
- **Todo se calcula "a fecha de corte"** (as-of), con parámetros explícitos.
- **Salidas inmutables por corrida.**

Formato intermedio: Parquet, que requiere `uv add pyarrow`, o CSV.

## 4. Skills reutilizables

Cada una sería `skills/<nombre>/SKILL.md`, con instrucciones y el comando `uv run` correspondiente.

| Skill | Propósito |
|---|---|
| `perfilar-csv` | Perfilar uno o varios CSV y actualizar `docs/data_dictionary.md` |
| `ingerir-lote` | Registrar un lote nuevo: huella, manifiesto, contrato de esquema, cuarentena si falla, página en `wiki/sources/`, entrada en el log |
| `validar-calidad` | Ejecutar los contratos y reglas Q01–Q17, generar el reporte de validación y actualizar `wiki/quality/` |
| `ejecutar-corrida` | Correr el pipeline para una `fecha_corte` y una configuración, escribir `runs/<run_id>/` y actualizar `latest.json` si todo pasa |
| `comparar-corridas` | Altas y bajas en el top-N, cambios de posición, Spearman/Kendall entre corridas |
| `mantener-wiki` | Operaciones de ingesta, consulta y archivo de respuestas según `llm-wiki.md` |
| `lint-wiki` | Detectar contradicciones, páginas huérfanas, enlaces rotos, afirmaciones sin fuente y métricas sin página |
| `registrar-decision` | Crear un registro de decisión D0x en `wiki/decisions/` y anotarlo en el log |

## 5. Estrategia para incorporar nuevos CSV sin perder histórico

1. **Solo se añade, nunca se reemplaza.** Cada entrega va a una carpeta de lote nueva. Los archivos actuales se quedan donde están y se registran como L0000.
2. **Manifiesto con huellas.** Si una huella ya está registrada, el archivo no se reprocesa (idempotencia). Si cambia la huella de un archivo ya registrado, el proceso se bloquea y se avisa.
3. **Contrato de esquema por entidad**: columnas, tipos, dominios y patrones de ID. Si no se cumple, el lote va a cuarentena y no hay fallo silencioso.
4. **Semántica de fusión por entidad**:
   - `interacciones` y `publicaciones`: eventos solo de anexión por ID. Si un mismo ID llega con contenido distinto, es un conflicto: se registra y gana el lote más reciente, conservando la versión anterior.
   - `relaciones`: la clave es el par. Falta saber si cada entrega es una **foto completa** (lo que desaparece cuenta como baja) o un **delta**; es la pregunta abierta Q11.
   - `usuarios`: SCD tipo 2 sobre los atributos que cambian (`verificado`, `nicho`, `pais`).
5. **Corridas inmutables y comparables.** Cada corrida declara qué lotes y qué `fecha_corte` usó. Las corridas previas nunca se tocan y cada corrida nueva trae su comparación con la vigente.
6. **Regla de censura.** Las publicaciones con menos de 14 días respecto a la `fecha_corte` no entran en las tasas de engagement.
7. **Wiki.** Cada lote genera su página de fuente, actualiza la calidad, el índice y el log, y deja una página en `wiki/runs/`.

## 6. Fases de implementación, validaciones y entregables

| Fase | Contenido | Validaciones clave | Entregables |
|---|---|---|---|
| F0 ✔ | Diagnóstico y diseño | Huellas sin cambios | Diccionario, AGENTS.md, estructura, wiki base, este plan |
| F1 Fundaciones | Dependencias (`uv add`), `config/metricas_v1.yaml`, manifiesto con L0000, esqueletos de skills, registros de decisión D01… de lo que se apruebe | `sha256sum -c` OK; `uv run` funciona desde cero | Manifiesto, configuración, `skills/*`, `wiki/decisions/` |
| F2 Ingesta y staging | Lectura tipada, linaje, banderas Q01/Q15, contratos | Filas por archivo = 650 / 8 133 / 5 200 / 35 810; integridad referencial 100 %; conteos de banderas iguales al diccionario (p. ej. 3 687 seguimientos previos al registro) | `processed/staging`, reporte de calidad, `wiki/sources/`, `wiki/quality/` |
| F3 Core y features | Modelo dimensional; métricas I, A, R y V | Σ grado de entrada = 8 133; pares mutuos = 106; totales por tipo de interacción cuadran; ningún valor imputado | `processed/core`, `features/<fecha_corte>`, `wiki/metrics/` |
| F4 Scores y rankings | Influencia, valor de seguidor (global y por cuenta), por nicho | Sensibilidad de pesos (±25 %, solapamiento del top-20, Spearman); contraste con `verificado` excluido; determinismo (dos corridas iguales dan el mismo hash) | `outputs/runs/<run_id>/`, `latest.json`, `wiki/runs/` |
| F5 Reportes y dashboard | Informe por corrida y tablero | Las cifras del tablero coinciden con `outputs` | `reports/<run_id>/`, `dashboards/` |
| F6 Prueba incremental | Lote de prueba sintético (en `tests/fixtures`) con altas, duplicados y un esquema roto | Idempotencia; la cuarentena funciona; raw y corridas previas intactos; se genera la comparación | Tests con pytest y documentación del proceso P3 |
| F7 Cierre | Lint de la wiki, páginas `entities/` del top-N y síntesis final | Sin huérfanas ni afirmaciones sin fuente | Wiki consolidada y respuesta documentada a P1–P3 |

## 7. Riesgos, preguntas y decisiones pendientes

**Riesgos**
- Datos sintéticos con variables "oráculo" (`calidad_contenido`, `verificado`) que pueden dictar el ranking.
- Fechas de registro poco fiables (Q01).
- La reciprocidad de seguimiento discrimina poco (Q04).
- La escasez de interacción dentro del grafo (Q03) debilita V2.
- La censura de borde (Q15).
- Sesgo por país: Bolivia es el 40,6 % de los perfiles.
- Pesos arbitrarios sin una "verdad de referencia" con la que validarlos.

**Preguntas para el proveedor de datos o para ti**
1. ¿Los nuevos `relaciones.csv` serán fotos completas o deltas? ¿Habrá bajas?
2. ¿Qué miden exactamente `impresiones` (únicas o totales, acumuladas a qué fecha) y `calidad_contenido`?
3. ¿Cada cuánto y en qué forma llegarán los datos nuevos?
4. ¿"Seguidor de alto valor" significa valor global como seguidor o valor para cuentas concretas (por ejemplo, el top-N de influencia)?
5. ¿Hay que segmentar por nicho y por país en los entregables?

## Punto de decisión

Antes de empezar F1 necesito que apruebes o ajustes lo siguiente:

1. **D01 Fecha de corte.** Recomiendo `2025-10-01`, la última interacción, para alinear contenido y grafo. Así quedan fuera 1 573 seguimientos posteriores (19,3 %). La alternativa es usar ventanas distintas para el grafo y para el contenido.
2. **D02 `fecha_registro`.** Marcarla con bandera pero no usarla para filtrar ni puntuar.
3. **D03 Definición de P2.** Valor global del seguidor (V1) y además valor por cuenta (V2) para el top-N de influencia. Hay que elegir N; propongo 20.
4. **D04 Pesos de interacción**: 1 / 2 / 3 / 4. Y normalización por percentiles.
5. **D05 Variables excluidas del score**: `verificado` (solo como contraste), coordenadas, y `calidad_contenido` como componente opcional sometido a sensibilidad.
6. **D06 Regla de censura** de 14 días para el engagement.
7. **D07 Dependencias a añadir con `uv add`**: `pyarrow` (Parquet), `networkx` (PageRank), `pyyaml` (configuración), `pytest` (dev). La tecnología del dashboard está por decidir: Streamlit o HTML estático con Plotly.
8. **D08 Formato intermedio**: Parquet (recomendado) o CSV.
9. **D09 Versionado en git**: `data/outputs/` y `reports/` se versionan y `data/processed/` va a `.gitignore`. Esto implica editar `.gitignore`.
10. **D10 Convenciones nuevas**: lotes en `data/raw/lotes/<AAAAMMDD>_<nn>/`, manifiesto en `data/processed/_control/`, carpetas extra `config/` y `tests/`, y `run_id = <fecha_corte>_<timestamp>`.
11. **D11 (opcional)**: crear un `CLAUDE.md` mínimo que importe `AGENTS.md`, para que Claude Code cargue las reglas automáticamente.

Con estas decisiones aprobadas, las registro como D01–D11 en `wiki/decisions/` y empiezo F1.