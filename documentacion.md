# Documentación técnica — Motor de análisis de valores humanos (LLMs)

> **Objetivo**
> Este módulo orquesta varios *agents* LLM para detectar la presencia de dimensiones de valores humanos en un texto, así como sus subdimensiones y polaridad. La documentación está pensada para *knowledge transfer* y mantenimiento: explica arquitectura, contratos entre clases, formatos de entrada/salida, errores esperables y cómo extender el sistema.

---

## Índice

1. [Visión general](#visión-general)
2. [Arquitectura y flujo](#arquitectura-y-flujo)
3. [Configuración (`prompts.yaml`)](#configuración-promptyaml)
4. [Clases y funciones](#clases-y-funciones)

   * [PromptManager](#promptmanager)
   * [AgentManager](#agentmanager)
   * [TextAnalyzer](#textanalyzer)
5. [Contratos con dependencias externas](#contratos-con-dependencias-externas)
6. [Formato de salida](#formato-de-salida)
7. [Ejecución local y requisitos](#ejecución-local-y-requisitos)
8. [Errores, logging y diagnósticos](#errores-logging-y-diagnósticos)
9. [Pruebas recomendadas](#pruebas-recomendadas)
10. [Extensión y personalización](#extensión-y-personalización)
11. [Limitaciones conocidas y mejoras planificadas](#limitaciones-conocidas-y-mejoras-planificadas)

---

## Visión general

El módulo consta de tres piezas principales:

* **PromptManager**: carga y sirve *prompts* (plantillas) desde un YAML.
* **AgentManager**: inicializa y expone instancias de `Agent` (uno por tarea).
* **TextAnalyzer**: orquesta el análisis por dimensión utilizando los *agents* y consolida el resultado.

Dimensiones soportadas: `openness`, `enhancement`, `conservation`, `transcendence`.
Para cada dimensión se consulta:

1. Presencia de la dimensión (bool).
2. Subdimensiones (JSON booleans por claves `"1"` a `"7"`).
3. Polaridad (bool).

---

## Arquitectura y flujo

### Diagrama (alto nivel)

```
           +----------------+
           |  prompts.yaml  |
           +--------+-------+
                    |
             load / get
                    v
+-------------------+-------------------+
|             PromptManager             |
+-------------------+-------------------+
                    |
           system_prompt / user_prompt
                    v
+-------------------+-------------------+
|              AgentManager             |
|  - crea Agent por nombre              |
+-------------------+-------------------+
   |           |           |         |
   v           v           v         v
agente_*   agente_*   ... agente_polaridad (Agent)

                (orquestación)
                       ^
                       |
+-------------------------------------------+
|               TextAnalyzer                |
| - analyze_text_flow()                      |
| - process_dimension(dimension)             |
| - run_analysis(text) -> JSON               |
+-------------------------------------------+
```

### Secuencia por dimensión (resumen)

1. `TextAnalyzer.process_dimension()` obtiene `main_agent`, `sub_agent` y `polaridad_agent`.
2. Construye `main_prompt` y ejecuta `main_agent(main_prompt)`.
3. Infiera presencia con `'true' in <respuesta>.lower()`.
4. Si presente:

   * Ejecuta `sub_agent(sub_prompt)` y parsea JSON.
   * Ejecuta `polaridad_agent(pol_prompt)` y evalúa `'true' in …`.
5. Agrega resultados a un diccionario por dimensión.

---

## Configuración (`prompts.yaml`)

El archivo **YAML** debe incluir una clave raíz `agents` con un objeto por cada agente. Cada agente **debe** declarar:

* `system_prompt`: instrucciones del sistema para el LLM.
* `user_prompt`: plantilla de entrada del usuario; se interpolan parámetros con `str.format(**kwargs)`.

### Esquema esperado (mínimo)

```yaml
agents:
  agente_openness:
    system_prompt: |
      Eres un clasificador. Devuelve "true" o "false" indicando si hay apertura a la experiencia.
    user_prompt: |
      Texto: "{input_text}"
      Responde solo con true o false.
  agente_subdimensiones_openness:
    system_prompt: |
      Devuelve JSON con subdimensiones 1..7 como booleanos.
    user_prompt: |
      Texto: "{input_text}"
      Formato estricto:
      {"1": true, "2": false, "3": false, "4": true, "5": false, "6": false, "7": true}
  agente_enhancement:
    system_prompt: "..."; user_prompt: "..."
  agente_subdimensiones_enhancement:
    system_prompt: "..."; user_prompt: "..."
  agente_conservation:
    system_prompt: "..."; user_prompt: "..."
  agente_subdimensiones_conservation:
    system_prompt: "..."; user_prompt: "..."
  agente_transcendence:
    system_prompt: "..."; user_prompt: "..."
  agente_subdimensiones_transcendence:
    system_prompt: "..."; user_prompt: "..."
  agente_polaridad:
    system_prompt: |
      Determina si la polaridad para la dimensión {dimension_name} es positiva (true) o negativa (false).
    user_prompt: |
      Texto: "{input_text}"
      Responde solo con true o false.
```

> **Placeholders disponibles**
>
> * `{input_text}`: texto a analizar.
> * `{dimension_name}`: nombre de la dimensión (solo para `agente_polaridad`).

### Ruta por defecto del YAML

* `"/mnt/c/Users/BorjaEsteveMolner/Documents/human_value_detection_llms/prompts.yaml"`
  Puede y **debe** sobreescribirse en entornos no-Windows o CI usando el parámetro `config_path` de `TextAnalyzer` o `PromptManager`.

---

## Clases y funciones

### PromptManager

**Propósito**: cargar los *prompts* desde YAML y suministrarlos formateados.

**Constructor**

```python
PromptManager(config_path: str = DEFAULT_PATH)
```

* **Efectos**: lee el YAML al inicializar y guarda en `self.prompts`.
* **Errores**:

  * `FileNotFoundError` si el archivo no existe.
  * `ValueError` si el YAML es inválido (`yaml.YAMLError` envuelto).
  * `KeyError` indirectamente si falta la clave `agents`.

**Métodos**

* `_load_prompts() -> Dict[str, Any]`
  Lee el YAML y retorna `config['agents']`. Maneja:

  * `FileNotFoundError`
  * `yaml.YAMLError` (reenvía como `ValueError`)

* `get_agent_prompts(agent_name: str) -> Dict[str, str]`
  Devuelve el bloque de *prompts* para `agent_name`.

  * **Errores**: `KeyError` si el agente no existe.

* `format_user_prompt(agent_name: str, **kwargs) -> str`
  Obtiene `user_prompt` y aplica `str.format(**kwargs)`.

  * **Errores**: `KeyError` si no existe el agente; `KeyError` de placeholders si faltan kwargs.

* `get_system_prompt(agent_name: str) -> str`
  Retorna el `system_prompt` del agente.

  * **Errores**: `KeyError` si no existe el agente.

---

### AgentManager

**Propósito**: inicializar y servir instancias `Agent` configuradas con su `system_prompt`.

**Constructor**

```python
AgentManager(model, prompt_manager: PromptManager)
```

* **Parámetros**

  * `model`: instancia de modelo (p.ej. `OllamaModel`) compatible con `Agent`.
  * `prompt_manager`: para recuperar `system_prompt`.
* **Efectos**: inicializa `self.agents` en `_initialize_agents()`.

**Inicialización**

* `_initialize_agents()` crea agentes para:

  ```
  agente_openness,
  agente_subdimensiones_openness,
  agente_enhancement,
  agente_subdimensiones_enhancement,
  agente_conservation,
  agente_subdimensiones_conservation,
  agente_transcendence,
  agente_subdimensiones_transcendence,
  agente_polaridad
  ```
* **Mensajes**: imprime ✓/⚠ en consola según existan *prompts*.

**Métodos**

* `get_agent(agent_name: str) -> Agent`
  Devuelve la instancia `Agent`.

  * **Errores**: `KeyError` si no está disponible (p.ej. faltó en el YAML).

> **Contrato de `Agent`** (ver sección [Contratos](#contratos-con-dependencias-externas)): el código asume que `Agent` es *callable* y retorna un objeto con `message['content'][0]['text']`.

---

### TextAnalyzer

**Propósito**: ofrece la API de alto nivel para procesar un texto a través de todas las dimensiones.

**Constructor**

```python
TextAnalyzer(model, config_path: str = DEFAULT_PATH)
```

* Crea internamente `PromptManager` y `AgentManager`.

**API pública**

* `run_analysis(text: str) -> str`
  Ejecuta el flujo completo y devuelve **JSON serializado** (pretty-printed, UTF-8).

  * Internamente llama a `analyze_text_flow()`.
  * **Errores**: captura cualquier excepción y devuelve `{"error": "<mensaje>"}`.

* `analyze_text_flow(input_text: str) -> Dict[str, Any]`
  Itera por `["openness", "enhancement", "conservation", "transcendence"]` y llama a `process_dimension()` para cada una.

  * En caso de error por dimensión, incluye:

    ```json
    {
      "dimension_name": false,
      "subdimensiones": {"1": false, ... "7": false},
      "polaridad": false,
      "error": "<mensaje>"
    }
    ```

* `process_dimension(input_text: str, dimension_name: str) -> Dict[str, Any]`
  Orquesta los tres agentes necesarios (principal, subdimensiones, polaridad).

  * **Mapeo de agentes**:

    * `"openness"` → `("agente_openness", "agente_subdimensiones_openness")`
    * `"enhancement"` → `("agente_enhancement", "agente_subdimensiones_enhancement")`
    * `"conservation"` → `("agente_conservation", "agente_subdimensiones_conservation")`
    * `"transcendence"` → `("agente_transcendence", "agente_subdimensiones_transcendence")`

  * **Detección de presencia**:

    ```python
    is_dimension_present = 'true' in main_result.message['content'][0]['text'].lower()
    ```

    > *Nota*: frágil ante falsos positivos (p.ej. “not true”, “true/false”).

  * **Subdimensiones**: espera JSON puro (sin texto extra). Si falla el `json.loads`, rellena con `{"1": false, ..., "7": false}` y loguea error.

  * **Polaridad**: misma heurística `'true' in ...`.

  * **Errores**:

    * `ValueError` si `dimension_name` no es reconocido.
    * `KeyError`/`IndexError` si el objeto devuelto por `Agent` no tiene la estructura esperada.
    * `json.JSONDecodeError` si el agente de subdimensiones no devuelve JSON limpio.

---

## Contratos con dependencias externas

El código depende de **`strands`**:

* `from strands import Agent`
  Se asume:

  * **Construcción**: `Agent(model=<model>, system_prompt=<str>)`.
  * **Llamado**: `Agent(prompt_str)` retorna un objeto (p.ej. `InferenceResult`) con la forma:

    ```python
    {
      "message": {
        "content": [
          {"text": "<respuesta>"}
        ]
      }
    }
    ```

    o equivalente accesible vía `obj.message['content'][0]['text']`.

* `from strands.models.ollama import OllamaModel`
  Se asume:

  * **Construcción**: `OllamaModel(host="<url>", model_id="<id>")`.
  * **Uso**: compatible con `Agent`.

> **Recomendación**: si estas interfaces cambian, adaptar el acceso al texto de respuesta y documentar la nueva estructura. Para tests, *mockear* `Agent` para retornar objetos con esa forma.

---

## Formato de salida

El **JSON** devuelto por `run_analysis()` tiene esta estructura:

```json
{
  "openness": {
    "openness": true,
    "subdimensiones": {
      "1": true, "2": false, "3": false, "4": true, "5": false, "6": false, "7": true
    },
    "polaridad": true
  },
  "enhancement": {
    "enhancement": false,
    "subdimensiones": {"1": false, "2": false, "3": false, "4": false, "5": false, "6": false, "7": false},
    "polaridad": false
  },
  "conservation": { "...": "..." },
  "transcendence": { "...": "..." }
}
```

> **Importante**: La clave booleana de presencia **repite** el nombre de la dimensión (p.ej. `"openness": true` dentro del objeto `openness`). Mantener este contrato si hay consumidores aguas abajo.

---

## Ejecución local y requisitos

### Requisitos

* Python 3.9+ (recomendado)
* Dependencias:

  * `PyYAML` (`yaml`)
  * Paquete interno `strands` (incluye `Agent` y `OllamaModel`)
  * Servidor **Ollama** accesible si se usa `OllamaModel`
* Acceso al archivo `prompts.yaml`

### Ejemplo de ejecución (bloque `__main__`)

```python
if __name__ == "__main__":
    ollama_model = OllamaModel(
        host="http://localhost:11434",
        model_id="llama3.2:3b"
    )

    input_text = (
      "I love exploring new ideas and cultures, and I turn every lesson into a "
      "challenge to push myself and achieve more ambitious goals."
    )

    print("Iniciando análisis...")
    analyzer = TextAnalyzer(ollama_model)
    resultado = analyzer.run_analysis(input_text)
    print("Resultado:")
    print(resultado)
```

### Uso programático (sugerido)

```python
model = OllamaModel(host="http://localhost:11434", model_id="llama3.2:3b")
analyzer = TextAnalyzer(model, config_path="/ruta/a/prompts.yaml")
json_str = analyzer.run_analysis("Texto a analizar")
data = json.loads(json_str)
```

---

## Errores, logging y diagnósticos

### Excepciones que puede lanzar

* **Carga de configuración**

  * `FileNotFoundError`: YAML no encontrado.
  * `ValueError`: error de parseo YAML.
  * `KeyError`: falta `agents` o `agent_name`.

* **Ejecución de agentes**

  * `KeyError`/`IndexError`: estructura inesperada en `message['content'][0]['text']`.
  * `json.JSONDecodeError`: subagente no devuelve JSON válido.

* **Parámetros inválidos**

  * `ValueError`: dimensión no reconocida.

### Trazas actuales

* Se usa `print()` extensivamente (inicio de dimensión, resultados crudos, etc.).
  **Mejora recomendada**: sustituir por `logging` con niveles `INFO/DEBUG/WARNING/ERROR` y posibilidad de activar verbosidad por CLI/env.

---

## Pruebas recomendadas

> **Estrategia**: *mockear* `Agent` para controlar las respuestas y cubrir casos felices y de error.

### Unit tests (ejemplos de casos)

1. **PromptManager**

   * Carga correcta del YAML válido.
   * `FileNotFoundError` cuando no existe.
   * `ValueError` con YAML corrupto.
   * `KeyError` al pedir agente inexistente.
   * `format_user_prompt` con y sin placeholders requeridos.

2. **AgentManager**

   * Inicialización con YAML con todos los agentes.
   * Advertencia y exclusión cuando falta un agente en YAML.
   * `get_agent` con nombre desconocido → `KeyError`.

3. **TextAnalyzer.process\_dimension**

   * Presencia detectada (`"true"`) y `"false"`.
   * Subdimensiones: JSON válido → mapeo; JSON inválido → fallback 1..7 = false.
   * Polaridad: `"true"`/`"false"` en diferentes mayúsculas y con ruido (asegurar `.lower()`).
   * Error de dimensión no reconocida.

4. **TextAnalyzer.analyze\_text\_flow**

   * Un agente lanza excepción → bloque `error` por dimensión relleno.
   * Flujo completo con mocks deterministas.

5. **run\_analysis**

   * Serializa correctamente y captura excepciones globales.

> **Sugerencia**: usar `pytest` y `unittest.mock` para `Agent`. Asegurar cobertura de ramas de excepción.

---

## Extensión y personalización

### Añadir una nueva dimensión

1. Definir **dos** agentes en `prompts.yaml`:

   * `agente_<dim>`
   * `agente_subdimensiones_<dim>`
2. Añadir el par al `agent_names` de `AgentManager._initialize_agents()`.
3. Añadir el mapeo en `TextAnalyzer.process_dimension()`:

   ```python
   agent_mapping["<dim>"] = ("agente_<dim>", "agente_subdimensiones_<dim>")
   ```
4. Ajustar prompts (asegurar JSON puro para subdimensiones).

### Cambiar el modelo por agente (futuro)

* El TODO enumera permitir modelos distintos por agente. Hoy todos comparten `self.model`.
  **Diseño propuesto**: aceptar un `models_by_agent: Dict[str, Model]` opcional en `AgentManager` o cargar `model_id` por agente desde el YAML.

### Robustecer el parsing

* Sustituir heurística `'true' in text.lower()` por:

  * Respuestas estrictas `true/false` en prompts (ya recomendado).
  * Post-procesamiento con regex `^\s*(true|false)\s*$`.
  * **Ideal**: `agent.structured_output(PydanticModel)` (ver TODO).

### Concurrencia

* `analyze_text_flow` procesa dimensiones **secuencialmente**.
  Si `Agent` es IO-bound, se puede paralelizar por dimensión con `asyncio`/`gather` o `concurrent.futures.ThreadPoolExecutor`.

### CLI (futuro)

* Añadir interfaz `typer`:

  ```
  hvd analyze --text "..." --config /ruta/prompts.yaml --model-id llama3.2:3b
  --format json --verbose
  ```

### Observabilidad

* Reemplazar `print` por `logging` (logger por módulo).
* Métricas: latencia por agente, tasa de parseo fallido, ratio de `true`/`false`.
* Trazabilidad: IDs de petición y *prompts* con *redaction* de datos sensibles.

---

## Limitaciones conocidas y mejoras planificadas

Refleja los `TODO` del código y comentarios adicionales:

* [ ] **True/false frágil**: la heurística `'true' in ...` produce falsos positivos/negativos.
* [ ] **Salida estructurada**: añadir `agent.structured_output()` (Pydantic) para evitar `json.loads` frágil.
* [ ] **Manejo de errores** más robusto y tipificado; *retries* con *backoff* para llamadas a LLM.
* [ ] **Refactor** para legibilidad y modularidad (separar adapters de `Agent` y *parsers*).
* [ ] **Modelo por agente** configurable externamente.
* [ ] **Sustituir prints por logging** con niveles.
* [ ] **CLI** para uso desde terminal.
* [ ] **Tests de integración** con Ollama y *golden files* de salida.
* [ ] **Internacionalización**: prompts multilenguaje y detección automática del idioma de entrada.
* [ ] **Validación de YAML** con esquema (p.ej. `cerberus` o `pydantic-yaml`).

---

## Notas de implementación útil (quick reference)

* **Acceso a texto de respuesta**:

  ```python
  text = result.message['content'][0]['text']
  ```

  Ajustar si la estructura del `Agent` cambia.

* **Fallback de subdimensiones**:

  ```python
  {str(i): False for i in range(1, 8)}
  ```

* **Dimensiones soportadas**:

  ```python
  ["openness", "enhancement", "conservation", "transcendence"]
  ```

* **Rutas**: ajustar `config_path` si no existe la ruta por defecto (WSL/Windows).

---

## Ejemplo de extremo a extremo

1. **Configurar** `prompts.yaml` conforme al esquema.
2. **Inicializar** el modelo Ollama y el analizador:

```python
model = OllamaModel(host="http://localhost:11434", model_id="llama3.2:3b")
analyzer = TextAnalyzer(model, config_path="/abs/path/prompts.yaml")
```

3. **Ejecutar**:

```python
print(analyzer.run_analysis("Texto de ejemplo con señales de apertura y superación."))
```

4. **Consumir** el JSON en tu pipeline (ETL, API, dashboard, etc.).

---

> **Contacto para mantenimiento**
>
> * Revisar `prompts.yaml` ante cambios de dominio o *guidelines*.
> * Alinear versiones de `strands` y del servidor Ollama.
> * Actualizar tests si se modifican estructuras de respuesta de `Agent`.
