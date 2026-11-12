# Diccionario de datos: `data/raw/`

> **Estado:** diagnóstico inicial (Fase 0). Fecha del perfilado: 2026-11-11.
> **Alcance:** solo lectura de `data/raw/`. No se modificó ningún archivo crudo.
> **Método:** perfilado ad hoc con `pandas` (`uv run python`), leyendo cada CSV de dos formas: como texto (`dtype=str`, sin conversión de nulos) para detectar vacíos y tokens raros, y con inferencia de tipos para rangos y estadísticos.
> Las cifras de este documento deben volver a calcularse con el validador de la Fase 1 y compararse con estas.

Convención: **[O]** = observado en los datos · **[S]** = supuesto · **[F]** = información faltante.

---

## 1. Inventario de archivos

| Archivo | Filas (sin cabecera) | Columnas | Tamaño | Codificación | Grano (una fila =) | SHA-256 (prefijo) |
|---|---:|---:|---:|---|---|---|
| `usuarios.csv` | 650 | 11 | 89 665 B | UTF-8 (sin BOM), LF | un perfil | `8fc43295…5c737` |
| `publicaciones.csv` | 5 200 | 7 | 286 566 B | UTF-8 (sin BOM), LF | una publicación | `05014e6c…4da59` |
| `interacciones.csv` | 35 810 | 5 | 1 774 224 B | ASCII, LF | un evento de interacción de un usuario sobre una publicación | `ae234e72…c0209` |
| `relaciones.csv` | 8 133 | 3 | 219 632 B | ASCII, LF | una arista dirigida "A sigue a B" | `eb03d660…54c8` |
| `diccionario_datos.csv` | 7 | 3 | 468 B | UTF-8 (sin BOM), LF | una descripción de campo (metadatos) | `693b468c…d772` |

SHA-256 completos (línea base para detectar cambios en los crudos):

```
8fc432955f431ec31865049a4757116819bfc050b78bb2de6556642e59a5c737  usuarios.csv
05014e6ce0ee24b0d1a3c57c7de1d7de7170f0b6615d582f1ae52b69a584da59  publicaciones.csv
ae234e72572030017f2e85a08d9bee98089da78d59b76726d4cc4c7ce3fc0209  interacciones.csv
eb03d660d33bb02d2a9e6d71170fb102779af081ec8b3c17c7c166248eb654c8  relaciones.csv
693b468c30fb941f03161a34d8cfdcc61091339f63d947477974eed6ce9fd772  diccionario_datos.csv
```

Formato común [O]: separador `,`, cabecera en la primera fila, comillas dobles solo cuando el valor contiene comas (`usuarios.csv`: `organizacion`, `ocupacion`; `diccionario_datos.csv`), fin de archivo con salto de línea, sin filas duplicadas exactas, sin celdas vacías, sin tokens de nulo (`NA`, `null`, `-`, etc.) y sin espacios sobrantes en ningún archivo.

**Naturaleza de los datos [O]:** el propio `diccionario_datos.csv` describe las claves como *sintéticas*, `verificado` como *ficticio* y `calidad_contenido`/`impresiones` como *simulados*. Los nombres, ocupaciones y organizaciones siguen el patrón de la librería Faker (en inglés). **Todos los resultados son, por tanto, un ejercicio metodológico y no describen personas reales.**

---

## 2. Diccionario por archivo

### 2.1 `usuarios.csv`: dimensión de perfiles

Clave primaria: `usuario_id` (650 valores únicos, sin vacíos).

