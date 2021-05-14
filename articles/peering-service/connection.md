---
title: Conexión de Azure Peering Service
description: Obtenga información sobre la conexión de Microsoft Azure Peering Service
services: peering-service
author: gthareja
ms.service: peering-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 04/07/2021
ms.author: gatharej
ms.openlocfilehash: 9463a0967b9cfc25f189dd3efac80e26428866fa
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108202666"
---
# <a name="peering-service-connection"></a>Conexión de Peering Service

Una conexión normalmente hace referencia a un conjunto de información lógico que identifica a una instancia de Peering Service. Se define al especificar los atributos siguientes:

- Nombre lógico
- Asociado de conectividad
- Ubicación del servicio principal del asociado de conectividad
- Ubicación del servicio de copia de seguridad del asociado de conectividad
- Ubicación física del cliente
- Prefijos IP

El cliente puede establecer una sola conexión o varias conexiones según los requisitos. Una conexión también se usa como una unidad de recopilación de telemetría. Por ejemplo, para recibir alertas de telemetría, el cliente debe definir la conexión que se va a supervisar.

> [!Note]
> Al suscribirse a Peering Service, se analiza la telemetría de Windows y Microsoft 365 para proporcionar medidas de latencia para los prefijos seleccionados. Actualmente, los datos de telemetría se admiten para prefijos de tamaño /24 o superior únicamente.
>Para obtener más información sobre la telemetría de conexión, vea [Telemetría de conexión de Peering Service](connection-telemetry.md).
>

## <a name="how-to-create-a-peering-service-connection"></a>¿Cómo crear una conexión de Peering Service?

**Escenario**: imagine una sucursal distribuida entre diferentes ubicaciones geográficas, como se muestra en la ilustración siguiente. Aquí, el cliente debe proporcionar un nombre lógico, el nombre del proveedor de servicios (SP), la ubicación física del cliente y los prefijos IP que haya (propiedad del cliente o asignados por el proveedor de servicios) asociados a una única conexión.  Las ubicaciones del servicio principal y de copia de seguridad con asociado ayudan a definir la ubicación de servicio preferida para el cliente. Este proceso debe repetirse para crear Peering Service para otras ubicaciones.

![Conexiones con redundancia geográfica](./media/peering-service-connection/peering-service-connections.png)

> [!Note]
> El filtrado de nivel de estado se tiene en cuenta para la ubicación física del cliente cuando la conexión se encuentra geográficamente en los Estados Unidos.
>

## <a name="next-steps"></a>Pasos siguientes

Para conocer el procedimiento paso a paso para registrar la conexión de Peering Service, consulte [Creación de Peering Service mediante Azure Portal](azure-portal.md).

Para más información sobre la telemetría de la conexión de Peering Service, consulte [Telemetría de la conexión de Peering Service](connection-telemetry.md).

Para acceder a la telemetría, consulte [Acceso a la telemetría de conexión](measure-connection-telemetry.md).
