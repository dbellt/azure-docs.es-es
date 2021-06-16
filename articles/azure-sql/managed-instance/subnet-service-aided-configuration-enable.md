---
title: Habilitación de la configuración de subred asistida por servicio para Instancia administrada de Azure SQL
description: Habilitación de la configuración de subred asistida por servicio para Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.date: 03/12/2020
ms.openlocfilehash: 26f9dd2991f575669f580fc92cfc29259ca54be6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110707211"
---
# <a name="enabling-service-aided-subnet-configuration-for-azure-sql-managed-instance"></a>Habilitación de la configuración de subred asistida por servicio para Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La configuración de subred asistida por servicio proporciona administración de la configuración de red automatizada para las subredes que hospedan instancias administradas. Con la configuración de subred asistida por servicio, el usuario tiene control total sobre el acceso a datos (flujos de tráfico de TDS) mientras que la instancia administrada asume la responsabilidad de garantizar el flujo ininterrumpido de tráfico de administración para cumplir el Acuerdo de Nivel de Servicio.

Los grupos de seguridad de red y las reglas de tabla de enrutamiento configurados automáticamente son visibles para el cliente y se anotan con el prefijo _Microsoft.Sql-managedInstances_UseOnly_ _.

La configuración asistida por servicio se habilita automáticamente cuando se activa la [delegación de subred](../../virtual-network/subnet-delegation-overview.md) para el proveedor de recursos `Microsoft.Sql/managedInstances`.

> [!IMPORTANT] 
> Cuando la delegación de subred está activada, no puede desactivarla hasta que quite el último clúster virtual de la subred. Para obtener más información sobre cómo eliminar un clúster virtual, consulte el siguiente [artículo](virtual-cluster-delete.md#delete-a-virtual-cluster-from-the-azure-portal).

> [!NOTE] 
> Dado que la configuración de subred asistida por servicio es una característica esencial para el mantenimiento del Acuerdo de Nivel de Servicio, a partir del 1 de mayo de 2020, no se podrán implementar instancias administradas en subredes no delegadas en el proveedor de recursos de instancia administrada. El 1 de julio de 2020 todas las subredes que contienen instancias administradas se delegarán automáticamente al proveedor de recursos de instancia administrada. 

## <a name="enabling-subnet-delegation-for-new-deployments"></a>Habilitación de la delegación de subred para nuevas implementaciones
Para implementar la instancia administrada en una subred vacía, debe delegarla en el proveedor de recursos `Microsoft.Sql/managedInstances`, tal y como se describe en el siguiente [artículo](../../virtual-network/manage-subnet-delegation.md). _Tenga en cuenta que en el artículo de referencia se usa el proveedor de recursos `Microsoft.DBforPostgreSQL/serversv2` como ejemplo. En su lugar, deberá usar el proveedor de recursos `Microsoft.Sql/managedInstances`._

## <a name="enabling-subnet-delegation-for-existing-deployments"></a>Habilitación de la delegación de subred para implementaciones existentes

Para habilitar la delegación de subred para la implementación de instancia administrada existente, debe averiguar en qué subred de la red virtual se encuentra. 

Para averiguarlo, puede consultar `Virtual network/subnet` en la hoja del portal `Overview` de su instancia administrada.

Como alternativa, puede ejecutar los siguientes comandos de PowerShell para averiguarlo. Reemplace **subscription-id** por su identificador de suscripción. Reemplace también **rg-name** por el grupo de recursos de la instancia administrada y **mi-name** por el nombre de la instancia administrada.

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi.SubnetId
```

Cuando encuentre la subred de la instancia administrada, debe delegarla en el proveedor de recursos `Microsoft.Sql/managedInstances`, tal y como se describe en el siguiente [artículo](../../virtual-network/manage-subnet-delegation.md). _Tenga en cuenta que en el artículo de referencia se usa el proveedor de recursos `Microsoft.DBforPostgreSQL/serversv2` como ejemplo. En su lugar, deberá usar el proveedor de recursos `Microsoft.Sql/managedInstances`._


> [!IMPORTANT]
> La habilitación de la configuración asistida por servicio no provoca una conmutación por error ni una interrupción de la conectividad de las instancias administradas que ya están en la subred.