| Columna | Tipo aparente | Únicos | Faltantes | Ejemplos | Dominio / rango observado | Notas |
|---|---|---:|---:|---|---|---|
| `usuario_id` | string (ID) | 650 | 0 | `USR0001`, `USR0002` | `USR0001`–`USR0650`, secuencia completa sin huecos | **PK**. Descrito como "clave sintética del perfil". |
| `usuario` | string | 650 | 0 | `raquelziemannd0001` | patrón `^[a-z]+\d{4}$`; los 4 dígitos finales coinciden con `usuario_id` en los 650 casos | Clave alternativa (handle). |
| `nombre` | string | 650 | 0 | `Raquel Ziemann DDS` | ~124 con títulos o sufijos (DDS, MD, PhD, Jr., II…) | PII sintética. No aporta a las métricas. |
| `ocupacion` | string (categoría libre) | 416 | 0 | `Oceanographer`, `Therapist, music` | texto libre en inglés | Cardinalidad alta; uso analítico limitado. |
| `organizacion` | string | 643 | 0 | `Fritsch Ltd`, `Price, Senger and Reinger` | 7 nombres repetidos 2 veces (14 filas) | Repeticiones probablemente casuales (Faker); no implican relación. |
| `pais` | categoría | 5 | 0 | `Perú`, `Bolivia` | Bolivia 264 · Perú 111 · Colombia 95 · Argentina 90 · Chile 90 | Bolivia está sobrerrepresentada (40,6 %). |
| `nicho` | categoría | 6 | 0 | `Tecnología` | Gastronomía 123 · Viajes 115 · Educación 111 · Moda 108 · Deportes 98 · Tecnología 95 | Mismo dominio que `publicaciones.tema`. |
| `fecha_registro` | fecha `YYYY-MM-DD` | 571 | 0 | `2024-02-18` | 2018-01-05 → 2026-06-23 | Todas parsean. Incoherente con la actividad (ver §5). |
| `latitud` | float | 650 | 0 | `11.81229` | −89,64 → 89,80 | Distribuida por todo el globo (ver §5). |
| `longitud` | float | 650 | 0 | `10.83439` | −179,73 → 178,87 | Ídem. |
| `verificado` | booleano (`TRUE`/`FALSE`) | 2 | 0 | `TRUE` | TRUE 151 (23,2 %) · FALSE 499 | "Indicador ficticio". Correlaciona con alcance (ver §4). |

### 2.2 `publicaciones.csv`: hechos de contenido

Clave primaria: `publicacion_id` (5 200 únicos).

| Columna | Tipo aparente | Únicos | Faltantes | Ejemplos | Dominio / rango observado | Notas |
|---|---|---:|---:|---|---|---|
| `publicacion_id` | string (ID) | 5 200 | 0 | `PUB000001` | `PUB000001`–`PUB005200`, sin huecos | **PK**. |
| `autor_id` | string (FK) | 646 | 0 | `USR0137` | 100 % existe en `usuarios` | **FK → usuarios.usuario_id**. 4 usuarios sin publicaciones: `USR0197`, `USR0229`, `USR0612`, `USR0626`. |
| `fecha_publicacion` | fecha | 626 | 0 | `2025-02-19` | 2024-01-01 → 2025-09-17 | 2024: 3 000 · 2025: 2 200 (septiembre de 2025 incompleto: 140). |
| `tipo_contenido` | categoría | 4 | 0 | `Video` | Video 1 789 · Imagen 1 687 · Texto 969 · Historia 755 | Mediana de impresiones casi igual entre tipos (288–306). |
| `tema` | categoría | 6 | 0 | `Educación` | Viajes 920 · Educación 887 · Moda 887 · Gastronomía 875 · Deportes 835 · Tecnología 796 | En el 81,9 % de las publicaciones el tema coincide con el `nicho` del autor. |
| `calidad_contenido` | float [0,1] | 720 | 0 | `0.656` | 0,033 → 0,978; media 0,463; 3 decimales | "Señal simulada entre 0 y 1". **Origen desconocido [F]**: ¿es un dato de entrada o un resultado? |
| `impresiones` | entero ≥ 0 | 1 335 | 0 | `556` | 20 → 7 336; mediana 299; media 479 (asimetría a la derecha) | "Exposición simulada". No hay ceros ni negativos. |

### 2.3 `interacciones.csv`: hechos de engagement

Clave primaria: `interaccion_id` (35 810 únicos).

