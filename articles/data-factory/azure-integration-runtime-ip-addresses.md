---
title: Direcciones IP de Azure Integration Runtime
description: Obtenga información sobre las direcciones IP a las que debe autorizar el tráfico de entrada con el fin de configurar correctamente los firewalls para proteger el acceso de red a los almacenes de datos.
ms.author: lle
author: lrtoyou1223
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/06/2020
ms.openlocfilehash: 872a8b0124a8ad1c86b201ca90b391d03bb76921
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485970"
---
# <a name="azure-integration-runtime-ip-addresses"></a>Direcciones IP de Azure Integration Runtime

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Las direcciones IP que usa Azure Integration Runtime dependen de la región en la que se encuentra su entorno de ejecución de integración de Azure. *Todos* los entornos de ejecución de integración de Azure de una misma región usan los mismos intervalos de direcciones IP.

> [!IMPORTANT]  
> Los flujos de datos y Azure Integration Runtime, que habilitan instancias de Virtual Network administradas, no admiten el uso de intervalos IP fijos.
>
> Puede usar estos intervalos IP para la ejecución de movimientos de datos, canalizaciones y actividades externas. Estos intervalos IP se pueden usar para el filtrado de almacenes de datos, grupos de seguridad de red (NSG) o firewalls para el acceso de entrada desde Azure Integration Runtime. 

## <a name="azure-integration-runtime-ip-addresses-specific-regions"></a>Direcciones IP de Azure Integration Runtime: Regiones específicas

Permita el tráfico desde las direcciones IP enumeradas para Azure Integration Runtime en la región específica en la que se encuentran los recursos. Puede obtener una lista de intervalos de IP de etiquetas de servicio en el [vínculo de descarga de intervalo IP de etiquetas de servicio](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). Por ejemplo, si la región de Azure es **AustraliaEast**, puede obtener una lista de intervalos IP en **DataFactory. AustraliaEast**.


## <a name="known-issue-with-azure-storage"></a>Problema conocido con Azure Storage

* Al conectarse a una cuenta de Azure Storage, las reglas de red IP no surten ningún efecto en las solicitudes que se originan en el entorno de ejecución de integración de Azure de la misma región que la cuenta de almacenamiento. Para más información, [consulte este artículo](../storage/common/storage-network-security.md#grant-access-from-an-internet-ip-range). 

  En su lugar, se recomienda usar [servicios de confianza al conectarse a Azure Storage](https://techcommunity.microsoft.com/t5/azure-data-factory/data-factory-is-now-a-trusted-service-in-azure-storage-and-azure/ba-p/964993). 

## <a name="next-steps"></a>Pasos siguientes

* [Consideraciones de seguridad para el movimiento de datos en Azure Data Factory](data-movement-security-considerations.md)