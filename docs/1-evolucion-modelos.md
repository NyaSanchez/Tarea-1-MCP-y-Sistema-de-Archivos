# 1. Evolución de los modelos de lenguaje

## ¿Qué es un modelo de lenguaje?

Un **modelo de lenguaje (LM, *Language Model*)** es un sistema estadístico que aprende
la distribución de probabilidad de secuencias de texto: dado un fragmento de texto,
predice cuál es la palabra (o, más precisamente, el *token*) más probable que continúa
esa secuencia. Los primeros modelos de lenguaje (como los modelos *n-grama*) estimaban
esa probabilidad contando la frecuencia con la que unas palabras seguían a otras en un
corpus de texto. Eran útiles para tareas acotadas (corrección ortográfica, predicción de
texto en teclados), pero no podían capturar relaciones de largo alcance dentro de un
texto ni generalizar bien a contextos nuevos.

Con la llegada de las redes neuronales, y en particular de la arquitectura
**Transformer** (Vaswani et al., 2017), los modelos de lenguaje empezaron a representar
las palabras como vectores en un espacio continuo y a usar un mecanismo de
**autoatención** (*self-attention*) que permite que cada palabra de una secuencia "mire"
a todas las demás para decidir cuánto peso darles al construir su representación. Esto
resolvió el problema de las dependencias de largo alcance y permitió entrenar modelos
mucho más profundos de forma paralelizable en hardware como GPUs y TPUs.

## De LM a LLM

Un **modelo de lenguaje grande (LLM, *Large Language Model*)** es, en esencia, un modelo
de lenguaje basado en Transformer entrenado con:

- **Muchos más parámetros** (de millones a cientos de miles de millones).
- **Corpus de entrenamiento masivos** (fragmentos representativos de una gran parte del
  texto disponible públicamente en internet, libros, código, etc.).
- **Mucho más cómputo** de entrenamiento (miles de GPUs/TPUs corriendo durante semanas o
  meses).

Este salto de escala no es solo cuantitativo: a partir de cierto tamaño, los LLM
empiezan a mostrar **capacidades emergentes** — comportamientos que no estaban presentes
(o eran muy débiles) en versiones más pequeñas del mismo modelo, como seguir
instrucciones complejas en lenguaje natural, traducir entre idiomas sin haber sido
entrenados explícitamente para ello, o resolver problemas paso a paso. Este fenómeno es
lo que popularizó el término "LLM" como una categoría distinta dentro de los modelos de
lenguaje, y lo que llevó a que empezaran a usarse como asistentes de propósito general en
lugar de solo como componentes dentro de sistemas más pequeños (por ejemplo, correctores
o traductores).

## Modelos con razonamiento explícito

Una evolución más reciente son los **modelos con razonamiento explícito** (a veces
llamados *reasoning models*), que —antes de dar una respuesta final— generan una cadena
de pasos intermedios de razonamiento (*chain-of-thought*) que el modelo usa como
"borrador" para llegar a una conclusión más confiable, especialmente en tareas de
matemáticas, lógica o programación.

Es importante remarcar un punto que se presta a confusión: **esta capacidad de razonar
explícitamente no aparece automáticamente por el simple hecho de aumentar el tamaño del
modelo.** Un modelo enorme entrenado únicamente para predecir el siguiente token no
necesariamente aprende a razonar de forma estructurada. La capacidad de razonamiento
explícito proviene de dos elementos adicionales y deliberados:

1. **Técnicas de entrenamiento específicas.** Se afina el modelo (por ejemplo, mediante
   aprendizaje por refuerzo con recompensas verificables, o *fine-tuning* sobre ejemplos
   de razonamiento paso a paso) para que aprenda a producir y a apoyarse en esas cadenas
   de pasos intermedios antes de responder, en lugar de "adivinar" la respuesta de
   inmediato.
2. **Cómputo adicional en el momento de la inferencia.** Estos modelos dedican más
   tiempo (y por tanto más cómputo) a "pensar" antes de contestar: generan tokens de
   razonamiento internos que no necesariamente se muestran al usuario final, y ese
   cómputo extra en inferencia está correlacionado con mejoras medibles en tareas
   complejas, de forma similar a como una persona resuelve mejor un problema si se toma
   el tiempo de escribir su razonamiento en un papel en lugar de responder de memoria al
   primer intento.

En resumen: escalar el tamaño de un modelo mejora su conocimiento general y fluidez,
pero el razonamiento explícito y confiable es el resultado de decisiones de diseño
específicas en el entrenamiento y de asignar más cómputo al momento de generar la
respuesta, no una consecuencia automática de tener más parámetros.

## Referencias

- Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser,
  Ł., & Polosukhin, I. (2017). *Attention is all you need*. Advances in Neural
  Information Processing Systems, 30.
