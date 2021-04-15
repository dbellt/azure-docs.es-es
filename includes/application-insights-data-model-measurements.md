---
author: mrbullwinkle
ms.service: application-insights
ms.topic: include
ms.date: 11/09/2018
ms.author: mbullwin
ms.openlocfilehash: 91141e314bf276d9138157c8a9d85d5262ac5907
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186073"
---
Colección de medidas personalizadas. Use esta colección para notificar la medida designada asociada al elemento de telemetría. Casos de uso típicos:
- el tamaño de carga útil de la telemetría de dependencia
- el número de elementos en la cola procesados por la telemetría de solicitudes
- tiempo que el cliente tardó en completar los pasos del asistente de finalización de la telemetría de eventos.

Puede consultar las [medidas personalizadas](https://analytics.applicationinsights.io/demo?q=H4sIAAAAAAAAA2WLOw6DMAyGd07hZoLeoRPqyMaGGAL8aiPhGCV2kKoeHsHK%2Bj1myyr8LoiaqfrT%2FkUCzRft4LMl8OUeL3LuLLIx%2BxR%2BIF8%2BtcoiNq2o78vgWuFthQaJ1AeGGxt6UlBwKxa1qQ6EpLhAfQAAAA%3D%3D&timespan=PT24H) en el análisis de aplicaciones:

```
customEvents
| where customMeasurements != ""
| summarize avg(todouble(customMeasurements["Completion Time"]) * itemCount)
```

 > [!NOTE]
 > Las medidas personalizadas están asociadas al elemento de telemetría al que pertenecen. Están sujetas a muestreo con el elemento de telemetría que contiene esas mediciones. Para realizar el seguimiento de una medida que tiene un valor independiente de otros tipos de telemetría, use la [telemetría de métricas](../articles/azure-monitor/app/api-custom-events-metrics.md).

Longitud máxima de la clave: 150
