# 🧠 Human Value Detection with LLMs (ValueEval)

Este proyecto tiene como objetivo detectar valores humanos en textos mediante el uso de modelos de lenguaje grandes (LLMs), en el contexto de las tareas **ValueEval @ SemEval 2023** y **CLEF 2024 (Touché Lab)**. Se basa en la teoría de Schwartz, que categoriza 19 valores humanos en 4 dimensiones principales.

## 📌 Objetivo General

Desarrollar y evaluar estrategias basadas en LLMs para identificar valores humanos en frases de textos, empleando enfoques de Zero-Shot, Few-Shot con varios ejemplos y documentación extra.

## 🎯 Objetivos Específicos

- Clasificar frases según los valores humanos presentes (basado en la teoría de Schwartz).
- Determinar la polaridad: si el valor está siendo promovido o restringido.
- Comparar el rendimiento de diferentes LLMs en Zero-Shot, Few-Shot y Documentación Extra.
- Analizar errores, sesgos y efectividad práctica.

## 📦 Requisitos previos

Antes de comenzar, asegurate de tener instalados:

- [`pyenv`](https://github.com/pyenv/pyenv)
- [`poetry`](https://python-poetry.org/docs/)

## 💻 Entorno recomendado: WSL + VS Code

Este proyecto está pensado para ejecutarse desde un entorno **WSL (Windows Subsystem for Linux)**.

🔧 **Recomendación**: Usar Visual Studio Code con la extensión oficial de WSL:

1. Instalar la extensión “**WSL**” en VS Code.
2. Presionar `Ctrl + Shift + P` y seleccionar: Open Folder in WSL

## ⚙️ Configuración del entorno

### 1. Clonar el repositorio y cambiar a `develop`

```bash
git clone https://github.com/tu-usuario/human_value_detection_llms.git
cd human_value_detection_llms
git checkout develop
```

### 2. Crear una rama basada en develop

```bash
git checkout -b feat/setup-entorno

# Sube tu rama al remoto
git push -u origin feat/setup-entorno
```

El desarrollo se hace en la rama que se crea en base a develop. Cuando está todo testeado, se hace PR a develop y si todo está bien, se hace a main. Cuantos menos PR a main mejor,

### 3. Verificar versión de Python
El archivo .python-version ya está incluido en el repo. pyenv lo leerá automáticamente y usará Python 3.11.9.

Si no tenés esa versión instalada:

```bash
pyenv install 3.11.9
```

### 4. Instalar dependencias

```bash
poetry install
```

### 5. Activar entorno de dependencias

```bash
poetry shell
```

### 6. Ejecutar scripts

Una vez dentro del entorno (poetry shell):

```bash
python scripts/nombre_script.py
```

O, sin activar el entorno explícitamente:

```bash
poetry run python scripts/nombre_script.py
```
Por ejemplo, para ejecutar el script actual de clasificación:

```bash
poetry run python hvdetector/analyze.py
```

### 7. Variables de entorno

Este proyecto puede usar un archivo `.env` para variables sensibles.

Ejemplo `.env`:

```bash
API_KEY=tu_clave_secreta
```

Y en tus scripts:

```python
from dotenv import load_dotenv
load_dotenv()
```

Asegurate de que `.env` esté en `.gitignore` (ya lo está por defecto).



## 📁 Estructura del Repositorio

```
├── DATA/                    # Datasets anotados (train, val, test)
│   ├── data_human_value/        # Datasets finales transformados y trabajados
│   ├── prompts/                 # Prompts diseñados para cada enfoque
├── CÓDIGO/                  # Código relacionado con el TFG
│   ├── models/                  # Configs o checkpoints de modelos usados
│   ├── scripts/                 # Scripts de inferencia y evaluación
│   ├── notebooks/               # Experimentos exploratorios y EDA
│   ├── results/                 # Resultados (predicciones, métricas)
├── DOCS/                    # Documentación extendida
└── README.md                # Este archivo
```

## 🧠 Taxonomía de Valores Humanos

Basado en la teoría de Schwartz, el proyecto trabaja con 19 valores humanos agrupados en 4 dimensiones:

| Dimensión            | Valores                                                                  |
|----------------------|--------------------------------------------------------------------------|
| Openness to Change   | Self-Direction (Thought), Self-Direction (Action), Stimulation, Hedonism |
| Self-enhancement     | Achievement, Power (Dominance),Power (Resources), Face |
| Conservation         | Security (Personal), Security (Societal), Conformity (Rules), Conformity (Interpersonal), Tradition, Humility |
| Self-transcendence   | Universalism (Concern), Universalism (Nature), Universalism (Tolerance), Benevolence (Dependability), Benevolence (Caring)  |

## 📦 Dataset

- **Nombre**: ValueEval
- **Origen**: SemEval 2023 y CLEF 2024 (Touché Lab)
- **Idiomas**: EN, ES, FR, DE, IT, TR, HE, EL, NL, BG
- **Anotadores**: 70+ expertos en valores humanos
- **Tamaño**: 2648 textos segmentados en ~74,000 frases
- **Formato**: Frase + Etiquetas binarias por valor + Polaridad (attained/constrained)

## 🧪 Métodos Usados

### 1. **Zero-Shot Learning**
- Sin ejemplos, usando descripciones explícitas del valor.
- Ejemplo de modelo: GPT-4 con instrucciones tipo CoT (Chain of Thought).

### 2. **Few-Shot Learning**
- Con 1, 3, 5 y 10 ejemplos. También versión con definiciones + ejemplos.
- Modelos evaluados: ChatGPT, Claude, Gemini, LLaMA.

## ⚙️ Herramientas y Frameworks

- **LLMs**: OpenAI, Claude, LLaMA, Gemini, Mistral, Phi, etc.
- **Frameworks LLMs**: Ollama
- **Agentes**: LangGraph, LangChain, CrewAI
- **Evaluación**: scikit-learn, pandas
- **Entorno local**: Ollama, Docker, FastAPI

## 📊 Métricas de Evaluación

- **Macro F1-Score** (principal)
- Precisión, Recall, Matriz de Confusión
- Análisis por clase (valor) y polaridad
- Tiempo de inferencia en RAG vs otros métodos

## ❓ Preguntas Clave

- ¿Qué LLM es más preciso y generalizable en esta tarea?
- ¿Cuál es el impacto del número de ejemplos en Few-Shot?
- ¿Cómo mitigar sesgos hacia valores frecuentes?
- ¿Qué diseño de prompt optimiza el rendimiento?

## 📌 Resultados Relevantes

- GPT-4 obtuvo mejor rendimiento en polaridad (subtarea 2).
- Modelos multilingües como XLM-RoBERTa destacaron en detección (subtarea 1).
- El uso de prompts contextuales mejoró métricas frente a los directos.

## 📚 Referencias

- ValueEval @ CLEF 2024: [Enlace](https://touche.webis.de)
- ValueEval @ SemEval 2023: [Enlace](https://semeval.github.io/)
- Schwartz Value Theory: [Wikipedia](https://en.wikipedia.org/wiki/Theory_of_Basic_Human_Values)

## 🧠 Créditos

Desarrollado como parte de un Trabajo Fin de Grado sobre la Detección de Valores Humanos mediante LLMs. Basado en los trabajos de Touché 2024 y SemEval 2023, junto con fundamentos teóricos en Psicología Social.

----
