# 3. MCP frente a una API

Este es el punto central de la investigación: entender con precisión en qué se parecen
y, sobre todo, en qué se diferencian el Model Context Protocol (MCP) y una API
tradicional.

## ¿Qué es una API?

Una **API (Application Programming Interface)** es, en esencia, un **contrato entre dos
programas**: define qué operaciones (*endpoints*) existen, qué parámetros recibe cada
una, qué formato de respuesta devuelve y qué reglas de autenticación aplican. Ese
contrato queda documentado (por ejemplo, en una especificación OpenAPI/Swagger, o en una
página de documentación como la de Stripe o la de GitHub) y es responsabilidad de **la
persona que desarrolla** leerlo, entenderlo y decidir, de antemano, en tiempo de
programación:

- Qué endpoint se va a llamar (`GET /repos/{owner}/{repo}/issues`, por ejemplo).
- En qué momento del flujo del programa se va a llamar.
- Cómo se arma la petición (encabezados, cuerpo, parámetros).
- Cómo se interpreta y se usa la respuesta.

Todo eso queda **fijo en el código fuente** de la aplicación. Si mañana se quiere que la
aplicación llame a un endpoint distinto, o en un orden distinto, hay que modificar y
volver a desplegar el código. La API en sí misma no "decide" nada: solo expone
capacidades; quien decide cuándo y cómo usarlas es siempre el programa que la consume.

## ¿Qué es MCP?

El **Model Context Protocol (MCP)** es un **protocolo abierto**, basado en el estándar
**JSON-RPC 2.0**, mediante el cual un **servidor** publica un **catálogo de
herramientas** (*tools*) — junto con recursos y plantillas de *prompt*, ver el punto 4 —
describiendo para cada una su nombre, una descripción en lenguaje natural y un esquema
formal (JSON Schema) de los parámetros que acepta.

La diferencia clave con una API tradicional es que ese catálogo **no lo lee ni lo
interpreta una persona en tiempo de programación**: lo descubre el **modelo de lenguaje**
en tiempo de ejecución, a través de una llamada estándar del protocolo
(`tools/list`), y es el propio modelo quien decide —con base en la petición en lenguaje
natural del usuario y en las descripciones de cada herramienta— **cuál invocar y con qué
argumentos**, generando una llamada (`tools/call`) que el cliente ejecuta contra el
servidor. Nadie escribió de antemano en el código "si el usuario pide esto, llama a
tal endpoint": esa decisión la toma el modelo, dinámicamente, en cada conversación.

## Tabla comparativa

| Criterio | API tradicional | MCP |
|---|---|---|
| **¿Quién decide qué se invoca?** | La persona que desarrolla, en tiempo de programación (la lógica está fija en el código). | El modelo de lenguaje, en tiempo de ejecución, según la petición del usuario. |
| **Descubrimiento de capacidades** | Manual: se lee documentación externa (Swagger/OpenAPI, docs web) antes de programar. | Dinámico: el cliente pide la lista de herramientas al servidor con `tools/list` y las recibe con nombre, descripción y esquema. |
| **Acoplamiento cliente–servicio** | Alto: el cliente está programado contra los endpoints exactos de esa API específica. | Bajo: cualquier cliente compatible con MCP puede hablar con cualquier servidor MCP sin código nuevo, siempre que el servidor exponga sus herramientas. |
| **Formato de los mensajes** | Varía por API (REST/JSON, SOAP/XML, GraphQL, gRPC...); no hay un estándar único. | Estandarizado: siempre JSON-RPC 2.0, sobre uno de los transportes definidos por el protocolo (stdio o Streamable HTTP). |
| **Autenticación y consentimiento** | Definida por cada API (API keys, OAuth, tokens...); el consentimiento del usuario final rara vez es parte del protocolo mismo. | El protocolo contempla el consentimiento como parte del diseño: los clientes (como Claude Desktop) suelen pedir confirmación humana antes de ejecutar una herramienta, además de los esquemas de autenticación que use cada servidor. |
| **Reutilización entre aplicaciones distintas** | Baja: cada aplicación que quiera usar esa API debe implementar su propio cliente para ella. | Alta: un mismo servidor MCP (por ejemplo, el de sistema de archivos) puede conectarse, sin cambios, a Claude Desktop, VS Code, Cursor o cualquier otro host compatible. |

## MCP no sustituye a las APIs

Es fundamental dejar esto explícito, porque es uno de los errores conceptuales más
comunes al hablar del tema: **MCP no reemplaza ni vuelve obsoletas a las APIs.** Un
servidor MCP, en la inmensa mayoría de los casos, **envuelve** una API, una base de
datos o un recurso que ya existía. Por ejemplo, un servidor MCP que da acceso a GitHub
por debajo sigue llamando a la API REST de GitHub; lo único que agrega MCP es una capa
por encima que:

1. Describe esas capacidades en un formato que un modelo puede descubrir e interpretar
   por sí mismo.
2. Estandariza la forma en la que un modelo puede invocarlas (JSON-RPC 2.0, con el mismo
   patrón `tools/list` + `tools/call` sin importar qué haya detrás).

En otras palabras: **la API sigue siendo el recurso; MCP es la capa que lo hace
descubrible y utilizable por un modelo de lenguaje**, no una alternativa que compita con
ella ni una tecnología que la sustituya.

## MCP no es de un solo proveedor

Otro error frecuente es presentar MCP como una tecnología propietaria de Anthropic (la
empresa que lo creó). Si bien Anthropic lo diseñó y lo publicó como estándar abierto en
noviembre de 2024, desde entonces ha sido adoptado por un ecosistema muy amplio de
plataformas de distintos proveedores —entre ellas ChatGPT, Cursor, Gemini, Microsoft
Copilot y Visual Studio Code (Anthropic, 2025)—, y el 9 de diciembre de 2025 Anthropic
donó formalmente el protocolo a la **Agentic AI Foundation**, un fondo dirigido bajo la
**Linux Foundation**, cofundado junto con **Block** y **OpenAI**, y con el respaldo de
Google, Microsoft, AWS, Cloudflare y Bloomberg (Anthropic, 2025). Es decir: hoy la
gobernanza técnica de MCP ya no depende de una sola empresa, sino de un consorcio
multiproveedor bajo una fundación sin fines de lucro, de forma similar a como Kubernetes
o Node.js son gobernados fuera del control exclusivo de quien los creó originalmente.

## Referencias

- Anthropic. (2025, 9 de diciembre). *Donating the Model Context Protocol and
  establishing the Agentic AI Foundation*. https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation
- JSON-RPC Working Group. (2010). *JSON-RPC 2.0 Specification*. https://www.jsonrpc.org/specification
