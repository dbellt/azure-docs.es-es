---
title: Escalabilidad - Azure Event Hubs | Microsoft Docs
description: En este artículo se proporciona información sobre cómo escalar Azure Event Hubs mediante el uso de particiones y unidades de procesamiento.
ms.topic: article
ms.date: 05/26/2021
ms.openlocfilehash: ef894e0f14c140691b43da121a1983017ab03150
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110616468"
---
# <a name="scaling-with-event-hubs"></a>Escalado con Event Hubs

Hay dos factores que influyen en el escalado con Event Hubs.
* Unidades de rendimiento (nivel estándar) o unidades de procesamiento (nivel prémium) 
* Particiones

## <a name="throughput-units"></a>Unidades de procesamiento

La capacidad de rendimiento de Event Hubs se controla mediante *unidades de rendimiento*. Las unidades de procesamiento son unidades de capacidad adquiridas previamente. Un único procesamiento le permite:

* Entrada: hasta 1 MB por segundo o 1000 eventos por segundo, lo que ocurra primero.
* Salida: hasta 2 MB por segundo o 4096 eventos por segundo.

Si supera la capacidad de las unidades de rendimiento adquiridas, la entrada se limitará y se devolverá una excepción [ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception). La salida no produce excepciones de limitación, pero sigue estando limitada a la capacidad de las unidades de rendimiento adquiridas. Si recibe excepciones de tasa de publicación o espera ver una salida superior, compruebe cuántas unidades de rendimiento adquirió para el espacio de nombres. Puede administrar las unidades de procesamiento en la hoja **Escala** de los espacios de nombres en [Azure Portal](https://portal.azure.com). También puede administrar unidades de procesamiento mediante programación con las [API de Event Hubs](./event-hubs-samples.md).

Las unidades de procesamiento se adquieren previamente y se facturan por hora. Cuando se adquieren, las unidades de procesamiento se facturan durante un período mínimo de una hora. Se pueden adquirir hasta 20 unidades de procesamiento para un espacio de nombres de Event Hubs y compartir entre todos los centros de eventos del espacio de nombres.

La característica de **inflado automático** de Event Hubs escala verticalmente y de forma automática mediante el aumento del número de unidades de procesamiento para responder a las necesidades de utilización. Al aumentar las unidades de rendimiento, se evitan escenarios de limitación en los que nos encontramos con:

- Velocidades de entrada de datos que superan las unidades de rendimiento establecidas.
- Velocidades de solicitud de salida de datos que superan las unidades de rendimiento establecidas.

El servicio Event Hubs aumenta el rendimiento cuando la carga aumenta más allá del umbral mínimo, sin que se produzca ningún problema de las solicitudes con errores de ServerBusy. 

Para más información sobre la característica de inflado automático, consulte [Escalado automático de las unidades de rendimiento](event-hubs-auto-inflate.md).

## <a name="processing-units"></a>Unidades de procesamiento

 [Event Hubs Premium](./event-hubs-premium-overview.md) proporciona un rendimiento superior y un mejor aislamiento en un entorno PaaS multiinquilino administrado. Los recursos de un nivel Premium están aislados en el nivel de CPU y memoria para que cada carga de trabajo de inquilino se ejecute en aislamiento. Este contenedor de recursos recibe el nombre de *unidad de procesamiento* (PU). Puede comprar 1, 2, 4, 8 o 16 unidades de procesamiento para cada espacio de nombres de Event Hubs Premium. 

Lo que se pueda ingerir y transmitir con una unidad de procesamiento depende de varios factores, como los productores, los consumidores, la velocidad a la que se ingiere y procesa, etc. Una unidad de procesamiento puede ofrecer aproximadamente una capacidad principal de entrada de ~5-10 MB/s y de salida de 10-20 MB/s, dado que hay suficientes particiones para que el almacenamiento no sea un factor de limitación.  

Para saber cómo configurar las unidades de procesamiento para un espacio de nombres de nivel prémium, consulte [Configuración de unidades de procesamiento](configure-processing-units-premium-namespace.md).

> [!NOTE]
> Para más información sobre las cuotas y los límites, consulte [Cuotas y límites de Azure Event Hubs](event-hubs-quotas.md).

## <a name="partitions"></a>Particiones
[!INCLUDE [event-hubs-partitions](../../includes/event-hubs-partitions.md)]




## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de Event Hubs, visite los vínculos siguientes:

- [Escalado automático de unidades de procesamiento para un espacio de nombres de nivel estándar](event-hubs-auto-inflate.md)
- [Configuración de unidades de procesamiento para un espacio de nombres de nivel prémium](configure-processing-units-premium-namespace.md)
