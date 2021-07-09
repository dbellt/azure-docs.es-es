---
title: archivo de inclusión
description: archivo de inclusión
services: azure-monitor
author: rboucher
tags: azure-service-management
ms.topic: include
ms.date: 05/03/2021
ms.author: robb
ms.custom: include file
ms.openlocfilehash: 1d9ec2d05374412c17493b47f3bb71524f8cf93f
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110630880"
---
| Recurso | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| Alertas de métricas (clásico) |100 reglas de alertas activas por suscripción. | Llame al soporte técnico. |
| Alertas de métricas |5000 reglas de alertas activas por suscripción tanto en la nube pública de Azure, como en las nubes de Azure China 21Vianet y Azure Government. Si alcanza este límite, examine si puede usar [alertas de varios recursos del mismo tipo](../articles/azure-monitor/alerts/alerts-metric-overview.md#monitoring-at-scale-using-metric-alerts-in-azure-monitor).<br/>5000 series temporales de métricas por regla de alertas. | Llame al soporte técnico. |
| Alertas de registros de actividad | 100 reglas de alerta activas por suscripción (este número no se puede aumentar). | Igual que el predeterminado. |
| Alertas de registro | 1000 reglas de alertas activas por suscripción. 1000 reglas de alertas activas por recurso. | Llame al soporte técnico. |
| Longitud de la descripción de las reglas de alertas y las reglas de acción| Alertas de búsqueda de registros: 4096 caracteres<br/>Todas las demás, 2048 caracteres | Igual que el predeterminado. |

### <a name="alerts-api"></a>API de alertas
Las alertas de Azure Monitor disponen de varios límites de protección frente a los usuarios que realizan un número excesivo de llamadas. Este comportamiento puede sobrecargar potencialmente los recursos de back-end del sistema y poner en peligro la capacidad de respuesta del servicio. Los límites siguientes están diseñados para proteger a los clientes frente a interrupciones y garantizar un nivel de servicio coherente. Los límites y la limitación de usuarios están diseñados para afectar solo a un escenario de uso extremo y no deben ser de importancia para un uso normal.

| Resource | Límite predeterminado | Límite máximo |
| --- | --- | --- |
| GET alertsSummary | 50 llamadas por minuto y por suscripción | Igual que el predeterminado. | 
|   GET alerts (sin especificar un identificador de alerta) | 100 llamadas por minuto y por suscripción | Igual que el predeterminado. | 
|   Todas las demás llamadas | 1000 llamadas por minuto y por suscripción | Igual que el predeterminado. | 

