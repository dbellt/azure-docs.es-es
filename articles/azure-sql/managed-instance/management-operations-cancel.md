---
title: Cancelación de operaciones de administración
titleSuffix: Azure SQL Managed Instance
description: Aprenda a cancelar las operaciones de administración de Azure SQL Managed Instance.
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
ms.openlocfilehash: 13e0190b330d1cae0d819ee28c9c8fabe98e8dcb
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689745"
---
# <a name="canceling-azure-sql-managed-instance-management-operations"></a>Cancelación de operaciones de administración de Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance ofrece la capacidad de cancelar algunas [operaciones de administración](management-operations-overview.md), como cuando se implementa una nueva instancia administrada o se actualizan las propiedades de una instancia. 

## <a name="overview"></a>Información general

 Todas las operaciones de administración se pueden clasificar de la siguiente manera:

- Implementación de una instancia (creación de instancia nueva).
- Actualización de una instancia (cambio de las propiedades de una instancia, como núcleos virtuales o almacenamiento reservado).
- Eliminación de una instancia.

Puede [supervisar el progreso y el estado de las operaciones de administración](management-operations-monitor.md) y cancelar algunas de ellas si es necesario. 

En la tabla siguiente se resumen las operaciones de administración, tanto si se pueden cancelar como si no, y su duración general típica:

Category  |Operación  |Cancelable  |Duración de cancelación estimada  |
|---------|---------|---------|---------|
|Implementación |Creación de instancias |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (De uso general) |No |  |
|Actualizar |Escalado o reducción vertical del almacenamiento de una instancia (Crítico para la empresa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (De uso general) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (Crítico para la empresa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Actualizar |Cambio en el nivel de servicio de una instancia (De uso general a Crítico para la empresa y viceversa) |Sí |El 90 % de las operaciones finaliza en 5 minutos. |
|Eliminar |Eliminación de una instancia |No |  |
|Eliminar |Eliminación de un clúster virtual (como operación iniciada por el usuario) |No |  |

## <a name="cancel-management-operation"></a>Cancelación de una operación de administración

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para cancelar operaciones de administración mediante Azure Portal, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Vaya a la hoja de **información general** del servicio SQL Managed Instance. 
1. Seleccione el cuadro **Notificación** que aparece junto a la operación en curso para abrir la página **Operación en curso**. 

   :::image type="content" source="media/management-operations-cancel/open-ongoing-operation.png" alt-text="Seleccione el cuadro Operación en curso para abrir la página Operación en curso.":::

1. Seleccione **Cancelar la operación** en la parte inferior de la página. 

   :::image type="content" source="media/management-operations-cancel/cancel-operation.png" alt-text="Seleccione Cancelar para cancelar la operación.":::

1. Confirme que quiere cancelar la operación. 


Si la solicitud de cancelación se realiza correctamente, se cancela la operación de administración y se produce un error. Recibirá una notificación de que la cancelación se realiza correctamente o no.

![Resultado de la operación de cancelación](./media/management-operations-cancel/canceling-operation-result.png)


Si no se puede cancelar la solicitud o el botón Cancelar no está activo, significa que la operación de administración entró en un estado que no se puede cancelar y que finalizará en breve.  La operación de administración continuará su ejecución hasta que se complete.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si aún no tiene instalado Azure PowerShell, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

Para cancelar la operación de administración, debe especificar el nombre de la operación. Por lo tanto, use primero el comando get para recuperar la lista de operaciones y luego cancele la operación específica.

```powershell-interactive
$managedInstance = "<Your-instance-name>"
$resourceGroup = "<Your-resource-group-name>"

$managementOperations = Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup

foreach ($mo in $managementOperations ) {
    if($mo.State -eq "InProgress" -and $mo.IsCancellable){
        $cancelRequest = Stop-AzSqlInstanceOperation -ResourceGroupName $resourceGroup -ManagedInstanceName $managedInstance -Name $mo.Name
        Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup -Name $mo.Name
    }
}
```

Para obtener una explicación detallada de los comandos, consulte [Get-AzSqlInstanceOperation](/powershell/module/az.sql/get-azsqlinstanceoperation) y [Stop-AzSqlInstanceOperation](/powershell/module/az.sql/stop-azsqlinstanceoperation).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Si no ha instalado aún la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Para cancelar la operación de administración, debe especificar el nombre de la operación. Por lo tanto, use primero el comando get para recuperar la lista de operaciones y luego cancele la operación específica.

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName |
   --query "[?state=='InProgress' && isCancellable].{Name: name}" -o tsv |
while read -r operationName; do

az sql mi op cancel -g yourResourceGroupName --mi yourInstanceName -n $operationName
done
```

Para obtener una explicación detallada de los comandos, consulte [az sql mi op](/cli/azure/sql/mi/op).

---

## <a name="canceled-deployment-request"></a>Solicitud de implementación cancelada

Con la versión 2020-02-02 de la API, en cuanto se acepta la solicitud de creación de la instancia, la instancia comienza a existir como un recurso, independientemente del progreso del proceso de implementación (el estado de la instancia administrada es **de aprovisionamiento**). Si cancela la solicitud de implementación de instancia (nueva creación de instancia), la instancia administrada pasará del estado **de aprovisionamiento** a **FailedToCreate**.

Las instancias que no se han podido crear todavía están presentes como recurso y: 

- No se cobran
- No cuentan a efectos de los límites de recursos (cuota de núcleo virtual o subred)


> [!NOTE]
> Para minimizar el ruido en la lista de recursos o instancias administradas, elimine las instancias que no se han implementado o las instancias con implementaciones canceladas. 


## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
