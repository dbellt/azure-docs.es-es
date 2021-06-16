---
title: 'Plantillas de Azure Resource Manager: Azure SQL Database e Instancia administrada de Azure SQL'
description: Use las plantillas de Azure Resource Manager para crear y configurar Azure SQL Database e Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: deployment-configuration
ms.custom: overview-samples sqldbrb=2
ms.devlang: ''
ms.topic: guide
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: mathoma
ms.date: 05/24/2021
ms.openlocfilehash: 84a489222ed8d68c8c49d240cc22bd35f443227a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705720"
---
# <a name="azure-resource-manager-templates-for-azure-sql-database--sql-managed-instance"></a>Plantillas de Azure Resource Manager para Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Las plantillas de Azure Resource Manager le permiten definir la infraestructura como código e implementar sus soluciones en la nube de Azure para Azure SQL Database e Instancia administrada de Azure SQL.

## <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

En la tabla siguiente se incluyen vínculos a plantillas de Azure Resource Manager para Azure SQL Database.

|Vínculo |Descripción|
|---|---|
| [SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-database-transparent-encryption-create) | Esta plantilla de Azure Resource Manager crea una base de datos única en Azure SQL Database y configura las reglas del firewall IP en el nivel de servidor. |
| [Server](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-logical-server) | Esta plantilla de Azure Resource Manager crea un servidor para Azure SQL Database. |
| [Grupo elástico](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-elastic-pool-create) | Esta plantilla permite implementar un grupo elástico y asignarle bases de datos. |
| [Grupos de conmutación por error](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-with-failover-group) | Esta plantilla crea dos servidores, una base de datos única y un grupo de conmutación por error en Azure SQL Database.|
| [Detección de amenazas](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-threat-detection-db-policy-multiple-databases) | Esta plantilla permite implementar un servidor y un conjunto de bases de datos con Detección de amenazas habilitado, con una dirección de correo electrónico para las alertas de cada base de datos. Detección de amenazas forma parte de la oferta de Advanced Threat Protection (ATP) de SQL y proporciona una capa de seguridad que responda a amenazas potenciales a través de las bases de datos y servidores.|
| [Auditoría a Azure Blob Storage](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-auditing-server-policy-to-blob-storage) | Esta plantilla le permite implementar un servidor con la auditoría habilitada para escribir los registros de auditoría en un almacenamiento de blobs. La auditoría de Azure SQL Database y realiza un seguimiento de eventos de base de datos y los escribe en un registro de auditoría que se puede colocar en su cuenta de Azure Storage, en el área de trabajo de OMS o en Event Hubs.|
| [Auditoría a Azure Event Hub](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-auditing-server-policy-to-eventhub) | Esta plantilla le permite implementar un servidor con la auditoría habilitada para escribir los registros de auditoría en un centro de eventos existente. Para enviar eventos de auditoría a Event Hubs, establezca la configuración de auditoría con `Enabled` `State` y establezca `IsAzureMonitorTargetEnabled` como `true`. Configure también los valores de diagnóstico con la categoría de registros `SQLSecurityAuditEvents` en la base de datos `master` (para la auditoría de nivel de servidor). La auditoría realiza un seguimiento de los eventos de base de datos y los escribe en un registro de auditoría que se puede colocar en su cuenta de Azure Storage, en el área de trabajo de OMS o en Event Hubs.|
| [Azure Web App con SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/web-app-sql-database) | Este ejemplo crea una base de datos y una aplicación web de Azure gratuitas en Azure SQL Database, en el nivel de servicio "Básico".|
| [Azure Web App y Redis Cache con SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-redis-cache-sql-database) | Esta plantilla crea una aplicación web, una instancia de Redis Cache y una base de datos en el mismo grupo de recursos y crea dos cadenas de conexión en la aplicación web para la base de datos y Redis Cache.|
| [Importación de datos del almacenamiento de blobs mediante ADF V2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-v2-blob-to-sql-copy) | Esta plantilla de Azure Resource Manager crea una instancia de Azure Data Factory V2 que copia datos de Azure Blob Storage a SQL Database.|
| [Clúster de HDInsight con una base de datos](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.hdinsight/hdinsight-linux-with-sql-database) | Esta plantilla permite crear un clúster de HDInsight, un servidor SQL lógico, una base de datos y dos tablas. Esta plantilla se usa en el artículo [Uso de Apache Sqoop con Hadoop en HDInsight](../../hdinsight/hadoop/hdinsight-use-sqoop.md). |
| [Aplicación de lógica de Azure que ejecuta un procedimiento almacenado de SQL según una programación](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.logic/logic-app-sql-proc) | Esta plantilla permite crear una aplicación lógica que ejecutará un procedimiento almacenado de SQL de forma programada. Los argumentos para el procedimiento se pueden colocar en la sección del cuerpo de la plantilla.|

## <a name="azure-sql-managed-instance"></a>[Instancia administrada de Azure SQL](#tab/managed-instance)

En la tabla siguiente se incluyen vínculos a plantillas de Azure Resource Manager para Instancia administrada de Azure SQL.

|Vínculo|Descripción|
|---|---|
| [Instancia administrada de SQL en una red virtual nueva](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sqlmi-new-vnet) | Esta plantilla de Azure Resource Manager crea una nueva red virtual de Azure configurada y una instancia administrada en la red virtual. |
| [Entorno de red de Instancia administrada SQL](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-managed-instance-azure-environment) | Esta implementación creará una red virtual de Azure con dos subredes: una que se dedicará a las instancias administradas y otra en la que se pueden colocar otros recursos (por ejemplo, máquinas virtuales, entornos de App Service, etc.). Esta plantilla creará un entorno de red configurado correctamente en el que puede implementar instancias administradas. |
| [Instancia administrada de SQL con conexión P2S](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sqlmi-new-vnet-w-point-to-site-vpn) | Esta implementación creará una red virtual de Azure con dos subredes, `ManagedInstance` y `GatewaySubnet`. Instancia administrada de SQL se implementará en la subred ManagedInstance. Se creará una puerta de enlace de red virtual en la subred `GatewaySubnet` y se configurará para la conexión VPN de punto a sitio. |
| [Instancia administrada de SQL con una máquina virtual](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sqlmi-new-vnet-w-jumpbox) | Esta implementación creará una red virtual de Azure con dos subredes, `ManagedInstance` y `Management`. Instancia administrada de SQL se implementará en la subred `ManagedInstance`. La máquina virtual con la versión más reciente de SQL Server Management Studio (SSMS) se implementarán en la subred `Management`. |

---
