# 🧠 Human Value Detection with LLMs (ValueEval)

Este proyecto tiene como objetivo detectar valores humanos en textos mediante el uso de modelos de lenguaje grandes (LLMs), en el contexto de las tareas **ValueEval @ SemEval 2023** y **CLEF 2024 (Touché Lab)**. Se basa en la teoría de Schwartz, que categoriza 19 valores humanos en 4 dimensiones principales.

## 📌 Objetivo General

Desarrollar y evaluar estrategias basadas en LLMs para identificar valores humanos y su polaridad (fomentado o restringido) en frases, empleando enfoques de Zero-Shot, Few-Shot y RAG (Retrieval-Augmented Generation).

## 🎯 Objetivos Específicos

* Clasificar frases según los valores humanos presentes (basado en la teoría de Schwartz).
* Determinar la polaridad: si el valor está siendo promovido o restringido.
* Comparar el rendimiento de diferentes LLMs en Zero-Shot, Few-Shot y RAG.
* Analizar errores, sesgos y efectividad práctica.

## 📁 Estructura del Repositorio

```
├── data/                    # Datasets anotados (train, val, test)
├── prompts/                 # Prompts diseñados para cada enfoque
├── scripts/                 # Scripts de inferencia y evaluación
├── models/                  # Configs o checkpoints de modelos usados
├── codigo/                  # Experimentos exploratorios y EDA
├── results/                 # Resultados (predicciones, métricas)
├── docs/                    # Documentación extendida
└── README.md                # Este archivo
```

## 🧠 Taxonomía de Valores Humanos

Basado en la teoría de Schwartz, el proyecto trabaja con 19 valores humanos agrupados en 4 dimensiones:

| Dimensión          | Valores Ejemplo                     |
| ------------------ | ----------------------------------- |
| Openness to Change | Self-direction, Stimulation         |
| Self-enhancement   | Achievement, Power, Hedonism        |
| Conservation       | Security, Conformity, Tradition     |
| Self-transcendence | Universalism, Benevolence, Humility |

Más detalles disponibles en [`docs/Valores_Schwartz.md`](docs/Valores_Schwartz.md).

## 📦 Dataset

* **Nombre**: ValueEval
* **Origen**: SemEval 2023 y CLEF 2024 (Touché Lab)
* **Idiomas**: EN, ES, FR, DE, IT, TR, HE, EL, NL, BG
* **Anotadores**: 70+ expertos en valores humanos
* **Tamaño**: 2648 textos segmentados en \~74,000 frases
* **Formato**: Frase + Etiquetas binarias por valor + Polaridad (attained/constrained)

## 🧪 Métodos Usados

### 1. **Zero-Shot Learning**

* Sin ejemplos, usando descripciones explícitas del valor.
* Ejemplo de modelo: GPT-4 con instrucciones tipo CoT (Chain of Thought).

### 2. **Few-Shot Learning**

* Con 1, 3, 5 y 10 ejemplos. También versión con definiciones + ejemplos.
* Modelos evaluados: ChatGPT, Claude, Gemini, LLaMA.

### 3. **RAG (Retrieval-Augmented Generation)**

* Utiliza documentación sobre Schwartz y anotaciones para mejorar la inferencia.
* Implementado con LangChain y vectores FAISS.

### 4. **Fine-Tuning y Ensambles (Experimentos previos)**

* Modelos como XLM-RoBERTa, DeBERTa y RoBERTa.
* Upsampling y cascadas en modelos finos como en Touché 2024.

## ⚙️ Herramientas y Frameworks

* **LLMs**: OpenAI, Claude, LLaMA, Gemini, Ollama
* **RAG & Agentes**: LangChain, FAISS, CrewAI
* **Evaluación**: scikit-learn, pandas
* **Entorno local**: Ollama, Docker, FastAPI
* **Cloud**: AWS SageMaker, GCP AI Platform

## 📊 Métricas de Evaluación

* **Macro F1-Score** (principal)
* Precisión, Recall, Matriz de Confusión
* Análisis por clase (valor) y polaridad
* Tiempo de inferencia en RAG vs otros métodos

## ❓ Preguntas Clave

* ¿Qué LLM es más preciso y generalizable en esta tarea?
* ¿Cuál es el impacto del número de ejemplos en Few-Shot?
* ¿Cómo mitigar sesgos hacia valores frecuentes?
* ¿Qué diseño de prompt optimiza el rendimiento?

## 📌 Resultados Relevantes

* GPT-4 obtuvo mejor rendimiento en polaridad (subtarea 2).
* Modelos multilingües como XLM-RoBERTa destacaron en detección (subtarea 1).
* El uso de prompts contextuales mejoró métricas frente a los directos.

## 📚 Referencias

* ValueEval @ CLEF 2024: [Enlace](https://touche.webis.de)
* ValueEval @ SemEval 2023: [Enlace](https://semeval.github.io/)
* Schwartz Value Theory: [Wikipedia](https://en.wikipedia.org/wiki/Theory_of_Basic_Human_Values)

## 🧠 Créditos

Desarrollado como parte de un Trabajo Fin de Grado sobre la Detección de Valores Humanos mediante LLMs. Basado en los trabajos de Touché 2024 y SemEval 2023, junto con fundamentos teóricos en Psicología Social.

---

¿Quieres que también te lo prepare en formato `.md` descargable o que lo convierta en una web tipo GitHub Pages?
