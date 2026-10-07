# Laboratorio de Datos — Trabajos Prácticos

Dos proyectos de la materia **Laboratorio de Datos** (Lic. en Ciencias de Datos, FCEyN-UBA, 2do cuatrimestre 2024):

| Proyecto | Pregunta | Técnicas | Resultado principal |
|---|---|---|---|
| [TP1 — Migración y representación argentina](#tp1--migración-y-representación-argentina) | ¿Argentina tiene más sedes diplomáticas en los países con mayor flujo migratorio? | Limpieza y calidad de datos (GQM), normalización, modelo entidad-relación, SQL, visualización | Los países con más sedes tienden a mostrar mayor flujo migratorio, con excepciones |
| [TP2 — Clasificación de dígitos (TMNIST)](#tp2--clasificación-de-dígitos-tmnist) | ¿Qué tan bien se pueden clasificar dígitos escritos en miles de tipografías distintas? | EDA, selección de atributos por varianza, KNN, árboles de decisión, validación cruzada | Exactitud de **93.5%** sobre datos no vistos (10 clases) |

**Stack:** Python · pandas · SQL (`inline_sql`) · scikit-learn · matplotlib · seaborn

---

## TP1 — Migración y representación argentina

**Pregunta:** ¿existe relación entre los flujos migratorios con cada país y la cantidad de representaciones (embajadas, consulados) que Argentina mantiene allí? La respuesta sirve, por ejemplo, para detectar destinos con mucha migración y poca representación.

**Datos**
- [Global Bilateral Migration — World Bank](https://databank.worldbank.org/source/global-bilateral-migration)
- [Representaciones argentinas en el exterior — datos.gob.ar](https://datos.gob.ar/dataset/exterior-representaciones-argentinas)

**Qué hice**
1. **Calidad de datos.** Medí cada problema con la metodología GQM (*Goal–Question–Metric*) antes y después de corregirlo:
   - ~4.7% de los valores de migración faltantes (marcados como `..`), documentados como no recuperables.
   - Horarios de atención cargados en la columna equivocada en el 49.2% de las filas; al reubicarlos, el problema bajó al 44.2% (el resto falta en la fuente).
   - Una decena de representaciones distintas de "valor nulo" en códigos postales (`0`, `s/c`, `no existe`, `-----`, …), unificadas en una sola (84% → 0% de inconsistencia).
   - Una fila desplazada una celda y nombres de países sin estandarizar, corregidos.
2. **Modelado.** Analicé en qué forma normal estaba cada tabla (una no cumplía 1FN por tener varias redes sociales en una misma celda), diseñé un modelo entidad-relación y separé los datos en tablas normalizadas.
3. **Reportes en SQL.** Escribí cuatro consultas: sedes y flujo migratorio por país, promedio de flujo por región, diversidad de redes sociales por país y detalle de redes por sede. Los resultados completos están en `tp1/Anexo Reportes/`.
4. **Visualización.** Cantidad de sedes por región, distribución del flujo migratorio por región y relación sedes ↔ flujo. Un *scatterplot* no mostraba nada claro, así que pasé a *boxplots* agrupados por cantidad de sedes.

**Resultados**
- Los países con más sedes argentinas tienden a tener mayor flujo migratorio (datos del año 2000). Los países con 11 sedes son una excepción, aunque son solo 3 casos.
- América del Sur y Europa Occidental concentran la mayoría de las sedes.
- La relación encontrada es una **correlación**, no una relación causal.

📄 Informe completo: [`tp1/Informe TP1 - Laboratorio de Datos.pdf`](tp1/Informe%20TP1%20-%20Laboratorio%20de%20Datos.pdf)

---

## TP2 — Clasificación de dígitos (TMNIST)

**Pregunta:** ¿se pueden clasificar dígitos del 0 al 9 cuando cada uno aparece escrito en 2990 tipografías distintas (normales, negrita, cursiva, solo contorno)?

**Datos:** [TMNIST — Typeface MNIST (Kaggle)](https://www.kaggle.com/datasets/nimishmagre/tmnist-typeface-mnist). Son 29 900 imágenes de $28 \times 28$ píxeles en escala de grises, con 10 clases balanceadas.

**Qué hice**
1. **Análisis exploratorio.**
   - Calculé la varianza de cada píxel: la información se concentra en el centro de la imagen y hay píxeles que valen siempre 0.
   - Construí "dígitos promedio" y comparé las diferencias entre ellos para anticipar qué pares se iban a confundir (por ejemplo, 3 con 8).
2. **Clasificación binaria (0 vs 1) con KNN.**
   - Usé la varianza como criterio para seleccionar atributos.
   - Comparé valores de $k$ entre 1 y 20 y distintos subconjuntos de píxeles, midiendo exactitud, precisión, recall y F1 en train y test.
   - Los píxeles de mayor varianza dieron sistemáticamente mejores métricas.
3. **Clasificación multiclase con árboles de decisión.**
   - Separé un conjunto *held-out* del 20% que no se usó en ningún momento del ajuste.
   - Hice validación cruzada con 5 *folds* sobre profundidades de 1 a 10, con los criterios Gini y Entropy.
   - Elegí el modelo con Entropy y profundidad 10.

**Resultados**
- **Exactitud de 0.9346 en el conjunto held-out** para las 10 clases.
- Los pares que más se confundieron fueron 6/5, 8/3, 2/1 y 9/8, lo cual coincide con lo que anticipaba el análisis exploratorio.

📄 Informe completo: [`tp2/informe_tp2_sinnombre.pdf`](tp2/informe_tp2_sinnombre.pdf)

---

## Estructura del repo

```
LDD_TPs/
├── tp1/
│   ├── tp1_codigo_inlinesql.py         # modelado, reportes SQL y visualizaciones
│   ├── Tablas Limpias/                 # datasets ya limpios
│   ├── Anexo Reportes/                 # resultados de las consultas SQL (CSV)
│   └── Informe TP1 - Laboratorio de Datos.pdf
└── tp2/
    ├── tmnist_sinnombre.py             # EDA, KNN, árboles y validación cruzada
    └── informe_tp2_sinnombre.pdf
```

## Cómo reproducirlo

```bash
pip install pandas numpy matplotlib seaborn scikit-learn inline-sql
```

- **TP1:** antes de correr el script, ajustá la variable `carpeta` en `tp1_codigo_inlinesql.py` para que apunte a `tp1/Tablas Limpias/`.
- **TP2:** descargá el CSV de TMNIST desde Kaggle y ajustá `file_path` en `tmnist_sinnombre.py`. Algunos experimentos se activan comentando o descomentando bloques del script.
