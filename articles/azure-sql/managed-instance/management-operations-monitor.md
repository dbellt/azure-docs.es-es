---
title: Supervisión de operaciones de administración
titleSuffix: Azure SQL Managed Instance
description: Conozca las distintas formas de supervisar las operaciones de administración de Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, bonova, MashaMSFT
ms.date: 09/03/2020
ms.openlocfilehash: df11946ccc76e59a858b365ff3fe9b8fac833e92
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689685"
---
# <a name="monitoring-azure-sql-managed-instance-management-operations"></a>Supervisión de operaciones de administración de Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance permite la supervisión de [operaciones de administración](management-operations-overview.md) que se usan para implementar nuevas instancias administradas, actualizar propiedades de instancias o eliminar instancias que ya no son necesarias. 

## <a name="overview"></a>Información general

Todas las operaciones de administración se pueden clasificar de la siguiente manera:

- Implementación de una instancia (creación de instancia nueva).
- Actualización de una instancia (cambio de las propiedades de una instancia, como núcleos virtuales o almacenamiento reservado).
- Eliminación de una instancia.

La mayoría de las operaciones de administración son [operaciones de larga duración](management-operations-overview.md#duration). Por lo tanto, es necesario supervisar el estado o seguir el progreso de los pasos de la operación. 

Hay varias maneras de supervisar las operaciones de administración de instancias administradas:

- [Implementaciones de grupos de recursos](../../azure-resource-manager/templates/deployment-history.md)
- [Registro de actividad](../../azure-monitor/essentials/activity-log.md)
- [API de operaciones de instancia administrada](#managed-instance-operations-api)


En la tabla siguiente se comparan las opciones de supervisión de operaciones de administración: 

| Opción | Retención | Admite cancelación | Crear | Actualizar | Eliminar | Cancelar | Pasos |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Implementaciones de grupos de recursos | Infinite<sup>1</sup> | No<sup>2</sup> | Visible | Visible | No es visible | Visible | No es visible |
| Registro de actividades | 90 días | No | Visible | Visible | Visible | Visible |  No es visible |
| API de operaciones de instancia administrada | 24 horas | [Sí](management-operations-cancel.md) | Visible | Visible | Visible | Visible | Visible |
|  |  |  |  |  |  |  | |

<sup>1</sup> El historial de implementaciones de un grupo de recursos está limitado a 800 implementaciones.

<sup>2</sup> Las implementaciones de grupos de recursos admiten la operación de cancelación. Sin embargo, debido a la lógica de cancelación, solo se cancelará una operación programada para la implementación después de realizar la acción de cancelación. La implementación en curso no se cancela cuando se cancela la implementación del grupo de recursos. Como la implementación de instancia administrada consiste en un paso de larga duración (desde la perspectiva de Azure Resource Manager), la cancelación de la implementación del grupo de recursos no cancelará la implementación de la instancia administrada y la operación se completará. 

## <a name="managed-instance-operations-api"></a>API de operaciones de instancia administrada

Las API de operaciones de administración están diseñadas especialmente para supervisar las operaciones. La supervisión de operaciones de instancia administrada puede ofrecer información sobre los parámetros de la operación y los pasos de la operación, además de [cancelar operaciones específicas](management-operations-cancel.md). Además de los detalles de la operación y el comando de cancelación, esta API se puede usar en scripts de automatización con implementaciones de varios recursos; según el paso del progreso, se puede iniciar una implementación de recursos dependiente.

Estas son las API: 

| Comando | Descripción |
| --- | --- |
|[Operaciones de Instancia administrada de SQL Database: obtener](/rest/api/sql/managedinstanceoperations/get)|Obtiene una operación de administración en una instancia administrada.|
|[Operaciones de Instancia administrada de SQL Database: cancelar](/rest/api/sql/managedinstanceoperations/cancel)|Cancela la operación asincrónica en la instancia administrada.|
|[Operaciones de Instancia administrada de SQL Database: mostrar por instancia administrada](/rest/api/sql/managedinstanceoperations/listbymanagedinstance)|Obtiene una lista de las operaciones realizadas en la instancia administrada.|

> [!NOTE]
> Use la versión 2020-02-02 de la API para ver la operación de creación de instancia administrada en la lista de operaciones. Se trata de la versión predeterminada que se usa en Azure Portal y los paquetes más recientes de PowerShell y la CLI de Azure.

## <a name="monitor-operations"></a>Supervisión de operaciones

# <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, use la página de **información general** de la instancia administrada para supervisar las operaciones de instancia administrada. 

Por ejemplo, la **operación de creación** está visible al inicio del proceso de creación en la página de **información general**: 

![Progreso de la creación de instancia administrada](./media/management-operations-monitor/monitoring-create-operation.png)

Seleccione **Operación en curso** para abrir la página **Operación en curso** y ver las operaciones de **creación** o **actualización**. También puede [Cancelar](management-operations-cancel.md) operaciones desde esta página.  

![Detalles de la operación de instancia administrada](./media/management-operations-monitor/monitoring-operation-details.png)

> [!NOTE]
> Las operaciones de creación enviadas a través de Azure Portal, PowerShell, la CLI de Azure u otras herramientas que usan la API de REST versión 2020-02-02 [pueden cancelarse](management-operations-cancel.md). Las versiones de la API de REST anteriores a 2020-02-02 que se usen para enviar una operación de creación iniciarán la implementación de la instancia, pero la implementación no se mostrará en la API de operaciones y no se podrá cancelar.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

El cmdlet Get-AzSqlInstanceOperation obtiene información sobre las operaciones de una instancia administrada. Puede ver todas las operaciones de una instancia administrada o ver una operación específica proporcionando el nombre de la operación.

```powershell-interactive
$managedInstance = "yourInstanceName"
$resourceGroup = "yourResourceGroupName"

$managementOperations = Get-AzSqlInstanceOperation `
    -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup
```

Para obtener una explicación detallada de los comandos, consulte [Get-AzSqlInstanceOperation](/powershell/module/az.sql/get-azsqlinstanceoperation).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En la lista az sql mi op se obtiene una lista de las operaciones realizadas en la instancia administrada. Si no ha instalado aún la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName 
```

Para obtener una explicación detallada de los comandos, consulte [az sql mi op](/cli/azure/sql/mi/op).

---

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
