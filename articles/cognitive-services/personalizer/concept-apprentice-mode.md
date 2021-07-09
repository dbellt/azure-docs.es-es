---
title: 'Modo de aprendiz: Personalizer'
description: Aprenda a usar el modo de aprendiz para obtener confianza en un modelo sin cambiar ningún código.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: 8ef7137460c997b3685e75d5a37b7949fee86255
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111895755"
---
# <a name="use-apprentice-mode-to-train-personalizer-without-affecting-your-existing-application"></a>Uso del modo de aprendiz para el entrenamiento de Personalizer sin que ello afecte a la aplicación existente

Debido a la naturaleza del aprendizaje de refuerzo **real**, un modelo de Personalizer solo se puede entrenar en un entorno de producción. Al implementar un nuevo caso de uso, el modelo de Personalizer no funciona de forma eficaz porque se requiere tiempo para que el modelo esté suficientemente entrenado.  El **modo de aprendiz** es un comportamiento de aprendizaje que simplifica esta situación y le permite aumentar la confianza en el modelo, sin que el desarrollador cambie nada de código.

[!INCLUDE [Important Blue Box - Apprentice mode pricing tier](./includes/important-apprentice-mode.md)]

## <a name="what-is-apprentice-mode"></a>¿Qué es el modo de aprendiz?

Es similar a la forma en que un aprendiz aprende de un patrón y con experiencia puede mejorar. El modo de aprendiz es un _comportamiento_ que permite a Personalizer aprender observando los resultados obtenidos de la lógica de la aplicación existente.

Personalizer entrena mediante la imitación de la misma salida que la aplicación. A medida que fluyen los eventos, Personalizer puede _ponerse al día_ con la aplicación existente sin que ello afecte a la lógica y los resultados existentes. Las métricas, disponibles en Azure Portal y la API, le ayudan a comprender el rendimiento a medida que el modelo aprende.

Una vez que Personalizer ha aprendido y alcanzado cierto nivel de comprensión, el desarrollador puede cambiar el comportamiento del modo de aprendiz al modo en línea. En ese momento, Personalizer comienza a influir en las acciones de Rank API.

## <a name="purpose-of-apprentice-mode"></a>Propósito del modo de aprendiz

El modo de aprendiz le proporciona confianza en el servicio Personalizer y sus capacidades de Machine Learning, y garantiza que el servicio recibe información de la que se puede aprender, sin poner en peligro el tráfico en línea.

Las dos razones principales para usar el modo de aprendiz son las siguientes:

* Mitigación de los **arranques en frío**: El modo de aprendiz ayuda a administrar y evaluar el costo del tiempo de aprendizaje de un "nuevo" modelo (cuando no se devuelve la mejor acción y no se logra un nivel de efectividad satisfactorio alrededor del 60-80 %).
* **Validación de las características de acción y contexto**: las características enviadas en acciones y en el contexto pueden ser inadecuadas o inexactas (escasas, excesivas, incorrectas o demasiado específicas para entrenar Personalizer para alcanzar el grado de eficacia ideal). Use las [evaluaciones de características](concept-feature-evaluation.md) para buscar y corregir problemas con las características.

## <a name="when-should-you-use-apprentice-mode"></a>¿Cuándo se debe usar el modo de aprendiz?

Use el modo de aprendiz para entrenar Personalizer para mejorar su eficacia en los siguientes escenarios, a la vez que impide que Personalizer afecte a la experiencia de los usuarios:

* Está implementando Personalizer en un caso de uso nuevo.
* Ha cambiado significativamente las características que envía en contexto o acciones.
* Ha cambiado significativamente cuándo y cómo se calculan las recompensas.

El modo de aprendiz no es una manera eficaz de medir el impacto de Personalizer en las puntuaciones de recompensa. Para medir el grado de eficacia de Personalizer en la elección de la mejor acción posible para cada llamada de Rank, use [evaluaciones sin conexión](concepts-offline-evaluation.md).

## <a name="who-should-use-apprentice-mode"></a>¿Quién debe usar el modo de aprendiz?

El modo de aprendiz es útil para desarrolladores, científicos de datos y responsables de la toma de decisiones empresariales:

* Los **desarrolladores** pueden usar el modo de aprendiz para garantizar que Rank API y Reward API se usan correctamente en la aplicación, y que las características que se envían a Personalizer desde la aplicación no contienen errores ni características no pertinentes, como una marca de tiempo o un elemento UserID.

* Los **científicos de datos** pueden usar el modo de aprendiz para comprobar que las características son eficaces para entrenar los modelos de Personalizer y que los tiempos de espera de recompensas no son demasiado largos o cortos.

* Los **responsables de la toma de decisiones empresariales** pueden usar el modo de aprendiz para evaluar el potencial de Personalizer para mejorar los resultados (es decir, las recompensas) en comparación con la lógica de negocios existente. Esto les permite tomar una decisión informada que afecta a la experiencia del usuario, donde los ingresos reales y la satisfacción del usuario están en juego.

