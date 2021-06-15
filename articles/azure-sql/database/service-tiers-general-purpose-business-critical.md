---
title: Niveles de servicio de uso general y crítico para la empresa
titleSuffix: Azure SQL Database & SQL Managed Instance
description: En el artículo se describen los niveles de servicio de uso general y crítico para la empresa en el modelo de compra basado en núcleo virtual, usado por Azure SQL Database e Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service-overview
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: dimitri-furman
ms.author: dfurman
ms.reviewer: mathoma
ms.date: 12/14/2020
ms.openlocfilehash: 1ccf83fa98ec7024e74bd9829e241b8596b1fbf6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693202"
---
# <a name="azure-sql-database-and-azure-sql-managed-instance-service-tiers"></a>Niveles de servicio de Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database e Instancia administrada de Azure SQL se basna en la arquitectura del motor de base de datos de SQL Server que se ajusta al entorno en la nube para garantizar una disponibilidad del 99,99 %, incluso en los casos de error de la infraestructura. En Azure SQL Database e Instancia administrada de Azure SQL se usan dos niveles de servicio, cada uno con un modelo de arquitectura diferente. Estos niveles de servicio son:

- [De uso general](service-tier-general-purpose.md), diseñado para la mayoría de cargas de trabajo guiadas por el presupuesto.
- [Crítico para la empresa](service-tier-business-critical.md), diseñado para cargas de trabajo de baja latencia con alta resistencia a errores y conmutaciones por error rápidas.

Azure SQL Database tiene un nivel de servicio adicional: 

- [Hiperscala](service-tier-hyperscale.md), diseñado para la mayoría de las cargas de trabajo empresariales; proporciona almacenamiento altamente escalable, escalado horizontal de lectura y capacidades rápidas de restauración de las bases de datos.

Este artículo se analizan las diferencias entre los niveles de servicio, las consideraciones de almacenamiento y copia de seguridad de uso general, y los niveles de servicio críticos para el negocio en el modelo de compra basado en núcleo virtual.

## <a name="service-tier-comparison"></a>Comparación de niveles de servicio

En la tabla siguiente se describen las diferencias principales entre los niveles de servicio de la última generación (Gen5). Tenga en cuenta que las características del nivel de servicio pueden ser diferentes en SQL Database e Instancia administrada de SQL.

