---
title: Puntuación de seguridad de Azure Security Center
description: Descripción de Puntuación de seguridad de Azure Security Center y sus controles de seguridad
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: article
ms.date: 05/05/2021
ms.author: memildin
ms.openlocfilehash: 9879b5154866436e3216243b123b3575e6cd3df4
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754926"
---
# <a name="secure-score-in-azure-security-center"></a>Puntuación de seguridad de Azure Security Center

## <a name="introduction-to-secure-score"></a>Introducción a Puntuación de seguridad

Azure Security Center tiene dos objetivos principales: 

- ayudarle a entender su situación de seguridad actual
- ayudarle a mejorar la seguridad de forma eficaz

La característica principal de Security Center que le ayuda a conseguir estos objetivos es la **puntuación de seguridad**.

Security Center evalúa continuamente los recursos, suscripciones y la organización en busca de problemas de seguridad. A continuación, agrega todos los resultados a una sola puntuación para que pueda conocer de un vistazo la situación de la seguridad actual: cuanto mayor sea la puntuación, menor será el nivel de riesgo identificado.

La puntuación de seguridad se muestra en las páginas de Azure Portal como un valor de porcentaje, pero los valores subyacentes también se presentan claramente:

:::image type="content" source="./media/secure-score-security-controls/single-secure-score-via-ui.png" alt-text="Puntuación de seguridad general como se muestra en el portal":::

Para aumentar la seguridad, revise la página de recomendaciones de Security Center en busca de las acciones pendientes necesarias para aumentar la puntuación. Cada recomendación incluye instrucciones para ayudarle a corregir el problema específico.

Las recomendaciones se agrupan en **controles de seguridad**. Cada control es un grupo lógico de recomendaciones de seguridad relacionadas y refleja las superficies de ataque vulnerables. La puntuación solo mejora cuando corrige *todas* las recomendaciones para un solo recurso de un control. Para ver cómo protege su organización cada todas y cada una de las superficies expuesta a ataques, revise las puntuaciones de cada control de seguridad.

