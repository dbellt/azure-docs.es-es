---
title: Implementación de Instancia administrada de SQL en un grupo de instancias
titleSuffix: Azure SQL Managed Instance
description: En este artículo se describe cómo crear y administrar grupos de Instancia administrada de Azure SQL (versión preliminar).
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: mathoma
ms.date: 09/05/2019
ms.openlocfilehash: 60afa287a96425ec0a3aead7e5affa6e046b7cbd
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689711"
---
# <a name="deploy-azure-sql-managed-instance-to-an-instance-pool"></a>Implementación de Instancia administrada de Azure SQL en un grupo de instancias
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se proporcionan detalles sobre cómo crear un [grupo de instancias](instance-pools-overview.md) y cómo implementar Instancia administrada de Azure SQL para ello. 

## <a name="instance-pool-operations"></a>Operaciones de grupos de instancias

En la tabla siguiente se muestran las operaciones disponibles relacionadas con los grupos de instancias y su disponibilidad en Azure Portal y PowerShell.

|Get-Help|Azure portal|PowerShell|
|:---|:---|:---|
|Creación de un grupo de instancias|No|Sí|
|Actualización de un grupo de instancias (número limitado de propiedades)|No |Sí |
|Comprobación del uso de un grupo de instancias y sus propiedades|No|Sí |
|Eliminación de un grupo de instancias|No|Sí|
|Creación de una instancia administrada dentro de un grupo de instancias|No|Sí|
|Actualización del uso de un recurso para una instancia administrada|Sí |Sí|
|Comprobación del uso y las propiedades de una instancia administrada|Sí|Sí|
|Eliminación de una instancia administrada del grupo|Sí|Sí|
|Creación de una base de datos en una instancia dentro del grupo|Sí|Sí|
|Eliminación de una base de datos en Instancia administrada de SQL|Sí|Sí|

[Comandos de PowerShell](/powershell/module/az.sql/) disponibles:

|Cmdlet |Descripción |
|:---|:---|
|[New-AzSqlInstancePool](/powershell/module/az.sql/new-azsqlinstancepool/) | Crea un grupo de Instancia administrada de SQL. |
|[Get-AzSqlInstancePool](/powershell/module/az.sql/get-azsqlinstancepool/) | Devuelve información sobre un grupo de instancias. |
|[Set-AzSqlInstancePool](/powershell/module/az.sql/set-azsqlinstancepool/) | Establece las propiedades de un grupo de instancias en Instancia administrada de SQL. |
|[Remove-AzSqlInstancePool](/powershell/module/az.sql/remove-azsqlinstancepool/) | Quita un grupo de instancias en Instancia administrada de SQL. |
|[Get-AzSqlInstancePoolUsage](/powershell/module/az.sql/get-azsqlinstancepoolusage/) | Devuelve información sobre el uso de grupos de Instancia administrada de SQL. |


