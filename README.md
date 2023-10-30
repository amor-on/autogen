- [Agentes y roles](#agentes-y-roles)
  - [ConversableAgent](#conversableagent)
    - [Características](#características)
    - [Ejemplos de uso](#ejemplos-de-uso)
  - [UserProxyAgent](#userproxyagent)
    - [Características](#características-1)
    - [Ejemplos de uso](#ejemplos-de-uso-1)
      - [math\_user\_proxy\_agent](#math_user_proxy_agent)
        - [Características](#características-2)
        - [Ejemplo de uso](#ejemplo-de-uso)
      - [retrieve\_user\_proxy\_agent](#retrieve_user_proxy_agent)
        - [Características](#características-3)
        - [Ejemplo de uso](#ejemplo-de-uso-1)
      - [QdrantRetrieveUserProxyAgent](#qdrantretrieveuserproxyagent)
        - [Características](#características-4)
        - [Ejemplo de uso](#ejemplo-de-uso-2)
  - [AssistantAgent](#assistantagent)
    - [Características](#características-5)
    - [Ejemplos de uso](#ejemplos-de-uso-2)
      - [retrieve\_assistant\_agent](#retrieve_assistant_agent)
        - [Características](#características-6)
        - [Ejemplo de uso](#ejemplo-de-uso-3)
      - [TeachableAgent](#teachableagent)
        - [Características](#características-7)
        - [Ejemplo de uso](#ejemplo-de-uso-4)
      - [TextAnalyzerAgent](#textanalyzeragent)
        - [Características](#características-8)
        - [Ejemplo de uso](#ejemplo-de-uso-5)
  - [GroupChatManager](#groupchatmanager)
    - [Características](#características-9)
    - [Ejemplo de uso](#ejemplo-de-uso-6)
- [Funciones](#funciones)
  - [Registro de funciones](#registro-de-funciones)
    - [Pasos](#pasos)
    - [Ejemplo](#ejemplo)
  - [Definición de funciones](#definición-de-funciones)
    - [Pasos](#pasos-1)
    - [Ejemplo](#ejemplo-1)
  - [Configuración de funciones](#configuración-de-funciones)
    - [Pasos](#pasos-2)
    - [Ejemplo](#ejemplo-2)
  - [Uso de funciones](#uso-de-funciones)
    - [Pasos](#pasos-3)


# Agentes y roles

## ConversableAgent

El `ConversableAgent` es una clase para agentes conversacionales genéricos que pueden configurarse como asistentes o proxies de usuario. Después de recibir cada mensaje, el agente enviará una respuesta al emisor a menos que el mensaje sea un mensaje de terminación. Clases como `AssistantAgent` y `UserProxyAgent` son subclases de esta, configuradas con diferentes ajustes predeterminados.

### Características

- Puede auto-responder a los mensajes a menos que sean mensajes de terminación.
- Capacidad de modificar la respuesta automática sobrescribiendo el método `generate_reply`.
- Posibilidad de habilitar/deshabilitar la respuesta humana en cada turno configurando `human_input_mode` en "NEVER" o "ALWAYS".
- Puede personalizar la forma de obtener la entrada humana sobrescribiendo el método `get_human_input`.
- Puede modificar la manera de ejecutar bloques de código, un solo bloque de código o llamadas a funciones sobrescribiendo los métodos `execute_code_blocks`, `run_code` y `execute_function` respectivamente.
- Para personalizar el mensaje inicial cuando comienza una conversación, sobrescriba el método `generate_init_message`.

### Ejemplos de uso

```python
agent = ConversableAgent(
    name="miAgente",
    system_message="You are a helpful AI Assistant.",
    human_input_mode="ALWAYS",
    function_map={"say_hello": lambda: "Hello World!"}
)
```

## UserProxyAgent
El `UserProxyAgent` actúa como un agente proxy para el usuario, con la capacidad de ejecutar código y proporcionar retroalimentación a otros agentes. Es una subclase de `ConversableAgent`, configurada para solicitar entrada humana en cada mensaje recibido, y tiene la capacidad de auto-responder con ejecución de código, mientras que las respuestas automáticas basadas en LLM están deshabilitadas por defecto.

### Características

- Está configurado con `human_input_mode` en "ALWAYS", lo que significa que siempre solicita entrada humana cuando recibe un mensaje.
- Las respuestas automáticas basadas en LLM están deshabilitadas por defecto.
- La ejecución de código está habilitada por defecto, lo que permite que el agente ejecute código y provea feedback.
- Permite personalizar la forma de obtener la entrada humana sobrescribiendo el método `get_human_input`.
- Puede modificar la forma de ejecutar bloques de código, un solo bloque de código o llamadas a funciones sobrescribiendo los métodos `execute_code_blocks`, `run_code` y `execute_function`.
- Posibilidad de personalizar el mensaje inicial de la conversación sobrescribiendo el método `generate_init_message`.

### Ejemplos de uso

```python
proxy_agent = UserProxyAgent(
    name="userProxy",
    function_map={"calculate": lambda x: x*2},
    code_execution_config={
        "work_dir": "/path_to_working_directory",
        "use_docker": True,
        "timeout": 10
    }
)
```

#### math_user_proxy_agent

`math_user_proxy_agent` es un agente experimental diseñado específicamente para manejar problemas matemáticos. Hereda las características de `UserProxyAgent` y ofrece capacidades adicionales para resolver problemas matemáticos mediante la ejecución de código Python o consultas a Wolfram Alpha.

##### Características

- Puede gestionar y resolver problemas matemáticos.
- Ofrece una interacción automática sin solicitar entrada humana, configurado con `human_input_mode` en "NEVER" por defecto.
- Tiene la capacidad de generar un mensaje inicial basado en un problema y tipo de indicación proporcionados, permitiendo opciones como resolver el problema directamente con Python, resolverlo paso a paso con Python, o resolverlo usando herramientas como Python o Wolfram Alpha.
- Puede ejecutar bloques de código Python y guardar códigos anteriores para ser ejecutados junto con el nuevo código.
- Puede ejecutar consultas individuales en Wolfram Alpha y devolver la salida.
- Se ha incorporado una característica que limita el número máximo de consultas inválidas por paso, configurado por `max_invalid_q_per_step`.

##### Ejemplo de uso

```python
math_agent = MathUserProxyAgent(
    name="MathChatAgent",
    max_invalid_q_per_step=2
)

# Generar un mensaje inicial para resolver un problema matemático
problem_to_solve = "Resuelve la ecuación cuadrática x^2 - 5x + 6 = 0"
prompt_message = math_agent.generate_init_message(problem=problem_to_solve, prompt_type="default")

# Ejecutar un código Python
result = math_agent.execute_one_python_code("2 + 2")

# Realizar una consulta en Wolfram Alpha
wolfram_output = math_agent.execute_one_wolfram_query("integral of x^2 dx")
```

#### retrieve_user_proxy_agent

El `retrieve_user_proxy_agent` es un agente especializado en la recuperación de documentos basados en un problema o consulta proporcionada. Funciona tomando un problema y buscando en un conjunto de documentos para encontrar respuestas o soluciones relevantes.

##### Características

- Permite personalizar la configuración de recuperación con diferentes opciones, como el tipo de tarea, el cliente de la base de datos, el modelo a utilizar, y más.
- Proporciona la capacidad de integrar bases de datos vectoriales personalizadas, permitiendo a los usuarios sobrescribir funciones para adaptarse a sus propias bases de datos.

##### Ejemplo de uso

```python
from agentchat.contrib.retrieve_user_proxy_agent import RetrieveUserProxyAgent

# Configuración de recuperación
retrieve_config = {
    "task": "qa",
    "model": "gpt-4",
    "chunk_token_size": 100,
    "context_max_tokens": 200,
    "chunk_mode": "multi_lines",
    "embedding_model": "all-MiniLM-L6-v2",
    "customized_prompt": None,
    "customized_answer_prefix": "",
    "update_context": True,
    "get_or_create": False
}

# Inicializar el agente
agent = RetrieveUserProxyAgent(
    name="RetrieveChatAgent",
    human_input_mode="ALWAYS",
    retrieve_config=retrieve_config
)

# Generar un mensaje inicial
problem = "What is the Pythagorean theorem?"
prompt = agent.generate_init_message(problem)
print(prompt)
```

#### QdrantRetrieveUserProxyAgent
`QdrantRetrieveUserProxyAgent` es una clase diseñada para interactuar con un agente de chat y recuperar información utilizando Qdrant, una solución de búsqueda de similitud. Es una extensión de `RetrieveUserProxyAgent` y proporciona métodos específicos para interactuar con Qdrant.

##### Características
- **Modos de Entrada del Humano**: Puede configurar cómo y cuándo se solicita la entrada del usuario con las opciones "ALWAYS", "TERMINATE", "NEVER".
- **Configuración de Recuperación**: Configuración detallada para ajustar cómo se recuperan los datos, incluyendo el tipo de tarea, la ruta de los documentos, el nombre de la colección, el modelo a utilizar, entre otros.
- **Recuperación de Documentos**: Ofrece un método para recuperar documentos basados en un problema dado con opciones de filtrado.
- **Creación de Colecciones Qdrant**: Permite crear una colección Qdrant a partir de todos los archivos en un directorio dado.
- **Consulta a Qdrant**: Realiza una búsqueda de similitud con filtros en una colección Qdrant.

##### Ejemplo de uso
```python
from agentchat.contrib.qdrant_retrieve_user_proxy_agent import QdrantRetrieveUserProxyAgent

# Instanciar el agente
agent = QdrantRetrieveUserProxyAgent(name="QdrantAgent", human_input_mode="ALWAYS")

# Crear una colección Qdrant desde un directorio
agent.create_qdrant_from_dir(dir_path="/path/to/docs/directory")

# Recuperar documentos basados en un problema específico
results = agent.retrieve_docs(problem="¿Qué es Qdrant?", n_results=5)

# Mostrar resultados
for doc in results:
    print(doc["document"])
```

## AssistantAgent

`AssistantAgent` es una subclase de `ConversableAgent` que está configurada con un mensaje de sistema predeterminado, diseñado específicamente para resolver tareas utilizando LLM. Esta herramienta es capaz de sugerir bloques de código en Python y de ayudar en su depuración. Por defecto, el `human_input_mode` está configurado en "NEVER" y no ejecuta código automáticamente, por lo que espera que el usuario ejecute el código propuesto.

### Características

- Diseñado para interactuar y resolver tareas con LLM, incluyendo la sugerencia de bloques de código en Python y su depuración.
- No ejecuta código por defecto, lo que significa que es el usuario quien debe ejecutar cualquier código sugerido.
- Configurable a través de varios parámetros, como el mensaje del sistema, la configuración de LLM, y condiciones de terminación.

### Ejemplos de uso

```python
agent = AssistantAgent(name="Assistant",
                       system_message="Ayudante de código",
                       llm_config=None,
                       human_input_mode="NEVER",
                       code_execution_config=False)

response = agent.handle({"content": "¿Cómo puedo invertir una lista en Python?"})
print(response)
```

#### retrieve_assistant_agent

`RetrieveAssistantAgent` es una subclase experimental de `AssistantAgent`. Está configurado con un mensaje de sistema predeterminado específicamente diseñado para resolver tareas con LLM. Al igual que el `AssistantAgent`, puede sugerir bloques de código en Python y ayudar en su depuración. No ejecuta código de forma automática, lo que significa que espera que el usuario lleve a cabo la ejecución de cualquier código propuesto.

##### Características

- Es una versión experimental y especializada del `AssistantAgent` para la recuperación y resolución de tareas con LLM.
- Por defecto, no ejecuta el código automáticamente, delegando la responsabilidad de ejecución al usuario.

##### Ejemplo de uso

```python
retrieve_agent = RetrieveAssistantAgent(name="RetrieveAgent",
                                        system_message="Ayudante de recuperación",
                                        human_input_mode="NEVER",
                                        code_execution_config=False)

response = retrieve_agent.handle({"content": "¿Cómo puedo filtrar elementos de una lista en Python?"})
print(response)
```

#### TeachableAgent

El `TeachableAgent` es una subclase de `ConversableAgent` que utiliza una base de datos vectorial para recordar y recuperar las enseñanzas de los usuarios. El agente tiene la capacidad de aprender de los comentarios de los usuarios, almacenarlos como memos y recuperarlos según sea necesario. Aunque se ha diseñado principalmente para interacciones individuales, aún no ha sido probado en configuraciones de chat grupal.

##### Características

- Utiliza una base de datos vectorial para recordar enseñanzas de los usuarios.
- Capaz de aprender de los comentarios de los usuarios y decidir qué enseñanzas almacenar.
- Proporciona métodos para la gestión de memos, como almacenamiento, recuperación y pre-población de la base de datos.
- Se integra con `TextAnalyzerAgent` para analizar textos según instrucciones específicas.

##### Ejemplo de uso

```python
agent = TeachableAgent(name="TeachableBot", 
                       system_message="You are a helpful AI assistant that remembers user teachings from prior chats.", 
                       llm_config=None, 
                       teach_config={"verbosity": 1, "reset_db": True})

response = agent.handle({"content": "Teach me that the sky is blue."})
print(response)

feedback = agent.learn_from_user_feedback()
print(feedback)

related_memos = agent.retrieve_relevant_memos("Tell me about the sky.")
print(related_memos)
```

#### TextAnalyzerAgent

`TextAnalyzerAgent` es una subclase de `ConversableAgent` diseñada específicamente para analizar textos según las instrucciones proporcionadas. Este agente se especializa en llevar a cabo análisis detallados de fragmentos de texto y proporcionar información pertinente basada en las instrucciones dadas.

##### Características

- Diseñado para analizar textos según instrucciones específicas proporcionadas.
- No solicita la intervención humana por defecto (configurado para "NEVER").

##### Ejemplo de uso

```python
agent = TextAnalyzerAgent(name="TextAnalyzer", 
                          system_message="I am here to analyze the text you provide.")

analysis_instructions = "Extract main ideas"
response = agent.analyze_text("The sun rises in the east and sets in the west.", analysis_instructions)
print(response)
```

## GroupChatManager

`GroupChatManager` es un agente de gestión de chat en vista previa que supervisa y gestiona el flujo de una conversación grupal que involucra a múltiples agentes. Con mecanismos incorporados, el manager toma decisiones sobre el flujo de la conversación, especialmente sobre qué agente debe intervenir a continuación, garantizando así un flujo de conversación dinámico y coherente.

### Características

- **Selección Dinámica de Hablantes**: El manager decide dinámicamente qué agente debe hablar a continuación basándose en el contexto de la conversación y los mensajes intercambiados.
- **Filtro de Llamada a Función**: Cuando un mensaje sugiere una llamada a función y `func_call_filter` está activado, el siguiente agente a hablar será aquel que contenga la función correspondiente en su `function_map`.
- **Control del Admin**: En caso de una interrupción, un agente administrador toma control de la conversación.
- **Métodos para la Selección de Hablantes**: Métodos integrados como `select_speaker_msg` y `select_speaker` ayudan a determinar y anunciar el próximo hablante.
- **Rotación de Agentes**: Permite un flujo circular o secuencial de agentes en la conversación.

### Ejemplo de uso

```python
from agentchat import GroupChat, GroupChatManager, ConversableAgent

# Creando algunos agentes de ejemplo
agent1 = ConversableAgent(name="Agent1")
agent2 = ConversableAgent(name="Agent2")
group_chat_config = GroupChat(agents=[agent1, agent2], max_round=5, admin_name="Admin")

# Creando el manager
manager = GroupChatManager()

# Ejecutando el chat grupal
manager.run_chat(config=group_chat_config)
```

# Funciones

## Registro de funciones

### Pasos

1. Utiliza el método `register_function` del framework AutoGen.
2. Proporciona un argumento `function_map` que es un diccionario, donde las claves son los nombres de las funciones y los valores son las implementaciones de las funciones.

### Ejemplo

```python
# register the functions
user_proxy.register_function(
    function_map={
        "python": exec_python,
        "sh": exec_sh,
    }
)
```

## Definición de funciones

### Pasos

1. Define la función propia.
2. Implementa la lógica deseada para esa función dentro de su definición.

### Ejemplo

```python
def exec_python(cell):
    # ... implementation ...

def exec_sh(script):
    # ... implementation ...
```

## Configuración de funciones

### Pasos

1. Define las funciones registradas en la configuración `llm_config`.
2. Describe detalles sobre los parámetros que requiere cada función y proporciona una descripción de lo que hace cada función.

### Ejemplo

```python
llm_config = {
    "functions": [
        {
            "name": "python",
            "description": "run cell in ipython and return the execution result.",
            # ... more configuration ...
        },
        {
            "name": "sh",
            "description": "run a shell script and return the execution result.",
            # ... more configuration ...
        },
    ],
    # ... more configuration ...
}
```

## Uso de funciones

### Pasos

1. En el contexto de una conversación, permite que los agentes aprovechen las funciones registradas.
2. Utiliza estas funciones para ejecutar código o realizar acciones específicas según las necesidades de la conversación.