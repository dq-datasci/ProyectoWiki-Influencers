---
titulo: Respuesta en el chat de la 2.ª ejecución (sin preámbulo)
nota: copia literal; el plan completo está en wiki/plan-accion.md de la rama experimento-u3t4
---

## Fase 0 terminada: diagnóstico y diseño, pendiente de tu aprobación

Leí `docs/llm-wiki.md` y los 5 CSV en modo solo lectura. Los 5 hashes SHA-256 de `data/raw/` son iguales antes y después del trabajo, así que no se modificó nada. No implementé pipeline, scores, rankings ni dashboard, y no hice commit (rama `experimento-u3t4`; todo queda sin seguimiento en git).

**Lo que creé** (todo en `/home/dq-datasci/Projects/ProyectoWiki-rerun/`):
- `docs/data_dictionary.md`: contiene:
  - inventario de los archivos con sus hashes de referencia;
  - columna por columna: tipo, valores únicos, faltantes, ejemplos y rango;
  - claves y relaciones;
  - observaciones A1–A7;
  - problemas de calidad Q1–Q11;
  - información faltante F1–F7 y supuestos S1–S4.
- `AGENTS.md`: reglas para no tocar los datos crudos, para que todo se pueda regenerar (uv, configuración versionada, misma entrada da la misma salida), para guardar cada corrida en `data/outputs/runs/<run_id>/` con un `manifest.json` sin sobrescribir las anteriores, y para mantener la wiki (índice, log, formato de las páginas).
- `wiki/index.md`, `wiki/log.md`, `wiki/objectives.md`, más `wiki/plan-accion.md` con el plan completo.
- Carpetas `wiki/{fuentes,metricas,hallazgos,decisiones}/`, `skills/`, `data/processed/`, `data/outputs/`, `src/`, `reports/` y `dashboards/`, cada una con un `.gitkeep`.

## Plan de acción (versión completa en `wiki/plan-accion.md`)

**1. Resumen y modelo de los datos.** Son 650 usuarios, 5 200 publicaciones (de enero de 2024 a septiembre de 2025), 35 810 interacciones (entre 0 y 14 días después de publicar) y 8 133 relaciones de seguimiento (de 2022 a agosto de 2026). En forma están limpios: no hay vacíos ni duplicados, y todas las claves que apuntan a otra tabla existen.

Los problemas están en la coherencia de los datos:
- **Fechas de registro:** 1 098 publicaciones (21 %), 7 359 interacciones (21 %) y 3 687 seguimientos (45 %) son anteriores a la fecha de registro del usuario. Por eso `fecha_registro` no es fiable.
- **Coordenadas:** están repartidas por todo el mundo y no coinciden con el país; solo 33 de 650 caen en Sudamérica.
- **Seguimiento y actividad casi no se cruzan:** solo el 2,3 % de las interacciones viene de alguien que sigue al autor.
- **Reciprocidad baja:** hay 106 pares que se siguen mutuamente y un 9,4 % de reciprocidad en interacciones.
- **`verificado` es ficticio pero pesa mucho:** las cuentas verificadas tienen unas 2,2 veces más seguidores (21,4 frente a 9,8 de media).

El modelo es: el usuario publica, interactúa con publicaciones y sigue a otros usuarios. De ahí salen dos redes: una de seguimiento y otra de "quién interactúa con qué autor".

**2. Métricas preliminares y si se pueden calcular hoy.**
- **Influencia (I1–I11):** seguidores, PageRank, impresiones, engagement total y por impresión, audiencia única y constancia se pueden calcular. Compartidos, calidad y crecimiento solo con limitaciones. Alcance externo, viralidad y sentimiento no, porque faltan datos.
- **Actividad (A1–A7):** todas se pueden calcular; la recencia depende de la fecha de corte.
- **Reciprocidad (R1–R4):** seguimiento mutuo, reciprocidad de interacción y la cruzada se pueden calcular; la temporal solo con limitaciones.
- **Valor del seguidor (V1–V5):** se pueden calcular, pero V1 (si el seguidor interactúa con la cuenta que sigue) tiene muy poca señal: solo 773 de 8 133 relaciones tienen alguna interacción.
- **Scores propuestos:** el de influencia combina cuatro bloques (red, alcance, resonancia y amplificación) sobre percentiles. El de seguidor tiene dos niveles: uno global por perfil y otro por relación seguidor→cuenta.

