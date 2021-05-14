---
title: Disponibilidad de recursos por región
description: Disponibilidad de recursos de proceso y memoria para el servicio Azure Container Instances en diferentes regiones de Azure.
ms.topic: article
ms.date: 04/27/2020
ms.custom: references_regions
ms.openlocfilehash: 2bf5735c1e50ae31f70b68d7b926e82badeb3933
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108015994"
---
# <a name="resource-availability-for-azure-container-instances-in-azure-regions"></a>Disponibilidad de recursos para Azure Container Instances en las regiones de Azure

En este artículo se detalla la disponibilidad de los recursos de proceso, memoria y almacenamiento de Azure Container Instances en las regiones de Azure y del sistema operativo de destino. Para obtener una lista general de las regiones disponibles para Azure Container Instances, consulte las [regiones disponibles](https://azure.microsoft.com/regions/services/).

Los valores que se presentan son los recursos máximos disponibles por cada implementación de un [grupo de contenedores](container-instances-container-groups.md). Los valores están actualizados en el momento de la publicación.

> [!NOTE]
> Los grupos de contenedores creados dentro de estos límites de recursos están sujetos a disponibilidad en la región de implementación. Cuando una región está sometida a mucha carga, puede experimentar un error al implementar instancias. Para mitigar este tipo de error de implementación, intente implementar las instancias con una configuración de menos recursos o bien pruebe a realizar la implementación en un momento posterior o en una región diferente con los recursos disponibles.

Para obtener información sobre las cuotas y otros límites en las implementaciones, vea [Cuotas y límites de Azure Container Instances](container-instances-quotas.md).

## <a name="linux-container-groups"></a>Grupos de contenedores de Linux

Están disponibles las siguientes regiones y recursos máximos para los grupos de contenedores con contenedores de Linux en implementaciones generales, implementaciones de [red virtual de Azure](container-instances-vnet.md) e implementaciones con [recursos de GPU](container-instances-gpu.md) (versión preliminar).

> [!IMPORTANT]
> El número máximo de recursos de una región es diferente en función de la implementación. Por ejemplo, una región puede tener un tamaño de CPU y memoria máximo diferente en una implementación de red virtual de Azure que en una implementación general. Esa misma región también puede tener un conjunto diferente de valores máximos para una implementación con recursos de GPU. Compruebe el tipo de implementación antes de buscar los valores máximos de la región en las tablas siguientes.

| Region | Uso máximo de CPU | Memoria máxima (GB) | CPU máxima de VNET | Memoria máxima de VNET (GB) | Almacenamiento (GB) | SKU de GPU (versión preliminar) |
| -------- | :---: | :---: | :----: | :-----: | :-------: | :----: |
| Este de Australia | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Sur de Brasil | 4 | 16 | 2 | 8 | 50 | No aplicable |
| Centro de Canadá | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Centro de la India | 4 | 16 | 4 | 4 | 50 | V100 |
| Centro de EE. UU. | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Este de Asia | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Este de EE. UU. | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |
| Este de EE. UU. 2 | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Centro de Francia | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Centro-oeste de Alemania | 3 | 16 | No aplicable | No aplicable | 50 | No aplicable |
| Japón Oriental | 2 | 8 | 4 | 16 | 50 | No aplicable |
| Centro de Corea del Sur | 4 | 16 | No aplicable | No aplicable | 50 | No aplicable |
| Centro-Norte de EE. UU | 2 | 3,5 | 4 | 16 | 50 | K80, P100, V100 |
| Norte de Europa | 4 | 16 | 4 | 16 | 50 | K80 |
| Centro-sur de EE. UU. | 4 | 16 | 4 | 16 | 50 | V100 |
| Sudeste de Asia | 4 | 16 | 4 | 16 | 50 | P100, V100 |
| Sur de la India | 4 | 16 | No aplicable | No aplicable | 50 | K80 |
| Norte de Suiza | 3 | 16 | No aplicable | No aplicable | 50 | No aplicable |
| Sur de Reino Unido | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Norte de Emiratos Árabes Unidos | 3 | 16 | No aplicable | No aplicable | 50 | No aplicable |
| Centro-Oeste de EE. UU.| 4 | 16 | 4 | 16 | 50 | No aplicable |
| Oeste de Europa | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |
| Oeste de EE. UU. | 4 | 16 | 4 | 16 | 50 | No aplicable |
| Oeste de EE. UU. 2 | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |

Los recursos máximos siguientes están disponibles para un grupo de contenedores implementado con [recursos de GPU](container-instances-gpu.md) (versión preliminar).

> [!IMPORTANT]
> En este momento, las implementaciones con recursos de GPU no se admiten en una implementación de Azure Virtual Network y solo están disponibles en grupos de contenedores de Linux.

| SKU de GPU | Recuento de GPU | Uso máximo de CPU | Memoria máxima (GB) | Almacenamiento (GB) |
| --- | --- | --- | --- | --- |
| K80 | 1 | 6 | 56 | 50 |
| K80 | 2 | 12 | 112 | 50 |
| K80 | 4 | 24 | 224 | 50 |
| P100, V100 | 1 | 6 | 112 | 50 |
| P100, V100 | 2 | 12 | 224 | 50 |
| P100, V100 | 4 | 24 | 448 | 50 |

## <a name="windows-container-groups"></a>Grupos de contenedor de Windows

Las siguientes regiones y recursos máximos están disponibles para los grupos de contenedores con contenedores de Windows Server [admitidos y en versión preliminar](container-instances-faq.md#what-windows-base-os-images-are-supported).

> [!IMPORTANT]
> En este momento, las implementaciones con grupos de contenedores de Windows no se admiten en una implementación de Azure Virtual Network.

###  <a name="windows-server-2016"></a>Windows Server 2016

> [!NOTE]
> Para obtener más información sobre los hosts 1B, 2B y 3B, consulte [Compatibilidad de versiones de host y contenedor](/virtualization/windowscontainers/deploy-containers/update-containers#host-and-container-version-compatibility).

| Region | Número máximo de CPU de 1B/2B | Máximo de memoria (GB) de 1B/2B |Número máximo de CPU de 3B | Máximo de memoria (GB) de 3B | Almacenamiento (GB) |
| -------- | :---: | :---: | :----: | :-----: | :-------: |
| Este de Australia | 2 | 8 | 2 | 8 | 20 |
| Sur de Brasil | 4 | 16 | 4 | 16 | 20 |
| Centro de Canadá | 2 | 8 | 2 | 3,5 | 20 |
| Centro de la India | 2 | 3,5 | 2 | 3,5 | 20 |
| Centro de EE. UU. | 2 | 8 | 2 | 3,5 | 20 |
| Este de Asia | 2 | 3,5 | 2 | 3,5 | 20 |
| Este de EE. UU. | 2 | 8 | 2 | 8 | 20 |
| Este de EE. UU. 2 | 2 | 8 | 4 | 16 | 20 |
| Japón Oriental | 4 | 16 | 4 | 16 | 20 |
| Centro de Corea del Sur | 4 | 16 | 4 | 16 | 20 |
| Centro-Norte de EE. UU | 2 | 8 | 4 | 16 | 20 |
| Norte de Europa | 2 | 8 | 2 | 8 | 20 |
| Centro-sur de EE. UU. | 2 | 8 | 2 | 8 | 20 |
| Sudeste de Asia | N/D | N/D | 2 | 3,5 | 20 |
| Sur de la India | 2 | 3,5 | 2 | 3,5 | 20 |
| Sur de Reino Unido | 2 | 8 | 2 | 3,5 | 20 |
| Centro-Oeste de EE. UU. | 2 | 8 | 2 | 8 | 20 |
| Oeste de Europa | 4 | 16 | 4 | 16 | 20 |
| Oeste de EE. UU. | 2 | 8 | 2 | 8 | 20 |
| Oeste de EE. UU. 2 | 2 | 8 | 2 | 3,5 | 20 |


### <a name="windows-server-2019-ltsc"></a>Windows Server 2019 LTSC

> [!NOTE]
> Para obtener más información sobre los hosts 1B, 2B y 3B, consulte [Compatibilidad de versiones de host y contenedor](/virtualization/windowscontainers/deploy-containers/update-containers#host-and-container-version-compatibility).

| Region | Número máximo de CPU de 1B/2B | Máximo de memoria (GB) de 1B/2B |Número máximo de CPU de 3B | Máximo de memoria (GB) de 3B | Almacenamiento (GB) |
| -------- | :---: | :---: | :----: | :-----: | :-------: |
| Este de Australia | 4 | 16 | 4 | 16 | 20 |
| Sur de Brasil | 4 | 16 | 4 | 16 | 20 |
| Centro de Canadá | 4 | 16 | 4 | 16 | 20 |
| Centro de la India | 4 | 16 | 4 | 16 | 20 |
| Centro de EE. UU. | 4 | 16 | 4 | 16 | 20 |
| Este de Asia | 4 | 16 | 4 | 16 | 20 |
| Este de EE. UU. | 4 | 16 | 4 | 16 | 20 |
| Este de EE. UU. 2 | 2 | 3,5 | 2 | 3,5 | 20 |
| Centro de Francia | 4 | 16 | 4 | 16 | 20 |
| Japón Oriental | N/D | N/D | 4 | 16 | 20 |
| Centro de Corea del Sur | 4 | 16 | 4 | 16 | 20 |
| Centro-Norte de EE. UU | 4 | 16 | 4 | 16 | 20 |
| Norte de Europa | 4 | 16 | 4 | 16 | 20 |
| Centro-sur de EE. UU. | 4 | 16 | 4 | 16 | 20 |
| Sudeste de Asia | 4 | 16 | 4 | 16 | 20 |
| Sur de la India | 4 | 16 | 4 | 16 | 20 |
| Sur de Reino Unido | 4 | 16 | 4 | 16 | 20 |
| Centro-Oeste de EE. UU. | 4 | 16 | 4 | 16 | 20 |
| Oeste de Europa | 4 | 16 | 4 | 16 | 20 |
| Oeste de EE. UU. | 4 | 16 | 4 | 16 | 20 |
| Oeste de EE. UU. 2 | 2 | 8 | 4 | 16 | 20 |

## <a name="next-steps"></a>Pasos siguientes

Informe al equipo si le gustaría ver regiones adicionales o una mayor disponibilidad de los recursos en [aka.ms/aci/feedback](https://aka.ms/aci/feedback).

Para obtener información sobre cómo solucionar problemas de la implementación de instancias de contenedor, vea [Solución de problemas en las implementaciones en Azure Container Instances](container-instances-troubleshooting.md).


[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
