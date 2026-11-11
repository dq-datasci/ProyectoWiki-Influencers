---
titulo: Objetivos del proyecto
tipo: objetivos
actualizado: 2026-11-11
fuentes: [docs/data_dictionary.md]
estado: borrador — pendiente de aprobación
---

# Objetivos

## Preguntas de negocio

| # | Pregunta | Unidad de análisis | Producto esperado |
|---|---|---|---|
| P1 | ¿Qué cuentas tienen mayor potencial de influencia? | Perfil (como autor / nodo seguido) | Ranking de influencia con desglose por componente y por nicho |
| P2 | ¿Qué perfiles son seguidores de alto valor por actividad y reciprocidad? | Perfil (como seguidor) y par seguidor→cuenta | Ranking de valor de seguidor (global) y, por decidir, por cuenta influyente |
| P3 | ¿Cómo se actualizan los resultados al recibir nuevos datos? | Lote de datos / corrida | Proceso documentado de ingesta incremental, corridas versionadas y comparación entre corridas |

## Alcance

- Dentro: los cuatro CSV de `data/raw/` (usuarios, relaciones, publicaciones, interacciones) y lotes futuros con el mismo contrato.
- Fuera (por ahora): datos externos, análisis geográfico (coordenadas no fiables, ver Q05 en el [diccionario de datos](../docs/data_dictionary.md)), análisis de texto o sentimiento (no hay texto).

## Criterios de éxito (propuestos)

1. Cada métrica tiene definición escrita, insumos trazables y su limitación documentada.
2. Los rankings son reproducibles: misma `fecha_corte` + mismos lotes + misma configuración ⇒ mismo resultado.
3. Los rankings son estables ante pequeñas variaciones de pesos (análisis de sensibilidad documentado).
4. Añadir un lote nuevo no altera `data/raw/` existente ni corridas previas, y produce un informe de cambios.
5. La wiki refleja el estado actual: índice completo, log al día, sin páginas huérfanas.

## Restricciones

- `data/raw/` es inmutable (ver `AGENTS.md` §3).
- Entorno `uv`, Python 3.12.
- Datos sintéticos: los resultados ilustran el método, no describen un mercado real.

## Estado

F0 (diagnóstico y diseño) completada el 2026-11-11; plan de acción pendiente de aprobación.
