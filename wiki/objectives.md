---
titulo: Objetivos del proyecto
tipo: decision
estado: borrador
actualizado: 2026-11-11
fuentes: [docs/data_dictionary.md, data/raw/diccionario_datos.csv]
---

# Objetivos del proyecto

## Contexto

Proyecto local de BI sobre una red social **sintética** (650 perfiles, 5 200 publicaciones, 35 810 interacciones y 8 133 relaciones de seguimiento). Los datos crudos están en `data/raw/` y son inmutables. Ver [diccionario de datos](../docs/data_dictionary.md).

## Preguntas de negocio

| # | Pregunta | Unidad de análisis | Resultado esperado |
|---|---|---|---|
| P1 | ¿Qué cuentas tienen mayor **potencial de influencia**? | Cuenta (como autor y como nodo del grafo) | Ranking explicable por componentes: red, alcance, resonancia y amplificación. |
| P2 | ¿Qué perfiles son **seguidores de alto valor** por actividad y reciprocidad? | Perfil como seguidor (global) y relación seguidor→cuenta | Ranking global de seguidores y, para cada cuenta influyente, sus seguidores más valiosos. |
| P3 | ¿Cómo se **actualizan** los resultados al recibir nuevos datos? | Corrida del pipeline / lote de datos | Proceso reproducible de ingesta, histórico de corridas y comparación de rankings entre corridas. |

## Criterios de éxito (propuestos, pendientes de aprobación)

1. **Trazabilidad:** cada cifra de un reporte o de la wiki remite a un archivo crudo o a un `run_id`.
2. **Reproducibilidad:** con los mismos crudos y la misma configuración se obtienen outputs idénticos.
3. **Explicabilidad:** cada score se descompone en métricas componentes visibles.
4. **Robustez:** los rankings top-N se mantienen razonablemente estables ante cambios moderados de pesos (análisis de sensibilidad documentado).
5. **Incrementalidad:** agregar un CSV nuevo no altera ni borra crudos ni corridas anteriores, y permite comparar el antes y el después.
6. **Honestidad:** las limitaciones de los datos (ver [diccionario §5–6](../docs/data_dictionary.md)) se declaran junto a los resultados.

## Fuera de alcance (por ahora)

- Datos de plataformas reales, APIs externas o enriquecimiento web.
- Análisis de texto o sentimiento (no hay texto de comentarios).
- Predicción o *machine learning* supervisado (no hay variable objetivo).
- Análisis geográfico con coordenadas (son incoherentes con `pais`; Q5).

## Estado

- Fase 0 (diagnóstico y diseño): **completada y pendiente de aprobación.** Ver el [plan de acción](plan-accion.md).
- Las decisiones aprobadas se registrarán aquí y en [log](log.md).
