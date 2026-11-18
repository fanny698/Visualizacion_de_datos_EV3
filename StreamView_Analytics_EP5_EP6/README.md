# StreamView Analytics · EP5 y EP6

Proyecto de **ADY1104 Visualización de Datos** (Duoc UC). Dashboard interactivo en **Power BI** e informe ejecutivo sobre el catálogo integrado de películas y series de StreamView Analytics (31.991 títulos: 16.000 películas y 15.991 series).

**Equipo:** Sebastián Aird · María Calfileo · Stephanie Chamorro · Rocío Cruces
**Docente:** Alicia Ramos

**Pregunta central:** ¿qué géneros, países y tipos de contenido debería priorizar StreamView Analytics en adquisición y promoción, con cifras que resistan una auditoría?

## Estructura

```
data/          raw/ (CSV originales), processed/ (catalogo_integrado.csv y resúmenes), looker/
notebooks/     01 y 02: análisis de popularidad por género (películas y series)
dashboard/     StreamView_Dashboard_v3_1.pbix (8 páginas) y app.py
images/        figuras de los notebooks; dashboard/ (capturas); informe/ (figuras del informe)
src/           preprocessing.py, analysis.py, export_looker.py
Documentos/    Informe_Ejecutivo_EP5_EP6_StreamView.docx y entregables de etapas previas
requirements.txt
```

## Reglas de negocio aplicadas

- `popularity` es un índice relativo de interés, no reproducciones.
- Toda medida de valoración exige `vote_count >= 10` (valoración promedio global: 6,63).
- La popularidad por género se pondera por 1/n géneros del título.
- Nunca se promedia popularidad entre películas y series sin segmentar por `content_type`.
- La muestra trae como máximo 1.000 películas y 1.000 series por año de estreno: las líneas por año son planas por construcción y no miden crecimiento real.

## Dashboard (8 páginas)

Resumen ejecutivo · Análisis por género · Evolución del catálogo · Desempeño financiero · Valoración vs. popularidad · Mercados: país e idioma · Catálogo · Insights ejecutivos. Capturas en `images/dashboard/`.

## Cómo reproducir

1. `python -m venv .venv`, activar el entorno y `pip install -r requirements.txt`.
2. Ejecutar en orden los notebooks de `notebooks/` para regenerar `data/processed/catalogo_integrado.csv`.
3. Abrir `dashboard/StreamView_Dashboard_v3_1.pbix` en Power BI Desktop y actualizar el origen de datos si cambió la ruta.
4. Verificar las medidas `Vote Average Válido` y `Popularidad promedio` (ponderada) y ejecutar Actualizar.
