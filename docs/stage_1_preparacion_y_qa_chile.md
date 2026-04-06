# Stage 1 — Preparación y QA de datos municipales (Chile)

## Contexto del problema

El proyecto **Aggregated_MUNIFI_OECD** busca analizar la eficiencia y sostenibilidad fiscal de gobiernos locales (municipios) a partir de datos financieros públicos. Para Chile, el objetivo es construir un panel de datos (2010–2021) que permita:

- Comparar municipios en términos de gasto, ingresos, composición y balances.
- Identificar patrones de eficiencia, dependencia de transferencias y rigidez del gasto.
- Generar un pipeline replicable a otros países del mismo dataset.

El dataset de origen es **“MUNIFI Data Query - DISAGGREGATED DATA_Chile.csv”**, proporcionado por la OECD, que contiene observaciones anuales desagregadas por municipio y tipo de medida fiscal.

---

## Dataset de origen

- **Archivo**: `MUNIFI Data Query - DISAGGREGATED DATA_Chile.csv`
- **Filas**: 351 900
- **Columnas clave**: `Country`, `TL 2 Name`, `TL 3 Name`, `Municipality`, `REF_AREA`, `TIME_PERIOD`, `MEASURE`, `UNIT_MEASURE`, `Unit of measure`, `OBS_VALUE`
- **Período**: 2010–2021
- **Municipios**: 345
- **Unidades disponibles** (`UNIT_MEASURE`):
  - `XDC`: moneda nacional (monto total)
  - `XDC_PS`: moneda nacional per cápita
  - `USD_PPP`: dólares PPP (monto total)
  - `USD_PPP_PS`: dólares PPP per cápita
- **Medidas fiscales (`MEASURE`)**: incluye gastos (`E1`, `E11`, `E12`, `E111`, `E115`, `E121`) e ingresos (`R1`, `R11`, `R12`, `R13`, `R14`), además de otras variables (`EC*`, `POP`).

---

## Tecnologías utilizadas

- **Python 3**
- **pandas** para manipulación y transformación de datos
- **numpy** para operaciones vectorizadas
- **Jupyter Notebook** para el análisis exploratorio y la documentación del pipeline
- **Markdown** para documentación y resúmenes
- **Git** (implícito) para control de versiones

---

## Objetivo del análisis (Stage 1)

En esta primera etapa, el objetivo es **preparar y validar los datos** para que sean consistentes, comparables y listos para el análisis exploratorio posterior (Stage 2). Se busca:

1. Garantizar una **llave única** para cada observación (municipio-año-medida-unidad).
2. Separar los datos por unidad para evitar mezclar escalas.
3. Validar la **cobertura temporal y por indicador**.
4. Detectar y marcar valores inusuales (negativos en `R14`).
5. Verificar la **consistencia contable** (totales vs componentes).
6. Generar **features base** (balances, ratios) y exportar datasets listos para modelar.

---

## Pasos seguidos

### Paso 1 — Diagnóstico de duplicados lógicos
- Se probó una llave inicial sin incluir `UNIT_MEASURE`.
- Se detectó que cada combinación municipio-año-medida aparecía **5 veces**.
- Al inspeccionar, se vio que la multiplicidad se explica por **5 unidades** distintas.
- Decisión: **separar por unidad** para tener una llave única.

### Paso 2 — Creación de 4 paneles anchos (por unidad)
- Se pivotó el dataset largo a formato ancho (panel) para cada `UNIT_MEASURE`.
- Índice del panel: `['Country','TL 2 Name','TL 3 Name','Municipality','REF_AREA','TIME_PERIOD']`.
- Columnas del panel: cada `MEASURE` como columna (`E1`, `R1`, etc.).
- Shapes resultantes: `(4140, 22)` para cada unidad (345 municipios × 12 años).

### Paso 3 — Cobertura y definición de panel balanceado
- Se verificó que **todos los municipios tienen 2010–2021 completo**.
- No hay nulos en `E1` ni `R1` en los paneles.
- Conclusión: el panel `XDC` ya es balanceado por cobertura y core measures.

### Paso 4 — Diagnóstico de valores negativos
- Se detectaron **80 valores negativos** en todo el dataset.
- Todos pertenecen a `MEASURE = R14` (Income from assets), distribuidos por igual entre las 4 unidades.
- Decisión: **conservar los negativos** y crear un flag `flag_r14_negative` para análisis posterior.

### Paso 5 — Validaciones jerárquicas (contables)
- Se verificó que los cierres contables cumplan una tolerancia del 1%:
  - `E1 ≈ E11 + E12`
  - `E11 ≈ E111 + E115`
  - `R1 ≈ R11 + R12 + R13 + R14`
- **Resultado**: 0 filas con inconsistencias (todos cierran perfectamente).
- Tolerancia adoptada como estándar: `tol = 0.01`.

### Paso 6 — Creación de features base y exportación
Para cada panel, se calcularon:
- `balance = R1 - E1`
- `balance_ratio = balance / R1`
- Ratios de composición del gasto (`current_exp_ratio`, `capital_exp_ratio`, etc.)
- Ratios de composición de ingresos (`tax_ratio`, `grants_ratio`, etc.)
- Se mantuvo `flag_r14_negative` y se agregó `UNIT_MEASURE` para trazabilidad.

Se exportaron 4 CSVs a `data/processed/`:
- `chile_panel_XDC_features.csv`
- `chile_panel_XDC_PS_features.csv`
- `chile_panel_USD_PPP_features.csv`
- `chile_panel_USD_PPP_PS_features.csv`

---

## Resultados y entregables

- **4 datasets listos para análisis** (panel ancho + features + flags), uno por unidad.
- **Documentación de la etapa** (este archivo y `docs/fase_1_unidades_y_paneles.md`).
- **Pipeline replicable** para otros países (mismos pasos, misma lógica de unidad + QA).
- **Base sólida para el Stage 2**: análisis exploratorio, clustering, rankings y modelado.

---

## Próximos pasos (Stage 2)

- Cargar `panel_xdc_features.csv` como base para el EDA.
- Explorar distribuciones, tendencias temporales, rankings municipales.
- Analizar eficiencia y sostenibilidad fiscal con las features generadas.
- Comparar resultados entre `XDC` y `XDC_PS` (montos vs per cápita).

---

## Notas metodológicas

- **Unidades**: se recomienda usar `XDC` para análisis de montos y `XDC_PS` para análisis per cápita. `USD_PPP` y `USD_PPP_PS` quedan disponibles para comparabilidad cross-country.
- **Negativos**: los valores negativos en `R14` se marcaron, no se eliminaron, para poder evaluar si afectan el análisis de sostenibilidad.
- **Cobertura**: Chile tiene cobertura completa (2010–2021) para todos los municipios y medidas core, lo cual es ideal para un panel balanceado.
- **Escalabilidad**: el pipeline está diseñado para aplicarse a otros países del mismo dataset, ajustando solo las rutas de archivos y, eventualmente, tolerancias si hubiere inconsistencias.
