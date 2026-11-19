# 🎬 StreamView Analytics · EP5 y EP6

Proyecto de la asignatura **ADY1104 Visualización de Datos** (Duoc UC, 2026). Solución integral de visualización para el caso **StreamView Analytics**: un **dashboard interactivo en Power BI** (8 páginas) y un **informe ejecutivo**, construidos sobre el catálogo integrado de películas y series de la plataforma.

| | |
|---|---|
| **Equipo** | Sebastián Aird · María Calfileo · Stephanie Chamorro · Rocío Cruces |
| **Docente** | Alicia Ramos |
| **Evaluaciones** | EP5 (encargo: informe y dashboard) · EP6 (presentación y defensa) |
| **Herramientas** | Power BI Desktop · Python (pandas, matplotlib, seaborn, plotly, streamlit) · Jupyter |

---

## 1. Problema de negocio

Marketing, la Gerencia de Contenidos y Adquisición y la Dirección consultan hoy reportes construidos por separado, con criterios distintos de limpieza y agregación. Eso produce versiones distintas de un mismo indicador (por ejemplo, la "popularidad promedio" de un país cambia según se pondere o no por género, o según se mezclen películas con series).

**Pregunta central:** ¿qué géneros, países y tipos de contenido debería priorizar StreamView Analytics en adquisición y promoción, con cifras que resistan una auditoría?

**Decisión que apoya:** el comité trimestral de contenidos define en qué géneros, países y tipos concentrar las próximas adquisiciones y campañas.

## 2. Objetivos

- Integrar los catálogos de películas y series en una sola fuente analítica, documentando con evidencia propia los problemas de calidad.
- Construir KPIs y visualizaciones que respondan las preguntas analíticas priorizadas.
- Diseñar un dashboard de ocho vistas con filtros, navegación e indicadores consistentes entre páginas.
- Evaluar críticamente la solución y corregir, con evidencia recalculada, las observaciones docentes de EV2 y EV3.

## 3. Datos

| Fuente | Archivo | Registros | Variables |
|---|---|---|---|
| Películas | `data/raw/netflix_movies_detailed_up_to_2025.csv` | 16.000 | 18 |
| Series | `data/raw/netflix_tv_shows_detailed_up_to_2025.csv` | 16.000 | 16 |
| Catálogo integrado | `data/processed/catalogo_integrado.csv` | 31.991 (16.000 películas y 15.991 series) | 19 |

**Integración.** La clave analítica es `content_type + show_id` (397 `show_id` aparecen tanto en películas como en series, pero son contenidos distintos). En series se detectaron 9 `show_id` repetidos (18 filas) con el mismo título y metadatos; se consolidan en una sola fila promediando `popularity`. La auditoría está en `data/processed/auditoria_calidad_ids.csv`. Los archivos de `data/raw/` no se modifican.

**Calidad de datos y tratamiento**

| Problema | Evidencia | Decisión |
|---|---|---|
| Títulos sin votos | 4.560 de 31.991 (14,25%) con `vote_count = 0` | Advertir: se excluyen de toda valoración |
| Pocos votos | Chile: 3,96 sin umbral → 6,53 con `vote_count >= 10` (n = 45) | Limpiar: umbral de 10 votos |
| Datos financieros parciales | Solo 3.540 de 16.000 películas (22,1%) con `budget` y `revenue` > 0 | Advertir y limitar el análisis financiero a ese subconjunto |
| Géneros múltiples | Un título con 3 géneros suma 3 veces; en Drama-Brasil el promedio pasa de 97,2 a 118,1 | Limpiar: ponderar por 1/n géneros |
| Muestra con tope | 1.000 películas y 1.000 series por año de estreno | Advertir: nota metodológica en el dashboard |
| Variable vacía | `duration` vacía en el 100% de las películas | Descartar |

## 4. Reglas de negocio

- `popularity` es un índice relativo de interés, **no** reproducciones.
- Toda medida de valoración exige `vote_count >= 10` (valoración promedio global: **6,63**; popularidad promedio: **42,63**).
- La popularidad por género se pondera por **1/n géneros** del título.
- Nunca se promedia popularidad entre películas y series sin segmentar por `content_type`.
- Los indicadores financieros usan solo películas con `budget` y `revenue` disponibles a la vez.
- Los rankings por género exigen una muestra mínima (100 títulos; 30 películas con datos financieros en el ROI).

## 5. Dashboard (Power BI)

Archivo: `dashboard/StreamView_Dashboard_v3_1.pbix`. Capturas en `images/dashboard/`.

