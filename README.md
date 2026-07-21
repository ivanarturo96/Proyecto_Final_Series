_**MAESTRÍA EN CIENCIAS DE LA INTELIGENCIA ARTIFICIAL - FIUNA, SAN LORENZO, 2026**_

_**PROYECTO FINAL DE SERIES TEMPORALES**_

_**ALUMNO: IVÁN GONZÁLEZ**_

_**PROFESORES:**_

* _**SEBASTIÁN GRILLO**_

* _**ENRIQUE PAIVA**_

* _**DIEGO STALDER**_

_**MOMBEU:**_ Inserción Narrativa de la Cultura Guaraní para Modelos de Lenguaje

_**DESCRIPCIÓN DEL PROBLEMA:**_

Los Modelos de Lenguaje de Gran Escala (LLMs) poseen un amplio conocimiento general, pero presentan una importante limitación al momento de representar la riqueza cultural de comunidades con baja presencia digital, como la cultura guaraní. Si bien en Paraguay gran parte de la población ha crecido escuchando o hablando guaraní, la cosmovisión, los relatos tradicionales y el conocimiento ancestral asociado a la lengua rara vez se encuentran disponibles en formatos estructurados que puedan ser utilizados por sistemas de inteligencia artificial.

Además, existe un desafío adicional: los usuarios difícilmente realizan consultas explícitas sobre mitología o cultura guaraní. En la mayoría de los casos, el contexto cultural debe surgir de manera natural durante una conversación sobre cualquier tema, sin interrumpir el flujo del diálogo. A esto se suma la escasez de recursos necesarios para digitalizar, validar y estructurar este conocimiento mediante expertos.

Mombeu propone una solución basada en una capa narrativa que se integra sobre un LLM existente. En lugar de reemplazar modelos como GPT, Llama o Claude, el sistema aprende cuándo es apropiado enriquecer una respuesta con una referencia a la cultura guaraní utilizando contenido previamente validado por especialistas, manteniendo la naturalidad de la conversación.

_**DATASETS UTILIZADOS:**_

El proceso de entrenamiento combina dos fuentes de información complementarias.

1. Corpus de Cultura Paraguaya

Se construyó un corpus basado en el libro Ñande Ypykuéra, considerado una fuente autoritativa sobre la cosmovisión guaraní. Este corpus contiene relatos, explicaciones, traducciones y descripciones de personajes, tradiciones y elementos propios de la cultura paraguaya y guaraní.

Este conjunto constituye la fuente de conocimiento cultural utilizada durante el entrenamiento.

2. UltraChat en Español

Como fuente de conversaciones generales se utilizó una selección de aproximadamente 30.000 conversaciones en español provenientes del dataset UltraChat.

Estas conversaciones representan interacciones reales que un usuario podría mantener con un asistente conversacional moderno y sirven como contexto donde posteriormente se insertan las continuaciones culturales.

_**METODOLOGÍA:**_

El pipeline completo consta de varias etapas destinadas a construir automáticamente un conjunto de entrenamiento de alta calidad.

1. Filtrado temático

En primer lugar, las conversaciones de UltraChat son clasificadas según su dominio temático (tecnología, ciencia, historia, educación, cultura, geografía, salud, política, entre otros).

Para aumentar la confiabilidad de la clasificación se utilizan dos modelos de clasificación zero-shot:

mDeBERTa_v3_base_xnli_multilingual_nli_2mi17

y

xlm_roberta_large_xnli_anli

Únicamente se conservan aquellas conversaciones en las que ambos modelos coinciden en la categoría asignada. De esta forma se obtiene un subconjunto de conversaciones con alta confianza temática.

2. Búsqueda por similitud semántica

Posteriormente, tanto las respuestas del asistente de UltraChat como los fragmentos culturales del corpus son transformados en embeddings utilizando tres modelos multilingües:

* paraphrase-multilingual-MiniLM-L12-v2

* paraphrase-xlm-r-multilingual-v1

* LaBSE

Para cada conversación se calcula la similitud coseno con todos los fragmentos culturales mediante cada modelo de embeddings. Finalmente, los puntajes obtenidos se promedian para seleccionar el fragmento cultural más relacionado semánticamente con la conversación original.

3. Balanceo del dataset

El número de ejemplos obtenidos por temática resulta altamente desbalanceado. Para evitar que algunas categorías dominen el entrenamiento, se implementó un mecanismo de selección basado en un puntaje compuesto por:

