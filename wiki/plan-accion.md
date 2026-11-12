---
titulo: Plan de acción (Fase 0)
tipo: decision
estado: borrador
actualizado: 2026-11-11
fuentes: [docs/data_dictionary.md, data/raw/usuarios.csv, data/raw/publicaciones.csv, data/raw/interacciones.csv, data/raw/relaciones.csv, data/raw/diccionario_datos.csv]
---

# Plan de acción: pendiente de aprobación

Las cifras provienen del perfilado del 2026-11-11 ([diccionario](../docs/data_dictionary.md)). [O] = observado · [S] = supuesto · [F] = faltante.

## 1. Resumen y modelo conceptual

**Resumen [O].** Hay cuatro tablas limpias en forma: sin vacíos, sin duplicados y con integridad referencial del 100 %. Describen una red cerrada y sintética de **650 perfiles** que publican **5 200** contenidos (ene-2024 a sep-2025), reciben **35 810** interacciones (0–14 días tras publicar) y forman **8 133** relaciones de seguimiento dirigidas (2022 a ago-2026). Los problemas no son de forma sino de **coherencia semántica**: el 21–45 % de los eventos son anteriores a `fecha_registro` (Q1–Q3), las coordenadas no corresponden al país (Q5) y el grafo de seguimiento está casi desacoplado del engagement (solo el 2,3 % de las interacciones vienen de seguidores del autor; A4).

**Modelo conceptual.**

```
                  SIGUE (relaciones: seguidor_id → seguido_id, fecha)
              ┌──────────────────────────────────────────┐
              ▼                                          │
        ┌──────────┐   PUBLICA (autor_id)   ┌──────────────────┐
        │ USUARIO  │───────────────────────►│   PUBLICACIÓN    │
        │ 650      │                        │ 5 200            │
        │ país,    │◄───────────────────────│ tipo, tema,      │
        │ nicho,   │  INTERACTÚA            │ calidad,         │
        │ verific. │  (interacciones:       │ impresiones      │
        └──────────┘   tipo, fecha)         └──────────────────┘

Grafos derivados:
  G_follow:  usuario → usuario      (8 133 aristas, 106 pares mutuos)
  G_engage:  usuario → autor        (33 557 pares; peso = interacciones)
```

Una misma persona tiene dos roles: **cuenta** (autor o nodo seguido: P1) y **seguidor** (consumidor o nodo que sigue: P2).

## 2. Métricas preliminares

Leyenda de factibilidad: ✅ calculable con los datos actuales · ⚠️ calculable con limitaciones · ❌ no calculable.

### 2.1 Influencia (por cuenta: P1)

| ID | Métrica | Definición | Factib. | Nota |
|---|---|---|---|---|
| I1 | Seguidores | grado de entrada en G_follow | ✅ | Solo dentro de la muestra (S4). Rango 1–39. |
| I2 | Centralidad de red | PageRank sobre G_follow | ✅ | Hay que elegir entre implementación propia (numpy) o `networkx` (D11). |
| I3 | Alcance | suma y mediana de `impresiones` por autor | ✅ | Simulado. No hay alcance único (F3). |
| I4 | Engagement recibido | interacciones recibidas, totales y ponderadas por tipo | ✅ | Los pesos por tipo son una decisión (D3). |
| I5 | Tasa de engagement | interacciones / impresiones | ✅ | Controla la exposición (A2: correlación 0,82). |
| I6 | Audiencia activa única | usuarios distintos que interactúan con el autor | ✅ | |
| I7 | Amplificación | compartidos por publicación | ⚠️ | No hay cascadas (F4): solo es un conteo. |
| I8 | Calidad media | media de `calidad_contenido` | ⚠️ | Origen desconocido (F5). ¿Se incluye? (D12). |
| I9 | Constancia | publicaciones por mes activo en la ventana | ✅ | |
| I10 | Crecimiento de audiencia | nuevos seguidores en la ventana | ⚠️ | Fechas poco confiables (Q3) y sin bajas (F2). |
| I11 | Influencia temática | I1–I6 dentro del nicho propio | ✅ | El 81,9 % de las publicaciones coincide con el nicho del autor. |
| — | Alcance externo, viralidad, sentimiento, conversión | — | ❌ | Faltan datos (F2–F4). |

**Score preliminar de potencial de influencia (versión `influencia_v1`, a aprobar en D4):** media ponderada de percentiles en cuatro pilares: *Red* (I1, I2), *Alcance* (I3), *Resonancia* (I5, I6) y *Amplificación* (I7). La propuesta inicial es igual peso por pilar, con análisis de sensibilidad. `verificado` queda **fuera** del score y se usa como validación externa (A1: las cuentas verificadas deberían salir más arriba, pero no de forma automática).

