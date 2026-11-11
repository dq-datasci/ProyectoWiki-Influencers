---
titulo: Índice de la wiki
tipo: indice
actualizado: 2026-11-11
estado: activo
---

# Índice

Catálogo de la wiki del proyecto de BI de influencia en redes sociales. Reglas de mantenimiento en
[AGENTS.md](../AGENTS.md) §7. Leer este índice antes de responder cualquier consulta.

## Base

| Página | Resumen |
|---|---|
| [objectives.md](objectives.md) | Preguntas de negocio P1–P3, alcance, criterios de éxito y restricciones. |
| [log.md](log.md) | Registro cronológico (solo anexión) de setup, diagnósticos, decisiones, ingestas y corridas. |

## Documentación de referencia (fuera de `wiki/`)

| Documento | Resumen |
|---|---|
| [docs/data_dictionary.md](../docs/data_dictionary.md) | Inventario de los 5 CSV crudos, columnas, tipos, claves, relaciones, cobertura temporal, problemas de calidad Q01–Q17, huellas SHA-256 del lote L0000. |
| [docs/llm-wiki.md](../docs/llm-wiki.md) | Patrón LLM Wiki en el que se basa esta wiki (fuentes crudas → wiki → esquema). |
| [AGENTS.md](../AGENTS.md) | Esquema y reglas: preservación de crudos, reproducibilidad, versionado, mantenimiento de la wiki. |

## Secciones (vacías hasta la aprobación del plan)

| Carpeta | Contenido previsto | Páginas |
|---|---|---:|
| `sources/` | Una página por archivo/lote fuente (L0000: usuarios, relaciones, publicaciones, interacciones). | 0 |
| `metrics/` | Definición de cada métrica (influencia, actividad, reciprocidad, valor de seguidor). | 0 |
| `quality/` | Problemas de calidad con estado; hoy viven en el diccionario (Q01–Q17). | 0 |
| `decisions/` | Registros de decisión D01… aprobados por el usuario. | 0 |
| `runs/` | Resumen y comparación de cada corrida del pipeline. | 0 |
| `entities/` | Cuentas destacadas (solo tras resultados aprobados). | 0 |
