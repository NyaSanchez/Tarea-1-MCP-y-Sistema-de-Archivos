# 4. Arquitectura de MCP

## Modelo host / cliente / servidor

MCP define tres roles:

- **Host**: la aplicación con la que interactúa el usuario final y que aloja al
  modelo de lenguaje. En este proyecto el host es **Claude Desktop**.
- **Cliente**: el componente, integrado en el host, que mantiene una conexión
  1:1 con un servidor MCP, negocia versión y capacidades, y traduce las
  decisiones del modelo en llamadas al protocolo. En Claude Desktop el cliente
  vive dentro de la misma aplicación; el usuario no lo ve como algo aparte.
- **Servidor**: el proceso que expone un catálogo de capacidades sobre un
  recurso concreto. En este proyecto es el servidor de referencia
  `@modelcontextprotocol/server-filesystem`, ejecutado como proceso hijo por
  Claude Desktop, que expone operaciones sobre un directorio del disco.

Un mismo host puede sostener conexiones simultáneas con varios servidores (uno
de sistema de archivos, otro de una base de datos, etc.); cada conexión
cliente-servidor es independiente de las demás.

## Primitivas del lado del servidor

- **Tools (herramientas)**: funciones que el modelo puede invocar para producir
  un efecto o un cálculo (leer un archivo, ejecutar una consulta). Cada una se
  anuncia con nombre, descripción y un esquema JSON de sus parámetros.
- **Resources (recursos)**: datos que el servidor pone a disposición para que
  el cliente los adjunte al contexto (el contenido de un archivo, una fila de
  una base de datos), identificados por una URI.
- **Prompts (plantillas de prompt)**: plantillas de instrucciones
  parametrizables, definidas por el servidor, que el usuario invoca de forma
  explícita para guiar una tarea recurrente.

## Primitivas del lado del cliente

- **Roots**: el cliente informa al servidor qué directorios (o URIs) están
  dentro de su ámbito de trabajo. Es, en la práctica, el mecanismo que
  delimita hasta dónde puede llegar un servidor de sistema de archivos.
- **Elicitation**: permite a un servidor pedirle al cliente que solicite al
  usuario un dato puntual a mitad de una operación, en lugar de exigir toda la
  información por adelantado.
- **Sampling**: permite a un servidor pedirle al cliente que ejecute una
  compleción del modelo por cuenta del servidor, de modo que sea el cliente
  quien controle el acceso y el costo del modelo.

## Transportes

- **stdio**: el servidor corre como proceso hijo local; cliente y servidor se
  comunican por entrada/salida estándar. Es el transporte que usa Claude
  Desktop para lanzar el servidor de sistema de archivos: no hay red de por
  medio, todo ocurre en la misma máquina.
- **Streamable HTTP**: el servidor es un proceso remoto expuesto por HTTP; el
  cliente le envía peticiones y el servidor puede responder con eventos en
  flujo. Se usa para servidores que no viven en la máquina del usuario.

## Versión de la especificación consultada

Este documento se escribió consultando la especificación oficial del Model
Context Protocol, revisión **2026-07-28** (publicada el 28 de julio de 2026).

Aclaración necesaria: a la fecha de esta investigación (septiembre de 2026)
esa es la revisión más reciente, pero introdujo cambios importantes frente a
la anterior (2025-11-25): eliminó el intercambio inicial de tipo *initialize*
a favor de un modelo sin estado, y marcó como **obsoletas** (deprecadas, no
eliminadas) a Roots, Sampling y Logging, con una ventana de al menos doce
meses antes de poder retirarlas por completo. Dado lo reciente del cambio, la
mayoría de los clientes y servidores en uso real —incluidos Claude Desktop y
el servidor de referencia de sistema de archivos usado en la Parte 2 de esta
tarea— siguen operando conforme a la revisión previa, en la que Roots es un
mecanismo activo y no obsoleto. Por eso este documento describe ambas
primitivas de cliente como vigentes en la práctica, aunque formalmente Roots
ya esté en proceso de retiro según la especificación más nueva.