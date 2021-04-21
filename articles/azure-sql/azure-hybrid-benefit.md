---
title: Ventaja híbrida de Azure
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Use licencias de SQL Server existentes para los descuentos de Azure SQL Database e Instancia administrada de SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=4
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake
ms.date: 02/16/2021
ms.openlocfilehash: b5f85e0dcb8ca70d5773b8f1c3b53e0b449ef013
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779208"
---
# <a name="azure-hybrid-benefit---azure-sql-database--sql-managed-instance"></a>Ventaja híbrida de Azure: Azure SQL Database e Instancia administrada de SQL
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

En el nivel de proceso aprovisionado del modelo de compra basado en núcleo virtual, puede intercambiar las licencias existentes por tarifas con descuento en Azure SQL Database e Instancia administrada de Azure SQL mediante la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). Esta ventaja de Azure le permite ahorrar hasta un 30 % o incluso más en SQL Database e Instancia administrada de SQL al usar las licencias de SQL Server con Software Assurance. En la página [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) hay una calculadora para determinar el ahorro.  Tenga en cuenta que Ventaja híbrida de Azure no se aplica a Azure SQL Database sin servidor.

> [!NOTE]
> Cambiar a Ventaja híbrida de Azure no requiere ningún tiempo de inactividad.

![Estructura de precios de núcleo virtual](./media/azure-hybrid-benefit/pricing.png)

## <a name="choose-a-license-model"></a>Elija un modelo de licencia

Con la Ventaja híbrida de Azure, puede elegir pagar solo por la infraestructura subyacente de Azure mediante su licencia existente de SQL Server para el motor de base de datos de SQL Server (precios de proceso básicos), o pagar tanto por la infraestructura subyacente como por la licencia de SQL Server (precio con licencia incluida).

Puede elegir o cambiar el modelo de licencias en Azure Portal: 
- En el caso de las bases de datos nuevas, seleccione **Configurar base de datos** durante el proceso de creación en la pestaña **Conceptos básicos** y seleccione la opción para ahorrar dinero.
- En el caso de las bases de datos existentes, seleccione **Configurar** en el menú **Configuración** y seleccione la opción para ahorrar dinero.

También puede configurar una base de datos nueva o existente mediante una de las siguientes API:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para establecer o actualizar el tipo de licencia mediante PowerShell:

- [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)
- [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)
- [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance)
- [Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para establecer o actualizar el tipo de licencia mediante la CLI de Azure:

- [az sql db create](/cli/azure/sql/db#az_sql_db_create)
- [az sql mi create](/cli/azure/sql/mi#az_sql_mi_create)
- [az sql mi update](/cli/azure/sql/mi#az_sql_mi_update)

# <a name="rest-api"></a>[REST API](#tab/rest)

Para establecer o actualizar el tipo de licencia mediante la API REST:

- [Databases - Create Or Update](/rest/api/sql/databases/createorupdate)
- [Databases - Update](/rest/api/sql/databases/update)
- [Managed Instances - Create Or Update](/rest/api/sql/managedinstances/createorupdate)
- [Managed Instances - Update](/rest/api/sql/managedinstances/update)

* * *


### <a name="azure-hybrid-benefit-questions"></a>Preguntas de la Ventaja híbrida de Azure

#### <a name="are-there-dual-use-rights-with-azure-hybrid-benefit-for-sql-server"></a>¿Hay derechos de doble uso con Ventaja híbrida de Azure para SQL Server?

Dispone de 180 días de derechos de doble uso de la licencia para asegurarse de que las migraciones se ejecutan sin problemas. Después de ese período de 180 días, solo puede usar la licencia de SQL Server en la nube en SQL Database. Ya no tiene derechos de uso dual local y en la nube.

#### <a name="how-does-azure-hybrid-benefit-for-sql-server-differ-from-license-mobility"></a>¿En qué se diferencia la Ventaja híbrida de Azure para SQL Server de la movilidad de licencias?

Ofrecemos ventajas de movilidad de licencias a los clientes de SQL Server con Software Assurance. Esto permite la reasignación de sus licencias a los servidores compartidos de un asociado. Puede usar esta ventaja en IaaS de Azure y AWS EC2.

La Ventaja híbrida de Azure para SQL Server se diferencia de la movilidad de licencias en dos áreas principales:

- Proporciona ventajas económicas para mover cargas de trabajo muy virtualizadas a Azure. Los clientes de SQL Server Enterprise Edition pueden obtener cuatro núcleos en Azure en la SKU de uso general por cada núcleo que posean en el entorno local para aplicaciones muy virtualizadas. La movilidad de licencias no ofrece ninguna ventaja especial sobre los costos de mover cargas de trabajo virtualizadas a la nube.
- Se proporciona para destinos PaaS en Azure (Instancia administrada de SQL) que son muy compatibles con SQL Server.

#### <a name="what-are-the-specific-rights-of-the-azure-hybrid-benefit-for-sql-server"></a>¿Cuáles son los derechos específicos de la Ventaja híbrida de Azure para SQL Server?

Los clientes de SQL Database y SQL Managed Instance tendrán asociados los siguientes derechos con la Ventaja híbrida de Azure para SQL Server:

|Superficie de licencia|¿Qué le permite obtener la Ventaja híbrida de Azure para SQL Server?|
|---|---|
|Clientes de núcleo de SQL Server Enterprise Edition con SA|<li>Puede pagar la tasa base sobre la SKU de Hiperescala, De uso general o Crítico para la empresa</li><br><li>1 núcleo local = 4 núcleos en la SKU de Hiperescala</li><br><li>1 núcleo local = 4 núcleos en la SKU De uso general</li><br><li>1 núcleo local = 1 núcleo en SKU Crítico para la empresa</li>|
|Clientes de núcleo de SQL Server Standard Edition con SA|<li>Puede pagar la tasa base sobre la SKU de Hiperescala, De uso general o Crítico para la empresa</li><br><li>1 núcleo local = 1 núcleo en la SKU de Hiperescala</li><br><li>1 núcleo local = 1 núcleo en la SKU De uso general</li><br><li>4 núcleos locales = 1 núcleo en SKU Crítico para la empresa</li>|
|||


## <a name="next-steps"></a>Pasos siguientes

- Para ayudarle a elegir una opción de implementación de Azure SQL, consulte [Elección de la opción de implementación adecuada en Azure SQL](azure-sql-iaas-vs-paas-what-is-overview.md).
- Para obtener una comparación de las características de SQL Database e Instancia administrada de SQL, vea [Características de SQL Database e Instancia administrada de SQL](database/features-comparison.md).