## <a name="comparing-behaviors---apprentice-mode-and-online-mode"></a>Comparación de comportamientos: modo de aprendiz y modo en línea

Aprendizaje cuando el modo de aprendiz difiere del modo en línea de las siguientes maneras.

|Área|Modo de aprendiz|Modo en línea|
|--|--|--|
|Impacto en la experiencia del usuario|Puede usar el comportamiento del usuario existente para el entrenamiento de Personalizer. Para ello, permita que observe (sin alterar) cuál habría sido su **acción predeterminada** y la recompensa obtenida. Esto significa que la experiencia de los usuarios y sus resultados empresariales no se verán afectados.|Muestra la acción superior devuelta desde la llamada a Rank para afectar al comportamiento del usuario.|
|Velocidad de aprendizaje|Personalizer aprenderá más lentamente en el modo de aprendiz que en el modo en línea. El modo de aprendiz solo puede aprender observando las recompensas que ha obtenido la **acción predeterminada**, lo que limita la velocidad de aprendizaje, ya que no se puede realizar ninguna exploración.|Aprende más rápido porque puede aprovechar el modelo actual y explorar nuevas tendencias.|
|"Tope" de eficacia del aprendizaje|Personalizer puede aproximarse, muy pocas veces igualar y nunca superar el rendimiento de la lógica de negocios base (el total de recompensas logrado por la **acción predeterminada** de cada llamada a Rank). Este "tope" de aproximación se reduce mediante la exploración. Por ejemplo, con la exploración en un 20%, es muy poco probable que el rendimiento del modo de aprendiz supere el 80% y el 60% es un objetivo razonable en el que pasar al modo en línea.|Personalizer debe superar las aplicaciones de línea de base y, con el tiempo, cuando se detenga, debe realizar una evaluación sin conexión y una evaluación de características para continuar obteniendo mejoras para el modelo. |
|Valor de Rank API para rewardActionId|La experiencia de los usuarios no se ve afectada, ya que _rewardActionId_ siempre es la primera acción que se envía en la solicitud Rank. En otras palabras, Rank API no hace nada visible para la aplicación durante el modo de aprendiz. Las Reward API de la aplicación no deben cambiar el modo en que se usa Reward API entre un modo y otro.|La experiencia de los usuarios se modificará en función del valor _rewardActionId_ que Personalizer elija para la aplicación. |
|Evaluaciones|Personalizer mantiene una comparación de los totales de recompensas que la lógica de negocios predeterminada obtiene y los totales de recompensas que Personalizer obtendría en el modo en línea en ese momento. Existe una comparación disponible en Azure Portal para ese recurso.|Evalúe la eficacia de Personalizer mediante [evaluaciones sin conexión](concepts-offline-evaluation.md), que le permiten comparar las recompensas totales que Personalizer ha obtenido con las recompensas potenciales de la línea de base de la aplicación.|

Nota sobre la eficacia del modo de aprendiz:

* La eficacia de Personalizer en el modo de aprendiz no suele acercarse al 100 % de la línea de base de la aplicación y nunca lo supera.
* Los procedimientos recomendados serían no intentar llegar al 100 %. El objetivo sería un intervalo del 60 al 80 % en función del caso de uso.

## <a name="limitations-of-apprentice-mode"></a>Limitaciones del modo de aprendiz
El modo de aprendiz entrena el modelo de Personalizer mediante la imitación del algoritmo existente que elige elementos de línea de base, con las características presentes en el contexto y las acciones usadas en las llamadas de Rank y los comentarios de las llamadas de Reward. Los siguientes factores afectarán a si el aprendiz de Personalizer aprende suficientes recompensas coincidentes o cuándo.

### <a name="scenarios-where-apprentice-mode-may-not-be-appropriate"></a>Escenarios en los que el modo de aprendiz puede que no sea adecuado:

* **Contenido elegido por la editorial**: en algunos escenarios, como los de noticias o entretenimiento, un equipo editorial podría asignar manualmente el elemento de la línea de base. Esto significa que personas humanas usan sus conocimientos más amplios sobre el mundo, y la comprensión de lo que puede ser un contenido atractivo, para elegir artículos o contenidos multimedia específicos de un grupo y marcarlos como artículos "preferidos" o "prominentes". Dado que estos editores no son un algoritmo y los factores que tienen en cuenta los editores pueden matizarse y no incluirse como características del contexto y las acciones, es poco probable que el modo de aprendiz pueda predecir la siguiente acción de la línea de base. En estas situaciones puede: ** Probar Personalizer en modo en línea: el hecho de que el modo de aprendiz no prediga líneas de base no implica que Personalizer no pueda lograr resultados igual de buenos o incluso mejores. Considere la posibilidad de poner Personalizer en modo en línea durante un período de tiempo o en una prueba A/B si dispone de la infraestructura y, a continuación, ejecute una evaluación sin conexión para evaluar la diferencia.
** Agregar consideraciones editoriales y recomendaciones como características: pregunte a los editores qué factores influyen en sus elecciones y vea si puede agregarlos como características en su contexto y acciones. Por ejemplo, los editores de una empresa de contenidos multimedia pueden destacar un contenido mientras una determinada celebridad está en las noticias: este conocimiento se podría agregar como una característica de contexto.

