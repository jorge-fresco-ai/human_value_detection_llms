# 🧠 Human Value Detection with LLMs (ValueEval)

Este proyecto tiene como objetivo estudiar y modelizar la detección de valores humanos en textos mediante Large Language Models (LLMs), en el contexto de **ValueEval (SemEval 2023 y CLEF 2024 – Touché Lab)**.

El trabajo se fundamenta en la **Teoría de los Valores Humanos de Schwartz**, considerando no solo la clasificación de valores, sino su estructura relacional, contextual y semántica.

---

## 📌 Objetivo General

Analizar y rediseñar el problema de detección de valores humanos desde una perspectiva estructural, contextual y probabilística, evaluando distintos enfoques de modelado (multilabel, multiclass, RAG y modelado continuo de valores).

---

## 🎯 Nuevos Objetivos Específicos

### 1️⃣ Reformulación del problema: Multilabel vs. Multiclass

* Evaluar si tratar el problema como **multiclass** (forzando una única etiqueta por frase) resulta más eficiente que el enfoque **multilabel**, asumiendo una posible pérdida del ~2% de casos con múltiples valores.
* Analizar el impacto en:

  * Macro F1
  * Estabilidad del modelo
  * Interpretabilidad
  * Complejidad computacional
* Determinar si la simplificación estructural compensa la pérdida de información.

---

### 2️⃣ Incorporación de RAG (Retrieval-Augmented Generation)

* Construir una **Knowledge Base estructurada** con:

  * Text chunks etiquetados
  * Definiciones formales de valores
  * Ejemplos anotados
  * Literatura académica
  * Recursos psicológicos y sociológicos

* Evaluar:

  * Qué tipo de fuente mejora el rendimiento.
  * Qué fuentes introducen ruido.
  * Impacto del tamaño del chunk.
  * Impacto del tipo de embedding.

* Comparar:

  * Zero-Shot tradicional
  * Few-Shot
  * RAG con distintas configuraciones

---

### 3️⃣ Modelado de valores como continuo circular

En lugar de tratar los valores como etiquetas independientes, se propone:

* Modelarlos como posiciones en el **círculo motivacional de Schwartz**.
* Introducir nociones de:

  * Distancia angular entre valores.
  * Probabilidad de coocurrencia basada en proximidad estructural.
  * Penalización diferenciada según distancia conceptual.

Explorar:

* Si los errores entre valores cercanos deben penalizarse menos.
* Si puede modelarse el problema como predicción en un espacio continuo (embedding semántico alineado con el círculo).

---

### 4️⃣ Modelado contextual a nivel documento

Reformular el problema desde dos perspectivas:

**A. Enfoque jerárquico**

* Calcular primero los valores predominantes del documento.
* Utilizar esa distribución como prior para clasificar cada frase.

**B. Enfoque alternativo**

* Tratar el problema directamente como detección de valores a nivel documento.
* Comparar rendimiento frente al enfoque frase a frase.

Evaluar:

* Ganancia en coherencia semántica.
* Reducción de inconsistencias internas.
* Impacto en métricas multilabel.

---

## 📦 Dataset

* **Nombre**: ValueEval
* **Origen**: SemEval 2023 y CLEF 2024 (Touché Lab)
* **Idiomas**: EN, ES, FR, DE, IT, TR, HE, EL, NL, BG
* **Tamaño**: ~74.000 frases
* **Anotación**:

  * 19 valores humanos (teoría de Schwartz)
  * Polaridad: attained / constrained
  * Posibilidad de múltiples valores por frase

---

## 🧪 Líneas Experimentales

### 🔬 Línea 1: Multilabel vs Multiclass

* Baselines clásicos.
* LLM prompting estructurado.
* Evaluación comparativa estadística.

### 🔬 Línea 2: RAG

* Comparativa por tipo de fuente.
* Análisis de ruido vs señal.
* Evaluación de latencia vs mejora métrica.

### 🔬 Línea 3: Modelado continuo

* Representación vectorial de valores.
* Penalización basada en distancia angular.
* Experimentos con métricas suavizadas.

### 🔬 Línea 4: Modelado jerárquico documento-frase

* Pipeline en dos etapas.
* Evaluación frente a enfoque plano.

---

## 📊 Métricas de Evaluación

* Macro F1 (principal)
* Micro F1
* Hamming Loss
* Exact Match Ratio
* Matriz de Confusión estructural
* Métrica suavizada basada en distancia en el círculo de Schwartz (propuesta experimental)

---

## ❓ Preguntas de Investigación

1. ¿Es justificable simplificar el problema a multiclass?
2. ¿Qué tipo de conocimiento externo mejora realmente un LLM?
3. ¿Puede modelarse la estructura circular de valores para reducir errores conceptuales?
4. ¿El contexto documental mejora la coherencia en la detección?
5. ¿Se comportan los LLMs mejor cuando se respeta la estructura psicológica subyacente?

---

## ⚙️ Herramientas

* LLMs: OpenAI, Claude, Gemini, LLaMA, Mistral, Phi
* Frameworks: LangChain, LangGraph
* Vector DB: FAISS / Chroma
* Evaluación: scikit-learn, pandas
* Infraestructura: Docker, Ollama

---

## 📌 Contribución Esperada

Este trabajo no solo evalúa modelos, sino que propone:

* Una reformulación estructural del problema.
* Un análisis de simplificación multiclass.
* Un enfoque continuo basado en teoría psicológica.
* Una evaluación crítica del uso real de RAG en tareas de valores humanos.

---

## 🧠 Créditos

Trabajo desarrollado como parte de un Trabajo Fin de Grado sobre modelado avanzado de valores humanos mediante LLMs, integrando fundamentos de Psicología Social, NLP y Arquitecturas de Recuperación de Información.
