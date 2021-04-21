---
title: Actualización de la versión del clúster de Azure Service Fabric
description: Obtenga información sobre las versiones de clúster en Azure Service Fabric, incluido un vínculo a las versiones más recientes del blog del equipo de Service Fabric.
ms.topic: troubleshooting
ms.date: 06/15/2020
ms.openlocfilehash: 1f06d7cb5f8fa1182969447168dfb4297473075a
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257541"
---
# <a name="upgrade-your-azure-service-fabric-cluster-version"></a>Actualización de la versión del clúster de Azure Service Fabric

Asegúrese de que el clúster siempre ejecute una versión admitida de Azure Service Fabric. El soporte técnico para la versión anterior de Service Fabric finaliza al menos 60 días después del lanzamiento de la nueva versión. Las versiones nuevas se anuncian en el [blog del equipo de Service Fabric](https://azure.microsoft.com/updates/?product=service-fabric).

Para cada versión del entorno de ejecución de Service Fabric, puede usar las versiones especificadas o anteriores de los paquetes del SDK/NuGet. Es posible que las versiones más recientes de los paquetes no puedan tener como destino clústeres más antiguos. Los clústeres más antiguos podrían tener cambios de características o protocolos que los entornos de paquetes más recientes no admitan.

Consulte los artículos siguientes para ver detalles sobre cómo hacer que el clúster ejecute una versión compatible de Service Fabric:

- [Actualización de un clúster de Azure Service Fabric](service-fabric-cluster-upgrade.md)
- [Actualización de la versión de Service Fabric que se ejecuta en el clúster](service-fabric-cluster-upgrade-windows-server.md)

## <a name="unsupported-versions"></a>Versiones no admitidas

### <a name="upgrade-alert-for-versions-between-57-and-6363"></a>Alerta de actualización para las versiones entre 5.7 y 6.3.63.*

Para mejorar la seguridad y la disponibilidad, la infraestructura de Azure ha hecho un cambio que puede afectar a los clientes de Service Fabric. Este cambio afecta a todos los clústeres de Service Fabric que ejecutan las versiones entre la  5.7 y la 6.3, ambas inclusive.

Hay disponible una actualización del entorno de ejecución de Service Fabric para todas las versiones de Service Fabric compatibles en todas las regiones. Actualice a una de las versiones admitidas más recientes antes del 19 de enero de 2021 para evitar interrupciones del servicio.

Si tiene un plan de soporte técnico y necesita ayuda técnica, acceda a través de los canales de soporte técnico de Azure. Abra una solicitud de soporte técnico de Azure Service Fabric y mencione este contexto en la incidencia.

#### <a name="if-you-dont-upgrade-to-a-supported-version"></a>Si no actualiza a una versión compatible

Los clústeres de Azure Service Fabric que se ejecutan en las versiones de 5.7 a 6.3.63.* no estarán disponibles si no se han actualizado antes del 19 de enero de 2021.

#### <a name="required-action"></a>Acción necesaria

Actualice a una versión de Service Fabric compatible para evitar el tiempo de inactividad o la pérdida de funcionalidad relacionados con este cambio. Asegúrese de que los clústeres ejecuten al menos las versiones siguientes para evitar problemas en su entorno.

> [!Note]
> **Todas las versiones de 7.2 incluyen los cambios necesarios**.
  
  | SO | Entorno de ejecución actual de Service Fabric en el clúster | Versión de revisión/actualización acumulativa |
  | --- | --- |--- |
  | Windows | 7.0.* | 7.0.478.9590 |
  | Windows | 7.1.* | 7.1.503.9590 |
  | Windows | 7.2.* | 7.2.* |
  | Ubuntu 16 | 7.0.* | 7.0.472.1  |
  | Linux Ubuntu 16.04 | 7.1.* | 7.1.455.1  |
  | Linux Ubuntu 18.04 | 7.1.* | 7.1.455.1804 |
  | Linux Ubuntu 16.04 | 7.2.* | 7.2.* |
  | Linux Ubuntu 18.04 | 7.2.* | 7.2.* |

### <a name="upgrade-alert-for-versions-later-than-63"></a>Alerta de actualización de versiones posteriores a 6.3

Para mejorar la seguridad y la disponibilidad, la infraestructura de Azure ha hecho un cambio que puede afectar a los clientes de Service Fabric. Este cambio afectará a todos los clústeres de Service Fabric que usan el [modo de red abierto para los contenedores](./service-fabric-networking-modes.md#set-up-open-networking-mode) y ejecutan las versiones de 6.3 a 7.0 o versiones admitidas no compatibles posteriores a 7.0. Hay disponible una actualización del entorno de ejecución de Service Fabric para todas las versiones de Service Fabric compatibles en todas las regiones.

#### <a name="if-you-dont-upgrade-to-a-supported-version"></a>Si no actualiza a una versión compatible

Los clústeres de Azure Service Fabric que se ejecutan en versiones sin cambios posteriores a 6.3 experimentarán pérdida de funcionalidad o interrupciones del servicio si no se actualizaron a una versión admitida antes del 19 de enero de 2021.
  
  - **En el caso de los clústeres que ejecutan una versión de Service Fabric superior a 6.3 que NO usan la característica de red abierta**, el clúster permanecerá activo.

 - **En el caso de clústeres que ejecutan una versión de Service Fabric posterior a la 6.3 y que usan la [característica de red abierta para contenedores](./service-fabric-networking-modes.md#set-up-open-networking-mode)** , el clúster podría dejar de estar disponible y cesar su actividad, lo que podría provocar interrupciones del servicio en las cargas de trabajo.
 
 -   **Para los clústeres que ejecutan [versiones de Windows entre 7.0.457 y 7.0.466 (ambas inclusive)](#supported-version-names) y el sistema operativo Windows tiene habilitada la característica de contenedores de Windows. NOTA: Las versiones de Linux 7.0.457, 7.0.464 y 7.0.465 no se ven afectadas**.
    - **Impacto**: el clúster dejará de funcionar, lo que podría provocar interrupciones del servicio para las cargas de trabajo.
    
#### <a name="required-action"></a>Acción necesaria

Para evitar tiempos de inactividad o pérdida de funcionalidad, asegúrese de que los clústeres ejecutan una de las siguientes versiones.

Las versiones de Service Fabric de la tabla contienen los cambios necesarios para evitar la pérdida de funcionalidad. Asegúrese de que está usando una de estas versiones.  

> [!Note]
> **Los clústeres de Azure Service Fabric que se ejecutan en la versión 6.5 tienen que realizar varias actualizaciones al mismo tiempo antes de cambiar la infraestructura para evitar la pérdida de funcionalidad en el clúster** . 
>   -   1. Actualice a 7.0.466. **Los clústeres que ejecutan el sistema operativo Windows que tiene habilitada la característica de contenedores de Windows NO PUEDEN estar en esta versión intermedia. Deben completar el paso siguiente (II) que aparece a continuación, es decir, actualizar para que esté en la versión más segura y compatible a fin de evitar interrupciones del servicio**.
>   -   2. Actualice a las versiones compatibles más recientes de la versión 7.0* (7.0.478) o cualquiera de las versiones superiores que se muestran a continuación.


> [!Note]
> **Todas las versiones de 7.2 incluyen los cambios necesarios**.

 | SO | Entorno de ejecución actual de Service Fabric en el clúster | Versión de revisión/actualización acumulativa |
  | --- | --- |--- |
  | Windows | 7.1.* | 7.1.503.9590 |
  | Windows | 7.2.* | 7.2.477.9590 |
  | Windows | 8.0.* | 8.0.514.9590 |
  | Linux Ubuntu 16.04 | 7.1.* | 7.1.455.1  |
  | Linux Ubuntu 18.04 | 7.1.* | 7.1.455.1804 |
  | Linux Ubuntu 16.04 | 7.2.* | 7.2.476.1 |
  | Linux Ubuntu 18.04 | 7.2.* | 7.2.476.1804 |
  | Linux Ubuntu 16.04 | 8.0.* | 8.0.513.1 |
  | Linux Ubuntu 18.04 | 8.0.* | 8.0.513.1804 |

## <a name="supported-versions"></a>Versiones compatibles

En la tabla siguiente se indican las versiones de Service Fabric y sus fechas de finalización de soporte.

| Entorno de tiempo de ejecución de Service Fabric en el clúster | Se puede actualizar directamente desde la versión del clúster |Versiones del SDK o paquete de NuGet compatibles | Finalización del soporte técnico |
| --- | --- |--- | --- |
| Todas las versiones de clústeres anteriores a 5.3.121 | 5.1.158.* |Versión 2.3 o anterior |20 de enero de 2017 |
| 5.3* | 5.1.158.* |Versión 2.3 o anterior |24 de febrero de 2017 |
| 5.4.* | 5.1.158.* |Versión 2.4 o anterior |10 de mayo de 2017       |
| 5.5.* | 5.4.164.* |Versión 2.5 o anterior |10 de agosto de 2017    |
| 5.6.* | 5.4.164.* |Versión 2.6 o anterior |13 de octubre de 2017   |
| 5.7.* | 5.4.164.* |Versión 2.7 o anterior |15 de diciembre de 2017  |
| 6.0.* | 5.6.205.* |Versión menor o igual que la 2.8 |30 de marzo de 2018     |
| 6.1.* | 5.7.221.* |Versión 3.0 o anterior |15 de julio de 2018      |
| 6.2.* | 6.0.232.* |Versión 3.1 o anterior |26 de octubre de 2018   |
| 6.3.* | 6.1.480.* |Versión 3.2 o anterior |31 de marzo de 2019  |
| 6.4.* | 6.2.301.* |Versión 3.3 o anterior |15 de septiembre de 2019 |
| 6.5.* | 6.4.617.* |Versión 3.4 o anterior |1 de agosto de 2020 |
| 7.0.466.* | 6.4.664.* |Versión 4.0 o anterior|31 de enero de 2021  |
| 7.0.466.* | 6.5.* |Versión 4.0 o anterior|31 de enero de 2021 |
| 7.0.470.* | 7.0.466.* |Versión 4.0 o anterior |31 de enero de 2021  |
| 7.0.472.* | 7.0.466.* |Versión 4.0 o anterior |31 de enero de 2021  |
| 7.0.478.* | 7.0.466.* |Versión 4.0 o anterior |31 de enero de 2021  |
| 7.1.409.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.417.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.428.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.456.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.458.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.459.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.503.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.1.510.* | 7.0.466.* |Versión 4.1 o anterior |31 de julio de 2021 |
| 7.2.413.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.432.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.433.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.445.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.452.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.457.* | 7.0.470.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 7.2.477.* | 7.0.478.* |Versión 4.2 o anterior |30 de noviembre de 2021 |
| 8.0.514.* | 7.1.510.* |Versión 5.0 o anterior |Versión actual; por lo tanto, sin fecha de finalización |

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

En la tabla siguiente se indican los sistemas operativos compatibles para las versiones de Service Fabric admitidas.

| Sistema operativo | Versión más antigua admitida de Service Fabric |
| --- | --- |
| Windows Server 2012 R2 | Todas las versiones |
| Windows Server 2016 | Todas las versiones |
| Windows Server 1709 | 6.0 |
| Windows Server 1803 | 6.4 |
| Windows Server 1809 | 6.4.654.9590 |
| Windows Server 2019 | 6.4.654.9590 |
| Linux Ubuntu 16.04 | 6.0 |
| Linux Ubuntu 18.04 | 7.1 |

## <a name="supported-version-names"></a>Nombres de versiones admitidas

En la tabla siguiente se indican los nombres de versión de Service Fabric y sus números de versión correspondientes.

| Nombre de versión | Número de versión de Windows | Número de versión de Linux |
| --- | --- | --- |
| 5.3 RTO | 5.3.121.9494 | No aplicable|
| 5.3 CU1 | 5.3.204.9494 | No aplicable|
| 5.3 CU2 | 5.3.301.9590 | No aplicable|
| 5.3 CU3 | 5.3.311.9590 | No aplicable|
| 5.4 CU2 | 5.4.164.9494 | No aplicable|
| 5.5 CU1 | 5.5.216.0    | No aplicable|
| 5.5 CU2 | 5.5.219.0 | No aplicable|
| 5.5 CU3 | 5.5.227.0 | No aplicable|
| 5.5 CU4 | 5.5.232.0 | No aplicable|
| 5.6 RTO | 5.6.204.9494 | No aplicable|
| 5.6 CU2 | 5.6.210.9494 | No aplicable|
| 5.6 CU3 | 5.6.220.9494 | No aplicable|
| 5.7 RTO | 5.7.198.9494 | No aplicable|
| 5.7 CU4 | 5.7.221.9494 | No aplicable|
| 6.0 RTO | 6.0.211.9494 | 6.0.120.1 |
| 6.0 CU1 | 6.0.219.9494 | 6.0.127.1 |
| 6.0 CU2 | 6.0.232.9494 | 6.0.133.1 |
| 6.1 CU1 | 6.1.456.9494 | 6.1.183.1 |
| 6.1 CU2 | 6.1.467.9494 | 6.1.185.1 |
| 6.1 CU3 | 6.1.472.9494 | No aplicable|
| 6.1 CU4 | 6.1.480.9494 | 6.1.187.1 |
| 6.2 RTO | 6.2.269.9494 | 6.2.184.1 |
| 6.2 CU1 | 6.2.274.9494 | 6.2.191.1 |
| 6.2 CU2 | 6.2.283.9494 | 6.2.194.1 |
| 6.2 CU3 | 6.2.301.9494 | 6.2.199.1 |
| 6.3 RTO | 6.3.162.9494 | 6.3.119.1 |
| 6.3 CU1 | 6.3.176.9494 | 6.3.124.1 |
| 6.3 CU1 | 6.3.187.9494 | 6.3.129.1 |
| 6.4 RTO | 6.4.617.9590 | 6.4.625.1 |
| 6.4 CU2 | 6.4.622.9590 | No aplicable|
| 6.4 CU3 | 6.4.637.9590 | 6.4.634.1 |
| 6.4 CU4 | 6.4.644.9590 | 6.4.639.1 |
| 6.4 CU5 | 6.4.654.9590 | 6.4.649.1 |
| 6.4 CU6 | 6.4.658.9590 | No aplicable|
| 6.4 CU7 | 6.4.664.9590 | 6.4.661.1 |
| 6.4 CU8 | 6.4.670.9590 | No aplicable|
| 6.5 RTO | 6.5.639.9590 | 6.5.435.1 |
| 6.5 CU1 | 6.5.641.9590 | 6.5.454.1 |
| 6.5 CU2 | 6.5.658.9590 | 6.5.460.1 |
| 6.5 CU3 | 6.5.664.9590 | 6.5.466.1 |
| 6.5 CU5 | 6.5.676.9590 | 6.5.467.1 |
| 7.0 RTO | 7.0.457.9590 | 7.0.457.1 |
| 7.0 CU2 | 7.0.464.9590 | 7.0.464.1 |
| 7.0 CU3 | 7.0.466.9590 | 7.0.465.1 |
| 7.0 CU4 | 7.0.470.9590 | 7.0.469.1 |
| 7.0 CU6 | 7.0.472.9590 | 7.0.471.1 |
| 7.0 CU9 | 7.0.478.9590 | 7.0.472.1 |
| 7.1 RTO | 7.1.409.9590 | 7.1.410.1 |
| 7.1 CU1 | 7.1.417.9590 | 7.1.418.1 |
| 7.1 CU2 | 7.1.428.9590 | 7.1.428.1 |
| 7.1 CU3 | 7.1.456.9590 | 7.1.452.1 |
| 7.1 CU5 | 7.1.458.9590 | 7.1.454.1 |
| 7.1 CU6 | 7.1.459.9590 | 7.1.455.1 |
| 7.1 CU8 | 7.1.503.9590 | 7.1.508.1 |
| 7.1 CU10 | 7.1.510.9590 | N/D |
| 7.2 RTO | 7.2.413.9590 | N/D |
| 7.2 CU2 | 7.2.432.9590 | 7.2.431.1 |
| 7.2 CU3 | 7.2.433.9590 | N/D |
| 7.2 CU4 | 7.2.445.9590 | 7.2.447.1 |
| 7.2 CU5 | 7.2.452.9590 | 7.2.454.1 |
| 7.2 CU6 | 7.2.457.9590 | 7.2.456.1 |
| 7.2 CU7 | 7.2.477.9590 | 7.2.476.1 |
| 8.0 RTO | 8.0.514.9590 | 8.0.513.1 |