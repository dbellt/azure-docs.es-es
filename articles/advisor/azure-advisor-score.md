---
title: Optimización de cargas de trabajo de Azure mediante Advisor Score
description: Use Azure Advisor Score para sacar el máximo partido a Azure.
ms.topic: article
ms.date: 09/09/2020
ms.openlocfilehash: e439e5283f2d311d9efb3b30fe157750d589664e
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787930"
---
# <a name="optimize-azure-workloads-by-using-advisor-score"></a>Optimización de cargas de trabajo de Azure mediante Advisor Score

## <a name="introduction-to-advisor-score"></a>Introducción a Advisor Score

Azure Advisor proporciona recomendaciones de procedimientos recomendados para las cargas de trabajo. Estas recomendaciones son personalizadas y accionables para ayudarle a:

* Mejorar la posición de las cargas de trabajo y optimizar las implementaciones de Azure.
* Evitar de forma proactiva los principales problemas gracias a los procedimientos recomendados.
* Evaluar las cargas de trabajo de Azure conforme a los cinco pilares del [Marco de buena arquitectura de Microsoft Azure](/azure/architecture/framework/).

Como característica principal de Advisor, Advisor Score puede ayudarle a lograr estos objetivos de manera eficaz y eficiente.

Para sacar el máximo partido a Azure, es fundamental entender en qué punto del recorrido de optimización de cargas de trabajo se encuentra. Debe saber qué servicios o recursos se consumen correctamente y cuáles no. Además, necesitará saber cómo clasificar por orden de prioridad las acciones, en función de las recomendaciones, para maximizar el resultado.

También es importante realizar un seguimiento de su progreso y que está realizando en este recorrido de optimización, así como presentar informes al respecto. Con Advisor Score puede realizar todas estas acciones fácilmente gracias a la nueva experiencia de ludificación.

Como consultor personalizado en la nube, Azure Advisor evalúa continuamente la telemetría de uso y la configuración de recursos para comprobar que se siguen los procedimientos recomendados del sector. Luego Advisor agrega sus hallazgos en una sola puntuación. Con esta puntuación puede saber de un vistazo si está llevando a cabo los pasos necesarios para compilar soluciones confiables, seguras y rentables.

La puntuación de Advisor consta de una puntuación general que se puede desglosar en cinco puntuaciones de categoría. Una puntuación de cada categoría de Advisor representa los cinco pilares del marco de buena arquitectura.

Puede realizar un seguimiento del progreso que realiza a lo largo del tiempo si visualiza la puntuación general y la de categoría con tendencias diarias, semanales y mensuales. También puede establecer puntos de referencia que le ayuden a alcanzar los objetivos.

 ![Captura de pantalla que muestra la página de Advisor Score.](./media/advisor-score-1.png)

## <a name="interpret-an-advisor-score"></a>Interpretación de una puntuación de Advisor

Advisor muestra la puntuación general y un desglose de categorías de Advisor, en porcentajes. Una puntuación del 100 % en cualquier categoría significa que todos los recursos que evaluó Advisor siguen los procedimientos recomendados de Advisor. En el otro extremo del espectro, una puntuación de 0 % significa que ninguno de los recursos evaluados por Advisor sigue las recomendaciones de este. Con estos intervalos de puntuación puede lograr fácilmente el flujo siguiente:

* **Advisor Score** ayuda a determinar cómo están rindiendo la carga de trabajo o las suscripciones en función de una puntuación de Advisor. También puede ver las tendencias históricas para comprender cuál es la suya.
* **Puntuación por categoría** de cada recomendación indica qué recomendaciones destacadas van a mejorar más la puntuación. Estos valores reflejan el peso de la recomendación y la facilidad de implementación prevista. Estos factores ayudan a asegurarse de sacar el máximo partido al tiempo disponible. También ayudan a priorizar.
* **Impacto de la puntuación de categoría** de cada recomendación ayuda a clasificar por orden de prioridad las acciones de corrección de cada categoría.

La contribución de cada recomendación a la puntuación de la categoría se muestra claramente en la página de **Advisor Score** de Azure Portal. Puede aumentar cada puntuación de categoría el punto porcentual que aparece en la columna **Posible aumento de puntuación**. Este valor refleja el peso de la recomendación dentro de la categoría y la facilidad de implementación prevista para abordar las tareas más sencillas. Centrarse en las recomendaciones con el mayor impacto en la puntuación le ayudará a mejorar el progreso con el tiempo.

![Captura de pantalla que muestra el impacto de la puntuación de Advisor.](./media/advisor-score-2.png)