Para usar PowerShell, [instale la versión más reciente de PowerShell Core](/powershell/scripting/install/installing-powershell#powershell) y siga las instrucciones para [instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps).

En el caso de las operaciones relacionadas con instancias dentro de grupos e instancias únicas, use los [comandos de instancia administrada](api-references-create-manage-instance.md#powershell-create-and-configure-managed-instances) estándar, pero la propiedad de *nombre del grupo de instancias* se debe rellenar al usar estos comandos para una instancia de un grupo.

## <a name="deployment-process"></a>Proceso de implementación

Para implementar una instancia administrada en un grupo de instancias, primero debe implementar el grupo, lo que supone una operación única de ejecución prolongada que dura lo mismo que la implementación de una [instancia única creada en una subred vacía](sql-managed-instance-paas-overview.md#management-operations). Después, puede implementar una instancia administrada en el grupo; se trata de una operación relativamente rápida que normalmente tarda cinco minutos como máximo. El parámetro del grupo de instancias se debe especificar de manera explícita como parte de esta operación.

En la versión preliminar pública, ambas acciones solo son compatibles con las plantillas de Azure Resource Manager y PowerShell. La experiencia de Azure Portal no está disponible actualmente.

Una vez implementada una instancia administrada en un grupo, *puede* usar Azure Portal para cambiar sus propiedades en la página de planes de tarifa.

## <a name="create-a-virtual-network-with-a-subnet"></a>Creación de una red virtual con una subred 

Para colocar varios grupos de instancias dentro de la misma red virtual, consulte los artículos siguientes:

- [Determinación del tamaño de la red virtual y la subred para Instancia administrada de Azure SQL](vnet-subnet-determine-size.md).
- Cree una subred y red virtual nuevas con la [plantilla de Azure Portal](virtual-network-subnet-create-arm-template.md) o siga las instrucciones para [preparar una red virtual existente](vnet-existing-add-subnet.md).
 

## <a name="create-an-instance-pool"></a>Creación de un grupo de instancias 

Después de completar los pasos anteriores, ya está preparado para crear un grupo de instancias.

Las restricciones siguientes se aplican a los grupos de instancias:

- Solo los grupos de instancias de uso general y Gen5 están disponibles en la versión preliminar pública.
- El nombre del grupo solo puede contener letras minúsculas, números y guiones, y no puede empezar con un guion.
- Si quiere usar la Ventaja híbrida de Azure, se aplica en el nivel de grupo de instancias. Puede establecer el tipo de licencia durante la creación del grupo o actualizarlo en cualquier momento después de la creación.

> [!IMPORTANT]
> La implementación de un grupo de instancias es una operación de larga duración que tarda aproximadamente 4,5 horas.

Para obtener parámetros de red:

```powershell
$virtualNetwork = Get-AzVirtualNetwork -Name "miPoolVirtualNetwork" -ResourceGroupName "myResourceGroup"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "miPoolSubnet" -VirtualNetwork $virtualNetwork
```

Para crear un grupo de instancias:

```powershell
$instancePool = New-AzSqlInstancePool `
  -ResourceGroupName "myResourceGroup" `
  -Name "mi-pool-name" `
  -SubnetId $subnet.Id `
  -LicenseType "LicenseIncluded" `
  -VCore 8 `
  -Edition "GeneralPurpose" `
  -ComputeGeneration "Gen5" `
  -Location "westeurope"
```

> [!IMPORTANT]
> Dado que la implementación de un grupo de instancias es una operación de larga duración, debe esperar hasta que se complete antes de ejecutar cualquiera de los pasos siguientes descritos en este artículo.

## <a name="create-a-managed-instance"></a>Creación de una instancia administrada

Después de implementar correctamente el grupo de instancias, es el momento de crear una instancia administrada dentro de él.

Para crear una instancia administrada, ejecute el comando siguiente:

```powershell
$instanceOne = $instancePool | New-AzSqlInstance -Name "mi-one-name" -VCore 2 -StorageSizeInGB 256
```

La implementación de una instancia dentro de un grupo tarda un par de minutos. Una vez creada la primera instancia, se pueden crear instancias adicionales:

```powershell
$instanceTwo = $instancePool | New-AzSqlInstance -Name "mi-two-name" -VCore 4 -StorageSizeInGB 512
```

## <a name="create-a-database"></a>Crear una base de datos 

Para crear y administrar bases de datos en una instancia administrada que se encuentre dentro de un grupo, use los comandos de instancia única.

Para crear una base de datos dentro de una instancia administrada:

```powershell
$poolinstancedb = New-AzSqlInstanceDatabase -Name "mipooldb1" -InstanceName "poolmi-001" -ResourceGroupName "myResourceGroup"
```


## <a name="get-pool-usage"></a>Obtención del uso del grupo 
 
Para obtener una lista de instancias dentro de un grupo:

```powershell
$instancePool | Get-AzSqlInstance
```


Para obtener el uso de los recursos de un grupo:

```powershell
$instancePool | Get-AzSqlInstancePoolUsage
```


Para obtener información general detallada sobre el uso del grupo y las instancias que contiene:

```powershell
$instancePool | Get-AzSqlInstancePoolUsage –ExpandChildren
```

Para enumerar las bases de datos de una instancia:

```powershell
$databases = Get-AzSqlInstanceDatabase -InstanceName "pool-mi-001" -ResourceGroupName "resource-group-name"
```


> [!NOTE]
> Para comprobar los límites sobre el número de bases de datos por grupo de instancias e instancia administrada que se implementan dentro del grupo, consulte la sección [Limitaciones de recursos](instance-pools-overview.md#resource-limitations).


## <a name="scale"></a>Escala 


Después de rellenar una instancia administrada con bases de datos, puede alcanzar los límites de instancias con respecto al almacenamiento o al rendimiento. En ese caso, si no se ha superado el uso del grupo, puede escalar la instancia.
Escalar una instancia administrada dentro de un grupo es una operación que tarda un par de minutos. El requisito previo para el escalado son núcleos virtuales y almacenamiento disponibles en el nivel de grupo de instancias.

Para actualizar el número de núcleos virtuales y el tamaño de almacenamiento:

```powershell
$instanceOne | Set-AzSqlInstance -VCore 8 -StorageSizeInGB 512 -InstancePoolName "mi-pool-name"
```


Para actualizar solo el tamaño de almacenamiento:

```powershell
$instance | Set-AzSqlInstance -StorageSizeInGB 1024 -InstancePoolName "mi-pool-name"
```

## <a name="connect"></a>Conectar 

Para conectarse a una instancia administrada en un grupo, se necesitan los dos pasos siguientes:

1. [Habilite el punto de conexión público para la instancia](#enable-the-public-endpoint).
2. [Agregue una regla de entrada al grupo de seguridad de red (NSG)](#add-an-inbound-rule-to-the-network-security-group).

Una vez completados ambos pasos, puede conectarse a la instancia de mediante una dirección de punto de conexión pública, un puerto y las credenciales proporcionadas durante la creación de la instancia. 

### <a name="enable-the-public-endpoint"></a>Habilitación del punto de conexión público

La habilitación del punto de conexión público para una instancia se puede realizar a través de Azure Portal o mediante el comando de PowerShell siguiente:


```powershell
$instanceOne | Set-AzSqlInstance -InstancePoolName "pool-mi-001" -PublicDataEndpointEnabled $true
```

Este parámetro también se puede establecer durante la creación de la instancia.

### <a name="add-an-inbound-rule-to-the-network-security-group"></a>Incorporación de una regla de entrada al grupo de seguridad de red 

Este paso se puede realizar a través de Azure Portal o mediante comandos de PowerShell y se puede hacer en cualquier momento después de preparar la subred para la instancia administrada.

Para información detallada, consulte [Permitir el tráfico del punto de conexión público en el grupo de seguridad de red](public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group).


## <a name="move-an-existing-single-instance-to-a-pool"></a>Cómo mover una instancia única existente a un grupo
 
Mover instancias dentro y fuera de un grupo es una de las limitaciones de la versión preliminar pública. Una solución alternativa es la restauración a un momento dado de bases de datos desde una instancia fuera de un grupo a una instancia que ya está en un grupo. 

Ambas instancias deben estar en la misma suscripción y región. Actualmente no se admite la restauración entre regiones y entre suscripciones.

Este proceso tiene un período de tiempo de inactividad.

Para mover bases de datos existentes:

1. Pause las cargas de trabajo de la instancia administrada desde la cual se realizará la migración.
2. Genere scripts para crear bases de datos del sistema y ejecútelos en la instancia que se encuentra dentro del grupo de instancias.
3. Realice una restauración a un momento dado de cada base de datos desde la instancia única a la instancia del grupo.

    ```powershell
    $resourceGroupName = "my resource group name"
    $managedInstanceName = "my managed instance name"
    $databaseName = "my source database name"
    $pointInTime = "2019-08-21T08:51:39.3882806Z"
    $targetDatabase = "name of the new database that will be created"
    $targetResourceGroupName = "resource group of instance pool"
    $targetInstanceName = "pool instance name"
       
    Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
      -ResourceGroupName $resourceGroupName `
      -InstanceName $managedInstanceName `
      -Name $databaseName `
      -PointInTime $pointInTime `
      -TargetInstanceDatabaseName $targetDatabase `
      -TargetResourceGroupName $targetResourceGroupName `
      -TargetInstanceName $targetInstanceName
    ```

4. Dirija la aplicación a la nueva instancia y reanude sus cargas de trabajo.

Si hay varias bases de datos, repita el proceso para cada base de datos.


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
- Para obtener información sobre la supervisión avanzada del rendimiento de las bases de datos de Instancia administrada de SQL con inteligencia integrada para la solución de problemas, consulte [Supervisión de instancias de Azure SQL Database con Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Para obtener información sobre precios, vea la página de [precios de Instancia administrada de SQL](https://azure.microsoft.com/pricing/details/sql-database/managed/).
