---
title: Diagnóstico y solución de problemas de Azure API Management
description: Aprenda a solucionar problemas de la API de Azure API Management con la herramienta de diagnóstico y solución de problemas de Azure Portal.
author: rzhang628
ms.service: api-management
ms.topic: article
ms.date: 02/05/2021
ms.author: rongzhang
ms.openlocfilehash: 01b932cf33883f186f9f4ff69bc78869cdb4cb18
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109516585"
---
# <a name="azure-api-management-diagnostics-overview"></a>Introducción sobre los diagnósticos de Azure API Management

Cuando crea y administra una API en Azure API Management, quiere estar preparado para cualquier problema que pueda surgir, desde errores "404 No encontrado" a errores "502 Puerta de enlace incorrecta". Los diagnósticos de API Management son una experiencia inteligente e interactiva que le ayuda a solucionar los problemas de la API publicada en APIM sin necesidad de configuración alguna. Cuando surgen problemas con las API publicadas, los diagnósticos de API Management le indicarán lo que pasa y le guiarán a la información correcta de forma que pueda solucionar el problema rápidamente.

Aunque esta experiencia es más útil si ha tenido problemas con la API en las últimas 24 horas, todos los gráficos de diagnóstico están siempre disponibles para su análisis.

[!NOTE] Actualmente no se admite el diagnóstico y la solución de problemas para el nivel de consumo.

## <a name="open-api-management-diagnostics"></a>Abra la página de diagnósticos de API Management

Para acceder a los diagnósticos de API Management, vaya a la instancia de servicio de API Management en [Azure portal](https://portal.azure.com). En el panel de navegación de la izquierda, seleccione **Diagnosticar y solucionar problemas**.

:::image type="content" source="media/diagnose-solve-problems/apim-diagnostic-home.png" alt-text="Captura de pantalla que muestra cómo ir a los diagnósticos.":::



## <a name="intelligent-search"></a>Búsqueda inteligente

Puede buscar los problemas en la barra de búsqueda de la parte superior de la página. La búsqueda también le permite encontrar las herramientas que pueden ayudarle a solucionar o resolver los problemas. 

:::image type="content" source="media/diagnose-solve-problems/intelligent-search.png" alt-text="Captura de pantalla de la búsqueda inteligente.":::


## <a name="troubleshooting-categories"></a>Categorías de solución de problemas

Puede efectuar la solución de problemas mediante categorías. Los problemas habituales relacionados con el rendimiento de la API, la puerta de enlace, las directivas de API y el nivel de servicio se pueden analizar dentro de cada categoría. Cada categoría proporciona también comprobaciones de diagnóstico más específicas. 

:::image type="content" source="media/diagnose-solve-problems/troubleshooting-category.png" alt-text="Captura de pantalla con información general sobre las categorías.":::


### <a name="availability-and-performance"></a>Disponibilidad y rendimiento

Compruebe los problemas de rendimiento y la disponibilidad de la API en esta categoría. Después de seleccionar este icono de categoría, se recomiendan algunas comprobaciones comunes en una interfaz interactiva. Haga clic en cada comprobación para profundizar en los detalles de cada problema. La comprobación también le proporcionará un gráfico que muestra el rendimiento de la API y un resumen de los problemas de rendimiento. Por ejemplo, el servicio de API puede haber tenido un error 5xx y haber agotado el tiempo de espera en la última hora en el back-end. 

:::image type="content" source="media/diagnose-solve-problems/category-interactive-search-1.png" alt-text="Captura de pantalla 1 de la comprobación de interfaz interactiva.":::



:::image type="content" source="media/diagnose-solve-problems/category-interactive-search-2.png" alt-text="Captura de pantalla 2 de la comprobación de interfaz interactiva.":::

### <a name="api-policies"></a>Directivas de API

Esta categoría detecta errores y le informa de los problemas de la directiva. 

Una interfaz interactiva similar le guía por las métricas de datos para ayudarle a solucionar problemas de la configuración de directivas de la API.

:::image type="content" source="media/diagnose-solve-problems/proxy-policies.png" alt-text="Captura de pantalla del icono de categorías de directivas de la API.":::

### <a name="gateway-performance"></a>Rendimiento de las puertas de enlace 

En el caso de las solicitudes o respuestas de puerta de enlace o de cualquier error 4xx o 5xx en la puerta de enlace, use esta categoría para supervisar y solucionar problemas. Del mismo modo, utilice la interfaz interactiva para profundizar en el área específica que desea comprobar en relación con el rendimiento de la puerta de enlace de API. 

:::image type="content" source="media/diagnose-solve-problems/gateway-performance-tile.png" alt-text="Captura de pantalla del icono de la categoría de rendimiento de la puerta de enlace.":::

### <a name="service-upgrade"></a>Actualización del servicio

Esta categoría comprueba qué nivel de servicio (SKU) está usando actualmente y le recuerda que actualice para evitar cualquier problema que pueda estar relacionado con ese nivel. La misma interfaz interactiva le ayuda a profundizar más en los gráficos y en el resultado de la comprobación de resumen. 

:::image type="content" source="media/diagnose-solve-problems/service-sku.png" alt-text="Captura de pantalla del icono de la categoría de actualización del servicio.":::

## <a name="search-documentation"></a>Buscar en la documentación

Además de las herramientas para diagnosticar y solucionar problemas, puede buscar documentación de solución de problemas relacionada con el problema. Después de ejecutar el diagnóstico del servicio, seleccione **Buscar documentación** en la interfaz interactiva. 

 :::image type="content" source="media/diagnose-solve-problems/search-documentation.png" alt-text="Captura de pantalla 1 de cómo usar la función Buscar documentación.":::


 :::image type="content" source="media/diagnose-solve-problems/search-documentation-2.png" alt-text="Captura de pantalla 2 de cómo usar Buscar documentación.":::


## <a name="next-steps"></a>Pasos siguientes

* Use también el [análisis de API](howto-use-analytics.md) para analizar el uso y el rendimiento de las API. 
* ¿Desea solucionar problemas de Web Apps con Diagnósticos? Lea al respecto [aquí](../app-service/overview-diagnostics.md)
* Use Diagnósticos para comprobar los problemas de Azure Kubernetes Services. Consulte [Diagnósticos en AKS](../aks/concepts-diagnostics.md)
* Publique sus preguntas o comentarios en [UserVoice](https://feedback.azure.com/forums/248703-api-management) agregando "[Diag]" en el título.