| Columna | Tipo aparente | Únicos | Faltantes | Ejemplos | Dominio / rango observado | Notas |
|---|---|---:|---:|---|---|---|
| `interaccion_id` | string (ID) | 35 810 | 0 | `INT0000001` | `INT0000001`–`INT0035810`, sin huecos | **PK**. |
| `usuario_id` | string (FK) | 650 | 0 | `USR0152` | 100 % existe en `usuarios` | **FK → usuarios.usuario_id** (quien interactúa). Los 650 usuarios interactúan (mínimo 10, máximo 106, mediana 54). |
| `publicacion_id` | string (FK) | 5 082 | 0 | `PUB004027` | 100 % existe en `publicaciones` | **FK → publicaciones.publicacion_id**. 118 publicaciones no tienen ninguna interacción. |
| `tipo_interaccion` | categoría | 4 | 0 | `Me_gusta` | Me_gusta 23 121 · Comentario 6 016 · Compartido 3 713 · Guardado 2 960 | El diccionario fuente dice "Me gusta" con espacio; los datos usan `Me_gusta` con guion bajo. |
| `fecha_interaccion` | fecha | 640 | 0 | `2024-03-28` | 2024-01-01 → 2025-10-01 | Solo fecha, sin hora. Se produce entre 0 y 14 días después de la publicación (mediana 7). |

Clave natural candidata: (`usuario_id`, `publicacion_id`, `tipo_interaccion`) → **0 duplicados [O]**. El par (`usuario_id`, `publicacion_id`) se repite 166 veces porque un usuario puede dar varios tipos de interacción a la misma publicación (p. ej., Me_gusta y Guardado), lo cual es legítimo.

### 2.4 `relaciones.csv`: grafo dirigido de seguimiento

Sin columna ID. Clave natural: (`seguidor_id`, `seguido_id`) → **0 duplicados [O]**.

| Columna | Tipo aparente | Únicos | Faltantes | Ejemplos | Dominio / rango observado | Notas |
|---|---|---:|---:|---|---|---|
| `seguidor_id` | string (FK) | 650 | 0 | `USR0001` | 100 % en `usuarios` | "Perfil que inicia la relación" (origen de la arista). Grado de salida 2–27, mediana 12. |
| `seguido_id` | string (FK) | 650 | 0 | `USR0605` | 100 % en `usuarios` | "Perfil que recibe la relación" (destino). Grado de entrada (seguidores) 1–39, mediana 11. |
| `fecha_seguimiento` | fecha | 1 685 | 0 | `2024-10-06` | 2022-01-01 → 2026-08-28 | Va más allá de la última interacción (2025-10-01): 1 573 aristas son posteriores. |

[O] 0 autoseguimientos. 106 pares mutuos (212 aristas recíprocas, 2,6 % de las aristas). 178 usuarios tienen al menos un seguimiento mutuo. Ningún usuario está aislado: todos siguen y son seguidos por al menos una cuenta.

[F] No hay eventos de *dejar de seguir* ni estado activo/inactivo. Se supone que el archivo es una foto acumulada de las relaciones vigentes **[S]**.

### 2.5 `diccionario_datos.csv`: metadatos de la fuente

| Columna | Tipo | Descripción |
|---|---|---|
| `archivo` | string | Nombre del CSV descrito (4 archivos). |
| `campo` | string | Nombre de la columna. |
| `descripcion` | string | Descripción breve. |

Solo documenta **7 de los 26 campos** de datos. Este documento completa el resto. No es un archivo analítico: se usa solo como referencia semántica.

---

## 3. Claves y relaciones

```
usuarios (PK usuario_id)
  ├─< publicaciones.autor_id            1 usuario : N publicaciones   (646 de 650 publican)
  ├─< interacciones.usuario_id          1 usuario : N interacciones   (650 de 650 interactúan)
  ├─< relaciones.seguidor_id            usuario como origen de la arista
  └─< relaciones.seguido_id             usuario como destino de la arista

publicaciones (PK publicacion_id)
  └─< interacciones.publicacion_id      1 publicación : N interacciones (5 082 de 5 200 reciben)

relaciones = tabla puente N:M usuarios×usuarios (grafo dirigido)
interacciones = tabla puente N:M usuarios×publicaciones → define también un grafo
                derivado usuario→autor (33 557 pares dirigidos distintos)
```

