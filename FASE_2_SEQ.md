# Fase 2: Diagrama de Secuencia de Flujos MCP

Este diagrama muestra la secuencia completa de interacciones entre un Cliente MCP, el Servidor MCP, la Colección de Herramientas, cada Herramienta concreta y el Sistema Operativo remoto.

```mermaid
sequenceDiagram
    participant Cliente as MCP Client
    participant Servidor as MCP Server
    participant Coleccion as ToolCollection
    participant Herramienta as Herramienta
    participant OS as Sistema Operativo

    %% Flujo de listado de herramientas
    Cliente->>Servidor: {"type":"list_tools","id":"1"}
    Servidor-->>Cliente: {"type":"tools","id":"1","tools":[...]}  

    %% Flujo de invocación de herramienta
    Cliente->>Servidor: {"type":"invoke_tool","id":"2","tool":{"name":"bash","type":"bash_20250124"},"input":{"command":"ls -la"}}
    Servidor-->>Cliente: {"type":"tool_use","id":"2","name":"bash","input":{"command":"ls -la"}}  

    %% Despacho interno de la ejecución
    Servidor->>Coleccion: run(name="bash", input={"command":"ls -la"})
    Coleccion->>Herramienta: __call__(command="ls -la")
    Herramienta->>OS: Ejecuta comando en Bash
    OS-->>Herramienta: stdout, stderr
    Herramienta-->>Coleccion: ToolResult(output, error, base64_image)
    Coleccion-->>Servidor: ToolResultBlock(id="2", content=[...])

    %% Retorno del resultado al cliente
    Servidor-->>Cliente: {"type":"tool_result","id":"2","output":"...","error":null,"image":null}

    %% Flujo opcional de latido (ping-pong)
    Note over Cliente,Servidor: Opcional: heartbeat / ping-pong para detección de caídas
``` 