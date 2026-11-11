---
title: "U3-T3 · Proyecto BI con Asistencia AI"
subtitle: "Influenciadores y seguidores: revisión crítica del plan propuesto por el agente IA"
author: "David Andrés Quintela Saavedra · Business Intelligence II"
date: "11 de noviembre de 2026"
lang: es
---

# 1. Resumen ejecutivo

Un agente de IA independiente (Claude Code) ejecutó el prompt propuesto sobre los datos de
[rubendcl/Influencers](https://github.com/rubendcl/Influencers) y la guía
[LLM Wiki de Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Hizo un
diagnóstico de calidad muy sólido, propuso una arquitectura reproducible y versionada, y terminó con
11 decisiones concretas para aprobar. Respetó las dos restricciones de la consigna: no tocó
`data/raw/` (las huellas son idénticas antes y después) y no implementó pipeline, scores ni rankings.

**Mi valoración.** El plan es **adecuado y técnicamente correcto**. Todas las cifras que lo sustentan
las verifiqué de forma independiente y coinciden. Pero **es más complejo de lo necesario** para la
escala del problema: 650 perfiles, unas 50 000 filas, datos sintéticos y un proyecto de curso. Por eso
propongo aprobarlo con una versión mínima (MVP) y algunos ajustes (sección 7).

El hallazgo más valioso del diagnóstico es que **las interacciones están casi desconectadas del grafo
de seguidores**: solo el 2,3 % de las interacciones viene de un seguidor del autor. Esto obliga a
redefinir qué significa "seguidor de alto valor".

# 2. Contexto y método de trabajo

| Elemento | Detalle |
|---|---|
| Carpeta del proyecto | `~/Projects/ProyectoWiki` (creada el 22-oct con `docs/`, `data/raw/` y los datos ya descargados) |
| Datos fuente | 5 CSV de `rubendcl/Influencers/data/raw`. Comprobé el 11-nov que los checksums coinciden con el repo de origen |
| Guía | `docs/llm-wiki.md`, idéntica al gist de Karpathy (checksum verificado) |
| Entorno | Proyecto **uv** (`pyproject.toml` + `uv.lock`, Python 3.12, pandas 3.1). Todo se ejecutó con `uv run` |
| Versionado | Git local con commits por fase, publicado en GitHub: <https://github.com/dq-datasci/ProyectoWiki-Influencers> |
| Ejecución del prompt | Una **instancia nueva de Claude Code, sin contexto previo**, recibió el prompt textual. Así la revisión de este informe no es una autoevaluación. Ver `entrega/prompt_utilizado.md` |
| Respuesta del agente | Copia literal en `entrega/plan_agente.md` |
| Verificación | Recalculé por mi cuenta las cifras clave del diagnóstico (sección 4) y comprobé las huellas de `data/raw/` antes y después |

Pasos de preparación que pide la tarea:

- ✓ Carpeta dedicada al proyecto
- ✓ Subcarpetas `/docs` y `/data`, y `/data/raw` dentro de `/data`
- ✓ CSV fuente copiados en `/data/raw`
- ✓ Guía MD descargada en `/docs`
- ✓ Prompt ejecutado por el agente IA

# 3. Qué se realizó: entregables pedidos frente a entregados

| Pedido en el prompt | ¿Cumplido? | Observación |
|---|:---:|---|
| Leer `docs/llm-wiki.md` y todos los CSV | ✓ | Lo usa explícitamente: el `AGENTS.md` se presenta como el "esquema" del patrón LLM Wiki |
| No modificar `data/raw/` | ✓ | Huellas MD5 y SHA-256 idénticas antes y después |
| No implementar pipeline, dashboard, rankings ni scores | ✓ | `src/`, `skills/`, `dashboards/` y `reports/` quedaron vacíos (solo `.gitkeep`) |
| `docs/data_dictionary.md` | ✓ | 220 líneas. Incluye inventario, columnas, tipos, ejemplos, faltantes, claves, diagrama ER, cobertura temporal, 17 problemas de calidad (Q01–Q17), información faltante, supuestos S1–S6 y huellas SHA-256 |
| `AGENTS.md` | ✓ | 130 líneas: preservación de crudos, reproducibilidad, versionado, incorporación de lotes, mantenimiento de la wiki y calidad del trabajo del agente |
| Estructura `wiki/ skills/ data/processed/ data/outputs/ src/ reports/ dashboards/` | ✓ | Además creó las subcarpetas `wiki/{sources,metrics,quality,decisions,runs,entities}` |
| `wiki/index.md`, `log.md`, `objectives.md` | ✓ | Con frontmatter YAML. El log es de solo anexión y tiene 2 entradas |
| Plan con las 7 secciones + Punto de decisión | ✓ | Las 7 secciones están, y el Punto de decisión trae 11 ítems (D01–D11) |
| Distinguir supuestos, limitaciones y faltantes | ✓ | Marca cada descripción como **(oficial)** o **(inferida)**. Tiene una sección de "Información faltante" y otra de "Supuestos de trabajo" |

**Conclusión:** cumplió el prompt al 100 % y sin excederse en lo prohibido.

# 4. Verificación independiente del diagnóstico

Antes de leer los resultados del agente perfilé los datos por mi cuenta. Después recalculé sus
afirmaciones principales.

| Afirmación del agente | Mi cálculo | ¿Coincide? |
|---|---|:---:|
| 650 / 8 133 / 5 200 / 35 810 filas; sin vacíos ni duplicados; integridad referencial 100 % | Igual | ✓ |
| Q01: 3 687 seguimientos (45,3 %) involucran a un perfil aún no registrado; 1 098 publicaciones y 7 359 interacciones anteriores al registro | 3 687 / 1 098 / 7 359 | ✓ |
| Q03: solo 838 interacciones (2,3 %) vienen de seguidores del autor; solo 773 de 8 133 aristas tienen alguna interacción | 838 / 773 | ✓ |
| Q04: 106 pares mutuos (212 aristas, 2,6 %) | 212 aristas recíprocas | ✓ |
| 33 557 pares de interacción usuario→autor; 3 140 recíprocos | 33 557 / 3 140 | ✓ |
| Q07: correlación 0,65 entre `verificado` y número de seguidores | 0,650 | ✓ |
| Q15: las interacciones llegan entre 0 y 14 días después de la publicación (mediana 7) | 0–14, mediana 7 | ✓ |
| Q12: Bolivia tiene 264 perfiles (40,6 %) | 264 | ✓ |
| `tema` coincide con el `nicho` del autor en el 81,9 % | 0,819 | ✓ |
| 1 573 seguimientos posteriores al 2025-10-01 | 1 573 | ✓ |
| Q05: coordenadas aleatorias en todo el planeta | Latitud de −89,6 a 89,8 y longitud de −179,7 a 178,9 | ✓ |

**No encontré ninguna cifra inventada o errónea.** Esto importa porque las decisiones del plan
(fecha de corte, exclusión de variables, reciprocidad de interacción) se apoyan en estos números.

# 5. Interpretación del plan propuesto

Leído en mis propias palabras, el agente propone lo siguiente.

**1. Una sola entidad, tres papeles.** Cada perfil es a la vez *cuenta seguida* (audiencia),
*autor* (contenido y alcance) y *seguidor/interactor* (actividad). Las preguntas P1 y P2 son
entonces dos miradas sobre los mismos 650 perfiles, apoyadas en dos redes:

- la red **explícita** de seguimiento (8 133 aristas);
- una red **derivada** de "quién interactúa con el contenido de quién" (33 557 pares).

**2. Datos estructuralmente limpios pero semánticamente sospechosos.** No hay nulos ni huérfanos.
Sí hay fechas imposibles (actividad anterior al registro), ventanas de tiempo desalineadas, coordenadas
aleatorias y variables "ficticias" o "simuladas" que pueden actuar como *oráculo*: `verificado` y
`calidad_contenido`. El agente concluye, con razón, que los datos son **sintéticos** y que el proyecto
demuestra un método, no describe un mercado.

**3. Métricas como catálogo antes que como fórmula.** Propone unas 25 métricas en 4 familias:

| Familia | Métricas |
|---|---|
| Influencia | I1–I10: seguidores, PageRank, impresiones, engagement ponderado, tasa de engagement, audiencia única, compartidos, calidad, crecimiento, ranking por nicho |
| Actividad | A1–A6 |
| Reciprocidad | R1–R4 |
| Valor de seguidor | V1–V3 |

Cada una dice si es calculable hoy (Sí, Parcial o No). Los componentes se normalizan por percentiles
y se combinan con pesos, pero cada componente se publica por separado para que el ranking sea
explicable.

**4. Reciprocidad de dos tipos.** Como la reciprocidad de *seguimiento* casi no existe (106 pares),
propone usar la reciprocidad de *interacción* (A comenta a B y B comenta a A) como señal principal.
Es una adaptación inteligente a los datos.

**5. Arquitectura en capas, inmutable y "as-of":**

```
raw (inmutable) → staging (tipado y con banderas) → core (modelo dimensional)
               → features por fecha de corte → runs/<run_id> (inmutables) + latest.json
```

`processed/` se reconstruye entero en cada corrida. Con este volumen es más simple y auditable que
hacer cargas incrementales. Todo se calcula a una `fecha_corte` explícita.

**6. Nuevos datos sin perder histórico.** Funciona así:

- Los datos nuevos se añaden en `data/raw/lotes/<fecha>_<nn>/`, nunca reemplazando archivos.
- Un manifiesto con huellas da idempotencia y bloquea si un archivo ya registrado cambia.
- Un contrato de esquema decide: lo que no cumple va a cuarentena.
- Cada entidad tiene su propia regla de fusión: los eventos se anexan, los usuarios usan SCD tipo 2 y
  las relaciones quedan pendientes (Q11: ¿foto completa o delta?).
- Cada corrida nueva trae su comparación con la anterior.

**7. Ocho fases (F0–F7)**, cada una con validaciones cuantitativas concretas (por ejemplo, "suma de
grados de entrada = 8 133" o "dos corridas iguales dan el mismo hash"), hasta una prueba incremental
con lotes sintéticos de prueba.

**8. Gobierno por decisiones.** No avanza sin que el usuario apruebe D01–D11. Las más importantes son:

| Decisión | Propuesta |
|---|---|
| D01 Fecha de corte | 2025-10-01 |
| D03 Definición de "seguidor de alto valor" | Global (V1) y por cuenta (V2) |
| D04 Pesos por tipo de interacción | Me_gusta 1, Guardado 2, Comentario 3, Compartido 4 |
| D05 Variables excluidas del score | `verificado` y coordenadas |
| D06 Regla de censura | 14 días |

En síntesis, el agente actúa como un **arquitecto prudente**: diagnostica, documenta, deja todo
trazable y devuelve las decisiones de negocio al dueño del proyecto.

# 6. Respuestas a las preguntas de la tarea

## 6.1 ¿El plan te parece adecuado?

**Sí.** Es adecuado en lo esencial:

- **Responde a las tres preguntas.** P1 con las métricas I, P2 con A + R + V y P3 con la estrategia
  de lotes y corridas versionadas.
- **Se apoya en evidencia verificable.** Todas las cifras que revisé coinciden (sección 4).
- **Ataca los riesgos reales del dataset**: fecha de corte, censura de borde y variables oráculo. No
  aplica métricas "de manual" sin mirar los datos.
- **Deja el control al usuario** mediante un Punto de decisión explícito, con una recomendación en
  cada ítem.

Le encuentro tres carencias:

1. **No propone cómo validar el "potencial" de influencia.** "Potencial" implica predecir. Los datos
   lo permiten: hay 1 573 seguimientos posteriores a la fecha de corte propuesta. El plan los descarta
   (D01), cuando podrían servir como **validación fuera de muestra**: ¿las cuentas que el score marca
   como influyentes a 2025-10-01 son las que más seguidores ganan después? Sin esto, la única
   validación es la sensibilidad a los pesos, que mide estabilidad pero no acierto.
2. **No hay una línea base simple.** Falta un ranking ingenuo, por ejemplo solo seguidores o solo
   engagement, contra el que comparar el score compuesto y demostrar que la complejidad aporta algo.
3. **La definición de P2 queda abierta.** Es la pregunta más ambigua, y el plan la plantea bien como
   decisión (D03). Pero el hallazgo Q03 cambia el sentido de la pregunta: un "seguidor" en estos datos
   casi nunca interactúa con quien sigue. Conviene consultarlo con el docente antes de implementar.

## 6.2 ¿El plan tiene la complejidad esperada, o es menos o más complejo de lo necesario?

**Es más complejo de lo necesario**, aunque la complejidad está bien construida y en parte la pide el
propio prompt (reproducibilidad, versionado, incorporación de nuevos datos).

| Componente | Propuesta | ¿Necesario a esta escala? |
|---|---|---|
| Fases | 8 (F0–F7) | Bastan 4: base, métricas y scores, incremental, reporte |
| Skills | 8 | 4 o 5 cubren lo mismo (6.4) |
| Datos de usuarios | SCD tipo 2 en `dim_usuario` | Basta conservar la foto completa de cada lote. SCD2 solo si llegan cambios de atributos |
| Ingesta | Cuarentena, contratos de esquema, manifiesto con huellas | El manifiesto y una validación de esquema simple sí. La cuarentena como carpeta es opcional |
| Comparación entre corridas | Spearman y Kendall | Uno basta (Spearman) más el solapamiento del top-N |
| Variedad de métricas | ~25 métricas | Bien como catálogo. Para el score final conviene elegir 4 o 5 por pregunta |
| Configuración | YAML versionado con pesos | Adecuado y barato |
| Corridas inmutables + `latest.json` | — | Adecuado: es el corazón de P3 |

Para 650 perfiles y unos 50 000 registros sintéticos, esto es arquitectura de producción empresarial.
Mi criterio es **mantener los principios** (inmutabilidad, as-of, corridas versionadas, wiki) y
**reducir los mecanismos** (SCD2, cuarentena, 8 skills, 8 fases). Así se llega antes a una respuesta
de P1 y P2 sin perder la capacidad de responder P3.

## 6.3 ¿Se planifica uso de wiki? ¿Para qué?

**Sí, y es uno de los ejes del plan.** El agente adapta el patrón de Karpathy (fuentes crudas →
wiki mantenida por el LLM → esquema) a un proyecto de datos:

| Componente del patrón | En este proyecto |
|---|---|
| Fuentes crudas inmutables | `data/raw/` y cada lote nuevo |
| Wiki | `wiki/` con 6 secciones y 3 páginas base |
| Esquema | `AGENTS.md` |
| `index.md` | Catálogo; hay que leerlo antes de responder cualquier consulta |
| `log.md` | Registro de solo anexión, con prefijo `## [fecha] operación \| título` para poder buscarlo con `grep` |
| Operaciones | *ingesta*, *consulta* (las buenas respuestas se archivan como página) y *lint* (contradicciones, huérfanas, afirmaciones sin fuente) |

Cada sección de la wiki tiene su función:

- `sources/`: una página por lote de datos (qué llegó, cuándo, con qué huella y qué problemas trae).
- `metrics/`: la definición viva de cada métrica, su fórmula, sus insumos y sus limitaciones.
- `quality/`: los problemas Q01–Q17 con estado (abierto, mitigado o cerrado).
- `decisions/`: registros de decisión D01… con contexto, opciones, decisión, fecha y quién la aprobó.
- `runs/`: el resumen de cada corrida y qué cambió en los rankings respecto a la anterior.
- `entities/`: las cuentas destacadas, solo cuando haya resultados aprobados.

**Para qué sirve:** como **memoria persistente y acumulativa del proyecto**. Cuando llegue un lote
nuevo (P3), el agente no re-deriva todo desde cero:

- actualiza la página de la fuente;
- marca los problemas de calidad nuevos o resueltos;
- registra la corrida;
- documenta qué cuentas subieron o bajaron y por qué.

La wiki es, en la práctica, la **respuesta documentada a P3** y la trazabilidad de cada decisión.

**Una observación:** en el patrón original la wiki *sintetiza conocimiento* de documentos. Aquí cumple
más bien una función de *gobierno del proyecto*: decisiones, calidad, corridas y definiciones. La
adaptación es razonable, pero el valor "compuesto" que describe Karpathy solo aparecerá cuando haya
varias corridas y lotes. Hoy la wiki tiene 3 páginas.

## 6.4 ¿Se planifica uso de skills? ¿Para qué?

**Sí, pero solo están diseñadas, no creadas.** `skills/` está vacío, lo cual es coherente con la
prohibición de implementar en esta fase. Se proponen 8 skills como `skills/<nombre>/SKILL.md`:

| Skill | Para qué | Pregunta que apoya |
|---|---|---|
| `perfilar-csv` | Perfilar CSV y actualizar el diccionario de datos | Diagnóstico |
| `ingerir-lote` | Registrar un lote: huella, manifiesto, esquema, cuarentena, página en la wiki | **P3** |
| `validar-calidad` | Ejecutar los contratos y las reglas Q01–Q17 | P3 |
| `ejecutar-corrida` | Correr el pipeline a una `fecha_corte` y publicar `runs/<run_id>` | P1, P2 |
| `comparar-corridas` | Altas y bajas en el top-N y correlación de rangos | **P3** |
| `mantener-wiki` | Operaciones de ingesta y consulta de la wiki | Todas |
| `lint-wiki` | Revisar la salud de la wiki | Todas |
| `registrar-decision` | Crear registros de decisión D0x | Gobierno |

**Para qué sirven:** para encapsular **procedimientos repetibles**, de modo que cualquier agente (o
persona) los ejecute siempre igual. Es la contraparte operativa de la reproducibilidad. Así la llegada
de nuevos datos se vuelve una secuencia mecánica: `ingerir-lote → validar-calidad →
ejecutar-corrida → comparar-corridas → mantener-wiki`.

**Observaciones críticas:**

- **Ubicación.** Claude Code descubre skills automáticamente en `.claude/skills/`, no en `skills/`.
  Si se dejan en `skills/`, hay que referenciarlas desde `AGENTS.md`/`CLAUDE.md` o enlazarlas
  simbólicamente. De lo contrario el agente no las "verá" como skills nativas. Lo mismo pasa con
  `AGENTS.md`: Claude Code no lo carga solo. El agente lo detectó y lo propuso como D11
  (`CLAUDE.md` que importe `AGENTS.md`).
- **Cantidad.** 8 es demasiado para el alcance. Se pueden fusionar en 4 o 5:
  - `ingerir-lote`, que incluya la validación;
  - `ejecutar-corrida`, que incluya la comparación;
  - `mantener-wiki`, que incluya lint y registro de decisiones;
  - `perfilar-csv`;
  - opcionalmente, `responder-consulta`, que archiva las respuestas en la wiki.

## 6.5 ¿Qué estructura del proyecto se propone?

Esta es la estructura creada (✔) y la propuesta para fases futuras (◇):

```
ProyectoWiki/
├── AGENTS.md                 ✔ esquema y reglas para cualquier agente
├── CLAUDE.md                 ◇ (D11) importaría AGENTS.md
├── pyproject.toml, uv.lock   ✔ entorno uv (Python 3.12)
├── config/metricas_v1.yaml   ◇ pesos, ventanas, umbrales, fecha_corte
├── data/
│   ├── raw/                  ✔ inmutable; lote inicial L0000
│   │   └── lotes/<AAAAMMDD>_<nn>/   ◇ lotes nuevos (solo anexión)
│   ├── processed/            ✔ regenerable (no versionado)
│   │   ├── _control/manifiesto_lotes.csv   ◇
│   │   ├── _cuarentena/      ◇
│   │   ├── staging/          ◇ tipado + linaje + banderas flag_*
│   │   ├── core/             ◇ dim_usuario, fact_seguimiento, fact_publicacion, fact_interaccion
│   │   └── features/<fecha_corte>/  ◇
│   └── outputs/              ✔
│       ├── runs/<run_id>/    ◇ scores, rankings, validaciones, comparación, metadata.json
│       └── latest.json       ◇ puntero a la corrida vigente
├── docs/                     ✔ llm-wiki.md, data_dictionary.md
├── wiki/                     ✔ index.md, log.md, objectives.md
│   └── sources/ metrics/ quality/ decisions/ runs/ entities/   ✔ (vacías)
├── skills/                   ✔ (vacía) → 8 skills/<nombre>/SKILL.md ◇
├── src/                      ✔ (vacía) → ingest, validate, transform, features, scores, compare, report, pipeline ◇
├── tests/                    ◇ pytest + fixtures sintéticos
├── reports/<run_id>/         ◇ informe por corrida
└── dashboards/               ◇ Streamlit o HTML + Plotly (por decidir)
```

La lógica de la estructura es una **separación por ciclo de vida**:

| Capa | Rutas |
|---|---|
| Lo que no cambia | `raw` |
| Lo que se regenera | `processed` |
| Lo que se versiona y conserva | `outputs/runs`, `reports`, `wiki` |
| Lo que define el comportamiento | `src`, `config`, `skills`, `AGENTS.md` |

Es una estructura estándar de proyecto de datos, al estilo *cookiecutter data science*, más la capa
de conocimiento (wiki y skills) propia del enfoque asistido por IA.

# 7. Apreciaciones personales, conclusiones y sugerencias

## Apreciaciones

- **Lo que más me sorprendió** es la calidad del diagnóstico. Un analista humano habría tardado horas
  en encontrar Q01 (actividad antes del registro), Q03 (interacciones desconectadas del grafo) y Q15
  (censura de borde). Los tres cambian decisiones de diseño: el agente no los encontró porque sí.
- **La disciplina con las restricciones** fue ejemplar. No tocó los datos crudos, no adelantó
  rankings, separó lo observado de lo inferido y dejó los commits al usuario. En un proyecto real, esto
  es lo que permite confiar en un asistente de IA.
- **El enfoque wiki + skills + AGENTS.md** convierte al agente de un "chat que responde" en un
  colaborador con memoria y procedimientos. Es la idea central de la guía de Karpathy y el plan la
  aplica de forma coherente.
- **El riesgo opuesto es la sobreingeniería.** El agente diseña para un escenario de producción
  (SCD2, cuarentena, 8 skills, 8 fases) que este proyecto no necesita hoy. Un asistente de IA genera
  complejidad "gratis", pero mantenerla y entenderla no es gratis para quien la recibe.

## Conclusiones

1. El prompt funcionó: la fase de **diagnóstico y diseño quedó completa, verificable y gobernada** por
   decisiones explícitas.
2. El plan **responde a las tres preguntas**. La más sólida es P3 (cómo se actualizan los resultados),
   que es precisamente donde la wiki y las skills aportan más.
3. El plan **es más complejo de lo necesario** pero no está mal planteado. Recortarlo es más fácil que
   tener que añadir rigor después.
4. Como los datos son **sintéticos** y con variables oráculo, los rankings finales valdrán como
   demostración de método. La validación honesta (sensibilidad + fuera de muestra) es tan importante
   como el score.

## Sugerencias concretas antes de aprobar

| # | Sugerencia | Ajusta |
|---|---|---|
| 1 | **Aprobar un MVP en 4 fases:** (a) base: manifiesto, configuración, `CLAUDE.md`; (b) features y scores P1/P2 con línea base simple; (c) prueba incremental con un lote sintético; (d) reporte, dashboard y lint de la wiki | Fases F1–F7 |
| 2 | **Validación fuera de muestra del "potencial":** calcular la influencia a 2025-10-01 y contrastarla con los seguidores ganados entre 2025-10 y 2026-08 (1 573 aristas) | D01, F4 |
| 3 | **Comparar contra una línea base ingenua** (solo seguidores) para justificar el score compuesto | F4 |
| 4 | **Crear las skills en `.claude/skills/`** (o enlazarlas) y aprobar D11 (`CLAUDE.md` → `@AGENTS.md`), para que Claude Code las cargue de verdad | D11, skills |
| 5 | **Reducir las skills a 4 o 5** fusionando validación con ingesta, comparación con corrida y lint/decisiones con la wiki | Skills |
| 6 | **Una sola fuente de verdad para la calidad:** mover Q01–Q17 a `wiki/quality/` y dejar en el diccionario solo un enlace, para evitar que se desincronicen | Wiki, docs |
| 7 | **Postergar SCD2 y la cuarentena** hasta que llegue un lote real con cambios. Basta guardar cada lote completo | Arquitectura |
| 8 | **Definir P2 con el docente:** ¿valor del seguidor en general, o para una cuenta concreta? Dado Q03, propongo medir la versión global (V1) y reportar la por cuenta (V2) solo como análisis secundario | D03 |
| 9 | **Pesos de interacción:** aceptar 1/2/3/4 como punto de partida. Así lo dejó el agente al no tener el dato de esfuerzo real: es arbitrario, por eso hay que publicar la sensibilidad | D04 |
| 10 | **Dashboard en HTML estático con Plotly** en lugar de Streamlit: no requiere servidor, se publica en Netlify o GitHub Pages como en la tarea U3-T1 y encaja con las corridas inmutables | D07 |

# Anexo A · Archivos de esta entrega

| Archivo | Contenido |
|---|---|
| `entrega/INFORME.md` / `INFORME.pdf` | Este informe |
| `entrega/prompt_utilizado.md` | Prompt textual y preámbulo operativo |
| `entrega/plan_agente.md` | Respuesta completa del agente (plan de acción), sin editar |
| `docs/data_dictionary.md` | Diccionario de datos generado por el agente |
| `AGENTS.md` | Reglas del proyecto generadas por el agente |
| `wiki/index.md`, `wiki/log.md`, `wiki/objectives.md` | Base de la wiki generada por el agente |

# Anexo B · Cómo reproducir

```bash
git clone https://github.com/dq-datasci/ProyectoWiki-Influencers.git
cd ProyectoWiki-Influencers
uv sync                                  # crea .venv con Python 3.12 + pandas
sed -n '/^```$/,/^```$/p' docs/data_dictionary.md | grep data/raw | sha256sum -c   # integridad de raw
uv run python -c "import pandas as pd; print(pd.read_csv('data/raw/usuarios.csv').shape)"
```

Historial de git: `e357965` (setup inicial) → `2736122` (resultado del agente, fase F0) → commit del
informe.
