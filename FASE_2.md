# Fase 2: Estudio del protocolo MCP

## Objetivos
- Comprender el protocolo Model Context Protocol (MCP).
- Extraer los mensajes, flujos y mecanismos de extensión.
- Mapear las funcionalidades de `mcp-computer` a la especificación MCP.

## Documentación revisada
- Introducción al protocolo MCP:
  https://raw.githubusercontent.com/modelcontextprotocol/docs/main/introduction.mdx
- Guía rápida de cliente MCP (Python):
  https://raw.githubusercontent.com/modelcontextprotocol/docs/main/quickstart/client.mdx

## Arquitectura general
MCP define un sistema cliente-servidor:
- El **MCP Client** inicia una conexión (HTTP/WebSocket).
- El **MCP Server** registra un conjunto de herramientas (tools).
- El cliente envía mensajes JSON de tipo solicitud y el servidor responde con JSON.
- Uso de identificadores únicos (`id`) para correlacionar peticiones y respuestas.

## Flujo de mensajes MCP
1. **list_tools**
   - Cliente 👉 Servidor:
     ```json
     {"type": "list_tools", "id": "1"}
     ```
   - Servidor 👉 Cliente:
     ```json
     {"type":"tools","id":"1","tools":[
       {"name":"bash","type":"bash_20250124"},
       {"name":"computer","type":"computer_20250124"},
       {"name":"str_replace_editor","type":"text_editor_20250124"}
     ]}
     ```

2. **invoke_tool**
   - Cliente 👉 Servidor:
     ```json
     {
       "type":"invoke_tool",
       "id":"2",
       "tool": {"name":"bash","type":"bash_20250124"},
       "input": {"command":"ls -la"}
     }
     ```
   - Servidor 👉 Cliente (tool_use acknowledgment opcional):
     ```json
     {"type":"tool_use","id":"2","name":"bash","input":{"command":"ls -la"}}
     ```

3. **tool_result**
   - Servidor 👉 Cliente:
     ```json
     {
       "type":"tool_result",
       "id":"2",
       "output":"total 0\ndrwxr-xr-x . .",
       "error":null
     }
     ```

4. **heartbeat / ping-pong**
   - Opcionalmente, el protocolo puede incluir mensajes de latido para detección de caídas.

## Registro de herramientas en el servidor
El servidor MCP expondrá las herramientas definidas en `mcp-computer`:

| Nombre              | Tipo MCP               | Descripción                                       |
| ------------------- | ---------------------- | ------------------------------------------------- |
| bash                | bash_20250124          | Ejecuta comandos en una sesión Bash persistente   |
| computer            | computer_20250124      | Interacción GUI (ratón, teclado, capturas)        |
| str_replace_editor  | text_editor_20250124   | Edición de archivos: buscar/reemplazar, insert    |

Todas las herramientas implementan `to_params()` para registrar su nombre y tipo y `__call__(...)` para ejecutar la acción.

## Mapeo de funcionalidades existentes
- El bucle de agente (`computer_use_demo/loop.py`) se sustituye por un servidor que distribuye las llamadas a cada herramienta.
- La lógica de iteración y parsing de bloques deja de residir en el cliente y pasa al servidor MCP.
- Cada herramienta se registra como un recurso accesible mediante mensajes JSON, desacoplando la lógica de UI del cliente.

---
**Próximos pasos**
- Crear el esqueleto del servidor MCP (endpoints HTTP/WebSocket).
- Implementar manejo de mensajes `list_tools`, `invoke_tool` y `tool_result`.
- Escribir pruebas unitarias de protocolo y ejemplos de cliente. 