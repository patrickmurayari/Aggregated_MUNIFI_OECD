# Aggregated_MUNIFI_OECD

Análisis de finanzas municipales de países OECD a partir del dataset MUNIFI (Municipal Finance). Este proyecto prepara, valida y explora datos fiscales desagregados para evaluar eficiencia, sostenibilidad y patrones de gasto e ingresos a nivel local.

## Contexto

El proyecto **Aggregated_MUNIFI_OECD** busca analizar la eficiencia y sostenibilidad fiscal de gobiernos locales (municipios) a partir de datos financieros públicos de la OECD. Para Chile, el objetivo es construir un panel de datos (2010–2021) que permita:

- Comparar municipios en términos de gasto, ingresos, composición y balances
- Identificar patrones de eficiencia, dependencia de transferencias y rigidez del gasto
- Generar un pipeline replicable a otros países del mismo dataset

## Dataset

- **Archivo**: `MUNIFI Data Query - DISAGGREGATED DATA_Chile.csv`
- **Período**: 2010–2021
- **Municipios**: 345
- **Unidades**: XDC (moneda nacional), XDC_PS (per cápita), USD_PPP, USD_PPP_PS
- **Medidas**: Gastos (E1, E11, E12, E111, E115, E121), Ingresos (R1, R11, R12, R13, R14), población y otras variables

## Estructura del proyecto

```
├── data/
│   ├── raw/           # Datos originales de OECD
│   └── processed/     # Datasets limpios y con features por unidad
├── notebooks/         # Análisis exploratorio y pipeline de datos
├── docs/              # Documentación por etapa
└── README.md
```

## Etapas del análisis

### Stage 1 — Preparación y QA ✅
- Diagnóstico de duplicados lógicos por unidad de medida
- Creación de 4 paneles anchos (uno por unidad)
- Validación de cobertura y consistencia contable
- Detección y marcado de valores negativos (R14)
- Generación de features base (balances, ratios)
- Exportación de datasets listos para análisis

**Resultados**: 4 datasets limpios con features (`data/processed/`) y documentación completa en `docs/stage_1_preparacion_y_qa_chile.md`.

### Stage 2 — Análisis exploratorio (próximo)
- Explorar distribuciones, tendencias temporales y rankings municipales
- Analizar eficiencia y sostenibilidad fiscal con las features generadas
- Comparar resultados entre unidades (montos vs per cápita)

## Tecnologías

- Python 3, pandas, numpy
- Jupyter Notebook para análisis reproducible
- Markdown para documentación
- Git para control de versiones

## Cómo usar este proyecto

1. **Datos crudos**: Colocar el archivo CSV de OECD en `data/raw/`
2. **Preparación**: Ejecutar el notebook de Stage 1 para generar datasets limpios
3. **Análisis**: Usar los CSV en `data/processed/` para exploración y modelado
4. **Documentación**: Revisar `docs/` para detalles metodológicos por etapa

## Escalabilidad

El pipeline está diseñado para replicarse a otros países del dataset MUNIFI, manteniendo la misma lógica de:
- Separación por unidades de medida
- Validaciones contables con tolerancia
- Generación de features comparables

## Próximos pasos

- Completar análisis exploratorio (Stage 2)
- Extender el pipeline a otros países OECD
- Desarrollar dashboards y visualizaciones interactivas