Si alguna recomendación de Advisor no es relevante para un recurso individual, puede posponer o descartar esa recomendación. Se excluye del cálculo de puntuación con la próxima actualización. Advisor también usará esta entrada como comentario adicional para mejorar el modelo.

## <a name="how-is-an-advisor-score-calculated"></a>¿Cómo se calcula una puntuación de Advisor?

Advisor muestra las puntuaciones de categoría y la puntuación general de Advisor como porcentajes. Una puntuación del 100 % en cualquier categoría significa que todos los recursos *que evaluó Advisor* siguen los procedimientos recomendados de Advisor. En el otro extremo del espectro, una puntuación de 0 % significa que ninguno de los recursos evaluados por Advisor sigue las recomendaciones de este.

**Cada una de las cinco categorías tiene una puntuación potencial máxima de 100.** La puntuación general de Advisor se calcula como la suma de cada puntuación de categoría aplicable, dividida por la suma de la puntuación potencial máxima de todas las categorías aplicables. En la mayoría de las suscripciones, eso significa que Advisor suma la puntuación de cada categoría y la divide por 500. Pero *cada puntuación de categoría se calcula solo si se usan los recursos que evalúa Advisor*.

### <a name="advisor-score-calculation-example"></a>Ejemplo de cálculo de la puntuación de Advisor

* **Puntuación de una sola suscripción:** este ejemplo es la media simple de todas las puntuaciones de categoría de Advisor de la suscripción. Si las puntuaciones de categoría de Advisor son **Costo** = 73, **Confiabilidad** = 85, **Excelencia operativa** = 77 y **Rendimiento** = 100, la puntuación de Advisor sería (73 + 85 + 77 + 100)/(4x100) = 0,84 o 84 %.
* **Puntuación de varias suscripciones:** cuando se seleccionan varias suscripciones, la puntuación general de Advisor generada son las puntuaciones de categoría agregadas ponderadas. Aquí, cada puntuación de la categoría de Advisor se agrega en función de los recursos consumidos por las suscripciones. Una vez que Advisor tiene las puntuaciones de categoría agregadas ponderadas, calcula una media simple para proporcionar una puntuación total de las suscripciones.

### <a name="scoring-methodology"></a>Metodología de puntuación

El cálculo de la puntuación de Advisor se puede resumir en cuatro pasos:

1. Advisor calcula el *costo comercial de los recursos afectados*. Estos recursos son los de las suscripciones que tienen al menos una recomendación en Advisor.
1. Advisor calcula el *costo comercial de los recursos evaluados*. Estos recursos son los supervisados por Advisor, tanto si tienen alguna recomendación como si no.
1. En cada tipo de recomendación, Advisor calcula la *proporción de recursos correcta*. Esta proporción es el costo comercial de los recursos afectados dividido por el costo comercial de los recursos evaluados.
1. Advisor aplica tres pesos adicionales a la proporción de recursos correctos en cada categoría:

   * Las recomendaciones con mayor impacto se ponderan con un peso mayor que las que tienen un impacto menor.
   * Los recursos con recomendaciones duraderas contarán más en la puntuación.
   * Los recursos que se posponen o descartan en Advisor se quitan completamente del cálculo de la puntuación.

