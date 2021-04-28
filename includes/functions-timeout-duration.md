---
title: archivo de inclusión
description: archivo de inclusión
services: functions
author: nzthiago
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2018
ms.author: nzthiago
ms.custom: include file
ms.openlocfilehash: 709a06e9c4c027d8d82bdcde25e14b121feac35a
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065618"
---
## <a name="function-app-timeout-duration"></a><a name="timeout"></a>Duración del tiempo de espera de una aplicación de función 

La duración del tiempo de espera de una aplicación de funciones se define mediante la propiedad `functionTimeout` en el archivo de proyecto [host.json](../articles/azure-functions/functions-host-json.md#functiontimeout). En la tabla siguiente se muestran los valores predeterminados y máximos en minutos para ambos planes y en las distintas versiones en tiempo de ejecución:

| Plan | Versión en tiempo de ejecución | Valor predeterminado | Máxima |
|------|---------|---------|---------|
| Consumo | 1.x | 5 | 10 |
| Consumo | 2.x | 5 | 10 |
| Consumo | 3.x | 5 | 10 |
| Premium | 1.x | Sin límite | Sin límite |
| Premium | 2.x | 30 | Sin límite |
| Premium | 3.x | 30 | Sin límite |
| App Service | 1.x | Sin límite | Sin límite |
| App Service | 2.x | 30 | Sin límite |
| App Service | 3.x | 30 | Sin límite |

> [!NOTE] 
> Independientemente de la configuración del tiempo de espera de la aplicación de función, 230 segundos es la cantidad de tiempo máxima que una función desencadenada por HTTP puede tardar en responder a una solicitud. Esto se debe al [tiempo de espera de inactividad predeterminado de Azure Load Balancer](../articles/app-service/faq-availability-performance-application-issues.md#why-does-my-request-time-out-after-230-seconds). Para tiempos de procesamiento más largos, considere la posibilidad de usar el [patrón asincrónico de Durable Functions](../articles/azure-functions/durable/durable-functions-overview.md#async-http) o [aplazar el trabajo real y devolver una respuesta inmediata](../articles/azure-functions/functions-best-practices.md#avoid-long-running-functions).
