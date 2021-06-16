---
title: Nivel de servicio Uso general
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Obtenga información sobre el nivel de servicio de uso general para Azure SQL Database y Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service-overview
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: mathoma
ms.date: 02/07/2019
ms.openlocfilehash: 9713b1d8635d678e650ec0b4f5aa448507e1da11
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110691532"
---
# <a name="general-purpose-service-tier---azure-sql-database-and-azure-sql-managed-instance"></a>Nivel de servicio de uso general: Azure SQL Database y Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

> [!NOTE]
> El nivel de servicio de uso general del modelo de compra basado en núcleo virtual se denomina nivel de servicio estándar en el modelo de compra basado en DTU. Para obtener una comparación entre el modelo de compra basado en núcleo virtual y el modelo de compra basado en DTU, vea [Modelos de compra y recursos](purchasing-models.md).

Azure SQL Database y Azure SQL Managed Instance se basan en la arquitectura del motor de base de datos de SQL Server, que se ha adaptado al entorno en la nube con el fin de garantizar una disponibilidad del 99,99 % incluso en los casos de error de la infraestructura. 

Estos son los dos niveles de servicio que Azure SQL Database e Instancia administrada de SQL usan: 

- De uso general
- Crítico para la empresa

Azure SQL Database tiene también un tercer nivel de servicio, que actualmente no está disponible para Instancia administrada de Azure SQL:

- Hiperescala

El modelo de arquitectura para el nivel de servicio de uso general se basa en la separación del proceso y el almacenamiento. Este modelo de arquitectura se basa en la alta disponibilidad y confiabilidad de Azure Blob Storage que replica de forma transparente los archivos de base de datos y garantiza que no se pierdan los datos si se produce algún error en la infraestructura subyacente.

En la siguiente imagen se muestran cuatro nodos en el modelo de arquitectura estándar con las capas separadas de proceso y almacenamiento.

![Separación de proceso y almacenamiento](./media/service-tier-general-purpose/general-purpose-service-tier.png)

En el modelo de arquitectura para el nivel de servicio de uso general, hay dos capas:

- Una capa de proceso sin estado que ejecuta el proceso `sqlservr.exe` y que solo contiene datos en caché y transitorios (por ejemplo, caché de planes, grupo de búferes o grupo de almacenes de columnas). Este nodo sin estado lo opera Azure Service Fabric, que inicializa el proceso, controla el estado del nodo y realiza la conmutación por error en otro lugar si es necesario.
- Una capa de datos con estado con archivos de base de datos (.mdf o .ldf) que se almacenan en Azure Blob Storage. Azure Blob Storage garantiza que no se pierdan los datos de ningún registro que se encuentre en un archivo de base de datos. Azure Storage tiene redundancia o disponibilidad de datos integrada que garantiza la conservación de los registros de una página o archivo de registro aunque se bloquee el proceso.

Siempre que se actualice un sistema operativo o un motor de base de datos, si se produce un error en la infraestructura subyacente, o si se detecta algún problema crítico en el proceso `sqlservr.exe`, Azure Service Fabric moverá el proceso sin estado a otro nodo de proceso sin estado. Hay un conjunto de nodos libres que esperan ejecutar un nuevo servicio de proceso si se produce una conmutación por error del nodo principal, a fin de minimizar el tiempo de conmutación por error. Esto no afecta a los datos de la capa de Azure Storage, y los archivos de registro o de datos se anexan al proceso inicializado recientemente. Este proceso garantiza una disponibilidad del 99,99 %, pero podría afectar al rendimiento de grandes cargas de trabajo que se ejecutan debido al tiempo de transición y al hecho de que el nuevo nodo se inicia con la memoria caché inactiva.

## <a name="when-to-choose-this-service-tier"></a>¿Cuándo elegir este nivel de servicio?

El nivel de servicio de uso general es un nivel de servicio predeterminado de Azure SQL Database y Azure SQL Managed Instance que se ha diseñado para la mayoría de las cargas de trabajo genéricas. Si necesita un motor de base de datos totalmente administrado con un Acuerdo de Nivel de Servicio de un 99,99 % de latencia de almacenamiento de entre 5 y 10 ms que coincida con SQL Server en una máquina virtual de Azure en la mayoría de los casos, el nivel de uso general es la opción más conveniente.

## <a name="next-steps"></a>Pasos siguientes

- Encontrar características de recursos (número de núcleos, E/S, memoria) del nivel estándar o de uso general en [SQL Managed Instance](../managed-instance/resource-limits.md#service-tier-characteristics), la base de datos única en el [modelo de núcleos virtuales](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) o el [modelo DTU](resource-limits-dtu-single-databases.md#single-database-storage-sizes-and-compute-sizes), o el grupo elástico en el [modelo de núcleos virtuales](resource-limits-vcore-elastic-pools.md#general-purpose---provisioned-compute---gen4) y el [modelo DTU](resource-limits-dtu-elastic-pools.md#standard-elastic-pool-limits).
- Obtenga información sobre los niveles [Crítico para la empresa](service-tier-business-critical.md) e [Hiperescala](service-tier-hyperscale.md).
- Obtenga información sobre [Service Fabric](../../service-fabric/service-fabric-overview.md).
- Para conocer más opciones de alta disponibilidad y recuperación ante desastres, consulte [Continuidad del negocio](business-continuity-high-availability-disaster-recover-hadr-overview.md).