### 2.2 Actividad (por perfil: insumo de P2)

| ID | Métrica | Definición | Factib. |
|---|---|---|---|
| A1 | Interacciones emitidas | conteo total y por tipo | ✅ (10–106 por usuario) |
| A2 | Actividad ponderada | Σ peso(tipo) | ✅ (D3) |
| A3 | Días activos / regularidad | días distintos con interacción; desviación del intervalo entre ellos | ✅ (solo fecha) |
| A4 | Diversidad | autores y temas distintos con los que interactúa | ✅ |
| A5 | Profundidad | proporción de comentarios, compartidos y guardados frente a "me gusta" | ✅ |
| A6 | Recencia | días entre la última interacción y la fecha de corte | ⚠️ depende de D1 |
| A7 | Producción propia | publicaciones del perfil | ✅ |

### 2.3 Reciprocidad (por perfil y por par: insumo de P2)

| ID | Métrica | Definición | Factib. | Nota |
|---|---|---|---|---|
| R1 | Seguimiento mutuo | % de las cuentas que sigue que lo siguen de vuelta | ✅ | Señal escasa: 106 pares; solo 178 perfiles tienen ≥1 mutuo. |
| R2 | Reciprocidad de interacción | % de los autores con los que interactúa que también interactúan con él | ✅ | 9,4 % de los pares (3 140). |
| R3 | Reciprocidad cruzada | A sigue a B y B interactúa con A (o al revés) | ✅ | |
| R4 | Reciprocidad temporal | días hasta el seguimiento de vuelta | ⚠️ | Fechas poco confiables (Q3). |

### 2.4 Valor de seguidores (P2)

| ID | Métrica | Definición | Factib. | Nota |
|---|---|---|---|---|
| V1 | Compromiso con la cuenta seguida | interacciones del seguidor sobre el contenido del seguido, posteriores al seguimiento | ⚠️ | Solo 773 de 8 133 aristas tienen alguna (A4). |
| V2 | Influencia propia del seguidor | I1/I2 del seguidor ("seguidor que amplifica") | ✅ | |
| V3 | Actividad general | A1–A5 | ✅ | |
| V4 | Reciprocidad | R1–R3 | ✅ | |
| V5 | Afinidad temática | nicho del seguidor = nicho de la cuenta, o % de interacciones en ese tema | ✅ | |
| — | Valor económico, retención, conversión | — | ❌ | Faltan datos (F2, F6). |

**Score preliminar de valor del seguidor (`valor_seguidor_v1`, a aprobar en D5):** dos niveles. (a) **Global por perfil**: actividad (A) + reciprocidad (R) + influencia propia (V2). (b) **Por relación seguidor→cuenta**: V1 + V4 + V5 para listar los mejores seguidores de cada cuenta influyente. Se define "alto valor" como percentil ≥ 90 (D13).

## 3. Arquitectura propuesta

```
data/raw/                     (inmutable; lote inicial en la raíz + entregas/<fecha>_<desc>/)
   │  validar-crudos: hash, esquema, PK/FK, rangos  → si hay un fallo crítico, la corrida se detiene
   ▼
data/processed/
   _registro/ingestas.csv     lote_id, archivo, tabla, sha256, filas, fecha_ingesta, estado
   staging/                   cada lote tipado + lote_id (solo anexión)
   modelo/                    tablas consolidadas "vigentes" + columnas flag_*:
                                dim_usuario (SCD2 si cambian atributos), fact_publicacion,
                                fact_interaccion, fact_relacion (primer_lote, ultimo_lote, activa)
   features/                  metricas_cuenta, metricas_perfil, aristas_seguidor_cuenta
   ▼
data/outputs/
   runs/<run_id>/             scores, rankings, sensibilidad, validaciones, manifest.json
   historico/                 métricas y rankings por run_id + fecha_corte (solo anexión)
   latest                     puntero a la última corrida válida
   ▼
reports/<run_id>/  ·  dashboards/ (lee latest)  ·  wiki/ (hallazgos citando run_id)
```

- **Cómputo:** recálculo completo desde todos los lotes en cada corrida. Con menos de 50 000 filas es instantáneo, y es más simple y reproducible que un cómputo incremental. La incrementalidad está en la **ingesta y el histórico**, no en el cálculo.
- **Formato:** Parquet en `processed/` y `outputs/`, más CSV para los rankings legibles (D7: requiere `pyarrow`).
- **Configuración:** archivo versionado con fecha de corte, ventana, pesos, top-N y umbrales.
- **Código (`src/`):** `io_raw`, `validacion`, `ingesta`, `modelo`, `metricas`, `scores`, `historico`, `reportes`, más un CLI `uv run python -m src.pipeline run`.