| Relación | Cardinalidad | Integridad referencial [O] |
|---|---|---|
| `publicaciones.autor_id` → `usuarios.usuario_id` | N:1 | 100 % (0 huérfanos) |
| `interacciones.usuario_id` → `usuarios.usuario_id` | N:1 | 100 % |
| `interacciones.publicacion_id` → `publicaciones.publicacion_id` | N:1 | 100 % |
| `relaciones.seguidor_id` → `usuarios.usuario_id` | N:1 | 100 % |
| `relaciones.seguido_id` → `usuarios.usuario_id` | N:1 | 100 % |
| `usuarios.nicho` ↔ `publicaciones.tema` | mismo dominio categórico | 6 = 6 valores, coinciden exactamente |

---

## 4. Observaciones relevantes para el análisis (no son errores)

| # | Observación [O] | Implicación |
|---|---|---|
| A1 | `verificado = TRUE` → media de 21,4 seguidores frente a 9,8; mediana de impresiones por publicación de 601 frente a 250; 86 interacciones recibidas por autor frente a 46. | Es un predictor fuerte del alcance. Como es *ficticio*, **no debe usarse como componente de un score** (sería circular o arbitrario). Sirve como variable de contraste. |
| A2 | Correlación de las interacciones por publicación con impresiones: 0,82; con `calidad_contenido`: 0,64. Correlación de impresiones con calidad: 0,50. | El engagement depende de la exposición, así que conviene normalizarlo (tasa por impresión). Tasas y conteos responden preguntas distintas. |
| A3 | La proporción de tipos de interacción es casi idéntica en todos los tipos de contenido (Me_gusta ≈ 64–66 %). | Ponderar por tipo de interacción es una decisión de negocio, no algo que salga de los datos. |
| A4 | Solo 838 de 35 810 interacciones (2,3 %) provienen de alguien que sigue al autor. Solo 773 de 8 133 aristas de seguimiento (9,5 %) tienen alguna interacción del seguidor sobre el contenido del seguido, y 534 de esas 838 interacciones ocurren después de la fecha de seguimiento. | El grafo social y el engagement son casi independientes (típico de datos sintéticos). El "valor de un seguidor por actividad hacia la cuenta seguida" tendrá muy poca señal; ver §6 y el plan. |
| A5 | Reciprocidad: 2,6 % en seguimiento (106 pares mutuos) y 9,4 % en interacción (3 140 de 33 557 pares usuario→autor tienen pares inversos). | Hay dos definiciones posibles de reciprocidad, con niveles distintos. Hay que decidir cuál se usa. |
| A6 | La ventana de interacción va de 0 a 14 días tras publicar. La última publicación es del 2025-09-17 y la última interacción, del 2025-10-01 (= +14). | No hay truncamiento por la derecha en el engagement: todas las publicaciones tuvieron su ventana completa. |
| A7 | Las ventanas temporales no se alinean: publicaciones de 2024-01 a 2025-09; seguimientos de 2022-01 a 2026-08; registros de 2018-01 a 2026-06. | Hay que fijar una **fecha de corte** explícita para las métricas de recencia y una ventana de análisis común. |

---

## 5. Problemas de calidad detectados

