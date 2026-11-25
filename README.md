# StreamView Analytics · EP5 y EP6

Proyecto de **ADY1104 Visualización de Datos** (Duoc UC, 2026). Solución de visualización para StreamView Analytics: un **dashboard interactivo en Power BI** (8 páginas), un **informe ejecutivo** y una **presentación**, construidos sobre el catálogo integrado de películas y series (31.991 títulos: 16.000 películas y 15.991 series).

| | |
|---|---|
| **Equipo** | Sebastián Aird · María Calfileo · Stephanie Chamorro · Rocío Cruces |
| **Docente** | Alicia Ramos |
| **Evaluaciones** | EP5 (encargo: informe y dashboard) · EP6 (presentación y defensa) |
| **Herramientas** | Power BI Desktop · Python (pandas, matplotlib, seaborn, plotly, streamlit) · Jupyter |

---

## 1. Estructura del proyecto

```
StreamView_Analytics_EP5_EP6/
├── data/
│   ├── raw/                    CSV originales de películas y series (no se modifican)
│   ├── processed/              catalogo_integrado.csv, resúmenes y auditoría de IDs
│   └── looker/                 streamview_generos_looker.csv → fuente del dashboard
├── notebooks/
│   ├── 01_analisis_popularidad_por_genero.ipynb          EDA de películas
│   └── 02_analisis_series_popularidad_por_genero.ipynb   EDA de series
├── src/
│   ├── preprocessing.py        limpieza e integración (crea catalogo_integrado.csv)
│   ├── export_looker.py        tabla título × género con pesos (crea la fuente del dashboard)
│   └── analysis.py             funciones de KPIs y agregaciones usadas en los notebooks
├── dashboard/
│   ├── StreamView_Dashboard_v3_1.pbix                    dashboard final (Power BI)
│   └── app.py                  versión exploratoria en Streamlit + Plotly
├── images/                     figuras del EDA generadas por los notebooks
│   ├── dashboard/              capturas de las 8 páginas del dashboard (01 a 08)
│   └── informe/                figuras del informe ejecutivo
├── Documentos/
│   ├── Informe_Ejecutivo_EP5_EP6_StreamView.pdf          informe ejecutivo (EP5)
│   ├── Presentacion_EP6_StreamView.pdf                   presentación / resumen ejecutivo (EP6)
│   └── etapas_previas/         entregables de EP3 (referencia)
├── requirements.txt
└── README.md
```

**Flujo de datos:**

```
data/raw/*.csv ──► src/preprocessing.py ──► data/processed/catalogo_integrado.csv
                                                     │
                                                     ▼
                         src/export_looker.py ──► data/looker/streamview_generos_looker.csv
                                                     │
                                                     ▼
                         dashboard/StreamView_Dashboard_v3_1.pbix ──► images/dashboard/ ──► informe y presentación
```

## 2. Problema de negocio, audiencia y objetivos

**Contexto.** Marketing, la Gerencia de Contenidos y Adquisición y la Dirección consultan reportes construidos por separado, con criterios distintos de limpieza y agregación. Eso produce versiones distintas de un mismo indicador. Por ejemplo, la popularidad promedio de un país cambia según se pondere o no por género, o según se mezclen películas con series.

**Pregunta central:** ¿qué géneros, países y tipos de contenido debería priorizar StreamView Analytics en adquisición y promoción, con cifras que resistan una auditoría?

**Eje de análisis:** género × país × tipo de contenido.

**Audiencia y decisión que apoya:** el comité trimestral de contenidos (Dirección, Contenidos y Adquisición, Marketing) decide en qué géneros, países y tipos concentrar las próximas adquisiciones y campañas.

**Objetivos**
- Integrar los catálogos de películas y series en una sola fuente analítica, documentando con evidencia propia los problemas de calidad.
- Construir KPIs y visualizaciones que respondan las preguntas analíticas priorizadas.
- Diseñar un dashboard de ocho vistas con filtros, navegación e indicadores consistentes entre páginas.
- Evaluar críticamente la solución y corregir, con cifras recalculadas, las observaciones docentes.

## 3. Datos

