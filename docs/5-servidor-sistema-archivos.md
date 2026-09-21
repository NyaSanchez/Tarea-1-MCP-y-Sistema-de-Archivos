# 5. El servidor de sistema de archivos

## "FS" no es parte del protocolo

Es importante no confundir el protocolo con una de sus implementaciones. MCP en
sí mismo no sabe nada de archivos: es un protocolo genérico de descubrimiento e
invocación de capacidades. El servidor de sistema de archivos
(`@modelcontextprotocol/server-filesystem`) es apenas uno de los **servidores
de referencia** que mantiene el propio proyecto MCP, igual que existen
servidores de referencia para bases de datos SQLite, para hacer fetch de URLs,
o para memoria persistente. "FS" podría dejar de existir mañana y el
protocolo seguiría intacto; lo que cambiaría es que ya no habría ese servidor
en particular ofreciendo esas herramientas.

## Herramientas que expone

El servidor de sistema de archivos (implementación en Node.js) publica estas
herramientas:

| Herramienta | Qué hace |
|---|---|
| `list_directory` | Lista el contenido de un directorio, marcando cada entrada como `[FILE]` o `[DIR]`. |
| `list_directory_with_sizes` | Igual que la anterior, pero agrega tamaños y totales. |
| `directory_tree` | Devuelve un árbol recursivo en JSON de un directorio. |
| `read_text_file` | Lee el contenido completo de un archivo de texto. |
| `read_media_file` | Lee una imagen o audio y lo transmite en base64. |
| `read_multiple_files` | Lee varios archivos en paralelo sin que un error tumbe toda la operación. |
| `write_file` | Crea un archivo nuevo o sobrescribe uno existente. |
| `edit_file` | Aplica ediciones puntuales (buscar/reemplazar) con vista previa tipo diff. |
| `create_directory` | Crea un directorio (y los directorios padre que falten). |
| `move_file` | Mueve o renombra archivos y directorios. |
| `search_files` | Busca recursivamente archivos por nombre o patrón. |
| `get_file_info` | Devuelve metadatos (tamaño, fechas, permisos) de un archivo o directorio. |
| `list_allowed_directories` | Devuelve la lista de directorios a los que el servidor tiene permitido el acceso. |

Estas herramientas cubren exactamente las operaciones que pide la Parte 2 de
esta tarea: listar, leer, crear/escribir, modificar, mover y buscar.

## Cómo se delimita su alcance

El servidor no tiene acceso al disco completo por diseño: al arrancar, recibe
como argumento uno o más directorios permitidos (*allowed directories*), por
ejemplo:

```
npx -y @modelcontextprotocol/server-filesystem "C:\Users\Nya\ruta\permitida"
```

Cualquier operación que intente salirse de esos directorios —incluso mediante
rutas relativas del tipo `../../` o enlaces simbólicos que apunten afuera— es
rechazada por el propio servidor antes de tocar el disco. Alternativamente,
algunos clientes que soportan la primitiva **Roots** (ver punto 4) pueden
comunicarle esos directorios de forma dinámica en tiempo de ejecución en vez
de fijarlos por línea de comandos; si el cliente no soporta Roots, el servidor
simplemente se queda con los directorios que recibió al iniciar.

## Por qué existe ese límite y qué pasaría sin él

El límite existe porque el servidor corre con los mismos permisos del usuario
que lo lanzó: si pudiera acceder a cualquier ruta del sistema, una instrucción
maliciosa (ya sea escrita directamente por el usuario, o inyectada a través
del contenido de un archivo que el modelo lee) podría leer credenciales,
sobrescribir archivos de configuración del sistema operativo o borrar datos
fuera del proyecto en el que se está trabajando. Sin esa restricción, el
"aislamiento" del punto 2 de esta investigación —que existe precisamente para
que un modelo no toque el disco sin control— quedaría anulado por completo en
cuanto se conectara este servidor. El directorio permitido es, en la práctica,
el único mecanismo que mantiene acotado el daño posible.
