---
ms.topic: include
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.date: 04/01/2021
ms.openlocfilehash: 547242c70986c3216a4052ca3763701264dd4282
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108211057"
---
## <a name="business-disaster-recovery"></a>Recuperación ante desastres de la empresa

En esta sección se describen las características de Azure Time Series Insights que mantienen las aplicaciones y los servicios en ejecución, incluso si se produce un desastre (lo que se conoce como *recuperación ante desastres de la empresa*).

### <a name="high-availability"></a>Alta disponibilidad

Como servicio de Azure, Azure Time Series Insights proporciona determinadas características de *alta disponibilidad* mediante el uso de redundancias al nivel de región de Azure. Por ejemplo, Azure admite funcionalidades de recuperación ante desastres mediante la característica de *disponibilidad entre regiones* de Azure.

Entre las características de alta disponibilidad adicionales proporcionadas mediante Azure (también disponibles para cualquier instancia de Azure Time Series Insights), se encuentran las siguientes:

- **Conmutación por error**: Azure proporciona [replicación geográfica y equilibrio de carga](/azure/architecture/resiliency/recovery-loss-azure-region).
- **Restauración de datos** y **recuperación del almacenamiento**: Azure proporciona [varias opciones para conservar y recuperar datos](/azure/architecture/resiliency/recovery-data-corruption).
- **Azure Site Recovery**: Azure proporciona características de recuperación a través de [Azure Site Recovery](../articles/site-recovery/index.yml).
- **Azure Backup**: [Azure Backup](../articles/backup/backup-architecture.md) admite la copia de seguridad local y en la nube de las máquinas virtuales de Azure.

Asegúrese de habilitar las características de Azure pertinentes para proporcionar una alta disponibilidad global entre regiones para los dispositivos y usuarios.

> [!NOTE]
> Si Azure está configurado para habilitar la disponibilidad entre regiones, no es necesario establecer ninguna configuración adicional de disponibilidad entre regiones en Azure Time Series Insights.

### <a name="iot-and-event-hubs"></a>IoT y centro de eventos

Algunos servicios IoT de Azure también incluyen características de recuperación ante desastres de empresa integradas:

- [Recuperación ante desastres de alta disponibilidad de Azure IoT Hub](../articles/iot-hub/iot-hub-ha-dr.md), incluida la redundancia entre regiones
- [Directivas de Azure Event Hubs](../articles/event-hubs/event-hubs-geo-dr.md)
- [Redundancia de Azure Storage](../articles/storage/common/storage-redundancy.md)

La integración de Azure Time Series Insights con los demás servicios proporciona oportunidades adicionales de recuperación ante desastres. Por ejemplo, los datos de telemetría enviados al centro de eventos pueden conservarse en una copia de seguridad de la base de datos de Azure Blob Storage.

### <a name="azure-time-series-insights"></a>Azure Time Series Insights

Existen varias maneras de mantener los datos, las aplicaciones y los servicios de Azure Time Series Insights en ejecución, incluso si se ha interrumpido.

Sin embargo, puede determinar que también se requiera una copia de seguridad completa del entorno de Azure Time Series, con los siguientes objetivos:

- Como una *instancia de conmutación por error* específicamente para que Azure Time Series Insights le redirija los datos y el tráfico
- Para conservar los datos y la información de auditoría

Por lo general, la mejor forma de duplicar un entorno de Azure Time Series Insights es crear un segundo entorno de Azure Time Series Insights en una región de Azure de copia de seguridad. Los eventos también se envían a este secundario desde el origen del evento principal. Asegúrese de usar un segundo grupo de consumidores dedicado. Siga las pautas de recuperación ante desastres de la empresa de ese origen, como se han descrito anteriormente.

Para crear un entorno duplicado:

1. Cree un entorno en una segunda región. Para más información, lea [Creación de un entorno de Azure Time Series Insights en Azure Portal](../articles/time-series-insights/time-series-insights-get-started.md).
1. Cree un segundo grupo de consumidores dedicado para el origen de eventos.
1. Conecte ese origen de eventos al nuevo entorno. Asegúrese de designar el segundo grupo de consumidores dedicado.
1. Revise la documentación de Azure Time Series Insights sobre [IoT Hub](../articles/time-series-insights/how-to-ingest-data-iot-hub.md) y [Event Hubs](../articles/time-series-insights/concepts-access-policies.md).

Si se produce un evento:

1. Si la región primaria resulta afectada durante un incidente de desastre, vuelva a enrutar las operaciones al entorno de copia de Azure Time Series Insights de respaldo.
1. Dado que los números de secuencia de centro se reinician a partir de 0 después de la conmutación por error, vuelva a crear el origen de eventos en ambas regiones o entornos con grupos de consumidores diferentes para evitar la creación de lo que parecerían eventos duplicados.
1. Elimine el origen de eventos principal, que ahora está inactivo, para liberar un origen de eventos disponible para su entorno. (Hay un límite de dos orígenes de eventos activos por entorno).
1. Use la segunda región para realizar copias de seguridad y recuperar todos los datos de telemetría y consulta de Azure Time Series Insights.

> [!IMPORTANT]
> Si se produce una conmutación por error:
>
> - También puede producirse un retraso.
> - Podría producirse un pico momentáneo en el procesamiento de los mensajes dado que las operaciones se vuelven a enrutar.
>
> Para más información, lea [Mitigación de la latencia en Azure Time Series Insights](../articles/time-series-insights/time-series-insights-environment-mitigate-latency.md).