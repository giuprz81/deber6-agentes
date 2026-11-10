# Equipo de agentes

## 1. Objetivo general

Diseñar un equipo de agentes especializados para revisar el archivo `usuarios.py` de una tienda en línea antes de crear un Pull Request.

El equipo tiene un agente orquestador y dos especialistas independientes. Ambos especialistas analizan el mismo archivo, pero tienen responsabilidades diferentes.

## 2. Integrantes del equipo

| Agente | Misión | Herramientas autorizadas | Entrada | Salida | Límites |
|---|---|---|---|---|---|
| Orquestador | Distribuir las tareas y consolidar los resultados. | Delegación a especialistas. | Ruta del archivo `usuarios.py`. | Tabla consolidada de hallazgos y casos de prueba. | No modifica archivos ni acepta resultados sin verificar. |
| Especialista 1: revisor-seguridad | Buscar claves o tokens escritos en el código y entradas de usuario sin validar. | Read, Grep, Glob. | Ruta del archivo `usuarios.py`. | Lista de hallazgos con línea, problema, evidencia y severidad. | Solo revisa `usuarios.py`; no escribe ni modifica archivos. |
| Especialista 2: revisor-pruebas | Proponer tres casos de prueba faltantes. | Read. | Ruta del archivo `usuarios.py`. | Tres propuestas con nombre y qué verifica cada una. | Solo analiza `usuarios.py`; no escribe ni ejecuta pruebas. |

## 3. Patrón de orquestación

Se utiliza un patrón paralelo para los dos especialistas porque sus actividades son independientes y ambos revisan el mismo archivo.

El orquestador inicia las dos tareas, espera sus resultados y posteriormente consolida la información.

### Flujo de trabajo

1. El usuario solicita revisar `usuarios.py`.
2. El orquestador asigna la revisión de seguridad al Especialista 1.
3. Simultáneamente, asigna la identificación de pruebas faltantes al Especialista 2.
4. Ambos especialistas analizan el archivo sin modificarlo.
5. El orquestador recibe los resultados.
6. El orquestador verifica y consolida los hallazgos.
7. Una persona revisa los resultados antes de aprobar la continuación del proceso.

## 4. Checkpoint humano

Se establece un control Human-in-the-loop antes de crear el Pull Request.

La persona responsable debe:

- Verificar que los hallazgos estén respaldados por el código.
- Comprobar que las propuestas de pruebas sean pertinentes.
- Revisar que ningún especialista haya modificado `usuarios.py`.
- Evaluar la severidad y prioridad de los problemas.
- Aprobar o rechazar la continuación del proceso.

Si existen hallazgos críticos sin resolver o resultados sin evidencia, el proceso se detiene hasta que sean revisados.

## 5. Manejo de fallos

El orquestador debe comprobar que cada especialista entregue el resultado solicitado.

Si un especialista falla, devuelve información incompleta o presenta hallazgos sin evidencia:

1. Registra el fallo.
2. No acepta el resultado como válido.
3. Puede solicitar una nueva revisión con instrucciones más precisas.
4. Si el problema persiste, informa que la revisión está incompleta.
5. Solicita intervención humana antes de continuar.

## 6. Protección del archivo

Los dos especialistas tienen permisos de lectura y búsqueda.

No están autorizados a modificar, crear o eliminar archivos.

El archivo `usuarios.py` debe permanecer sin cambios durante todo el proceso de revisión.

## 7. Evidencia de implementación

El Especialista 1 fue implementado como subagente local de Claude Code mediante:

`.claude/agents/revisor-seguridad.md`

Su ejecución produjo hallazgos verificables en `usuarios.py`, sin modificar el archivo.

El Especialista 2 se describe como parte del diseño del equipo y se plantea como agente remoto para el ejercicio A2A.

## 8. Comunicación entre agentes mediante A2A

### 8.1. Escenario

Se plantea que el Especialista 2, encargado de proponer pruebas faltantes para `usuarios.py`, funciona como un agente remoto perteneciente a otra organización.

El orquestador puede descubrir sus capacidades y delegarle la tarea mediante el protocolo Agent-to-Agent (A2A).

Este escenario es ilustrativo y no representa una conexión real con un servidor remoto.

### 8.2. Agent Card ilustrativa

La siguiente Agent Card describe las capacidades del agente remoto ficticio.

