---
titulo: Índice de la wiki
tipo: proceso
estado: vigente
actualizado: 2026-11-11
---

# Índice

Catálogo de todas las páginas de la wiki. Léelo antes de responder cualquier consulta. Las reglas de mantenimiento están en [AGENTS.md](../AGENTS.md) §5.

## Base

| Página | Resumen | Actualizado |
|---|---|---|
| [objectives.md](objectives.md) | Preguntas de negocio P1–P3, criterios de éxito, alcance y estado de la fase. | 2026-11-11 |
| [plan-accion.md](plan-accion.md) | Plan de acción de la Fase 0: métricas, arquitectura, skills, fases, riesgos y decisiones pendientes. **Borrador, pendiente de aprobación.** | 2026-11-11 |
| [log.md](log.md) | Registro cronológico de operaciones (setup, ingest, run, query, lint, decision). | 2026-11-11 |

## Documentación técnica (fuera de `wiki/`)

| Documento | Resumen |
|---|---|
| [docs/data_dictionary.md](../docs/data_dictionary.md) | Inventario y diccionario de los 5 CSV crudos, claves, relaciones, calidad (Q1–Q11), faltantes y supuestos. |
| [docs/llm-wiki.md](../docs/llm-wiki.md) | Patrón LLM-Wiki que sigue este proyecto. |
| [AGENTS.md](../AGENTS.md) | Esquema y reglas del proyecto para agentes. |

## Fuentes (`fuentes/`)
_Vacío. Habrá una página por lote o archivo crudo ingerido (lote inicial: 5 CSV)._

## Métricas (`metricas/`)
_Vacío. Habrá una página por métrica o score con definición versionada, fórmula, supuestos y `run_id` de referencia._

## Hallazgos (`hallazgos/`)
_Vacío. Aquí irán los resultados interpretados de cada corrida y las respuestas a consultas archivadas._

## Decisiones (`decisiones/`)
_Vacío. Habrá una página por decisión aprobada (D1–D12 del plan) con fecha y motivo._