similitud semántica,
relevancia temática,
peso asignado manualmente a cada categoría.

Este procedimiento permite mantener una distribución más uniforme entre dominios y priorizar ejemplos culturalmente relevantes.

A partir de aproximadamente 42.000 candidatos, se seleccionaron cerca de 5.000 ejemplos balanceados para la primera fase del entrenamiento.

4. Generación de continuaciones culturales

Cada ejemplo de entrenamiento no consiste únicamente en anexar información cultural al final de una respuesta.

En su lugar, se sigue un patrón narrativo compuesto por cuatro partes:

* Una frase de transición. Ej.: "Detente un segundo antes de seguir."

* La respuesta original del asistente. Ej.: "El bosque de Camp Clover alberga desde colibríes iridiscentes
hasta águilas que sobrevuelan el lago en busca de peces."


* Un enlace natural hacia la cultura guaraní. Ej.: "Ese canto que llena los árboles tiene un eco
en la **cultura guaraní** que pocos conocen."

* Un fragmento textual extraído literalmente del corpus cultural. Ej.: "El Pájaro de Campana es uno de los pájaros más
raros de nuestros bosques."

Este patrón permite enseñar al modelo cómo introducir referencias culturales de manera orgánica, evitando respuestas forzadas o fuera de contexto.

5. Estrategia de entrenamiento

El entrenamiento se realiza en dos etapas consecutivas.

* Supervised Fine-Tuning (SFT)

En la primera etapa el modelo aprende el patrón narrativo mediante aprendizaje supervisado utilizando el conjunto de ejemplos generados.

La implementación utiliza la librería TRL de Hugging Face junto con SFTTrainer para ajustar el modelo base.

División del conjunto de entrenamiento:

Entrenamiento: 2.700 ejemplos
Validación: 300 ejemplos

* Direct Preference Optimization (DPO)

Una vez obtenido el modelo ajustado mediante SFT, se realiza una segunda etapa utilizando Direct Preference Optimization (DPO).

En esta fase el modelo aprende a preferir las mejores continuaciones culturales a partir de pares de respuestas preferidas y rechazadas, refinando así la calidad narrativa del modelo.

La implementación utiliza DPOTrainer de la librería TRL.

División del conjunto:

Entrenamiento: 1.500 ejemplos

Validación: 200 ejemplos

Test: 300 ejemplos

_**MODELOS APLICADOS:**_

Durante el proyecto se evaluaron distintos modelos de lenguaje livianos (Small Language Models) con el objetivo de identificar cuál aprende mejor el patrón narrativo propuesto.

Los modelos considerados fueron:

* LiquidAI LFM2.5 350M

* LiquidAI LFM2 700M

Los scripts proporcionados utilizan principalmente LiquidAI LFM2.5-350M como modelo base para el entrenamiento.

El entrenamiento se implementa utilizando:

* Hugging Face Transformers
  
* TRL (SFTTrainer y DPOTrainer)
  
* Precisión bfloat16
  
* SDPA (Scaled Dot Product Attention)
  
* Publicación automática de checkpoints en Hugging Face Hub

Esta configuración permite entrenar un modelo compacto capaz de incorporar referencias culturales sin necesidad de utilizar modelos de gran tamaño.

_**RESULTADOS Y MÉTRICAS:**_

En el entrenamiento, las métricas evaluadas fueron:

num_tokens: Número total de tokens procesados hasta el momento.

loss: Pérdida promedio de entropía cruzada (cross-entropy loss) calculada sobre los tokens no enmascarados durante el intervalo de registro (logging) actual.

entropy: Entropía promedio de la distribución de probabilidad de los tokens predichos por el modelo, calculada sobre los tokens no enmascarados.

mean_token_accuracy: Proporción de tokens no enmascarados para los cuales la predicción de mayor probabilidad (top-1) del modelo coincide con el token correcto (ground truth).

La evaluación del modelo busca medir no solamente la calidad lingüística de las respuestas, sino principalmente su capacidad para incorporar correctamente información cultural.

En la evaluación, se implementaron tres métricas:

M0 - la generación debe mencionar 'cultura guaraní' (condición necesaria). Si no aparece, el score final es 0 sin importar M1 y M2.

M1 – Recuperación del fragmento cultural

