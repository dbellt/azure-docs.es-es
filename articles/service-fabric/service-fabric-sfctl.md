---
title: 'CLI de Service Fabric de Azure: sfctl'
description: Más información sobre sfctl, la interfaz de la línea de comandos de Azure Service Fabric. Incluye una lista de comandos y subgrupos.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 4721598961ae912e8f0a9ef2f61022e5feb39e6c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462256"
---
# <a name="sfctl"></a>sfctl
Comandos para administrar clústeres y entidades de Service Fabric. Esta versión es compatible con el entorno de ejecución de Service Fabric 7.0.

Los comandos siguientes siguen el patrón nombre-verbo. Consulte los subgrupos para más información.

## <a name="subgroups"></a>Subgrupos
|Subgrupo|Descripción|
| --- | --- |
| [application](service-fabric-sfctl-application.md) | Cree, elimine y administre aplicaciones y tipos de aplicaciones. |
| [chaos](service-fabric-sfctl-chaos.md) | Inicie, detenga e informe sobre el servicio de prueba de Chaos. |
| [cluster](service-fabric-sfctl-cluster.md) | Seleccione, administre y opere clústeres de Service Fabric. |
| [compose](service-fabric-sfctl-compose.md) | Cree, elimine y administre aplicaciones de Docker Compose. |
| [container](service-fabric-sfctl-container.md) | Ejecute los comandos relacionados con el contenedor en un nodo de clúster. |
| [eventos](service-fabric-sfctl-events.md) | Recupere eventos del almacén de eventos (si ya está instalado el servicio EventStore). |
| [is](service-fabric-sfctl-is.md) | Consulte y envíe comandos al servicio de infraestructura. |
| [node](service-fabric-sfctl-node.md) | Administre los nodos que forman un clúster. |
| [partition](service-fabric-sfctl-partition.md) | Consulte y administre las particiones para cualquier servicio. |
| [property](service-fabric-sfctl-property.md) | Almacene y consulte las propiedades con nombres de Service Fabric. |
| [replica](service-fabric-sfctl-replica.md) | Administre las réplicas que pertenecen a las particiones del servicio. |
| [rpm](service-fabric-sfctl-rpm.md) | Consulte y envíe comandos al servicio del administrador de reparaciones. |
| [sa-cluster](service-fabric-sfctl-sa-cluster.md) | Administre clústeres de Service Fabric independientes. |
| [service](service-fabric-sfctl-service.md) | Cree, elimine y administre servicios, tipos de servicio y paquetes de servicio. |
| [settings](service-fabric-sfctl-settings.md) | Configura los valores locales para esta instancia de sfctl. |
| [store](service-fabric-sfctl-store.md) | Realice operaciones básicas a nivel de archivo en el almacén de imágenes del clúster. |

## <a name="next-steps"></a>Pasos siguientes
- [Configuración](service-fabric-cli.md) de la CLI de Service Fabric.
- Obtenga información sobre cómo utilizar la CLI de Service Fabric con los [scripts de ejemplo](./scripts/sfctl-upgrade-application.md).