| Fuente | Archivo | Registros | Variables |
|---|---|---|---|
| Películas | `data/raw/netflix_movies_detailed_up_to_2025.csv` | 16.000 | 18 |
| Series | `data/raw/netflix_tv_shows_detailed_up_to_2025.csv` | 16.000 | 16 |
| Catálogo integrado | `data/processed/catalogo_integrado.csv` | 31.991 títulos | 19 |
| Fuente del dashboard | `data/looker/streamview_generos_looker.csv` | una fila por título × género | 27 |

**Integración.** La clave analítica es `content_type + show_id` (397 `show_id` aparecen tanto en películas como en series, pero son contenidos distintos). En series hay 9 `show_id` repetidos (18 filas) con el mismo título y metadatos: se consolidan en una sola fila promediando `popularity`. La auditoría está en `data/processed/auditoria_calidad_ids.csv`.

**Calidad de datos y tratamiento**

| Problema | Evidencia | Decisión |
|---|---|---|
| Títulos sin votos | 4.560 de 31.991 (14,25%) con `vote_count = 0` | Se excluyen de toda valoración |
| Pocos votos | Chile: 3,96 con todos sus títulos, 4,51 sin los de 0 votos y 6,53 con `vote_count >= 10` (n = 45) | Umbral de 10 votos |
| Datos financieros parciales | 3.540 de 16.000 películas (22,1%) con `budget` y `revenue` > 0 | Análisis financiero solo sobre ese subconjunto |
| Géneros múltiples | Un título con 3 géneros se cuenta 3 veces. Drama en Brasil: 118,1 con promedio simple, 97,2 ponderado | Ponderar por 1/n géneros |
| Muestra con tope | Máximo 1.000 películas y 1.000 series por año de estreno | No se grafica la cantidad por año; nota metodológica |
| Fecha de incorporación | El año de `date_added` coincide con `release_year` en el 100% de los títulos | No se analizan incorporaciones |
| Variable vacía | `duration` vacía en el 100% de las películas | Se descarta |

## 4. Reglas de negocio

- `popularity` es un índice relativo de interés, **no** reproducciones.
- Toda valoración exige `vote_count >= 10` y se calcula por título. Valoración promedio global: **6,63**. Popularidad promedio ponderada: **42,63**.
- La popularidad por género se pondera por **1/n géneros** del título.
- Nunca se promedia popularidad entre películas y series sin segmentar por `content_type`.
- Los indicadores financieros usan solo películas con `budget` y `revenue` a la vez.
- Los rankings exigen muestra mínima: 100 títulos por género o país; 30 películas con datos financieros para el ROI.

## 5. Dashboard (Power BI)

Archivo: `dashboard/StreamView_Dashboard_v3_1.pbix`. Capturas en `images/dashboard/`.

| # | Página | Qué responde |
|---|---|---|
| 1 | Resumen ejecutivo | ¿Cómo está el catálogo en general? |
| 2 | Análisis por género | ¿Qué géneros dominan el catálogo, cuáles son más populares y cuáles mejor valorados? |
| 3 | Evolución del catálogo | ¿Cómo cambian la popularidad y la valoración según el año de estreno, por tipo y género? |
| 4 | Desempeño financiero | ¿Qué géneros generan más ingresos y qué retorno logran? |
| 5 | Valoración vs. popularidad | ¿Los géneros mejor valorados son también los más populares? |
| 6 | Mercados: país e idioma | ¿Qué países e idiomas concentran popularidad y valoración, por tipo de contenido? |
| 7 | Catálogo | ¿Qué títulos concretos dieron ganancia o pérdida? |
| 8 | Insights ejecutivos | ¿Qué ocurre, por qué y qué decisiones tomar? |

