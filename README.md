# Modelo de Score de Quiebra Empresarial en el Ecuador

**Maestría en Business Intelligence y Ciencia de Datos**  
**Universidad de las Américas (UDLA)**  
**Autor:** Wladimir Heredia  
**Director de tesis:** Manuel Eugenio Morocho Cayamcela  

---

## Descripción

Este repositorio contiene el código, la documentación y los recursos asociados a la tesis de maestría titulada **"Modelo de Score de Quiebra Empresarial en el Ecuador"**. El trabajo desarrolla un sistema de puntuación (*score*) para estimar la probabilidad de quiebra de empresas ecuatorianas, combinando técnicas de aprendizaje automático con datos financieros y societarios provenientes de la Superintendencia de Compañías, Valores y Seguros (SCVS) del Ecuador.

El modelo principal es **XGBoost**, seleccionado tras una comparación sistemática con Regresión Logística, Random Forest, LightGBM, redes neuronales artificiales (ANN y DNN) y el modelo clásico Altman Z″. La puntuación resultante sigue la fórmula:

```
Score = (1 − P(quiebra)) × 1000
```

donde valores más altos indican mayor solidez financiera.

---

## Estructura del Repositorio

```
├── notebooks/
│   └── capstone-quiebra-final_v3.ipynb   # Notebook principal (pipeline completo)
├── docs/
│   ├── entregable_2_objeto_de_estudio.pdf
│   ├── entregable_3_planteamiento_problema.pdf
│   ├── entregable_4_analisis_exploratorio.pdf
│   └── activity_5_modelado_predictivo.pdf
├── models/
│   └── preprocessing_params.pkl           # Parámetros de preprocesamiento (para despliegue)
├── app/
│   └── app_streamlit.py                   # Aplicación de puntuación (Streamlit)
└── README.md
```

> **Nota:** El notebook principal y la base de datos están alojados en Kaggle. Ver los enlaces en la sección [Recursos](#recursos).

---

## Conjunto de Datos

El dataset utilizado proviene de registros financieros y societarios de empresas ecuatorianas, publicados por la SCVS y disponibles en Kaggle.

| Característica         | Detalle                                      |
|------------------------|----------------------------------------------|
| Fuente                 | Superintendencia de Compañías, Valores y Seguros (SCVS) |
| Período                | 2019 – 2023                                  |
| Observaciones          | ~115 000 (empresa × año)                     |
| Empresas únicas        | ~35 000                                      |
| Variable objetivo      | `quiebra_legal` (binaria: 1 = quiebra, 0 = activa) |
| Desbalance de clases   | Aproximadamente 13:1 (activa vs. quiebra)    |
| Tipo de dato           | Panel longitudinal                           |

**Dataset en Kaggle:** [herediawla/datos-empresas-ec](https://www.kaggle.com/datasets/herediawla/datos-empresas-ec)

---

## Pipeline Metodológico

```
Datos SCVS
    │
    ▼
Preprocesamiento
(imputación, codificación, escalado)
    │
    ▼
División temporal
(entrenamiento: 2019–2022 │ prueba: 2023)
    │
    ▼
Balanceo de clases
(SMOTE ratio 0.3 — solo en entrenamiento)
    │
    ▼
Entrenamiento y validación cruzada
(StratifiedGroupKFold agrupado por RUC)
    │
    ▼
Selección del mejor modelo
(XGBoost)
    │
    ▼
Interpretabilidad
(SHAP + Importancia por permutación)
    │
    ▼
Score = (1 − P(quiebra)) × 1000
```

---

## Resultados

### Comparación de Modelos (conjunto de prueba 2023)

| Modelo              | AUC-ROC | KS     | Sensibilidad (umbral 0.29) |
|---------------------|---------|--------|---------------------------|
| **XGBoost**         | **0.9248** | **0.7474** | **94.09 %**           |
| LightGBM            | 0.9181  | 0.7340 | 92.80 %                   |
| Random Forest       | 0.9102  | 0.7198 | 91.54 %                   |
| Red Neuronal (DNN)  | 0.8973  | 0.6985 | 89.21 %                   |
| Regresión Logística | 0.8641  | 0.6512 | 85.30 %                   |
| Altman Z″ (benchmark) | 0.6495 | 0.4103 | 68.75 %                 |

**Validación cruzada XGBoost:** AUC-ROC = 0.9057 ± 0.0053

### Variables Predictoras Principales (SHAP)

1. `end_activo` — Endeudamiento del activo
2. `liquidez_corriente` — Liquidez corriente
3. `end_patrimonial` — Endeudamiento patrimonial
4. `cobertura_interes` — Cobertura de intereses
5. `segmento_microempresa` — Segmento microempresa

---

## Reproducibilidad

### Requisitos

```
Python >= 3.9
scikit-learn
xgboost
lightgbm
imbalanced-learn
shap
pandas
numpy
matplotlib
seaborn
tensorflow / keras
joblib
```

Instalación rápida:

```bash
pip install -r requirements.txt
```

### Ejecución

1. Descargar el dataset desde Kaggle y ubicarlo en la carpeta `data/`.
2. Abrir el notebook `notebooks/capstone-quiebra-final_v3.ipynb`.
3. Ejecutar las celdas en orden secuencial.

> El notebook está disponible directamente en Kaggle para ejecución en la nube sin necesidad de configuración local.

---

## Decisiones de Diseño

- **Sin fuga de datos:** se excluyeron variables como `situacion_legal` y `anio_quiebra` por ser contemporáneas a la etiqueta objetivo.
- **División temporal estricta:** el conjunto de prueba corresponde al año 2023; no se realizó división aleatoria.
- **Agrupación por RUC:** la validación cruzada utiliza `StratifiedGroupKFold` para evitar que observaciones de una misma empresa aparezcan en entrenamiento y validación simultáneamente.
- **SMOTE únicamente en entrenamiento:** el balanceo de clases se aplicó solo sobre los datos de entrenamiento para no contaminar la evaluación.
- **Variable objetivo:** `quiebra_legal` refleja el estado legal en el período corriente. La indisponibilidad de datos t+1 impide usar estado futuro; esta limitación se documenta explícitamente en la tesis.

---

## Recursos

| Recurso | Enlace |
|---------|--------|
| Notebook en Kaggle | [capstone-quiebra-final-v3](https://www.kaggle.com/code/herediawla/capstone-quiebra-final-v3) |
| Dataset en Kaggle  | [herediawla/datos-empresas-ec](https://www.kaggle.com/datasets/herediawla/datos-empresas-ec) |

---

## Cita

Si este trabajo es de utilidad para su investigación, puede citarlo de la siguiente forma:

```
Heredia, W. (2026). Modelo de Score de Quiebra Empresarial en el Ecuador
[Tesis de maestría]. Universidad de las Américas.
```

---

## Licencia

Este repositorio se publica con fines académicos. El uso de los datos originales está sujeto a las condiciones de la Superintendencia de Compañías, Valores y Seguros del Ecuador.