Para más información, consulte [Cálculo de la puntuación de seguridad](secure-score-security-controls.md#how-your-secure-score-is-calculated) a continuación. 

## <a name="how-your-secure-score-is-calculated"></a>Cálculo de la puntuación de seguridad 

La contribución de cada control de seguridad en la puntuación de seguridad total aparece con claridad en la página de recomendaciones.

:::image type="content" source="./media/secure-score-security-controls/security-controls.png" alt-text="Controles de Azure Security Center y su impacto en la puntuación segura" lightbox="./media/secure-score-security-controls/security-controls.png":::

Para obtener todos los puntos posibles de un control de seguridad, todos sus recursos deben cumplir todas las recomendaciones de seguridad de dicho control. Por ejemplo, Security Center tiene varias recomendaciones sobre cómo proteger los puertos de administración. Deberá corregirlas todas para que la puntuación de seguridad mejore.

### <a name="example-scores-for-a-control"></a>Puntuaciones de ejemplo para un control

:::image type="content" source="./media/secure-score-security-controls/remediate-vulnerabilities-control.png" alt-text="Control de seguridad Aplicar actualizaciones del sistema" lightbox="./media/secure-score-security-controls/remediate-vulnerabilities-control.png":::


En este ejemplo:

| #  | Nombre                                           | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                |
|:-:|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | **Control de seguridad Corregir vulnerabilidades** | Este control agrupa varias recomendaciones relacionadas con el descubrimiento y la resolución de vulnerabilidades conocidas.                                                                                                                                                                                                                                                                                                                                   |
| 2 | **Puntuación máxima**                                  | El número máximo de puntos que puede obtener al completar todas las recomendaciones de un control. La puntuación máxima de un control indica la importancia relativa de ese control y es fija para cada entorno. Use los valores de puntuación máxima para evaluar la prioridad de los problemas.<br>Para obtener una lista de todos los controles y sus puntuaciones máximas, consulte [Controles de seguridad y sus recomendaciones](#security-controls-and-their-recommendations). |
| 3 | **Número de recursos**                        | Este control afecta a 35 recursos.<br>Para comprender la posible contribución de cada recurso, divida la puntuación máxima entre el número de recursos.<br>En este ejemplo, 6/35=0,1714<br>**Cada recurso aporta 0,1714 puntos**.                                                                                                                                                                                          |
| 4 | **Puntuación actual**                              | La puntuación actual de este control.<br>Puntuación actual=[Puntuación por recurso]*[Número de recursos en buen estado]<br> 0,1714 x 5 recursos con estado correcto = 0,86<br>Cada control contribuye a la puntuación total. En este ejemplo, el control contribuye con 0,86 puntos a la puntuación total de seguridad actual.                                                                                                                                               |
| 5 | **Mejora potencial de puntuación**                   | Los puntos restantes disponibles para el usuario en el control. Si corrige todas las recomendaciones de este control, la puntuación aumentará en un 9 %.<br>Mejora potencial de puntuación=[Puntuación por recurso]*[Número de recursos incorrectos]<br> 0,1714 x 30 recursos con estado incorrecto = 5,14<br>                                                                                                                                                        |
|   |                                                |                                                                                                                                                                                                                                                                                                                                                                                                                                            |



### <a name="calculations---understanding-your-score"></a>Cálculos: Descripción de la puntuación

|Métrica|Fórmula y ejemplo|
|-|-|
|**Puntuación actual del control de seguridad**|<br>![Ecuación para calcular la puntuación de un control de seguridad](media/secure-score-security-controls/secure-score-equation-single-control.png)<br><br>Cada control de seguridad individual contribuye a la puntuación de seguridad. Cada recurso afectado por una recomendación dentro del control contribuye a la puntuación actual de este. La puntuación actual de cada control es una medida del estado de los recursos que están *dentro* del control.<br>![Información sobre herramientas que muestra los valores usados al calcular la puntuación actual del control de seguridad](media/secure-score-security-controls/security-control-scoring-tooltips.png)<br>En este ejemplo, la puntuación máxima de 6 se divide entre 78, ya que esta es la suma de los recursos correctos e incorrectos.<br>6 / 78 = 0,0769<br>La multiplicación de esa cifra por el número de recursos correctos (4) da como resultado la puntuación actual:<br>0.0769 * 4 = **0.31**<br><br>|
|**Puntuación segura**<br>Suscripción única|<br>![Ecuación para calcular la puntuación de seguridad de una suscripción](media/secure-score-security-controls/secure-score-equation-single-sub.png)<br><br>![Puntuación de seguridad de una suscripción única con todos los controles habilitados](media/secure-score-security-controls/secure-score-example-single-sub.png)<br>En este ejemplo, hay una suscripción única con todos los controles de seguridad disponibles (una puntuación máxima posible de 60 puntos). La puntuación muestra 28 puntos de los 60 posibles y los 32 puntos restantes se reflejan en las cifras de "Posible aumento de puntuación" de los controles de seguridad.<br>![Lista de controles y posible aumento de puntuación](media/secure-score-security-controls/secure-score-example-single-sub-recs.png)|
|**Puntuación segura**<br>Varias suscripciones|<br>![Ecuación para calcular la puntuación de seguridad de varias suscripciones](media/secure-score-security-controls/secure-score-equation-multiple-subs.png)<br><br>Al calcular la puntuación combinada para varias suscripciones, Security Center incluye una *ponderación* para cada suscripción. Las ponderaciones relativas de las suscripciones se determinan mediante Security Center en función de factores como el número de recursos.<br>La puntuación actual de cada suscripción se calcula de la misma manera que para una sola suscripción, pero la ponderación se aplica como se muestra en la ecuación.<br>Cuando se ven varias suscripciones, la puntuación de seguridad evalúa todos los recursos de todas las directivas habilitadas y agrupa su impacto combinado en la puntuación máxima de cada control de seguridad.<br>![Puntuación de seguridad de varias suscripciones con todos los controles habilitados](media/secure-score-security-controls/secure-score-example-multiple-subs.png)<br>La puntuación combinada **no** es un promedio; en vez de eso, es la posición evaluada del estado de todos los recursos de todas las suscripciones.<br>También en este caso, si va a la página de recomendaciones y suma los puntos posibles disponibles, observará que esta es la diferencia entre la puntuación actual (24) y la puntuación máxima disponible (60).|


### <a name="which-recommendations-are-included-in-the-secure-score-calculations"></a>¿Qué recomendaciones se incluyen en los cálculos de la puntuación de seguridad?

Solo las recomendaciones integradas afectan a la puntuación de seguridad.

Las recomendaciones marcadas como **Versión preliminar** no se incluyen en los cálculos de la puntuación de seguridad. Aun así, deben corregirse siempre que sea posible, de tal forma que, cuando finalice el período de versión preliminar, contribuyan a la puntuación.

Un ejemplo de una recomendación en versión preliminar:

:::image type="content" source="./media/secure-score-security-controls/example-of-preview-recommendation.png" alt-text="Recomendación con la marca de versión preliminar":::

## <a name="improve-your-secure-score"></a>Mejora de su puntuación de seguridad

Para mejorar la puntuación de seguridad, corrija las recomendaciones de seguridad de la lista de recomendaciones. Puede corregir cada recomendación manualmente para cada recurso o bien usar la opción **Corregir** (cuando esté disponible) para resolver rápidamente un problema en varios recursos. Para más información, consulte [Corrección de recomendaciones](security-center-remediate-recommendations.md).

Otra forma de mejorar la puntuación y de asegurarse de que los usuarios no creen recursos que afecten negativamente a la puntuación es configurar las opciones de aplicar y denegar en las recomendaciones pertinentes. Más información en [Evitar errores de configuración con Aplicar/Denegar](prevent-misconfigurations.md).

## <a name="security-controls-and-their-recommendations"></a>Controles de seguridad y sus recomendaciones

En la tabla siguiente se enumeran los controles de seguridad de Azure Security Center. Para cada control, puede ver el número máximo de puntos que puede sumar a la puntuación de seguridad si corrige *todas* las recomendaciones enumeradas en el control, para *todos* los recursos. 

El conjunto de recomendaciones de seguridad proporcionadas con Security Center se adapta a los recursos disponibles en el entorno de cada organización. Las recomendaciones se pueden personalizar aún más [deshabilitando directivas](tutorial-security-policy.md#disable-security-policies-and-disable-recommendations) y [excluyendo recursos específicos de una recomendación](exempt-resource.md). 
 
Se recomienda que cada organización revise cuidadosamente sus iniciativas de Azure Policy asignadas. 

> [!TIP]
> Para obtener información detallada sobre cómo revisar y editar iniciativas, vea [Trabajo con directivas de seguridad](tutorial-security-policy.md). 

Aunque la iniciativa de seguridad predeterminada de Security Center se basa en los estándares y procedimientos recomendados del sector, hay escenarios en los que es posible que las recomendaciones integradas que se indican a continuación no se ajusten completamente a su organización. Por lo tanto, a veces será necesario ajustar la iniciativa predeterminada, sin comprometer la seguridad, para asegurarse de que está alineada con las directivas propias de su organización, así como con estándares del sector, estándares normativos y pruebas comparativas que tiene la obligación de cumplir.<br><br>
<div class="foo">

<style type="text/css"> .tg  {border-collapse:collapse;border-spacing:0;} .tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px; overflow:hidden;padding:10px 5px;word-break:normal;} .tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:18px; font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;} .tg .tg-cly1{text-align:left;vertical-align:middle} .tg .tg-lboi{border-color:inherit;text-align:left;vertical-align:middle} </style>

[!INCLUDE [security-center-controls-and-recommendations](../../includes/asc/security-control-recommendations.md)]

</div>




## <a name="secure-score-faq"></a>Preguntas frecuentes sobre la puntuación de seguridad

### <a name="if-i-address-only-three-out-of-four-recommendations-in-a-security-control-will-my-secure-score-change"></a>Si solo se corrigen tres de las cuatro recomendaciones de un control de seguridad, ¿cambiará mi puntuación de seguridad?
No. No cambiará hasta que corrija todas las recomendaciones para un único recurso. Para obtener la puntuación máxima de un control, debe corregir todas las recomendaciones de todos los recursos.

### <a name="if-a-recommendation-isnt-applicable-to-me-and-i-disable-it-in-the-policy-will-my-security-control-be-fulfilled-and-my-secure-score-updated"></a>Si una recomendación no es aplicable a mí y la deshabilito en la directiva, ¿se cumplirá mi control de seguridad y se actualizará mi puntuación de seguridad?
Sí. Se recomienda deshabilitar las recomendaciones cuando no son aplicables a su entorno. Para obtener instrucciones sobre cómo deshabilitar una recomendación específica, consulte [Deshabilitar las directivas de seguridad](./tutorial-security-policy.md#disable-security-policies-and-disable-recommendations).

### <a name="if-a-security-control-offers-me-zero-points-towards-my-secure-score-should-i-ignore-it"></a>Si un control de seguridad ofrece cero puntos a mi puntuación de seguridad, ¿debería ignorarlo?
En algunos casos verá una puntuación máxima del control mayor que cero, pero el impacto es nulo. Cuando la puntuación incremental para corregir recursos es insignificante, se redondea a cero. Aún así, no ignore estas recomendaciones ya que pueden aportarle mejoras en seguridad. La única excepción es el control de "procedimiento recomendado adicional". La corrección de estas recomendaciones no aumentará la puntuación, pero mejorará la seguridad en general.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se describe la puntuación de seguridad y los controles de seguridad que presenta. Para obtener material relacionado, consulte los siguientes artículos:

- [Más información sobre los distintos elementos de una recomendación](security-center-recommendations.md)
- [Recomendaciones de corrección](security-center-remediate-recommendations.md)
- [Ver las herramientas basadas en GitHub para trabajar mediante programación con la puntuación de seguridad](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score)


> [!div class="nextstepaction"]
> [Acceso y seguimiento de la puntuación segura](secure-score-access-and-track.md)