**Diseño y accesibilidad.** Fondo negro con rosa como color de marca. Películas en gris y series en rosa en todas las páginas. Barra de navegación en la misma posición en las 8 páginas. Segmentadores desplegables (tipo, año 2010–2025, país, idioma y género). Ejes y etiquetas en 12 pt como mínimo, en rosa claro (#FEB9D4). Ganancia en azul y pérdida en naranjo (paleta segura para daltonismo). Texto alternativo en todos los visuales.

**Medidas DAX clave**

```dax
Valoración promedio =
CALCULATE (
    AVERAGEX ( VALUES ( catalogo[content_key] ), CALCULATE ( MAX ( catalogo[vote_average] ) ) ),
    catalogo[vote_count] >= 10
)

Popularidad promedio =
DIVIDE ( SUM ( catalogo[popularidad_x_peso] ), SUM ( catalogo[peso] ) )
```

## 6. Hallazgos y recomendaciones

- **Volumen y popularidad son dimensiones distintas.** Drama tiene 14.769 títulos, pero en películas lidera la popularidad Adventure (35,1 frente a 15,8 de Drama).
- **En series lideran Soap (160) y Talk (123).** Son géneros exclusivos de series, con volumen moderado (631 y 654 títulos), y su promedio lo elevan pocos títulos muy populares (Soap: mediana 88,4 frente a un promedio de 155,5 sin ponderar). Sirven para campañas de series, no como señal general.
- **Popularidad y valoración casi no se relacionan** (r = 0,12). La relación más fuerte es presupuesto–ingresos (r = 0,75).
- **Mercados:** en series lideran Brasil (131,9) y México (98,9); en películas, Estados Unidos (25,0). Por valoración, el top 3 es Colombia (7,31), Japón (7,24) y México (7,22).
- **Calidad:** Documentary es el género de películas mejor valorado (7,10).
- **Recomendación:** series de Drama y Soap para Brasil y México; películas de Action y Adventure para Estados Unidos; Documentary como apuesta de calidad.

## 7. Cómo reproducir

Desde la carpeta `StreamView_Analytics_EP5_EP6/`:

1. Crear el entorno e instalar dependencias:
   ```bash
   python -m venv .venv
   .venv\Scripts\activate        # Windows  (macOS/Linux: source .venv/bin/activate)
   pip install -r requirements.txt
   ```
2. Regenerar los datos procesados (catálogo integrado y fuente del dashboard):
   ```bash
   python src/preprocessing.py
   python src/export_looker.py
   ```
   Los notebooks de `notebooks/` reproducen el análisis exploratorio y las figuras de `images/`.
3. Abrir `dashboard/StreamView_Dashboard_v3_1.pbix` en Power BI Desktop. En **Transformar datos → Configuración del origen de datos → Cambiar origen**, apuntar a `data/looker/streamview_generos_looker.csv` de esta carpeta (el archivo guarda la ruta absoluta del equipo donde se creó) y presionar **Actualizar**.
4. (Opcional) Versión Streamlit:
   ```bash
   python -m streamlit run dashboard/app.py
   ```

## 8. Evaluación crítica

- **Fortalezas:** una sola fuente integrada y auditada; reglas de negocio explícitas y aplicadas igual en todas las páginas; separación sistemática por tipo de contenido.
- **Limitaciones:** `popularity` es un proxy y no mide reproducciones; solo el 22,1% de las películas tiene datos financieros; la muestra con tope de 1.000 títulos por tipo y año no mide el crecimiento real del catálogo; no hay fecha real de incorporación.
- **Mejoras futuras:** pedir la extracción completa del catálogo; mostrar el tamaño de muestra en los tooltips; auditar `budget` y `revenue` en 0 frente a nulo; incorporar horas de reproducción, retención y costo de licencia por título.

## 9. Correcciones aplicadas tras la revisión docente (EV3)

| Indicador | Corrección |
|---|---|
| IE27 | Valoración global 6,63 en todos los productos; Chile explicado con los tres criterios (3,96 / 4,51 / 6,53); `date_added` coincide con `release_year` en el 100% de los títulos; Soap y Talk descritos con su volumen real y su sesgo; archivo entregado `v3_1.pbix`. |
| IE25 | Evolución con popularidad y valoración por año en lugar de líneas planas; dispersión presupuesto vs. ingresos con puntos semitransparentes; etiquetas legibles en "5 géneros con menos ingresos" (Western suma 925 millones). |
| IE29 | Textos sin cortes, tipografía mínima de 12 pt, texto alternativo en todos los visuales, ganancia y pérdida en azul y naranjo, separador de miles, colores distintos por género. |
| IE28 | Barra de navegación idéntica en todas las páginas, segmentadores compactos, figuras del informe con la paleta del dashboard. |
| IE30 e IE34 | Capturas de las 8 páginas; presentación centrada en la pregunta de negocio y el eje género × país × tipo, con la página de Mercados. |