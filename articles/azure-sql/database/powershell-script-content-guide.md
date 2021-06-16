---
title: Ejemplo de script de Azure PowerShell
description: Use ejemplos de scripts de Azure PowerShell que le ayudarán a crear y administrar recursos de Instancia administrada de Azure SQL y Azure SQL Database.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: PowerShell
ms.topic: sample
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 03/25/2019
ms.openlocfilehash: c68c94c8c4a8e5f9e80dbd69130c611a270d21d9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699386"
---
# <a name="azure-powershell-samples-for-azure-sql-database-and-azure-sql-managed-instance"></a>Ejemplos de Azure PowerShell para Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database e Instancia administrada de Azure SQL le permite configurar las bases de datos, instancias y grupos mediante Azure PowerShell.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 de AZ PowerShell o posterior. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

En la tabla siguiente se incluyen vínculos a scripts de Azure PowerShell para Azure SQL Database.

|Vínculo|Descripción|
|---|---|
|**Creación y configuración de bases de datos únicas y grupos elásticos**||
| [Creación de una base de datos única y configuración de una regla de firewall de nivel de servidor](scripts/create-and-configure-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell crea una base de datos única y configura una regla de firewall IP en el nivel de servidor. |
| [Creación de grupos elásticos y traslado de bases de datos agrupadas](scripts/move-database-between-elastic-pools-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell crea grupos elásticos, traslada las bases de datos agrupadas y cambia los tamaños de proceso.|
|**Configuración de la replicación geográfica y de la conmutación por error**||
| [Configuración y conmutación por error de una base de datos única mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell configura la replicación geográfica activa para una base de datos única y la conmuta por error a la réplica secundaria. |
| [Configuración y conmutación por error de una base de datos agrupada mediante la replicación geográfica activa](scripts/setup-geodr-and-failover-elastic-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell configura la replicación geográfica activa para una base de datos de un grupo elástico y la conmuta por error a la réplica secundaria. |
|**Configuración de un grupo de conmutación por error**||
| [Configuración de un grupo de conmutación por error para una sola base de datos](scripts/add-database-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell crea una base de datos y un grupo de conmutación por error, agrega la base de datos al grupo de conmutación por error y prueba la conmutación por error en el servidor secundario. |
| [Configuración de un grupo de conmutación por error para un grupo elástico](scripts/add-elastic-pool-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell crea una base de datos, la agrega a un grupo elástico, agrega el grupo elástico al grupo de conmutación por error y prueba la conmutación por error en el servidor secundario. |
|**Escalado de una base de datos única y un grupo elástico**||
| [Escalado de una base de datos única](scripts/monitor-and-scale-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell supervisa las métricas de rendimiento de una base de datos única, la escala a un tamaño de proceso mayor y crea una regla de alerta en una de las métricas de rendimiento. |
| [Escalado de un grupo elástico](scripts/monitor-and-scale-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell supervisa las métricas de rendimiento de un grupo elástico, lo escala a un tamaño de proceso mayor y crea una regla de alertas en una de las métricas de rendimiento. |
| **Detección de amenazas y auditoría** |
| [Configuración de detección de amenazas y auditoría](scripts/auditing-threat-detection-powershell-configure.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell configura las directivas de auditoría y detección de amenazas para una base de datos. |
| **Restauración, copia e importación de una base de datos**||
| [Restauración de una base de datos](scripts/restore-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell restaura una base de datos desde una copia de seguridad con redundancia geográfica y restaura una base de datos eliminada a la copia de seguridad más reciente. |
| [Copia de una base de datos en un nuevo servidor](scripts/copy-database-to-new-server-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell crea una copia de una base de datos existente en un nuevo servidor. |
| [Importación de una base de datos desde un archivo bacpac](scripts/import-from-bacpac-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell importa una base de datos a una base de datos de Azure SQL Database desde un archivo bacpac. |
| **Sincronización de datos entre bases de datos**||
| [Sincronización de datos entre bases de datos](scripts/sql-data-sync-sync-data-between-sql-databases.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell configura la sincronización de datos para realizar la sincronización entre varias bases de datos de Azure SQL Database. |
| [Sincronización de datos entre la base de datos SQL Database y SQL Server local](scripts/sql-data-sync-sync-data-between-azure-onprem.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell configura la sincronización de datos entre una base de datos de Azure SQL Database y una base de datos de SQL Server local. |
| [Actualización del esquema de sincronización de SQL Data Sync](scripts/update-sync-schema-in-sync-group.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell agrega o quita elementos del esquema de sincronización de Data Sync. |
|||

Obtenga más información sobre la [API de la base de datos única de Azure PowerShell](single-database-manage.md#powershell). 

## <a name="azure-sql-managed-instance"></a>[Instancia administrada de Azure SQL](#tab/managed-instance)

En la tabla siguiente se incluyen vínculos a ejemplos de scripts de Azure PowerShell para Instancia administrada de Azure SQL.

|Vínculo|Descripción|
|---|---|
|**Creación y configuración de instancias administradas**||
| [Creación y administración de una instancia administrada](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | Este script de PowerShell muestra cómo crear y administrar una instancia administrada mediante Azure PowerShell. |
| [Creación y administración de una instancia administrada con una plantilla de Azure Resource Manager](../managed-instance/create-template-quickstart.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell muestra cómo crear y administrar una instancia administrada mediante Azure PowerShell y una plantilla de Azure Resource Manager.|
| [Restauración de una base de datos a una instancia administrada de otra región con replicación geográfica](../managed-instance/scripts/restore-geo-backup.md) | Este script de PowerShell realiza una copia de seguridad de una base de datos y la restaura en otra región. Esto se conoce como escenario de recuperación ante desastres de una restauración geográfica. |
| **Configuración del cifrado de datos transparente**||
| [Administración del cifrado de datos transparente en una instancia administrada con la propia clave desde Azure Key Vault](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Este script de PowerShell configura el cifrado de datos transparente en el escenario Bring Your Own Key para Instancia administrada de Azure SQL, con una clave de Azure Key Vault.|
|**Configuración de un grupo de conmutación por error**||
| [Configuración de un grupo de conmutación por error para una instancia administrada](../managed-instance/scripts/add-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Este script de PowerShell crea dos instancias administradas, las agrega a un grupo de conmutación por error y, a continuación, prueba la conmutación por error de la instancia administrada principal en la instancia administrada secundaria. |
|||

Obtenga más información sobre [cmdlets de PowerShell para Instancia administrada de Azure SQL](../managed-instance/api-references-create-manage-instance.md#powershell-create-and-configure-managed-instances).

---

## <a name="additional-resources"></a>Recursos adicionales

Los ejemplos que se enumeran en esta página utilizan los [cmdlets de PowerShell](/powershell/module/az.sql/) para crear y administrar recursos de SQL Azure. En el módulo [sqlserver](/powershell/module/sqlserver/) se encuentran otros cmdlets para ejecutar consultas y realizar varias tareas de base de datos. Para más información, consulte el artículo sobre [SQL Server PowerShell](/sql/powershell/sql-server-powershell/).