## 4. Skills reutilizables

| Skill (`skills/<nombre>/SKILL.md`) | Propósito | Fase |
|---|---|---|
| `validar-crudos` | Hashes, esquema, PK/FK, rangos, flags temporales; informe de validación. | 1 |
| `ingestar-lote` | Registrar un CSV nuevo, detectar a qué tabla pertenece y si es foto completa o incremental, anexarlo a staging y reportar altas, cambios y conflictos. | 1, 5 |
| `construir-modelo` | Consolidar las tablas vigentes, con SCD2 y flags. | 2 |
| `calcular-metricas` | Métricas I/A/R/V versionadas. | 3 |
| `generar-scores-rankings` | Normalización, pesos, rankings top-N y sensibilidad. | 4 |
| `comparar-corridas` | Diferencias entre dos `run_id`: altas y bajas del top-N, Spearman/Kendall, cambios por componente. | 5 |
| `generar-reporte` | Informe Markdown/HTML de una corrida. | 6 |
| `mantener-wiki` | Operaciones ingest/query/lint según AGENTS.md §5. | Transversal |

## 5. Estrategia para nuevos CSV sin perder histórico

1. **Llegada:** cada entrega va a `data/raw/entregas/<AAAA-MM-DD>_<desc>/`, con los nombres de archivo estándar. Nunca reemplaza nada (AGENTS.md §2).
2. **Registro:** se calculan el hash y los conteos y se registra el lote en `_registro/ingestas.csv`. Si un hash ya existe, el archivo es un duplicado y se omite. Si un archivo ya registrado cambió, la corrida se detiene.
3. **Validación de esquema:** columnas y tipos iguales al contrato. Si hay columnas nuevas, se avisa y se piden instrucciones; no se adaptan en silencio.
4. **Consolidación por tabla (D6):**
   - `publicaciones` e `interacciones` (eventos): se anexan y se deduplican por PK. Si llega la misma PK con contenido distinto, es un **conflicto**: se registra y gana el lote más reciente, conservando la versión anterior en staging.
   - `usuarios` (dimensión): SCD2 con `vigente_desde_lote` y `vigente_hasta_lote` para cambios de nicho, país o `verificado`.
   - `relaciones`: si el lote es **foto completa**, las aristas que desaparecen se marcan `activa=false` (baja inferida). Si es **incremental**, solo se agregan.
5. **Recalcular, versionar y comparar:** nueva corrida → nuevo `run_id` → se anexa al histórico → informe `comparar-corridas` frente a la corrida anterior → página de hallazgo y entrada `ingest` en la wiki.
6. **Prueba:** en la Fase 5 se simula un segundo lote con *fixtures* en `tests/` (nunca en `data/raw/`) para verificar que nada anterior se pierde.

## 6. Fases, validaciones y entregables

| Fase | Contenido | Validaciones clave | Entregables |
|---|---|---|---|
| **0** Diagnóstico ✅ | Perfilado, diseño | — | `docs/data_dictionary.md`, `AGENTS.md`, wiki base, este plan |
| **1** Fundamentos | Dependencias, configuración, `src/` base, validador, registro del lote inicial, tests | Hashes = línea base; 650/5 200/35 810/8 133 filas; PK únicas; FK al 100 % | `skills/validar-crudos`, `skills/ingestar-lote`, informe de validación, `wiki/fuentes/lote-inicial.md` |
| **2** Modelo procesado | Tablas tipadas + `flag_*` | Sin pérdida de filas; conteos de flags = Q1 1 098, Q2 7 359, Q3 3 687 | `processed/modelo/*`, `skills/construir-modelo` |
| **3** Métricas | I, A, R, V (v1) | Σ interacciones recibidas = Σ emitidas = 35 810; 106 pares mutuos; PageRank suma 1; tests sobre un grafo de juguete | `features/*`, páginas `wiki/metricas/*` |
| **4** Scores y rankings | Normalización, pesos, sensibilidad, contraste con `verificado` | Rangos [0,1] o [0,100]; estabilidad del top-N ante ±20 % de pesos; sin `NaN` | `runs/<run_id>/`, rankings P1 y P2, `wiki/hallazgos/*` |
| **5** Incrementalidad | Histórico, comparación de corridas, simulación de lote 2 | Hashes del lote 1 sin cambios; corrida 1 intacta; diferencias coherentes | `skills/comparar-corridas`, informe de diferencias |
| **6** Reportes y dashboard | Informe por corrida, dashboard local | El dashboard lee solo de `latest`; las cifras cuadran con los outputs | `reports/`, `dashboards/` |
| **7** Cierre | Lint de la wiki y documentación de uso | Sin huérfanas ni cifras sin fuente | Wiki consolidada, guía de ejecución |