Advisor aplica este modelo en un nivel de categoría para proporcionar una puntuación para cada categoría. **Seguridad** usa un modelo de [puntuación de seguridad](../security-center/secure-score-security-controls.md#introduction-to-secure-score). Una media simple genera la puntuación final de Advisor.

## <a name="advisor-score-faqs"></a>Preguntas frecuentes sobre la puntuación de Advisor

### <a name="how-often-is-my-score-refreshed"></a>¿Con qué frecuencia se actualiza la puntuación?

La puntuación se actualiza al menos una vez al día.

### <a name="why-do-some-recommendations-have-the-empty---value-in-the-category-score-impact-column"></a>¿Por qué algunas recomendaciones tienen el valor "-" vacío en la columna de impacto de la puntuación de categoría?

Advisor no incluye inmediatamente las nuevas recomendaciones ni aquellas con cambios recientes en el modelo de puntuación. Tras un breve período de evaluación, normalmente algunas semanas, se incluyen en la puntuación.

### <a name="why-is-the-cost-score-impact-greater-for-some-recommendations-even-if-they-have-lower-potential-savings"></a>¿Por qué el impacto de la puntuación de los costos es mayor en algunas recomendaciones, aunque su potencial de ahorro sea reducido?

La puntuación **Costo** refleja el posible ahorro de recursos infrautilizados y la facilidad de implementación prevista de esas recomendaciones. Por ejemplo, se aplica un peso adicional a los recursos afectados que han estado inactivos durante más tiempo, aunque el ahorro potencial sea menor.

### <a name="why-dont-i-have-a-score-for-one-or-more-categories-or-subscriptions"></a>¿Por qué no tengo una puntuación en una o más categorías o suscripciones?

Advisor genera una puntuación solo para las categorías y suscripciones que tienen recursos evaluados por Advisor.

### <a name="what-if-a-recommendation-isnt-relevant"></a>¿Qué ocurre si una recomendación no es relevante?

Si descarta una recomendación de Advisor, se omitirá en el cálculo de la puntuación. El descarte de recomendaciones también ayuda a Advisor a mejorar la calidad de las recomendaciones.

### <a name="why-did-my-score-change"></a>¿Por qué cambió mi puntuación?

La puntuación puede cambiar si se corrigen los recursos afectados mediante la adopción de los procedimientos recomendados de Advisor. Si usted o cualquier persona con permisos en la suscripción ha modificado o creado nuevos recursos, es posible que también observe fluctuaciones en la puntuación. La puntuación se basa en una proporción entre los recursos afectados por el costo y el costo total de todos los recursos.

### <a name="how-does-advisor-calculate-the-retail-cost-of-resources-on-a-subscription"></a>¿Cómo calcula Advisor el costo minorista de los recursos de una suscripción?

Advisor usa las tarifas de pago por uso publicadas en [Precios de Azure](https://azure.microsoft.com/pricing/). Estas tarifas no reflejan ningún descuento aplicable. Las tarifas se multiplican por la cantidad de uso el último día en que se ha asignado el recurso. La omisión de los descuentos en el cálculo del costo de los recursos hace que la puntuación de Advisor sea similar en todas las suscripciones, los inquilinos y las inscripciones donde los descuentos pueden variar.

### <a name="do-i-need-to-view-the-recommendations-in-advisor-to-get-points-for-my-score"></a>¿Es necesario ver las recomendaciones de Advisor para obtener puntos para la puntuación?

No. La puntuación refleja si adopta los procedimientos recomendados por Advisor, aunque nunca vea esas recomendaciones en Advisor, y si adopta los procedimientos recomendados de manera activa.

### <a name="does-the-scoring-methodology-differentiate-between-production-and-dev-test-workloads"></a>¿La metodología de puntuación distingue entre cargas de trabajo de producción /desarrollo y de pruebas?

No, por ahora no. Pero puede descartar las recomendaciones sobre recursos individuales si esos recursos se usan para desarrollo y pruebas, y la recomendación no se aplica.

### <a name="can-i-compare-scores-between-a-subscription-with-100-resources-and-a-subscription-with-100000-resources"></a>¿Puedo comparar puntuaciones entre una suscripción con 100 recursos y una suscripción con 100 000 recursos?

La metodología de puntuación está diseñada para controlar el número de recursos de una combinación de suscripción y servicio. Las suscripciones con menos recursos pueden tener puntuaciones mayores o menores que las suscripciones con más recursos.

### <a name="what-does-it-mean-when-i-see-coming-soon-in-the-score-impact-column"></a>¿Qué significa "próximamente" en la columna de impacto de la puntuación?

Este mensaje significa que la recomendación es nueva y se está trabajando para incluirla en el modelo de puntuación de Advisor. Una vez que se incluya esta nueva recomendación en un cálculo de puntuación, verá el valor de impacto de la puntuación de la recomendación.

### <a name="does-my-score-depend-on-how-much-i-spend-on-azure"></a>¿Mi puntuación depende de cuánto gasto en Azure?

No. La puntuación no es necesariamente un reflejo de cuánto se gasta. Los gastos innecesarios dan lugar a una puntuación menor **Costo**.

## <a name="access-advisor-score"></a>Acceso a Advisor Score

Advisor Score se encuentra en versión preliminar pública en Azure Portal. En el panel izquierdo, en la sección **Advisor**, vea **Advisor Score**.

![Captura de pantalla que muestra el punto de entrada de Advisor Score.](./media/advisor-score-3.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las recomendaciones de Advisor, consulte:

* [Introducción a Advisor](advisor-overview.md)
* [Introducción a Advisor](advisor-get-started.md)
* [Recomendaciones sobre el costo de Advisor](advisor-cost-recommendations.md)
* [Recomendaciones sobre rendimiento de Advisor](advisor-performance-recommendations.md)
* [Recomendaciones sobre seguridad de Advisor](advisor-security-recommendations.md)
* [Recomendaciones de excelencia operativa de Advisor](advisor-operational-excellence-recommendations.md)
