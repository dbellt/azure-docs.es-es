---
title: Seguimiento de la puntuación de seguridad en Azure Security Center
description: Obtenga información sobre las distintas maneras de acceder a su puntuación de seguridad y hacerle un seguimiento en Azure Security Center.
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 02/25/2021
ms.openlocfilehash: 50ab34a73a7709a47efdf6904acf005cd5db9da5
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753600"
---
# <a name="access-and-track-your-secure-score"></a>Acceso y seguimiento de la puntuación segura

Encontrará la puntuación de seguridad general, así como la puntuación por suscripción, mediante Azure Portal o mediante programación, tal como se describe en las secciones siguientes:

> [!TIP]
> Para obtener una explicación detallada de cómo se calculan las puntuaciones, consulte [Cálculos: Descripción de la puntuación](secure-score-security-controls.md#calculations---understanding-your-score).

## <a name="get-your-secure-score-from-the-portal"></a>Obtención de la puntuación de seguridad desde el portal

Security Center muestra la puntuación de forma destacada en el portal: es el primer icono principal de la página de información general de Security Center. Al seleccionar este icono, irá a la página dedicada de puntuación de seguridad, donde verá la puntuación dividida por suscripciones. Seleccione una suscripción única para ver la lista detallada de las recomendaciones prioritarias y del posible impacto que su corrección tendrá en la puntuación de la suscripción. 

En resumen, la puntuación de seguridad se muestra en las siguientes ubicaciones en las páginas del portal de Security Center.

- En un icono en la página **Información general** (panel principal) de Security Center:

    :::image type="content" source="./media/secure-score-security-controls/score-on-main-dashboard.png" alt-text="Puntuación de seguridad en el panel de Security Center":::

- En la página dedicada **Puntuación de seguridad**, puede ver la puntuación de seguridad de la suscripción y los grupos de administración:

    :::image type="content" source="./media/secure-score-security-controls/score-on-dedicated-dashboard.png" alt-text="Puntuación de seguridad de las suscripciones en la página de puntuación de seguridad de Security Center":::

    :::image type="content" source="./media/secure-score-security-controls/secure-score-management-groups.png" alt-text="Puntuación de seguridad de los grupos de administración en la página de puntuación de seguridad de Security Center":::

    > [!NOTE]
    > Los grupos de administración para los que no tenga permisos suficientes mostrarán su puntuación como "Restringido". 

- En la parte superior de la página **Recomendaciones**:

    :::image type="content" source="./media/secure-score-security-controls/score-on-recommendations-page.png" alt-text="Puntuación de seguridad en la página de recomendaciones de Security Center":::

## <a name="get-your-secure-score-from-the-rest-api"></a>Obtención de la puntuación de seguridad desde la API de REST

Puede acceder a la puntuación a través de la API de puntuación segura. Los métodos de API proporcionan la flexibilidad necesaria para consultar los datos y crear su propio mecanismo de creación de informes de las puntuaciones seguras a lo largo del tiempo. Por ejemplo, puede usar [Secure Scores](/rest/api/securitycenter/securescores) API para obtener la puntuación de una suscripción específica. Además, puede usar [Secure Score Controls API](/rest/api/securitycenter/securescorecontrols) para mostrar los controles de seguridad y la puntuación actual de las suscripciones.

![Recuperación de una sola puntuación de seguridad a través de la API](media/secure-score-security-controls/single-secure-score-via-api.png)

Para ver ejemplos de herramientas basadas en la API de puntuación de seguridad, consulte el [área de puntuación de seguridad de la comunidad de GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score). 

## <a name="get-your-secure-score-from-azure-resource-graph"></a>Obtención de la puntuación de seguridad desde Azure Resource Graph

Azure Resource Graph proporciona acceso instantáneo a la información de los recursos en los entornos en la nube con funcionalidades sólidas para filtrar, agrupar y ordenar. Es una forma rápida y eficaz de consultar información en las suscripciones de Azure mediante programación o desde Azure Portal. [Más información sobre Azure Resource Graph](../governance/resource-graph/index.yml).

Para acceder a la puntuación de seguridad de varias suscripciones con Azure Resource Graph:

1. Desde Azure Portal, abra el **Explorador de Azure Resource Graph**.

    :::image type="content" source="./media/security-center-identity-access/opening-resource-graph-explorer.png" alt-text="Inicio de la página de recomendaciones de Azure Resource Graph Explorer**" :::

1. Escriba la consulta de Kusto (utilice los ejemplos siguientes como guía).

    - Esta consulta devuelve el identificador de la suscripción, la puntuación actual en puntos y como porcentaje, y la puntuación máxima de la suscripción. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores' 
        | extend current = properties.score.current, max = todouble(properties.score.max)
        | project subscriptionId, current, max, percentage = ((current / max)*100)
        ```

    - Esta consulta devuelve el estado de todos los controles de seguridad. Para cada control, obtendrá el número de recursos incorrectos, la puntuación actual y la puntuación máxima. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores/securescorecontrols'
        | extend SecureControl = properties.displayName, unhealthy = properties.unhealthyResourceCount, currentscore = properties.score.current, maxscore = properties.score.max
        | project SecureControl , unhealthy, currentscore, maxscore
        ```

1. Seleccione **Ejecutar consulta**.


## <a name="tracking-your-secure-score-over-time"></a>Seguimiento de la puntuación de seguridad a lo largo del tiempo

### <a name="secure-score-over-time-report-in-workbooks-page"></a>Informe de puntuación de seguridad a lo largo del tiempo en la página de libros

La página de libros de Security Center incluye un informe ya preparado para realizar un seguimiento visual de las puntuaciones de las suscripciones, los controles de seguridad, etc. Obtenga más información en [Creación de informes completos e interactivos de los datos de Security Center](custom-dashboards-azure-workbooks.md).

:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-snip.png" alt-text="Una sección del informe de puntuación de seguridad a lo largo del tiempo de la galería de libros de Azure Security Center":::

### <a name="power-bi-pro-dashboards"></a>Paneles de Power BI Pro

Si es un usuario de Power BI con una cuenta Pro, puede usar el panel de Power BI de **puntuación de seguridad a lo largo del tiempo** para realizar un seguimiento de la puntuación de seguridad a lo largo del tiempo e investigar los cambios.

> [!TIP]
> Puede encontrar este panel, así como otras herramientas para trabajar mediante programación con la puntuación de seguridad, en el área dedicada de la comunidad de Azure Security Center en GitHub: https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score

El panel contiene los dos informes siguientes para ayudarle a analizar el estado de seguridad:

- **Resumen de los recursos**: proporciona datos resumidos sobre el estado de los recursos.
- **Resumen de la puntuación de seguridad**: proporciona datos resumidos sobre el progreso de la puntuación. Use el gráfico "Secure score over time per subscription" para ver los cambios en la puntuación. Si observa un cambio drástico en la puntuación, consulte la tabla "detected changes that may affect your secure score" para buscar posibles cambios que puedan haber producido la variación. En esta tabla se presentan recursos eliminados, recursos recién implementados o recursos que su estado de seguridad cambió para una de las recomendaciones.

:::image type="content" source="./media/secure-score-security-controls/power-bi-secure-score-dashboard.png" alt-text="Panel de Power BI de puntuación de seguridad opcional a lo largo del tiempo para realizar un seguimiento de la puntuación de seguridad a lo largo del tiempo e investigar los cambios":::


## <a name="next-steps"></a>Pasos siguientes

En este artículo se describe cómo acceder a la puntuación de seguridad y realizar su seguimiento. Para obtener material relacionado, consulte los siguientes artículos:

- [Más información sobre los distintos elementos de una recomendación](security-center-recommendations.md)
- [Recomendaciones de corrección](security-center-remediate-recommendations.md)
- [Ver las herramientas basadas en GitHub para trabajar mediante programación con la puntuación de seguridad](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score)