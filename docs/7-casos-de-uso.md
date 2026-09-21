# 7. Casos de uso

A continuación, tres herramientas reales que implementan MCP actualmente, con
qué fin concreto lo usan cada una.

## 1. Claude Code

Claude Code es un agente de línea de comandos (con variantes de escritorio y
para VS Code/JetBrains) hecho por Anthropic. Permite registrar servidores MCP
con un comando como `claude mcp add <nombre> -- <comando-del-servidor>`, y a
partir de ahí el agente puede invocar, en la misma sesión de trabajo, tanto
sus herramientas nativas de archivo y terminal como las herramientas que
exponga cualquier servidor MCP conectado (por ejemplo, un servidor de GitHub
para abrir pull requests, o uno de base de datos para consultar un esquema).
El uso concreto de MCP aquí es extender lo que el agente puede hacer más allá
del propio repositorio local, sin que cada integración tenga que programarse
a mano dentro de Claude Code.

## 2. GitHub Copilot (en VS Code)

GitHub Copilot incorporó soporte para servidores MCP directamente en VS Code:
la persona desarrolladora agrega servidores desde un panel de configuración
(o desde un archivo `mcp.json` del proyecto) y Copilot descubre sus
herramientas igual que describe el punto 3 de esta investigación. Microsoft
reporta más de 60 servidores MCP listos para usarse desde ahí, incluyendo uno
para conectar con Dataverse. El caso de uso es el mismo patrón: en vez de que
cada integración (una base de datos, un CRM, un servicio interno) requiera una
extensión de VS Code hecha a la medida, un servidor MCP la expone una sola vez
y cualquier cliente compatible —Copilot incluido— la puede usar.

## 3. Google Antigravity

Google Antigravity es un entorno de desarrollo agéntico (no debe confundirse
con un modelo de lenguaje: aquí sí hablamos de una plataforma completa, con
editor, terminal y navegador integrados). Antigravity usa MCP para conectar a
sus agentes con Google Cloud, Firebase y otras fuentes de datos externas: un
agente que está construyendo una aplicación puede, a través de un servidor
MCP, consultar o modificar recursos de esos servicios como parte de la misma
tarea, sin que la persona desarrolladora tenga que salir del entorno para
hacerlo manualmente.

*(Nota de precisión: Qwen es una familia de modelos de lenguaje, no una
plataforma de desarrollo agéntico; por eso no se incluye aquí como "caso de
uso" — si se quisiera citar un ejemplo basado en Qwen, habría que nombrar la
herramienta concreta que lo integra, no el modelo en sí.)*

## Cómo editan repositorios completos sin subir archivos manualmente

En el flujo antiguo (copiar y pegar texto en un navegador), la persona era el
canal: tenía que abrir cada archivo, copiar su contenido, pegarlo en el chat,
esperar la respuesta, y volver a pegar el resultado de vuelta en su editor,
archivo por archivo. Estas tres herramientas eliminan ese canal manual porque
el agente tiene, mediante MCP (o herramientas nativas equivalentes construidas
sobre el mismo principio), acceso directo de lectura y escritura sobre el
árbol de archivos del proyecto: puede listar la estructura completa de un
repositorio, abrir varios archivos a la vez, aplicar cambios en cada uno, y
—cuando el entorno también le da acceso a una terminal— ejecutar los comandos
de Git para dejar esos cambios en el historial (`git add`, `git commit`), todo
dentro de la misma sesión de trabajo y sin que la persona mueva el contenido a
mano en ningún punto intermedio.