```json
{
  "name": "Especialista Remoto de Pruebas",
  "description": "Agente especializado en identificar casos de prueba faltantes en código Python.",
  "url": "https://agente.ejemplo/a2a",
  "version": "1.0.0",
  "protocolVersion": "1.0",
  "supportedInterfaces": [
    {
      "url": "https://agente.ejemplo/a2a",
      "protocolBinding": "JSONRPC",
      "protocolVersion": "1.0"
    }
  ],
  "capabilities": {
    "streaming": false,
    "pushNotifications": false
  },
  "skills": [
    {
      "id": "analisis-pruebas-python",
      "name": "Análisis de pruebas faltantes",
      "description": "Identifica escenarios de prueba que no están cubiertos en un módulo Python.",
      "tags": [
        "python",
        "testing",
        "pytest"
      ]
    }
  ],
  "defaultInputModes": [
    "text/plain"
  ],
  "defaultOutputModes": [
    "text/plain"
  ],
  "securitySchemes": {
    "bearerAuth": {
      "type": "http",
      "scheme": "bearer"
    }
  }
}
```

**Nota:** Simplificación didáctica; no es JSON conforme a una versión específica de A2A. La Agent Card, el dominio `agente.ejemplo` y el endpoint son ficticios y no representan una conexión real con un servidor remoto.

### 8.3. Descubrimiento y validación

El orquestador podría obtener la Agent Card desde:

`https://agente.ejemplo/.well-known/agent-card.json`

Antes de delegar una tarea debería:

1. Verificar que el dominio corresponde al proveedor esperado.
2. Comprobar la identidad y autenticidad del agente.
3. Revisar la versión del protocolo y las interfaces disponibles.
4. Verificar que el agente ofrece la habilidad `analisis-pruebas-python`.
5. Comprobar los requisitos de autenticación.
6. Confirmar que la información solicitada no contiene datos sensibles innecesarios.

Si el agente no ofrece la habilidad requerida o no cumple los requisitos de seguridad, el orquestador no debe enviarle la tarea.

### 8.4. Secuencia de comunicación

La comunicación propuesta sigue este orden:

1. El orquestador identifica que necesita propuestas de pruebas faltantes.
2. Obtiene y valida la Agent Card del agente remoto.
3. Envía una solicitud mediante A2A con el contexto necesario de `usuarios.py`.
4. El agente remoto procesa la solicitud y puede responder directamente mediante un mensaje o crear una Task.
5. Si crea una Task, el orquestador consulta o recibe sus actualizaciones de estado.
6. Cuando la tarea termina, el orquestador obtiene el resultado mediante el mensaje o Artifact correspondiente.
7. El orquestador incorpora el resultado a la consolidación.
8. Una persona revisa las propuestas antes de aprobar su utilización.

### 8.5. Estados de una tarea A2A

| Estado | Significado | Acción del orquestador |
|---|---|---|
| `submitted` | La tarea fue recibida y enviada a procesamiento. | Esperar su procesamiento. |
| `working` | El agente está ejecutando la tarea. | Consultar o recibir actualizaciones. |
| `completed` | La tarea finalizó correctamente. | Recibir y revisar el resultado. |
| `failed` | La ejecución falló. | Registrar el fallo y evaluar un nuevo intento. |
| `input-required` | Se necesita información adicional. | Proporcionar el contexto solicitado o cancelar. |
| `rejected` | El agente rechazó la solicitud. | Revisar el motivo y buscar otra alternativa. |
| `canceled` | La tarea fue cancelada. | Registrar la cancelación y detener el seguimiento. |

**Escenario alternativo:**

Si la tarea queda en `input-required`, el orquestador debe revisar qué información falta. Si es apropiado, proporciona el contexto adicional solicitado y permite continuar. Si no puede proporcionar esa información, cancela la tarea o solicita intervención humana.

Si la tarea termina en `failed`, no debe presentar resultados incompletos como si fueran una revisión exitosa.

### 8.6. Diferencia entre subagente local, A2A y MCP

| Característica | Subagente local | A2A | MCP |
|---|---|---|---|
| Propósito | Delegar trabajo dentro del entorno del agente. | Comunicar agentes independientes. | Conectar agentes con herramientas y recursos. |
| Comunicación | Delegación local del harness. | Protocolo entre agentes. | Protocolo cliente-servidor. |
| Identificación | Configuración local del subagente. | Agent Card. | Configuración y descubrimiento de servidores y herramientas MCP. |
| Ejemplo | `revisor-seguridad`. | Especialista remoto de pruebas. | Herramienta para consultar información o ejecutar operaciones autorizadas. |
| Infraestructura | Entorno local. | Servicio remoto y endpoint A2A. | Cliente y servidor MCP. |

A2A y MCP son complementarios, pero no intercambiables. A2A permite la comunicación entre agentes, mientras que MCP permite acceder a herramientas y recursos externos.