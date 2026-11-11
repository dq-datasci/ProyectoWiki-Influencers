# Prompt entregado al agente IA

El agente fue una instancia nueva e independiente de Claude Code (modelo Claude Opus 5.5), lanzada sin el
contexto de la conversación previa, para que la revisión posterior no fuera una autoevaluación.

## Preámbulo operativo añadido (no forma parte de la consigna)

> Contexto operativo (no forma parte de la consigna): tu directorio de proyecto es
> /home/dq-datasci/Projects/ProyectoWiki. Todas las rutas relativas del prompt se refieren a esa carpeta.
> Es un proyecto uv (Python 3.12, pandas ya instalado); si necesitas ejecutar Python usa
> `uv run python ...` desde esa carpeta y si necesitas otra librería agrégala con `uv add`. No uses pip ni
> Python del sistema. No hagas commits de git. Responde en español. Tu mensaje final debe ser la
> presentación completa del plan de acción (no un resumen de él), tal como se la presentarías al usuario.

Motivo del preámbulo: obligar a usar el entorno virtual `uv` del proyecto, dejar los commits en manos del
usuario y conseguir el plan completo como respuesta.

## Prompt propuesto por la tarea (textual)

Actúa como arquitecto de un proyecto local de Business Intelligence para análisis de redes sociales.

El proyecto debe responder:

¿Qué cuentas tienen mayor potencial de influencia?

¿Qué perfiles son seguidores de alto valor por actividad y reciprocidad?

¿Cómo se actualizan los resultados al recibir nuevos datos?

Lee docs/llm-wiki.md y todos los CSV de data/raw/.

data/raw/ es la fuente de verdad: no modifiques, renombres, elimines ni sobrescribas sus archivos. No implementes aún el pipeline, dashboard, rankings ni scores finales. Trabaja únicamente en la fase de diagnóstico y diseño, basándote en datos observables; identifica claramente cualquier supuesto, limitación o información faltante.

Crea:

docs/data_dictionary.md, con inventario de archivos, columnas, tipos aparentes, ejemplos, faltantes, claves, relaciones y problemas de calidad.

AGENTS.md, con reglas para preservar datos crudos, asegurar reproducibilidad, versionar resultados y mantener la wiki.

La estructura inicial de wiki/, skills/, data/processed/, data/outputs/, src/, reports/ y dashboards/.

wiki/index.md, wiki/log.md y wiki/objectives.md como base de la wiki.

Finalmente, presenta un plan de acción para mi aprobación que incluya:

Resumen y modelo conceptual de los datos.

Métricas preliminares para influencia, actividad, reciprocidad y valor de seguidores, indicando si pueden calcularse con los datos actuales.

Arquitectura propuesta de datos y resultados.

Skills reutilizables necesarias.

Estrategia para incorporar nuevos CSV sin perder histórico.

Fases de implementación, validaciones y entregables.

Riesgos, preguntas y decisiones pendientes.

Termina con una sección Punto de decisión, indicando qué debo aprobar antes de comenzar la implementación.