Cada fase termina con una entrada en `wiki/log.md` y un punto de revisión con el usuario.

## 7. Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Datos sintéticos con grafo y engagement independientes (A4) | V1 y la reciprocidad tienen poca señal, y los rankings de P2 pueden parecer arbitrarios | Reportar la escasez; apoyar P2 en actividad e influencia propia; separar el valor global del valor por relación |
| Pesos arbitrarios en los scores | Rankings discutibles | Pesos en la configuración, sensibilidad, descomposición por componente |
| `fecha_registro` incoherente (Q1–Q4) | Antigüedad y recencia sesgadas | Excluirla de las métricas; marcarla con flags |
| `verificado` ficticio pero muy correlacionado (A1) | Circularidad si se usa | Solo como contraste |
| Ventanas temporales desalineadas (A7, Q6) | Mezcla de periodos | Fecha de corte y ventana comunes (D1) |
| Contrato de nuevos CSV desconocido (F7) | Ingesta errónea (foto completa vs incremental) | Validación estricta y detención ante lo inesperado |
| Red cerrada de 650 perfiles (S4) | Métricas de red no extrapolables | Declararlo en cada reporte |

## 8. Preguntas y decisiones pendientes

| ID | Decisión | Propuesta por defecto |
|---|---|---|
| D1 | Fecha de corte y ventana de análisis | Corte en **2025-10-01** (último evento de engagement). Ventana 2024-01-01 → 2025-10-01. Seguimientos posteriores excluidos de V1/R y conservados para I1 con un flag. |
| D2 | Definición de reciprocidad | Usar las dos (R1 seguimiento y R2 interacción) como componentes separados. |
| D3 | Pesos por tipo de interacción | Me_gusta 1 · Guardado 2 · Comentario 3 · Compartido 4, con sensibilidad. |
| D4 | Composición del score de influencia | Cuatro pilares con igual peso sobre percentiles (§2.1). |
| D5 | Alcance de "seguidor de alto valor" | Ambos niveles: global por perfil y por relación seguidor→cuenta. |
| D6 | Ubicación y contrato de los nuevos CSV | `data/raw/entregas/<fecha>_<desc>/`. Hay que confirmar por archivo si es foto completa o incremental. |
| D7 | Formato y versionado de outputs | Parquet + CSV de rankings. En git: código, configuración, manifiestos y rankings pequeños; Parquet pesados en `.gitignore`. |
| D8 | Tecnología del dashboard | HTML estático con Plotly (sin servidor) frente a Streamlit (interactivo, más dependencias). |
| D9 | Coordenadas y `fecha_registro` | Excluirlas de las métricas. |
| D10 | Uso de `verificado` | Solo como contraste o validación. |
| D11 | Dependencias nuevas | `pyarrow`, `networkx`, `pytest` (+ `plotly`/`streamlit` según D8). |
| D12 | ¿Incluir `calidad_contenido` en el score de influencia? | No en v1; reportarla aparte hasta conocer su origen (F5). |
| D13 | Umbral de "alto valor" y tamaño del top-N | Percentil ≥ 90; top-20 en los reportes. |

Preguntas abiertas para el usuario: ¿existe un criterio de negocio de "influencia" o "alto valor" (p. ej., objetivo de campaña, nicho prioritario)? ¿Cada cuánto llegarán los nuevos datos y en qué forma? ¿Para quién son los reportes y el dashboard (docente o evaluación, negocio)?

## 9. Punto de decisión

Antes de comenzar la Fase 1 necesito tu aprobación de:

1. **El plan por fases** (§6) y el principio de recálculo completo con histórico solo de anexión (§3, §5).
2. **Las decisiones de diseño D1–D13**, ya sea aceptando los valores por defecto o indicando cambios. Las críticas para empezar son **D1** (fecha de corte), **D6** (ubicación y contrato de los nuevos CSV), **D7** (formato y versionado) y **D11** (dependencias). D2–D5, D12 y D13 pueden confirmarse antes de la Fase 3.
3. **Las reglas de `AGENTS.md`**, en especial la ubicación `data/raw/entregas/` para nuevos lotes y la política de no usar `verificado`, coordenadas ni `fecha_registro` en los scores.
4. **Si se hace commit** de la Fase 0 en la rama actual.