| ID | Severidad | Archivo(s) | Problema [O] | Magnitud | Tratamiento propuesto (sin tocar los crudos) |
|---|---|---|---|---|---|
| Q1 | **Alta** | publicaciones × usuarios | Publicación anterior al registro de su autor | 1 098 de 5 200 (21,1 %) | Marcar con `flag_pub_antes_registro`. No descartar. Tratar `fecha_registro` como poco confiable. |
| Q2 | **Alta** | interacciones × usuarios | Interacción anterior al registro de quien interactúa | 7 359 de 35 810 (20,6 %) | `flag_int_antes_registro`. Ídem. |
| Q3 | **Alta** | relaciones × usuarios | Seguimiento anterior al registro del seguidor (2 334), del seguido (2 260) o de cualquiera de los dos | 3 687 de 8 133 (45,3 %) | `flag_follow_antes_registro`. Ídem. |
| Q4 | Alta | usuarios | 65 usuarios registrados después de la última interacción (2025-10-01) tienen, aun así, 529 publicaciones y 3 583 interacciones | 65 usuarios | Consecuencia de Q1/Q2: **`fecha_registro` no sirve para calcular antigüedad ni para filtrar.** |
| Q5 | Media | usuarios | Coordenadas uniformes en todo el globo, incoherentes con `pais` (solo 33 de 650 caen en la caja aproximada de Sudamérica) | 617 de 650 | Excluir `latitud` y `longitud` de los análisis. Usar `pais` como única geografía. |
| Q6 | Media | relaciones | 1 573 seguimientos posteriores a la última interacción, y 1 149 con fecha de 2026 | 19,3 % | Definir la fecha de corte y la ventana común. Las métricas de "seguidor que interactúa" solo pueden usar aristas anteriores al 2025-10-01. |
| Q7 | Baja | publicaciones | Septiembre de 2025 incompleto (140 publicaciones frente a ~250/mes; termina el día 17) | 1 mes | Excluir el último mes parcial de las tendencias mensuales o marcarlo como parcial. |
| Q8 | Baja | interacciones / diccionario | `Me_gusta` (datos) frente a "Me gusta" (diccionario) | Cosmético | Normalizar etiquetas en `processed`. Documentar el mapeo. |
| Q9 | Baja | diccionario_datos | Documenta 7 de 26 campos | 19 campos | Cubierto por este documento. |
| Q10 | Baja | usuarios | Sesgo de país (Bolivia 40,6 %) | n/a | Tenerlo en cuenta al comparar por país (usar tasas, no conteos). |
| Q11 | Info | usuarios | `ocupacion` y `nombre` en inglés (Faker); `organizacion` con 7 repeticiones | n/a | No usar como dimensión analítica principal. |

**Sin problemas [O]:** no hay celdas vacías, duplicados de PK, duplicados de clave natural, FK huérfanas, autoseguimientos, autointeracciones (nadie interactúa con su propia publicación), interacciones anteriores a la publicación, fechas imposibles o futuras respecto al 2026-11-11, `calidad_contenido` fuera de [0,1], ni impresiones ≤ 0, y en ninguna publicación hay más interacciones que impresiones.

---

## 6. Información faltante y supuestos

| ID | Tipo | Descripción |
|---|---|---|
| F1 | [F] | **Fecha de extracción y periodo de referencia** de cada CSV. No se sabe si los archivos son fotos completas o incrementales. |
| F2 | [F] | Eventos de **dejar de seguir**; conteo de seguidores *externos* a la muestra de 650 perfiles. |
| F3 | [F] | **Alcance por usuario** (cuentas únicas alcanzadas); solo existen impresiones por publicación. |
| F4 | [F] | Hora de los eventos (solo hay fecha); texto de los comentarios; destino de los compartidos. |
| F5 | [F] | Definición y origen de `calidad_contenido` (¿es entrada o salida de otro modelo?). |
| F6 | [F] | Criterio de negocio que defina "alto valor" y los pesos por tipo de interacción. |
| F7 | [F] | Contrato de los nuevos CSV: ¿mismo esquema? ¿registros nuevos o fotos completas? ¿pueden cambiar filas existentes, como el nicho o `verificado`? |
| S1 | [S] | `relaciones.csv` representa relaciones vigentes a la fecha de extracción. |
| S2 | [S] | Los ID son estables entre entregas (`USR0001` es siempre el mismo perfil). |
| S3 | [S] | `impresiones` es un acumulado final de la publicación (ventana cerrada de 14 días; consistente con A6). |
| S4 | [S] | Las 650 cuentas forman una red cerrada: las métricas de red solo tienen sentido dentro de la muestra. |
