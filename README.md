# 🧠 Human Value Detection with LLMs (ValueEval)

Este proyecto tiene como objetivo detectar valores humanos en textos mediante el uso de modelos de lenguaje grandes (LLMs), en el contexto de las tareas **ValueEval @ SemEval 2023** y **CLEF 2024 (Touché Lab)**. Se basa en la teoría de Schwartz, que categoriza 19 valores humanos en 4 dimensiones principales.

## 📌 Objetivo General

Desarrollar y evaluar estrategias basadas en LLMs para identificar valores humanos y su polaridad (fomentado o restringido) en frases, empleando enfoques de Zero-Shot, Few-Shot con varios ejemplos y documentación extra.

## 🎯 Objetivos Específicos

- Clasificar frases según los valores humanos presentes (basado en la teoría de Schwartz).
- Determinar la polaridad: si el valor está siendo promovido o restringido.
- Comparar el rendimiento de diferentes LLMs en Zero-Shot, Few-Shot y Documentación Extra.
- Analizar errores, sesgos y efectividad práctica.

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

| Dimensión             | Valores Ejemplo                          |
|----------------------|-------------------------------------------|
| Openness to Change   | Self-direction, Stimulation               |
| Self-enhancement     | Achievement, Power, Hedonism              |
| Conservation         | Security, Conformity, Tradition           |
| Self-transcendence   | Universalism, Benevolence, Humility       |

**Openness to Change**
- Self-Direction (Thought)
- Self-Direction (Action)
- Stimulation
- Hedonism

**Self-Enhancement**
- Achievement
- Power (Dominance)
- Power (Resources)
- Face

**Conservation**
- Security (Personal)
- Security (Societal)
- Tradition
- Conformity (Rules)
- Conformity (Interpersonal)
- Humility

**Self-Transcendence**
- Benevolence (Dependability)
- Benevolence (Caring)
- Universalism (Concern)
- Universalism (Nature)
- Universalism (Tolerance)

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