**3. Arquitectura.** Los datos pasan por `raw` (sin tocar), luego `processed` (registro de lo ingerido, copias tipadas, modelo con columnas de aviso y métricas) y luego `outputs` (cada corrida en su carpeta, histórico acumulado y un puntero a la última). Reportes, dashboard y wiki leen de ahí. En cada corrida se recalcula todo, porque con menos de 50 000 filas es instantáneo.

**4. Skills.** `validar-crudos`, `ingestar-lote`, `construir-modelo`, `calcular-metricas`, `generar-scores-rankings`, `comparar-corridas`, `generar-reporte` y `mantener-wiki`.

**5. Nuevos CSV sin perder histórico.** Cada entrega llega a `data/raw/entregas/<fecha>_<desc>/` y se registra con su hash. Si un archivo ya registrado cambia, el proceso se detiene. Publicaciones e interacciones se agregan sin duplicar. Si cambian datos de un usuario, se guarda la versión anterior y la nueva. Si un archivo de seguimientos llega completo, las relaciones que desaparecen se marcan como bajas. Cada carga produce una corrida nueva que se compara con la anterior.

**6. Fases.**
- **F1:** base del proyecto y validador; comprueba hashes y conteos.
- **F2:** modelo procesado; los avisos deben dar 1 098, 7 359 y 3 687.
- **F3:** métricas; por ejemplo, las interacciones recibidas y emitidas deben sumar 35 810 cada una.
- **F4:** scores, rankings y prueba de qué pasa si cambian los pesos.
- **F5:** simulación de un segundo lote con datos de prueba en `tests/`, nunca en `data/raw/`.
- **F6:** reportes y dashboard.
- **F7:** revisión final de la wiki.

**7. Riesgos.** La poca señal para valorar seguidores, que los pesos son arbitrarios, que `verificado` crearía un razonamiento circular si entra en el score, que las ventanas de fechas no coinciden, que no sabemos cómo llegarán los nuevos CSV, y que las métricas de red solo valen dentro de estos 650 perfiles.

## Punto de decisión

Estas son las decisiones pendientes (D1–D13), con la propuesta por defecto de cada una:

| ID | Decisión | Propuesta por defecto |
|---|---|---|
| D1 | Fecha de corte | 2025-10-01, la última interacción |
| D2 | Qué cuenta como reciprocidad | Seguimiento mutuo y reciprocidad de interacción, por separado |
| D3 | Pesos por tipo de interacción | Me_gusta 1, Guardado 2, Comentario 3, Compartido 4 |
| D4 | Score de influencia | Cuatro bloques con el mismo peso |
| D5 | "Seguidor de alto valor" | Los dos niveles: global y por relación |
| D6 | Dónde y cómo llegan los nuevos CSV | `data/raw/entregas/<fecha>_<desc>/` |
| D7 | Formato y qué se guarda en git | Parquet y CSV; en git solo lo ligero |
| D8 | Dashboard | HTML estático con Plotly o Streamlit |
| D9 | Coordenadas y fecha de registro | Fuera de las métricas |
| D10 | `verificado` | Solo para contrastar resultados |
| D11 | Librerías nuevas | `pyarrow`, `networkx`, `pytest` |
| D12 | `calidad_contenido` en el score | No en la primera versión |
| D13 | Umbral de "alto valor" y top-N | Percentil 90; top-20 |

Para empezar la implementación tienes que aprobar:
1. El plan por fases y la idea de recalcular todo en cada corrida, con un histórico al que solo se agrega.
2. Las decisiones de la tabla. Las que necesito antes de la Fase 1 son **D1** (fecha de corte), **D6** (dónde llegan los nuevos CSV), **D7** (formato) y **D11** (librerías); las demás pueden esperar a la Fase 3.
3. Las reglas de `AGENTS.md`, sobre todo la carpeta `data/raw/entregas/` y dejar `verificado`, coordenadas y `fecha_registro` fuera de los scores.
4. Si quieres que haga commit de la Fase 0.

Además, me ayudaría saber si existe un criterio de negocio para "influencia" o "alto valor", con qué frecuencia y en qué formato llegarán los datos nuevos, y para quién son los reportes.

Las cifras salen de scripts de análisis que dejé en una carpeta temporal y no forman parte del proyecto; en la Fase 1 propongo convertirlos en el validador `validar-crudos`.