|-| Tipo de recurso | Uso general |  Hiperescala | Crítico para la empresa |
|:---:|:---:|:---:|:---:|:---:|
| **Más adecuado para** | |  Ofrece opciones de proceso y almacenamiento equilibradas adecuadas para un presupuesto limitado. | La mayoría de las cargas de trabajo empresariales. Escalado automático del tamaño de almacenamiento hasta 100 TB, escalado de procesos vertical y horizontal fluido, restauración rápida de bases de datos. | Aplicaciones de OLTP con una alta tasa de transacciones y latencia de E/S baja. Ofrece mayor resistencia a los errores y rapidez en las conmutaciones por error mediante varias réplicas actualizadas sincrónicamente.|
|  **Disponible en estos tipos de recurso:** ||SQL Database / Instancia administrada de SQL | Base de datos única de Azure SQL Database | SQL Database / Instancia administrada de SQL |
| **Tamaño de proceso**| SQL Database | 1 a 80 núcleos virtuales | 1 a 80 núcleos virtuales | 1 a 80 núcleos virtuales |
| | Instancia administrada de SQL | 4, 8, 16, 24, 32, 40, 64 y 80 núcleos virtuales | N/D | 4, 8, 16, 24, 32, 40, 64 y 80 núcleos virtuales |
| | Grupos de Instancia administrada de SQL | 2, 4, 8, 16, 24, 32, 40, 64 y 80 núcleos virtuales | N/D | N/D |
| **Tipo de almacenamiento** | All | Almacenamiento remoto Premium (por instancia) | Almacenamiento desacoplado con caché de SSD local (por instancia) | Almacenamiento SSD local extremadamente rápido (por instancia) |
| **Tamaño de la base de datos** | SQL Database | 5 GB – 4 TB | Hasta 100 TB | 5 GB – 4 TB |
| | Instancia administrada de SQL  | 32 GB–8 TB | N/D | 32 GB – 4 TB |
| **Tamaño de almacenamiento** | SQL Database | 5 GB – 4 TB | Hasta 100 TB | 5 GB – 4 TB |
| | Instancia administrada de SQL  | 32 GB–8 TB | N/D | 32 GB – 4 TB |
| **Tamaño de TEMPDB** | SQL Database | [32 GB por núcleo virtual](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) | [32 GB por núcleo virtual](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5) | [32 GB por núcleo virtual](resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen4) |
| | Instancia administrada de SQL  | [24 GB por núcleo virtual](../managed-instance/resource-limits.md#service-tier-characteristics) | N/D | Hasta 4 TB: [limitado en función del tamaño de almacenamiento](../managed-instance/resource-limits.md#service-tier-characteristics) |
| **Rendimiento de escritura de registros** | SQL Database | [1,875 MB/s por núcleo virtual (máximo 30 MB/s)](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) | 100 MB/s | [6 MB/s por núcleo virtual (máximo 96 MB/s)](resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen4) |
| | Instancia administrada de SQL | [3 MB/s por núcleo virtual (máximo 22 MB/s)](../managed-instance/resource-limits.md#service-tier-characteristics) | N/D | [4 MB/s por núcleo virtual (máximo 48 MB/s)](../managed-instance/resource-limits.md#service-tier-characteristics) |
|**Disponibilidad**|All| 99,99% |  [99,95 % con una réplica secundaria; 99,99 % con más réplicas](service-tier-hyperscale-frequently-asked-questions-faq.yml#what-slas-are-provided-for-a-hyperscale-database) | 99,99% <br/> [99,995 % con una base de datos única con redundancia de zona](https://azure.microsoft.com/blog/understanding-and-leveraging-azure-sql-database-sla/) |
|**Copias de seguridad**|All|RA-GRS, de 7 a 35 días (7 días de manera predeterminada). La retención máxima para el nivel Básico es 7 días. | RA-GRS, 7 días, recuperación a un momento dado (PITR) en un tiempo constante | RA-GRS, de 7 a 35 días (7 días de forma predeterminada) |
|**OLTP en memoria** | | N/D | N/D | Disponible |
|**Réplicas de solo lectura**| | 0 integradas <br> 0 a 4 con [replicación geográfica](active-geo-replication-overview.md) | 0 a 4 integradas | 1 integrada, incluida en el precio <br> 0 a 4 con [replicación geográfica](active-geo-replication-overview.md) |
|**Precios y facturación** | SQL Database | [El núcleo virtual, el almacenamiento reservado y el almacenamiento de copia de seguridad](https://azure.microsoft.com/pricing/details/sql-database/single/) se cobran. <br/>Las IOPS no se cobran. | Se cobran los [núcleos virtuales de cada réplica y el almacenamiento usado](https://azure.microsoft.com/pricing/details/sql-database/single/). <br/>IOPS todavía no se ha cargado. | [El núcleo virtual, el almacenamiento reservado y el almacenamiento de copia de seguridad](https://azure.microsoft.com/pricing/details/sql-database/single/) se cobran. <br/>Las IOPS no se cobran. |
|| Instancia administrada de SQL | [El núcleo virtual, el almacenamiento reservado y el almacenamiento de copia de seguridad](https://azure.microsoft.com/pricing/details/sql-database/managed/) se cobran. <br/>Las IOPS no se cobran.| N/D | [El núcleo virtual, el almacenamiento reservado y el almacenamiento de copia de seguridad](https://azure.microsoft.com/pricing/details/sql-database/managed/) se cobran. <br/>Las IOPS no se cobran.| 
|**Modelos de descuento**| | [Instancias reservadas](reserved-capacity-overview.md)<br/>[Ventaja híbrida de Azure](../azure-hybrid-benefit.md) (no disponible en suscripciones de desarrollo y pruebas)<br/>Suscripciones de Desarrollo/pruebas de [Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/) y [de pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)| [Ventaja híbrida de Azure](../azure-hybrid-benefit.md) (no disponible en suscripciones de desarrollo y pruebas)<br/>Suscripciones de Desarrollo/pruebas de [Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/) y [de pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)| [Instancias reservadas](reserved-capacity-overview.md)<br/>[Ventaja híbrida de Azure](../azure-hybrid-benefit.md) (no disponible en suscripciones de desarrollo y pruebas)<br/>Suscripciones de Desarrollo/pruebas de [Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/) y [de pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)|

Para obtener más información, consulte las diferencias detalladas entre los niveles de servicio en las páginas [Azure SQL Database (núcleo virtual)](resource-limits-vcore-single-databases.md), [bases de datos únicas en Azure SQL Database (DTU)](resource-limits-dtu-single-databases.md), [bases de datos agrupadas en Azure SQL Database (DTU)](resource-limits-dtu-single-databases.md), e [Instancia administrada de Azure SQL](../managed-instance/resource-limits.md).

> [!NOTE]
> Para obtener información sobre el nivel de servicio hiperescala en el modelo de compra basado en núcleo virtual, consulte el [nivel de servicio de hiperescala](service-tier-hyperscale.md). Para ver una comparación entre el modelo de compra basado en núcleo virtual y el modelo de compra basado en DTU, consulte [Modelos de compra y recursos](purchasing-models.md).

## <a name="data-and-log-storage"></a>Almacenamiento de datos y de registro

Los siguientes factores afectan a la cantidad de almacenamiento utilizado para los datos y los archivos de registro, y se aplican a De uso general y Crítico para la empresa. Para información detallada sobre el almacenamiento de datos y registros en Hiperescala, consulte [Nivel de servicio Hiperescala](service-tier-hyperscale.md).

- El almacenamiento asignado lo usan los archivos de datos (MDF) y los archivos de registro (LDF).
- Cada tamaño de proceso de una base de datos única admite un tamaño máximo de base de datos, con un tamaño máximo predeterminado de 32 GB.
- Cuando se configura el tamaño de base de datos única requerido (el tamaño del archivo MDF), el 30 % de almacenamiento adicional se agrega automáticamente para admitir archivos LDF.
- Puede seleccionar cualquier tamaño de base de datos única entre 10 GB y el máximo admitido.
  - En el caso del almacenamiento en los niveles de servicio estándar o de uso general, aumente o disminuya el tamaño en incrementos de 10 GB.
  - Para el almacenamiento en niveles de servicio premium o crítico para la empresa, aumente o disminuya el tamaño en incrementos de 250 GB.
- En el nivel de servicio de uso general, `tempdb` usa una SSD asociada y este costo de almacenamiento se incluye en el precio del núcleo virtual.
- En el nivel de servicio crítico para la empresa, `tempdb` comparte la SSD asociada con los archivos MDF y LDF, y el costo de almacenamiento de `tempdb` se incluye en el precio del núcleo virtual.
- En el nivel de servicio de DTU Premium, `tempdb` comparte la SSD conectada con archivos MDF y LDF.
- El tamaño de almacenamiento para una Instancia administrada de SQL debe especificarse en múltiplos de 32 GB.


> [!IMPORTANT]
> Se le cobra por el almacenamiento total asignado para los archivos MDF y LDF.

Para supervisar el tamaño total actual de los archivos MDF y LDF, use [sp_spaceused](/sql/relational-databases/system-stored-procedures/sp-spaceused-transact-sql). Para supervisar el tamaño actual de los archivos MDF y LDF individuales, use [sys.database_files](/sql/relational-databases/system-catalog-views/sys-database-files-transact-sql).

> [!IMPORTANT]
> En algunas circunstancias, puede que deba reducir una base de datos para reclamar el espacio no utilizado. Para obtener más información, consulte [Administración del espacio de archivo en Azure SQL Database](file-space-manage.md).

## <a name="backups-and-storage"></a>Copias de seguridad y almacenamiento

Para admitir las funcionalidades de restauración a un momento dado (PITR) y [retención a largo plazo (LTR)](long-term-retention-overview.md) de SQL Database e Instancia administrada de SQL, se asigna almacenamiento a las copias de seguridad de base de datos. Este almacenamiento se asigna por separado para cada base de datos y se factura como dos cargos independientes por base de datos.

- **PITR**: las copias de seguridad de base de datos individuales se copian en el [almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)](../../storage/common/geo-redundant-design.md) automáticamente. El tamaño de almacenamiento aumenta dinámicamente a medida que se crean nuevas copias de seguridad. El almacenamiento se usa para realizar cada cinco minutos copias de seguridad completas semanales, copias de seguridad diferenciales diarias y copias de seguridad de registros de transacciones. El consumo de almacenamiento depende de la tasa de cambio de la base de datos y del período de retención de las copias de seguridad. Puede configurar un período de retención diferente para cada base de datos de entre 7 y 35 días. Se ofrece una cantidad de almacenamiento mínimo igual al 100 % (1x) del tamaño de la base de datos sin costo adicional. En la mayoría de las bases de datos, esta cantidad es suficiente para almacenar copias de seguridad durante 7 días.
- **LTR**: también tiene la opción de configurar la retención a largo plazo de copias de seguridad completas hasta un máximo de 10 años para [SQL Managed Instance](long-term-retention-overview.md). Si ha instalado la directiva de LTR, estas copias de seguridad se almacenan en almacenamiento RA-GRS automáticamente, pero puede controlar la frecuencia con que se realizan las copias de seguridad. Para satisfacer los distintos requisitos de cumplimiento, puede seleccionar distintos períodos de retención para copias de seguridad semanales, mensuales o anuales. La configuración que elija determina la cantidad de almacenamiento que se usará para las copias de seguridad de LTR. Para estimar el costo del almacenamiento de LTR, se puede usar la calculadora de precios de LTR. Para más información, consulte [SQL Database long-term retention](long-term-retention-overview.md) (Retención a largo plazo de SQL Database).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los tamaños específicos de proceso y almacenamiento disponibles en los niveles de servicio De uso general y Crítico para la empresa, consulte: 

- [Límites de recursos basados en núcleos virtuales de Azure SQL Database](resource-limits-vcore-single-databases.md).
- [Límites de recursos basados en núcleos virtuales para bases de datos agrupadas en Azure SQL Database](resource-limits-vcore-elastic-pools.md).
- [Límites de recursos basados en núcleos virtuales para Instancia administrada de Azure SQL](../managed-instance/resource-limits.md).