### <a name="factors-that-will-improve-and-accelerate-apprentice-mode"></a>Factores que mejorarán y acelerarán el modo de aprendiz
Si el modo de aprendiz está aprendiendo y obteniendo recompensas coincidentes por encima de cero, pero parece que va creciendo lentamente (no llega al 60 % u 80 % de recompensas coincidentes en 2 semanas), es posible que el desafío disponga de muy pocos datos. La adopción de los pasos siguientes podría acelerar el aprendizaje. 

1. Agregar más eventos con recompensas positivas a lo largo del tiempo: el modo de aprendiz funciona mejor en aquellos casos de uso en los que la aplicación obtiene más de 100 recompensas positivas al día. Por ejemplo, si un sitio web que recompensa un clic tiene un 2 % de tasa de clickthrough, debe tener al menos 5000 visitas al día para tener un aprendizaje perceptible. También puede experimentar con una recompensa que sea más sencilla y que se produzca con más frecuencia. Por ejemplo, pasar de "¿Terminaron los usuarios de leer el artículo?" a "¿Empezaron los usuarios a leer el artículo?".
2. Agregar características diferenciadoras: puede realizar una inspección visual de las acciones en una llamada de Rank y sus características. ¿La acción de la línea de base tiene características que se diferencian de otras acciones? Si son muy parecidas, agregue más características que las hagan menos semejantes.
3. Reducción de acciones por evento: Personalizer usará la opción Explorar % para detectar preferencias y tendencias. Cuando una llamada a Rank tiene más acciones, la posibilidad de que se elija una acción para la exploración es menor. Reduzca el número de acciones enviadas en cada llamada de Rank a un número menor, a menos de 10. Puede ser un ajuste temporal para mostrar que el modo de aprendiz tiene los datos adecuados para que coincidan con las recompensas.



## <a name="using-apprentice-mode-to-train-with-historical-data"></a>Uso del modo de aprendiz para el entrenamiento con datos históricos

Si tiene una cantidad significativa de datos históricos, puede usarlos para el entrenamiento de Personalizer. Puede usar el modo de aprendiza para reproducir los datos a través de Personalizer.

Configure Personalizer en el modo de aprendiz y cree un script que llame a Rank con las características de acciones y de contexto de los datos históricos. Llame a Reward API en función de los cálculos de los registros de estos datos. Necesitará aproximadamente 50 000 eventos históricos para ver algunos resultados, pero se recomiendan 500 000 para una mayor confianza en los resultados.

Al realizar el entrenamiento a partir de datos históricos, se recomienda que los datos enviados (características de contexto y acciones, su diseño en el archivo JSON usado para las solicitudes de Rank y el cálculo de la recompensa en este conjunto de datos de aprendizaje) coincidan con los datos (características y cálculo de recompensas) disponibles de la aplicación existente.

Los datos sin conexión y post facto suelen ser más incompletos, presentan más ruido y se diferencian en el formato. Aunque el entrenamiento a partir de datos históricos es posible, los resultados de hacerlo pueden ser no concluyentes y no predecir correctamente el grado de aprendizaje de Personalizer, sobre todo si las características varían entre los datos pasados y la aplicación existente.

Normalmente, para Personalizer, en comparación con el entrenamiento con datos históricos, el cambio de comportamiento al modo de aprendiz y el aprendizaje a partir de una aplicación existente es una vía más eficaz para tener un modelo eficaz, con menos trabajo, ingeniería de datos y trabajo de limpieza.

## <a name="using-apprentice-mode-versus-ab-tests"></a>Uso del modo de aprendiz frente a las pruebas A/B

Solo es útil realizar pruebas A/B de los tratamientos de Personalizer una vez que se ha validado y está aprendiendo en el modo en línea. En el modo de aprendiz, solo se usa la **acción predeterminada**, lo que significa que todos los usuarios verán la experiencia de control de forma eficaz.

Aunque Personalizar sea solo el _tratamiento_, el mismo desafío se presenta cuando la validación de los datos es adecuada para el entrenamiento de Personalizer. En su lugar, se podría usar el modo de aprendiz, con un 100 % de tráfico y donde todos los usuarios obtengan la experiencia de control (sin alteraciones).

Cuando tiene un caso de uso que utiliza Personalizer y el aprendizaje en línea, los experimentos A/B le permiten realizar cohortes controladas y una comparación científica de los resultados, que pueden ser más complejos que las señales usadas para las recompensas. Una pregunta de ejemplo a la que podría responder una prueba A/B es la siguiente: `In a retail website, Personalizer optimizes a layout and gets more users to _check out_ earlier, but does this reduce total revenue per transaction?`

## <a name="next-steps"></a>Pasos siguientes

* Obtener información sobre [eventos activos e inactivos](concept-active-inactive-events.md)
