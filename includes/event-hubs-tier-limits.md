---
title: include file description: include file services: event-hubs author: spelluru ms.service: event-hubs ms.topic: include ms.date: 05/25/2021 ms.author: spelluru ms.custom: "include file","fasttrack-edit","iot","event-hubs"

---

En la tabla siguiente se muestran los límites que pueden ser diferentes para los niveles básico, estándar y dedicado. En la tabla, CU es [unidad de capacidad](../articles/event-hubs/event-hubs-dedicated-overview.md), PU es [unidad de procesamiento](../articles/event-hubs/event-hubs-scalability.md#processing-units) y TU es [unidad de rendimiento](../articles/event-hubs/event-hubs-scalability.md#throughput-units). 

| Límite | Básico | Estándar | Premium |  Dedicado |
| ----- | ----- | -------- | -------- | --------- | 
| Tamaño máximo de la publicación de Event Hubs | 256 KB | 1 MB | 1 MB |  1 MB |
| Número de grupos de consumidores por centro de eventos | 1 | 20 | 100 | 1000<br/>Sin límite por CU  |
| Número de conexiones asincrónicas por espacio de nombres | 100 | 5\.000 | 10 000 por unidad de procesamiento por PU | 100 000 por CU |
| Período de retención máximo de datos de eventos | 1 día | 7 días | 90 días<br/>1 TB por PU | 90 días<br/>10 TB por CU |
| TU, PU o CU máximas |20 TU | 20 TU | 16 PU | 20 CU |
| Número de particiones por centro de eventos | 32 | 32 | 100 por centro de eventos <br/>200 por PU | 1024 por centro de eventos<br/> 2000 por CU |
| Número de espacios de nombres por suscripción | 1000 | 1000 | 1000 | 1000 (50 por CU) |
| Número de centros de eventos por espacio de nombres | 10 | 10 | 100 | 1000 |
| Capturar | N/D | Pago por hora | Se incluye | Se incluye |
| Tamaño del registro de esquema (espacio de nombres), en megabytes | N/D | 25 | 100 | 1024 |
| Números de grupos de esquemas en un registro de esquemas o un espacio de nombres | N/D | 1: sin incluir el grupo predeterminado | 100 <br/>1 MB por esquema | 1000<br/>1 MB por esquema |
| Número de versiones del esquema en todos los grupos de esquemas | N/D | 25 | 1000 | 10000 |
| Rendimiento por unidad | Entrada: 1 MB/s o 1000 eventos por segundo<br/>Salida: 2 MB/s o 4096 eventos por segundo | Entrada: 1 MB/s o 1000 eventos por segundo<br/>Salida: 2 MB/s o 4096 eventos por segundo | Sin límites por PU * | Sin límites por CU * |

\* Depende de varios factores, como la asignación de recursos, el número de particiones, el almacenamiento, etc. 
 

> [!NOTE]
> Los eventos se pueden publicar de forma individual o por lotes. El límite de publicación (según la SKU) se aplica independientemente de si se trata de un evento único o un lote. Se rechazará la publicación de eventos que superen el umbral máximo.

