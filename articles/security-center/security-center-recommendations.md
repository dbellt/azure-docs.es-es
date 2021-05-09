---
title: Recomendaciones de seguridad en el Centro de seguridad de Azure
description: Este documento explica cómo las recomendaciones del Centro de seguridad de Azure ayudan a proteger los recursos de Azure y a cumplir con las directivas de seguridad.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 01/24/2021
ms.author: memildin
ms.openlocfilehash: da0826a349131d9d5716932abfc603be8103b788
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107903549"
---
# <a name="review-your-security-recommendations"></a>Examen de las recomendaciones de seguridad

En este tema se explica cómo ver y entender las recomendaciones de Azure Security Center para ayudar a proteger los recursos de Azure.

## <a name="monitor-recommendations"></a>Supervisión de recomendaciones<a name="monitor-recommendations"></a>

Security Center analiza el estado de seguridad de los recursos para identificar posibles vulnerabilidades. 

1. En el menú de Security Center, abra la página **Recomendaciones** para ver las recomendaciones aplicables a su entorno. Las recomendaciones se agrupan en controles de seguridad.

    :::image type="content" source="./media/security-center-recommendations/view-recommendations.png" alt-text="Recomendaciones agrupadas por control de seguridad" lightbox="./media/security-center-recommendations/view-recommendations.png":::

1. Para encontrar recomendaciones específicas para el tipo de recurso, la gravedad, el entorno u otros criterios que sean importantes para usted, utilice los filtros opcionales que se encuentran encima de la lista de recomendaciones.

    :::image type="content" source="media/security-center-recommendations/recommendation-list-filters.png" alt-text="Filtros para refinar la lista de recomendaciones de Azure Security Center":::

1. Expanda un control y seleccione una recomendación concreta para ver la página de detalles de las recomendaciones.

    :::image type="content" source="./media/security-center-recommendations/recommendation-details-page.png" alt-text="Página de detalles de recomendación" lightbox="./media/security-center-recommendations/recommendation-details-page.png":::

    La página incluye:

    1. En el caso de las recomendaciones admitidas, en la barra de navegación superior se muestran algunos o todos los botones siguientes:
        - **Aplicar** y **Denegar** (vea [Evitar errores de configuración con las recomendaciones Aplicar o Denegar](prevent-misconfigurations.md)).
        - **View policy definition** (Ver definición de directiva) para ir directamente a la entrada Azure Policy de la directiva subyacente.
    1. **Indicador de gravedad**
    1. **Intervalo de actualización** (si procede).
    1. **Recuento de recursos exentos** si existen exenciones para esta recomendación; se muestra el número de recursos exentos.
    1. **Descripción**: una breve descripción del problema
    1. **Pasos de corrección**: una descripción de los pasos manuales necesarios para corregir el problema de seguridad en los recursos afectados Para encontrar recomendaciones con la opción **Corregir**, puede seleccionar **Ver lógica de corrección** antes de aplicar la corrección sugerida a los recursos.
    1. **Recursos afectados**: los recursos se agrupan en pestañas:
        - **Recursos con estado correcto**: recursos relevantes que no se ven afectados o en los que ya se ha corregido el problema.
        - **Recursos con estado incorrecto**: recursos que todavía se ven afectados por el problema identificado.
        - **Recursos no aplicables**: recursos para los que la recomendación no puede dar una respuesta definitiva. La pestaña no aplicable también incluye las razones de cada recurso. 

            :::image type="content" source="./media/security-center-recommendations/recommendations-not-applicable-reasons.png" alt-text="Los recursos no aplicables con motivos":::.
    1. Botones de acción para corregir la recomendación o desencadenar una aplicación lógica.

## <a name="preview-recommendations"></a>Recomendaciones de la versión preliminar

Las recomendaciones marcadas como **Versión preliminar** no se incluyen en los cálculos de la puntuación de seguridad.

Aun así, deben corregirse siempre que sea posible, de tal forma que, cuando finalice el período de versión preliminar, contribuyan a la puntuación.

Un ejemplo de una recomendación en versión preliminar:

:::image type="content" source="./media/secure-score-security-controls/example-of-preview-recommendation.png" alt-text="Recomendación con la marca de versión preliminar":::
 
## <a name="next-steps"></a>Pasos siguientes

En este documento, se han presentando las recomendaciones de seguridad del Centro de seguridad. Para obtener información relacionada, visite:

- [Recomendaciones de corrección ](security-center-remediate-recommendations.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
- [Evitar errores de configuración con las recomendaciones Aplicar o Denegar](prevent-misconfigurations.md).
- [Creación de respuestas automáticas a alertas y recomendaciones con automatización del flujo de trabajo](workflow-automation.md): automatice las respuestas a las recomendaciones
- [Exclusión de un recurso de una recomendación](exempt-resource.md)
- [Guía de referencia sobre las recomendaciones de seguridad](recommendations-reference.md)