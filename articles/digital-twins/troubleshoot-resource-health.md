---
title: Información del estado de los recursos
titleSuffix: Azure Digital Twins
description: Consulte cómo usar Azure Resource Health para comprobar el estado de la instancia de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 10/6/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: f62e07bbc5b61ce07299f1d617b8aea1b331024e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475596"
---
# <a name="troubleshooting-azure-digital-twins-resource-health"></a>Solución de problemas de Azure Digital Twins: Estado de los recursos

[Azure Resource Health](../service-health/resource-health-overview.md) ayuda a diagnosticar problemas en los servicios que afectan a los recursos de Azure y a obtener soporte técnico para resolverlos. Informa sobre el mantenimiento actual y pasado de los recursos.

En este artículo se muestra cómo obtener al información de **estado del recurso** de las instancias de Azure Digital Twins.

## <a name="use-azure-resource-health"></a>Uso de Azure Resource Health

Azure Resource Health puede ayudarle a supervisar si la instancia de Azure Digital Twins está en funcionamiento. También permite saber si una interrupción regional afecta al estado de la instancia.

Para comprobar el estado de la instancia, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia de Azure Digital Twins. Escriba el nombre en la barra de búsqueda para encontrarla. 

2. En el menú de la instancia, seleccione _**Resource Health**_ en *Soporte técnico y solución de problemas*. Esto le llevará a la página para ver el historial de estado de los recursos. 

    :::image type="content" source="media/troubleshoot-resource-health/resource-health.png" alt-text="Captura de pantalla que muestra la página &quot;Resource Health&quot;. Hay una sección &quot;Historial de estado&quot; que muestra un informe diario de los últimos nueve días.":::

En la imagen anterior, esta instancia se muestra como *Disponible* y lo ha estado durante los últimos nueve días. Para más información sobre el estado *Disponible* y los otros tipos de estado que pueden aparecer, consulte [Introducción a Resource Health](../service-health/resource-health-overview.md).

También puede encontrar más información sobre las distintas comprobaciones que entran en el estado de los recursos para los diferentes tipos de recursos de Azure en [Tipos de recursos y comprobaciones de estado en Azure Resource Health](../service-health/resource-health-checks-resource-types.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte otras formas de supervisar la instancia de Azure Digital Twins en los siguientes artículos:
* [Solución de problemas: Visualización de métricas con Azure Monitor](troubleshoot-metrics.md)
* [Solución de problemas: Configuración de diagnósticos](troubleshoot-diagnostics.md).
* [Solución de problemas: Configuración de alertas](troubleshoot-alerts.md)
