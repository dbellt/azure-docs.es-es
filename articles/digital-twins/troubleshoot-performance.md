---
title: Solución de problemas de rendimiento
titleSuffix: Azure Digital Twins
description: Sugerencias para solucionar problemas de rendimiento de una instancia de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 5/11/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 65b1873c1ac4f6ce9b06aad20dbb86e5f4613886
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110116633"
---
# <a name="troubleshooting-azure-digital-twins-performance"></a>Solución de problemas de Azure Digital Twins: rendimiento

Si experimenta retrasos u otros problemas de rendimiento al trabajar con Azure Digital Twins, use las sugerencias de este artículo para ayudarle a solucionar problemas.

## <a name="isolate-the-source-of-the-delay"></a>Determinación del origen del retraso

Determine si el retraso viene de Azure Digital Twins u otro servicio de la solución. Para investigar esto, puede usar la métrica **latencia de API** en [Azure Monitor](../azure-monitor/essentials/quick-monitor-azure-resource.md) mediante Azure Portal. Para obtener instrucciones sobre cómo ver las métricas de Azure Monitor de una instancia de Azure Digital Twins, consulte [Solución de problemas de Azure Digital Twins: Métricas](troubleshoot-metrics.md).

## <a name="check-regions"></a>Comprobación de las regiones

Si la solución usa Azure Digital Twins en combinación con otros servicios de Azure (como Azure Functions), compruebe la región de implementación de cada servicio. Si los servicios se implementan en regiones diferentes, esto puede agregar retrasos a la solución. A menos que esté creando intencionadamente una solución distribuida, considere la posibilidad de implementar todas las instancias de servicio dentro de la misma región para evitar la introducción accidental de retrasos.

## <a name="leverage-logs"></a>Aprovechamiento de los registros

Azure Digital Twins puede recopilar los registros de una instancia de servicio para ayudar a supervisar su rendimiento, entre otros datos. Los registros se pueden enviar a [Log Analytics](../azure-monitor/logs/log-analytics-overview.md) o a un mecanismo de almacenamiento personalizado. Para habilitar el registro en la instancia, siga las instrucciones de [Solución de problemas de Azure Digital Twins: Registro de diagnóstico](troubleshoot-diagnostics.md). Puede analizar las marcas de tiempo de los registros para medir las latencias, evaluar si son coherentes y comprender su origen.

## <a name="check-api-frequency"></a>Comprobación de la frecuencia de la API

Otro factor que puede afectar al rendimiento es el tiempo que se necesita para volver a autorizar las llamadas de API. Tenga en cuenta la frecuencia de las llamadas de API. Si hay un intervalo de más de 15 minutos entre las llamadas, es posible que el sistema vuelva a autorizar en cada llamada, con lo que se tarda más tiempo en hacerlo. Puede evitarlo agregando un temporizador o algo similar en el código para asegurarse de que llama a Azure Digital Twins al menos una vez cada 15 minutos.

## <a name="contact-support"></a>Ponerse en contacto con soporte técnico

Si sigue experimentando problemas de rendimiento después de solucionar problemas con los pasos anteriores, puede crear una solicitud de soporte técnico de Ayuda y soporte técnico de Azure para obtener ayuda adicional para la solución de problemas. 

Siga estos pasos:

1. Recopile las [métricas](troubleshoot-metrics.md) y los [registros](troubleshoot-diagnostics.md) de la instancia.
2. Vaya a [Ayuda y soporte técnico de Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal. Use las indicaciones para proporcionar detalles del problema, consulte las soluciones recomendadas, comparta sus archivos de métricas o registro, y envíe cualquier otra información que el equipo de soporte técnico pueda usar para ayudar a investigar el problema. Para más información sobre la creación de solicitudes de soporte técnico, consulte [Creación de una solicitud de soporte técnico de Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte otras formas de solucionar problemas de la instancia de Azure Digital Twins en los siguientes artículos:
* [Solución de problemas: Visualización de métricas con Azure Monitor](troubleshoot-metrics.md)
* [Solución de problemas: Configuración de diagnósticos](troubleshoot-diagnostics.md).
* [Solución de problemas: Configuración de alertas](troubleshoot-alerts.md)
* [Solución de problemas de Azure Digital Twins: Estado de los recursos](troubleshoot-resource-health.md)
