---
title: Inicio manual de una conmutación por error en SQL Managed Instance
description: Obtenga información sobre cómo realizar manualmente la conmutación por error de las réplicas principal y secundaria en Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: seo-lt-2019, sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: mathoma
ms.date: 02/27/2021
ms.openlocfilehash: f66719ab3d7482fcc1df1665593d276a738257a9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697234"
---
# <a name="user-initiated-manual-failover-on-sql-managed-instance"></a>Conmutación por error manual iniciada por el usuario en SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica cómo realizar una conmutación por error manual de un nodo principal en los niveles de servicio De uso general (GP) y Crítico para la empresa (BC) de SQL Managed Instance, y cómo realizar una conmutación por error manual de un nodo de réplica de solo lectura secundario únicamente en el nivel de servicio BC.

## <a name="when-to-use-manual-failover"></a>Cuando se debe usar la conmutación por error manual

La [alta disponibilidad](../database/high-availability-sla.md) es una parte fundamental de la plataforma SQL Managed Instance que funciona de modo transparente para las aplicaciones de base de datos. Las conmutaciones por error desde nodos principales a secundarios en caso de degradación del nodo o de detección de errores, o durante las actualizaciones de software mensuales habituales son hechos esperados para todas las aplicaciones que usan SQL Managed Instance en Azure.