| Página | Qué responde |
|---|---|
| Resumen ejecutivo | ¿Cómo está el catálogo en general? |
| Análisis por género | ¿Qué géneros dominan el catálogo, cuáles son más populares y cuáles tienen mejor calidad? |
| Evolución del catálogo | ¿Cuánto contenido se estrena cada año y cómo cambia por tipo y género? |
| Desempeño financiero | ¿Qué géneros generan más ingresos y qué retorno logran? |
| Valoración vs. popularidad | ¿Los géneros mejor valorados son también los más populares? |
| Mercados: país e idioma | ¿Qué países e idiomas concentran popularidad y valoración? |
| Catálogo | ¿Qué títulos concretos dieron ganancia o pérdida? |
| Insights ejecutivos | ¿Qué ocurre, por qué, con qué evidencia y qué decisiones tomar? |

Todas las páginas tienen segmentadores (tipo de contenido, año 2010–2025, país, idioma y género), una frase guía bajo el título y la misma barra de navegación al pie. El diseño usa fondo negro y rosa como único color de marca.

**Medidas DAX clave**

```dax
Vote Average Válido = AVERAGEX(FILTER(catalogo, catalogo[vote_count] >= 10), catalogo[vote_average])
Peso por género = DIVIDE(1, catalogo[n_generos_titulo])
Popularidad promedio = DIVIDE(SUMX(catalogo, catalogo[popularity] * [Peso por género]),
                              SUMX(catalogo, [Peso por género]))
```

Existe además una versión exploratoria en **Streamlit + Plotly** (`dashboard/app.py`).

## 6. Hallazgos principales

- Volumen y popularidad son dimensiones distintas: Drama y Comedy dominan el catálogo, pero no la popularidad (en películas, Adventure 35,5 frente a Drama 16,7).
- En series lideran Soap (160), Talk (123) y News (92), géneros con pocos títulos.
- Popularidad y valoración casi no se relacionan (r = 0,12); la relación más fuerte es presupuesto–ingresos (r = 0,75).
- Con `vote_count >= 10`, el top 3 de países por valoración pasa a ser Colombia, Japón y México.
- "Latinoamérica lidera en popularidad" solo es cierto en series: en películas lidera Estados Unidos.
- Documentary es el género mejor valorado (7,10) y Soap el más popular, por lo que no coinciden.

## 7. Estructura del repositorio

```
.
├── data/
│   ├── raw/            CSV originales (sin modificar)
│   ├── processed/      catalogo_integrado.csv, resúmenes y auditoría de IDs
│   └── looker/         exportación para Looker Studio
├── notebooks/          01 y 02: EDA de popularidad por género (películas y series)
├── dashboard/          StreamView_Dashboard_v3_1.pbix y app.py (Streamlit)
├── src/                preprocessing.py, analysis.py, export_looker.py
├── images/             figuras de los notebooks
│   ├── dashboard/      capturas del dashboard
│   └── informe/        figuras del informe
├── Documentos/         informe ejecutivo EP5/EP6 y entregables de etapas previas
├── requirements.txt
└── README.md
```

## 8. Cómo reproducir

1. Crear y activar un entorno virtual, e instalar dependencias:
   ```bash
   python -m venv .venv
   pip install -r requirements.txt
   ```
2. Regenerar el catálogo integrado con `build_integrated_catalog()` de `src/preprocessing.py`, o ejecutar en orden los notebooks de `notebooks/`.
3. Abrir `dashboard/StreamView_Dashboard_v3_1.pbix` en Power BI Desktop, actualizar el origen de datos si cambió la ruta y ejecutar **Actualizar**.
4. (Opcional) Lanzar la versión Streamlit:
   ```bash
   python -m streamlit run dashboard/app.py
   ```

## 9. Limitaciones y mejoras

- **Limitaciones:** `popularity` es un proxy y no mide reproducciones; solo el 22,1% de las películas tiene datos financieros; la muestra con tope de 1.000 títulos por tipo y año no mide el crecimiento real del catálogo.
- **Mejoras priorizadas:** paleta apta para daltonismo en ganancia/pérdida; solicitar la extracción completa del catálogo; mostrar el tamaño de muestra en los tooltips; auditar `budget` y `revenue` en 0 frente a nulo; agregar texto alternativo a los visuales.
- **Datos adicionales deseables:** horas de reproducción, retención, costo de licencia por título y fecha real de incorporación por país.

## 10. Documentación

- Informe ejecutivo: `Documentos/Informe_Ejecutivo_EP5_EP6_StreamView.docx`.
- Pauta de evaluación: EP5 y EP6 de ADY1104 Visualización de Datos, Duoc UC.
