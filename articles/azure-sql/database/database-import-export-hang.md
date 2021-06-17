---
title: La importación y exportación de una base de datos tarda mucho tiempo
description: El servicio Import/Export de Azure SQL Database e Instancia administrada de Azure SQL tarda mucho tiempo en importar o exportar una base de datos
ms.custom: seo-lt-2019, sqldbrb=1
services: sql-database
ms.service: sql-db-mi
ms.subservice: data-movement
ms.topic: troubleshooting
author: v-miegge
ms.author: ramakoni
ms.reviewer: cawrites
ms.date: 09/27/2019
ms.openlocfilehash: a36fd722f38ec2253ada7e2f759597ea0df080a3
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110787377"
---
# <a name="azure-sql-database-and-managed-instance-importexport-service-takes-a-long-time-to-import-or-export-a-database"></a>El servicio Import/Export de Azure SQL Database e Instancia administrada tarda mucho tiempo en importar o exportar una base de datos

[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cuando se usa el servicio Import/Export, el proceso puede tardar más de lo esperado. En este artículo se describen las posibles causas de este retraso y métodos de solución alternativos.

## <a name="azure-sql-database-importexport-service"></a>Servicio Import/Export de Azure SQL Database

El servicio Import/Export de Azure SQL Database es un servicio web basado en REST que se ejecuta en cada centro de datos de Azure. Este es el servicio al que se llama cuando se usan las opciones [Importar base de datos](database-import.md#using-azure-portal) o [Exportar](./database-import.md#using-azure-portal) para trasladar la base de datos en Azure Portal. El servicio proporciona servicios de proceso y puesta en cola de solicitudes gratuitos para realizar importaciones y exportaciones entre Azure SQL Database y Azure Blob Storage.

Las operaciones de importación y exportación no representan una copia de seguridad de base de datos física tradicional, sino una copia de seguridad lógica de la base de datos que usa un formato de BACPAC especial. Este formato BACPAC le permite evitar tener que usar un formato físico que puede variar entre las versiones de Microsoft SQL Server, Azure SQL Database y Azure SQL Managed Instance.

## <a name="what-causes-delays-in-the-process"></a>¿Qué provoca retrasos en el proceso?

El servicio Import/Export de Azure SQL Database proporciona un número limitado de máquinas virtuales de proceso por región para procesar las operaciones de importación y exportación. La máquina virtual de proceso se hospeda por región para asegurarse de que la importación o exportación evita los cargos y los retrasos de ancho de banda entre regiones. Si se realizan demasiadas solicitudes al mismo tiempo en la misma región, se producen retrasos significativos en el procesamiento de las operaciones. El tiempo necesario para completar las solicitudes puede variar de unos segundos a varias horas.

> [!NOTE]
> Si una solicitud no se procesa en un plazo de cuatro días, el servicio la cancela automáticamente.

## <a name="recommended-solutions"></a>Soluciones recomendadas

Si las exportaciones de bases de datos se usan solo para la recuperación cuando hay una eliminación accidental de datos, todas las ediciones de Azure SQL Database proporcionan una funcionalidad de restauración de autoservicio a partir de las copias de seguridad generadas por el sistema. Pero si necesita estas exportaciones por otras razones, y si requiere un rendimiento de importación o exportación más rápido o más predecible, considere las siguientes opciones:

* [Exportar a un archivo BACPAC mediante la utilidad SQLPackage](./database-export.md#sqlpackage-utility).
* [Exportar a un archivo BACPAC mediante SQL Server Management Studio (SSMS)](./database-export.md#sql-server-management-studio-ssms).
* Ejecutar la importación o exportación de BACPAC directamente en el código mediante la API de Microsoft SQL Server Data-Tier Application Framework (DacFx). Para información adicional, consulte:
  * [Exportar una aplicación de la capa de datos](/sql/relational-databases/data-tier-applications/export-a-data-tier-application)
  * [Microsoft.SqlServer.Dac Namespace](/dotnet/api/microsoft.sqlserver.dac)
  * [Descarga de DacFx](https://www.microsoft.com/download/details.aspx?id=55713)

## <a name="things-to-consider-when-you-export-or-import-a-database"></a>Aspectos que se deben tener en cuenta al exportar o importar una base de datos

* Todos los métodos descritos en este artículo usan la cuota de unidad de transacción de base de datos (DTU), lo que provoca la limitación por parte del servicio Azure SQL Database. Puede [consultar las estadísticas de DTU de la base de datos en Azure Portal](./monitor-tune-overview.md#azure-sql-database-and-azure-sql-managed-instance-resource-monitoring). Si la base de datos ha alcanzado los límites de recursos, [actualice el nivel de servicio](./scale-resources.md) para agregar más recursos.
* Idealmente, debe ejecutar aplicaciones cliente (como la utilidad sqlpackage o la aplicación DAC personalizada) desde una máquina virtual que esté en la misma región que la base de datos. De lo contrario, podría experimentar problemas de rendimiento relacionados con la latencia de red.
* La exportación de tablas grandes sin índices agrupados puede ser un proceso muy lento o incluso producir un error. Este comportamiento se produce porque la tabla no se puede dividir y exportar en paralelo. En su lugar, debe exportarse en una única transacción, lo que provoca un rendimiento lento y un posible error durante la exportación, especialmente en el caso de tablas grandes.


## <a name="related-documents"></a>Documentos relacionados

[Consideraciones al exportar una base de datos](./database-export.md#considerations)