Evalúa si el modelo logra recuperar correctamente uno de los fragmentos válidos del corpus cultural.

M2 – Coherencia semántica

Evalúa si la continuación cultural mantiene una relación semántica natural con el contexto de la conversación original.

Finalmente, ambas métricas se combinan mediante un puntaje conjunto:

Score Final = M0 x M1 × M2

Este criterio prioriza respuestas que sean simultáneamente correctas desde el punto de vista cultural y coherentes dentro de la conversación.

**BENCHMARK del modelo LiquidAI/LFM2.5-350M:**

Ejemplos evaluados:               300

  _M0  Keyword 'cultura guaraní':    0.987  (296/300)_
  
  _M1  Retrieval rate:               0.880  (264/300)_
  
  _M2  Coherencia (dado M1=1):       0.621_
  
  _Score final (M0 × M1 × M2):      0.542_

Distribución score final:

  0.00-0.25  █████ 38
  
  0.25-0.50  █ 10
  
  0.50-0.75  ████████████████████████████████ 241
  
  0.75-1.00  █ 11

_**VISUALIZACIÓN Y EJEMPLOS:**_

El comportamiento esperado del modelo puede observarse mediante dos escenarios.

❌ Ejemplo incorrecto

Usuario:

¿Cómo puedo crear una promesa en JavaScript?

Aunque el modelo puede generar correctamente una referencia cultural sobre el Pájaro Campana, dicha continuación no guarda ninguna relación con programación, por lo que resulta artificial e innecesaria.

✅ Ejemplo correcto

Usuario:

¿Qué aves puedo ver en el bosque de Camp Clover?

En este caso, el asistente responde primero la consulta sobre las especies presentes en el bosque y posteriormente enlaza naturalmente la conversación con una referencia al Pájaro Campana, incorporando un fragmento auténtico de la cultura guaraní de forma fluida y contextualizada.

_Ejemplos:_

✓ CORRECTO  [idx=68]  score=0.776  M1=1.0  M2=0.776

PROMPT:     Mientras el grupo se adentraba más en los pasillos del carnaval, las sombras parecían volverse más oscuras, el aire más espeso con una presencia ominosa. De repente, escucharon sonidos de arañazos.

GENERACIÓN: Hay un detalle que suele pasar desapercibido cuando observamos cómo los monstruos acechan en la oscuridad, las criaturas demoníacas aparecen contorsionándose con una gracia antinatural mientras cazan a su presa, hasta que los adolescentes están acorralados.

CORPUS HIT: La fealdad, cabellera larga y sucia, palidez mortal y el olor fétido que despedía de Luisón causaba repugnancia y un terror.

✗ INCORRECTO  [idx=263]  score=0.000  M1=0.0  M2=0.576

PROMPT:     Dejo atrás la comodidad del hogar,
Mi corazón duele mientras vagabundeo.
Un extraño en una tierra nueva,
El peso de la soledad pesa en mi mano.

GENERACIÓN: Vale la pena detenerse un segundo a escuchar ese susurro que aparece justo frente a ti cuando todo parece oscurecer el camino. Las calles son extrañas y desagradables, no encuentro caras amigables, donde el mundo parece hostil, duro y cruel.

CORPUS HIT: Veamos lo que significa cada vocablo de Guarasy'ava, Guara significa raíz de la voz guaraní. sy significa madre. ava significa hombre. 

_**CONCLUSIÓN:**_

Mombeu demuestra que es posible especializar modelos de lenguaje de pequeño tamaño para incorporar narrativas culturales mediante una adecuada construcción del dataset y un entrenamiento basado en preferencias. En lugar de memorizar información enciclopédica, el modelo aprende un patrón narrativo que le permite decidir cuándo y cómo introducir referencias a la cultura guaraní dentro de conversaciones cotidianas.

La metodología combina clasificación temática, búsqueda por similitud semántica, modelos de embeddings, balanceo de datos, Supervised Fine-Tuning y Direct Preference Optimization, obteniendo un modelo capaz de generar respuestas culturalmente enriquecidas manteniendo la naturalidad del diálogo.

Como trabajo futuro, el proyecto plantea mejorar los mecanismos de recuperación del fragmento cultural más adecuado, ampliar el corpus con nuevas fuentes validadas por especialistas y realizar evaluaciones humanas sistemáticas que permitan medir tanto la fidelidad cultural como la percepción de naturalidad por parte de los usuarios.
