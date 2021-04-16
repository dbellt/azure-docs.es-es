---
title: Garantía de continuidad empresarial y recuperación ante desastres con regiones emparejadas de Azure
description: Garantía de la resistencia de las aplicaciones mediante el emparejamiento regional de Azure
author: martinekuan
manager: martinekuan
ms.service: multiple
ms.topic: conceptual
ms.date: 03/30/2021
ms.author: martinek
ms.custom: references_regions
ms.openlocfilehash: 9fda6f913fcb5325c811671cd6476dcbf2413766
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106058024"
---
# <a name="business-continuity-and-disaster-recovery-bcdr-azure-paired-regions"></a>Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure

## <a name="what-are-paired-regions"></a>¿Qué son las regiones emparejadas?

Una región de Azure consta de un conjunto de centros de datos implementados dentro de un perímetro definido por la latencia y conectados a través de una red dedicada de baja latencia.  Esto garantiza que los servicios de Azure dentro de una región de Azure ofrecen el mejor rendimiento y seguridad posibles.  

Una ubicación geográfica de Azure define un área definida del mundo que contiene al menos una región de Azure. Las zonas geográficas definen un mercado discreto, que normalmente contiene dos o más regiones, y conservan los límites de residencia de datos y cumplimiento.  [Aquí](https://azure.microsoft.com/global-infrastructure/regions/) encontrará más información sobre la infraestructura global de Azure.

Un par regional consta de dos regiones dentro de la misma ubicación geográfica. Azure serializa las actualizaciones de plataforma (mantenimiento planeado) entre pares regionales, lo que garantiza que solo se actualice una región de cada par cada vez. Si se produce una interrupción que afecte a varias regiones, se dará prioridad a la recuperación de al menos una región de cada par.

![AzureGeography](./media/best-practices-availability-paired-regions/GeoRegionDataCenter.png)

Algunos servicios de Azure aprovechan aún más las regiones emparejadas para garantizar la continuidad empresarial y protegerse frente a la pérdida de datos.  Azure proporciona varias [soluciones de almacenamiento](./storage/common/storage-redundancy.md#redundancy-in-a-secondary-region) que aprovechan las regiones emparejadas para garantizar la disponibilidad de los datos. Por ejemplo, el [almacenamiento con redundancia geográfica (GRS) de Azure](./storage/common/storage-redundancy.md#geo-redundant-storage) replica los datos en una región secundaria automáticamente, lo que garantiza que los datos sean duraderos incluso en caso de que la región principal no se pueda recuperar. 

Tenga en cuenta que no todos los servicios de Azure replican datos automáticamente, ni todos los servicios de Azure se revierten automáticamente de una región con errores a su par.  En tales casos, el cliente debe configurar la replicación y la recuperación.

## <a name="can-i-select-my-regional-pairs"></a>¿Puedo seleccionar mis pares regionales?

No. Algunos servicios de Azure se basan en pares regionales, como el [almacenamiento redundante](./storage/common/storage-redundancy.md) de Azure. Estos servicios no permiten crear nuevos emparejamientos regionales.  Del mismo modo, dado que Azure controla el mantenimiento planeado y la priorización de la recuperación para pares regionales, no puede definir sus propios pares regionales para aprovechar estos servicios. Sin embargo, puede crear su propia solución de recuperación ante desastres creando servicios en el número de regiones que desee y aprovechando los servicios de Azure para emparejarlos. 

Por ejemplo, puede usar servicios de Azure, como [AzCopy](./storage/common/storage-use-azcopy-v10.md), para programar copias de seguridad de datos en una cuenta de almacenamiento en una región diferente.  Con [Azure DNS y Azure Traffic Manager](./networking/disaster-recovery-dns-traffic-manager.md), los clientes pueden diseñar una arquitectura resistente para sus aplicaciones que sobrevivan a la pérdida de la región principal.

## <a name="am-i-limited-to-using-services-within-my-regional-pairs"></a>¿Estoy limitado al uso de servicios dentro de mis pares regionales?

No. Aunque un determinado servicio de Azure puede depender de un par regional, puede hospedar los demás servicios en cualquier región que satisfaga sus necesidades empresariales.  Una solución de almacenamiento GRS de Azure puede emparejar datos en el Centro de Canadá con un par del Este de Canadá mientras usa recursos de proceso ubicados en el Este de EE. UU.  

## <a name="must-i-use-azure-regional-pairs"></a>¿Debo usar los pares regionales de Azure?

No. Los usuarios pueden aprovechar los servicios de Azure para diseñar un servicio resistente sin depender de los pares regionales de Azure.  Sin embargo, se recomienda que configure la continuidad empresarial y recuperación ante desastres entre las parejas regionales para beneficiarse de las directivas de [aislamiento](./security/fundamentals/isolation-choices.md) y mejorar la [disponibilidad](./availability-zones/az-overview.md). Para aplicaciones que admiten varias regiones activas, recomendamos utilizar, siempre que sea posible, ambas regiones en un par de regiones. Esto asegura una disponibilidad óptima para las aplicaciones y minimizará el tiempo de recuperación en caso de desastre. Siempre que sea posible, diseñe la aplicación para obtener la [resistencia máxima](/azure/architecture/framework/resiliency/overview) y facilitar la [recuperación ante desastres](/azure/architecture/framework/resiliency/backup-and-recovery).

## <a name="azure-regional-pairs"></a>Pares regionales de Azure

| Geography | Par regional A | Par regional B  |
|:--- |:--- |:--- |
| Asia Pacífico |Asia Oriental (Hong Kong) | Sudeste Asiático (Singapur) |
| Australia |Este de Australia |Sudeste de Australia |
| Australia |Centro de Australia |Centro de Australia 2* |
| Brasil |Sur de Brasil |Centro-sur de EE. UU. |
| Brasil |Sudeste de Brasil* |Sur de Brasil |
| Canadá |Centro de Canadá |Este de Canadá |
| China |Norte de China |Este de China|
| China |Norte de China 2 |Este de China 2|
| Europa |Norte de Europa (Irlanda) |Oeste de Europa (Países Bajos) |
| Francia |Centro de Francia|Sur de Francia*|
| Alemania |Centro-oeste de Alemania |Norte de Alemania* |
| India |Centro de la India |Sur de la India |
| India |Oeste de la India |Sur de la India |
| Japón |Japón Oriental |Japón Occidental |
| Corea |Centro de Corea del Sur |Corea del Sur |
| Norteamérica |Este de EE. UU. |Oeste de EE. UU. |
| Norteamérica |Este de EE. UU. 2 |Centro de EE. UU. |
| Norteamérica |Centro-Norte de EE. UU |Centro-sur de EE. UU. |
| Norteamérica |Oeste de EE. UU. 2 |Centro-Oeste de EE. UU. |
| Noruega | Este de Noruega | Oeste de Noruega* |
| Sudáfrica | Norte de Sudáfrica |Oeste de Sudáfrica* |
| Suiza | Norte de Suiza |Oeste de Suiza* |
| Reino Unido |Oeste de Reino Unido |Sur de Reino Unido |
| Emiratos Árabes Unidos | Norte de Emiratos Árabes Unidos | Centro de Emiratos Árabes Unidos* |
| Departamento de Defensa de Estados Unidos |US DoD (este)* |US DoD (centro)* |
| US Gov |US Gov Arizona* |US Gov Texas* |
| US Gov |US Gov Iowa* |US Gov Virginia* |
| US Gov |US Gov Virginia* |US Gov Texas* |

(*) Ciertas regiones tienen acceso restringido para admitir escenarios de clientes específicos; por ejemplo, la recuperación ante desastres en el país. Estas regiones solo están disponibles cuando se solicitan mediante [la creación de una nueva solicitud de soporte técnico en Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

> [!Important]
> - Oeste de la India se empareja en una sola dirección. La región secundaria del India occidental es Sur de la India, pero la región secundaria esta última es Centro de la India.
> - Sur de Brasil es un caso único porque se empareja con una región fuera de su ubicación geográfica. La región secundaria de Sur de Brasil es Centro-sur de EE. UU. La región secundaria de Centro-sur de EE. UU. no es Sur de Brasil.


## <a name="an-example-of-paired-regions"></a>Un ejemplo de regiones emparejadas
La imagen siguiente ilustra una aplicación hipotética que utiliza el par regional para la recuperación ante desastres. Los números verdes resaltan las actividades entre regiones de tres servicios de Azure (Azure Compute, Storage y Database) y cómo están configurados para replicar entre regiones. Con los números de color naranja se resaltan los beneficios exclusivos de la implementación en regiones emparejadas.

![Información general sobre las ventajas de las regiones emparejadas](./media/best-practices-availability-paired-regions/PairedRegionsOverview2.png)

Ilustración 2: Pareja regional de Azure hipotética

## <a name="cross-region-activities"></a>Actividades entre regiones
Como se indica en la ilustración 2.

1. **Azure Compute (IaaS)** : debe aprovisionar recursos de procesos adicionales de antemano para asegurarse de que haya recursos disponibles en otra región durante un desastre. Para obtener más información, consulte [Guía técnica sobre resistencia en Azure](https://github.com/uglide/azure-content/blob/master/articles/resiliency/resiliency-technical-guidance.md). 

2. **Azure Storage**: si utiliza discos administrados, obtenga información sobre las [copias de seguridad entre regiones](/azure/architecture/resiliency/recovery-loss-azure-region#virtual-machines) con Azure Backup y la [replicación de máquinas virtuales](./site-recovery/azure-to-azure-tutorial-enable-replication.md) de una región a otra con Azure Site Recovery. Si está usando cuentas de almacenamiento, el almacenamiento con redundancia geográfica (GRS) se configura de manera predeterminada cuando se crea una cuenta de Azure Storage. Con GRS, los datos se replican automáticamente tres veces dentro de la región primaria y tres veces en la región emparejada. Para obtener más información, consulte [Opciones de redundancia de Azure Storage](storage/common/storage-redundancy.md).

3. **Azure SQL Database**: con la replicación geográfica de SQL Database, puede configurar la replicación asincrónica de las transacciones en cualquier región del mundo; sin embargo, se recomienda implementar estos recursos en una región emparejada para la mayoría de los escenarios de recuperación ante desastres. Para más información, consulte [Replicación geográfica en Azure SQL Database](./azure-sql/database/auto-failover-group-overview.md).

4. **Azure Resource Manager**: Resource Manager proporciona de forma inherente aislamiento lógico de los componentes entre regiones. Esto significa que es menos probable que los errores lógicos de una región afecten a otra.

## <a name="benefits-of-paired-regions"></a>Ventajas de las regiones emparejadas

5. **Aislamiento físico**: cuando sea posible, Azure prefiere por lo menos 300 millas de separación entre los centros de datos de un emparejamiento regional, aunque esto no será práctico o posible en todo el mundo. La separación del centro de datos físico reduce la probabilidad de que los desastres naturales, los disturbios civiles, los cortes del suministro eléctrico o las interrupciones de la red física afecten simultáneamente a ambas regiones. El aislamiento está sujeto a las restricciones geográficas (tamaño de la ubicación geográfica, disponibilidad de la infraestructura de red/energía, normativas, etc.).  

6. **Replicación proporcionada por la plataforma**: algunos servicios, como el almacenamiento con redundancia geográfica ofrecen replicación automática a la región emparejada.

7. **Orden de recuperación de región**: en el caso de una interrupción amplia, tiene prioridad la recuperación de una región por cada pareja. Se garantiza que, si las aplicaciones se implementan en regiones emparejadas, se dará prioridad a la recuperación de una de las regiones. Si una aplicación se implementa en regiones que no están emparejadas, podría retrasarse la recuperación; en el peor de los casos, es posible que las regiones elegidas sean las dos últimas en recuperarse.

8. **Actualizaciones secuenciales**: las actualizaciones planeadas del sistema de Azure se implementan en regiones emparejadas de forma secuencial (no al mismo tiempo) para minimizar el tiempo de inactividad, el efecto de los errores y la aparición de errores lógicos en el infrecuente caso de una actualización incorrecta.

9. **Residencia de datos**: una región reside dentro de la misma ubicación geográfica que su pareja (a excepción del Sur de Brasil) con el objeto de cumplir los requisitos de residencia de datos para fines de jurisdicción de impuestos y aplicación de la ley.