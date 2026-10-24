# Detector Multiclase de Noticias Falsas, Reales y Sátira en Español

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/brankarios/dl26-proyecto/blob/main/notebook/plantilla_proyecto.ipynb)
[![Hugging Face Model](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model%20Hub-yellow)](https://huggingface.co/brankarios/beto-news-false-satire-detector)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Proyecto final de la asignatura **Redes Neuronales Profundas (dl26)** en la **Universidad Central de Venezuela (UCV)**.

- **Integrantes:**
  - César Carios — C.I.: `30.136.117`
  - Erimar Reis — C.I.: `29.743.464`
  - Carlos Cao — C.I.: `28.655.925`

---

## Enlaces Principales

- **Modelo publicado en Hugging Face Hub:** [brankarios/beto-news-false-satire-detector](https://huggingface.co/brankarios/beto-news-false-satire-detector)
- **Modelo base preentrenado:** `dccuchile/bert-base-spanish-wwm-cased` (BETO)
- **Dataset de origen:** `gabrielhuav/Unified-and-Balanced-Spanish-Fake-News-Corpus`
- **Notebook reproducible:** [`notebook/plantilla_proyecto.ipynb`](notebook/plantilla_proyecto.ipynb)

---

## Resumen del Proyecto y Resultados

Este proyecto aborda la clasificación automática de notas de prensa en español en tres categorías mutuamente excluyentes: **Falsa (`0`)**, **Real (`1`)** y **Sátira (`2`)**. A diferencia de los detectores binarios tradicionales que suelen etiquetar la parodia humorística como desinformación, adaptamos **BETO** sobre un subconjunto curado, deduplicado y balanceado de **9.000 noticias** (3.000 por clase).

Durante el análisis exploratorio se detectó y mitigó un sesgo de longitud (*shortcut learning*), donde las noticias de sátira tenían una mediana de 369 palabras frente a 37–38 en las clases real y falsa. Se estandarizó la entrada eliminando cabeceras editoriales (`.-`) y URLs, y truncando cada documento a sus primeras **35 palabras** (`max_length = 48` tokens).

### Comparativa de Regímenes de Adaptación (Semilla `30136117`)

| Régimen | Parámetros Entrenables | Tiempo (GPU T4) | F1-Macro (Val) | F1-Macro (Test) |
| :--- | :---: | :---: | :---: | :---: |
| **1. Baseline desde cero** | 3.976.707 | 12 s | 0,7828 | 0,7764 |
| **2. Modelo base sin adaptar** | 0 | 14 s | 0,2246 | 0,2130 |
| **3. Feature extraction** | 2.307 | 121 s | 0,6952 | 0,6819 |
| **4. Fine-tuning completo (Publicado)** | **109.853.187** | **284 s** | **0,8502** | **0,8633** |
| **5. Adicional: Top-2 Capas + Pooler** | 14.768.643 | 128 s | 0,8420 | 0,8428 |

---

## Arquitectura del Repositorio

```text
dl26-proyecto/
├── notebook/plantilla_proyecto.ipynb   # Notebook reproducible con EDA, 5 regímenes y análisis
├── api/                                # Servicio FastAPI: descarga el modelo desde el Hub y sirve predicciones
│   ├── main.py                         # Endpoints GET /health y POST /predict/text
│   ├── inference.py                    # Carga única del pipeline en memoria e inferencia
│   ├── schemas.py                      # Validación Pydantic y estructura uniforme de errores
│   └── tests/test_api.py               # Pruebas automatizadas con modelo simulado (pytest)
├── cliente/                            # Aplicación web interactiva en Reflex que consume el API
│   ├── rxconfig.py
│   └── cliente/cliente.py              # Estado, llamadas HTTP al API, textos y extensión
├── requirements.txt                    # Dependencias con versiones exactas probadas
├── .env.example                        # Variables de entorno (MODEL_ID, TASK, API_URL)
└── Dockerfile                          # Contenedor para despliegue del API

```

---

## Adaptación y Extensión Implementada (`api/` y `cliente/`)

1. **Conexión con Hugging Face Hub:** El servicio FastAPI descarga `brankarios/beto-news-false-satire-detector` con `TASK=text` una sola vez al arrancar mediante `pipeline`. Ningún archivo de pesos viaja dentro del repositorio.
2. **Adaptación de la interfaz (`cliente/cliente/cliente.py`):** Se personalizaron `TITULO` y `DESCRIPCION` explicando en lenguaje llano qué clasifica el modelo (Falsa, Real, Sátira), qué entradas acepta (idealmente titular y entradilla de la noticia) y sus limitaciones (no verifica hechos en tiempo real ni reemplaza la verificación humana).
3. **Extensión propia:** Se incorporaron en la aplicación web **botones de ejemplos de entrada precargados** para cada una de las tres categorías (*Falsa*, *Real* y *Sátira*) que permiten probar y contrastar las probabilidades predichas por el modelo con un solo clic, junto con el preprocesamiento de texto alineado con el entrenamiento.

---

## Cómo Ejecutar el Proyecto

Requiere **Python 3.11 o 3.12**.

### 1. Instalar dependencias y configurar `.env`

```bash
pip install -r requirements.txt
cp .env.example .env

```

Asegúrate de que el archivo `.env` contenga los siguientes valores:

```env
MODEL_ID=brankarios/beto-news-false-satire-detector
TASK=text
API_URL=http://localhost:8000

```

### 2. Ejecutar las pruebas del API

```bash
pytest api/tests -q

```

### 3. Iniciar el API (FastAPI)

Desde la raíz del repositorio:

```bash
uvicorn api.main:app --reload

```

* Estado del servicio: `http://localhost:8000/health`
* Documentación interactiva (Swagger): `http://localhost:8000/docs`

### 4. Iniciar la aplicación web (Reflex)

En una segunda terminal:

```bash
cd cliente
reflex run

```

* La interfaz estará disponible en `http://localhost:3000`.

---

## Declaración de Uso de Inteligencia Artificial Generativa

En cumplimiento con las normas de la asignatura ([Reglas y restricciones](https://github.com/ucvia/dl26-proyecto/wiki/Reglas-y-restricciones)), se declara el uso de herramientas de IA generativa por componente:

| Componente | Herramienta Utilizada | Propósito y Condiciones |
| --- | --- | --- |
| **Funciones auxiliares del Notebook (`notebook/plantilla_proyecto.ipynb`)** | Gemini | Apoyo en el código de visualización (`matplotlib`/`seaborn` para gráficos de barras, curvas de aprendizaje y matriz de confusión) y expresiones regulares de limpieza. El código fue revisado, adaptado y comentado por el equipo. |
| **Documentación (`README.md` y Model Card)** | Gemini | Estructuración y formato de tablas Markdown|
| **Análisis de datos y discusión de resultados** | **Ninguna (Redacción 100% propia)** | El análisis exploratorio, la identificación del sesgo de longitud (*shortcut learning*), el diseño del régimen *Top-2 + Pooler* y toda la discusión de resultados son autoría exclusiva de los integrantes del equipo. |