Puede considerar la posibilidad de ejecutar una [conmutación por error manual](../database/high-availability-sla.md#testing-application-fault-resiliency) en SQL Managed Instance por algunos de los siguientes motivos:
- Probar la resistencia de la conmutación por error de la aplicación antes de realizar la implementación en producción
- Probar la resistencia frente a errores de sistemas de un extremo a otro en conmutaciones por error automáticas
- Probar cómo afecta la conmutación por error a las sesiones de base de datos existentes
- Comprobar si una conmutación por error cambia el rendimiento de un extremo a otro debido a los cambios en la latencia de la red
- En algunos casos de degradación del rendimiento de las consultas, la conmutación por error manual puede ayudar a mitigar el problema de rendimiento.

> [!NOTE]
> Asegurarse de que las aplicaciones sean resistentes a la conmutación por error antes de la implementación en producción ayudará a mitigar el riesgo de errores de la aplicación en producción y contribuirá a la disponibilidad de la aplicación para los clientes. Puede encontrar más información sobre cómo probar la preparación de las aplicaciones para la nube con la grabación de vídeo [Prueba de la preparación de las aplicaciones en la nube para la resistencia a la conmutación por error con SQL Managed Instance](https://youtu.be/FACWYLgYDL8).

## <a name="initiate-manual-failover-on-sql-managed-instance"></a>Inicio manual de la conmutación por error en SQL Managed Instance

### <a name="azure-rbac-permissions-required"></a>Se requieren permisos de RBAC de Azure

El usuario que inicia una conmutación por error debe tener uno de los roles de Azure siguientes:

- Rol Propietario de la suscripción, o
- Rol [Colaborador de Managed Instance](../../role-based-access-control/built-in-roles.md#sql-managed-instance-contributor) o
- Rol personalizado con el permiso siguiente:
  - `Microsoft.Sql/managedInstances/failover/action`

### <a name="using-powershell"></a>Usar PowerShell

La versión mínima de Az.Sql debe ser [v2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0). Considere la posibilidad de usar [Azure Cloud Shell](../../cloud-shell/overview.md) del Azure Portal que siempre tiene disponible la versión más reciente de PowerShell. 

Como requisito previo, use el siguiente script de PowerShell para instalar los módulos de Azure necesarios. Además, seleccione la suscripción en la que se encuentra la Instancia administrada de la que quiere realizar la conmutación por error.

```powershell
$subscription = 'enter your subscription ID here'
Install-Module -Name Az
Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount
Select-AzSubscription -SubscriptionId $subscription
```

Use el comando de PowerShell [Invoke-AzSqlInstanceFailover](/powershell/module/az.sql/invoke-azsqlinstancefailover) con el ejemplo siguiente para iniciar la conmutación por error del nodo principal, aplicable al nivel de servicio BC y GP.

```powershell
$ResourceGroup = 'enter resource group of your MI'
$ManagedInstanceName = 'enter MI name'
Invoke-AzSqlInstanceFailover -ResourceGroupName $ResourceGroup -Name $ManagedInstanceName
```

Use el siguiente comando de PS para conmutar por error el nodo secundario de lectura, aplicable solo al nivel de servicio BC.

```powershell
$ResourceGroup = 'enter resource group of your MI'
$ManagedInstanceName = 'enter MI name'
Invoke-AzSqlInstanceFailover -ResourceGroupName $ResourceGroup -Name $ManagedInstanceName -ReadableSecondary
```

### <a name="using-cli"></a>Uso de CLI

Asegúrese de tener instalados los scripts más recientes de la CLI.

Use el comando az sql mi failover de la CLI con el siguiente ejemplo para iniciar la conmutación por error del nodo principal, aplicable al nivel de servicio BC y GP.

```cli
az sql mi failover -g myresourcegroup -n myinstancename
```

Use el siguiente comando de la CLI para conmutar por error el nodo secundario de lectura, aplicable solo al nivel de servicio BC.

```cli
az sql mi failover -g myresourcegroup -n myinstancename --replica-type ReadableSecondary
```

### <a name="using-rest-api"></a>Uso de API REST

Para los usuarios avanzados, que quizás necesiten automatizar las conmutaciones por error de sus SQL Managed Instance con el fin de implementar una canalización de pruebas continua, o mitigadores de rendimiento automatizados, esta función se puede lograr mediante el inicio de la conmutación por error a través de una llamada de API. Consulte [Instancias administradas: API de REST de conmutación por error](/rest/api/sql/managed%20instances%20-%20failover/failover) para obtener más información.

Para iniciar la conmutación por error mediante una llamada API de REST, primero genere el token de autenticación con el cliente de API de su elección. El token de autenticación generado se usa como propiedad de autorización en el encabezado de la solicitud de API y es obligatorio.

El siguiente código es un ejemplo de URI de API de llamada:

```HTTP
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Sql/managedInstances/{managedInstanceName}/failover?api-version=2019-06-01-preview
```

Las siguientes propiedades deben pasarse en la llamada API:

| **Propiedad de API** | **Parámetro** |
| --- | --- |
| subscriptionId | Id. de suscripción en el que se implementa la instancia administrada. |
| resourceGroupName | Grupo de recursos que contiene la instancia administrada. |
| managedInstanceName | Nombre de instancia administrada. |
| replicaType | (Opcional) (Primary o ReadableSecondary). Estos parámetros representan el tipo de réplica que se va a conmutar por error: principal o secundaria legible. Si no se especifica, la conmutación por error se iniciará en la réplica principal de forma predeterminada. |
| api-version | Valor estático y actualmente debe ser "2019-06-01-preview". |

La respuesta de la API será una de las dos siguientes:

- 202 - Aceptado
- Uno de los errores de solicitud 400.

Se puede realizar un seguimiento del estado de la operación mediante la revisión de las respuestas de la API en los encabezados de respuesta. Para obtener más información, consulte [Estado de las operaciones asincrónicas de Azure](../../azure-resource-manager/management/async-operations.md).

## <a name="monitor-the-failover"></a>Supervisión de la conmutación por error

Para supervisar el progreso de la conmutación por error iniciada por el usuario para la instancia de BC, ejecute la siguiente consulta T-SQL en su cliente favorito (como SSMS) en SQL Managed Instance. Leerá la vista del sistema sys.dm_hadr_fabric_replica_states y las réplicas de informe disponibles en la instancia. Actualice la misma consulta después de iniciar la conmutación por error manual.

```T-SQL
SELECT DISTINCT replication_endpoint_url, fabric_replica_role_desc FROM sys.dm_hadr_fabric_replica_states
```

Antes de iniciar la conmutación por error, el resultado indicará la réplica principal actual en el nivel de servicio BC que contiene una principal y tres secundarias en el grupo de disponibilidad AlwaysOn. Tras la ejecución de una conmutación por error, para volver a ejecutar esta consulta necesitará indicar un cambio del nodo principal.

No podrá ver el mismo resultado con el nivel de servicio GP que el anterior que se muestra para BC. Esto se debe a que el nivel de servicio GP se basa en un solo nodo. Puede usar una consulta T-SQL alternativa que muestre la hora en que se inició el proceso SQL en el nodo de la instancia de nivel de servicio de GP:

```T-SQL
SELECT sqlserver_start_time, sqlserver_start_time_ms_ticks FROM sys.dm_os_sys_info
```

La breve pérdida de conectividad del cliente durante la conmutación por error, que normalmente dura menos de un minuto, será la indicación de la ejecución de la conmutación por error independientemente del nivel de servicio.

> [!NOTE]
> La finalización del proceso de conmutación por error (no de la breve falta de disponibilidad real) puede tardar varios minutos en el caso de cargas de trabajo de **alta intensidad**. Esto se debe a que el motor de la instancia se encarga de todas las transacciones actuales en la principal y se pone al día en el nodo secundario, antes de poder realizar la conmutación por error.

> [!IMPORTANT]
> Limitaciones funcionales de la conmutación por error manual iniciada por el usuario:
> - Podría haber una (1) conmutación por error iniciada en la misma Instancia administrada cada **15 minutos**.
> - En el caso de las instancias de BC, debe existir cuórum de réplicas para que se acepte la solicitud de conmutación por error.
> - En el caso de las instancias de BC, no es posible especificar en qué réplica secundaria legible se iniciará la conmutación por error.
> - No se permitirá la conmutación por error hasta que los sistemas de copia de seguridad automatizada completen la primera copia de seguridad completa para una nueva base de datos.
> - No se permitirá la conmutación por error si existe una restauración de la base de datos en curso.

## <a name="next-steps"></a>Pasos siguientes
- Puede encontrar más información sobre cómo probar la preparación de las aplicaciones para la nube con la grabación de vídeo [Prueba de la preparación de las aplicaciones en la nube para la resistencia a la conmutación por error con SQL Managed Instance](https://youtu.be/FACWYLgYDL8).
- Obtenga más información sobre la alta disponibilidad de la instancia administrada en [Alta disponibilidad para Azure SQL Managed Instance](../database/high-availability-sla.md).
- Para obtener información general, consulte [¿Qué es Azure SQL Managed Instance?](sql-managed-instance-paas-overview.md).
