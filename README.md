# Talent Retention Analytics — People Analytics (RRHH)

Caso de estudio de Machine Learning supervisado sobre el dataset **IBM HR Analytics Employee Attrition & Performance**, resuelto desde el rol simulado de **People Analytics Scientist**.

## Objetivo

Se resuelven dos tareas sobre el mismo dataset:

| Tarea | Target | Tipo | Pregunta de negocio |
|---|---|---|---|
| Regresión | `MonthlyIncome` | Numérico | ¿Qué factores explican el salario mensual y hay inconsistencias/inequidades? Apoya auditorías de equidad salarial. |
| Clasificación | `Attrition` | Binario (Yes/No) | ¿Qué empleados tienen mayor riesgo de renunciar? Alerta temprana de fuga de talento. |

## Dataset

- **Página Kaggle:** https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset
- **Descarga (Kaggle API):** https://www.kaggle.com/api/v1/datasets/download/pavansubhasht/ibm-hr-analytics-attrition-dataset

El notebook descarga el dataset automáticamente desde la Kaggle API al ejecutarse (con un enlace de respaldo verificado como idéntico byte a byte, por si la API no está disponible en el entorno de ejecución). No es necesario descargar nada a mano.

## Instrucciones para ejecutar

```bash
# 1) Crear y activar un entorno virtual (opcional pero recomendado)
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 2) Instalar dependencias
pip install -r requirements.txt

# 3) Abrir y ejecutar el notebook de punta a punta
jupyter notebook notebooks/modelo_final.ipynb
```

El notebook está diseñado para **correr de inicio a fin sin errores** y sin depender de archivos locales (usa rutas relativas y descarga el dataset por enlace).

## Estructura del repositorio

```
proyecto_machine_learning/
├── README.md              <- Este archivo: objetivo, dataset, instrucciones, resultados
├── requirements.txt        <- Entorno reproducible (pip install -r requirements.txt)
├── notebooks/
│   └── modelo_final.ipynb  <- Código documentado de extremo a extremo (EDA, leakage,
│                               pipelines, modelado, evaluación, interpretación)
├── report/                 <- Informe ejecutivo (PDF, 3-6 páginas) — pendiente
├── slides/                 <- Presentación (PDF, 8-12 diapositivas) — pendiente
└── src/                    <- (Opcional) Funciones de utilidad .py
```

## Resumen de resultados

### Regresión — `MonthlyIncome`

| Modelo | MAE | RMSE | R² |
|---|---:|---:|---:|
| Baseline (mediana) | 3392.70 | 5189.58 | -0.232 |
| Regresión Lineal | 895.37 | 1169.21 | 0.937 |
| Ridge (α óptimo por CV) | 896.39 | 1171.33 | 0.937 |
| Lasso (α óptimo por CV) | 896.57 | 1186.41 | 0.936 |

Lasso lleva exactamente a **0** el coeficiente de 21 de 48 variables (selección automática), evidenciando el efecto de la colinealidad detectada en el EDA entre variables de antigüedad y nivel jerárquico.

### Clasificación — `Attrition`

| Modelo | ROC-AUC | PR-AUC (Average Precision) |
|---|---:|---:|
| Baseline ("siempre No") | — | 0.160 |
| Regresión Logística (baseline formal, balanceada) | 0.802 | 0.555 |
| Regresión Logística ajustada por CV (`C` óptimo) | 0.812 | 0.589 |

- El dataset está desbalanceado (~16% de fuga), por lo que **no se usa Accuracy** como criterio de éxito.
- Se define un **umbral operativo de 0.55** (en vez del 0.5 por defecto), calibrado para minimizar un costo esperado de negocio en el que un Falso Negativo (perder talento sin anticipación) pesa 5 veces más que un Falso Positivo (una conversación de retención innecesaria).

### Principales hallazgos accionables

- El salario está dominado por `JobLevel` y `JobRole` (estructura por bandas).
- `OverTime = Yes` es uno de los factores que más incrementa el riesgo de fuga; también influyen la satisfacción laboral y el balance vida-trabajo.
- Variables sensibles (`Gender`, `MaritalStatus`, `Age`) fueron auditadas explícitamente en ambos modelos: su peso es bajo comparado con las variables estructurales, pero se recomienda monitoreo periódico como salvaguarda ética.

El detalle completo de metodología, decisiones de diseño y auditoría de cumplimiento está documentado en `notebooks/modelo_final.ipynb` y en la documentación técnica del proyecto.
