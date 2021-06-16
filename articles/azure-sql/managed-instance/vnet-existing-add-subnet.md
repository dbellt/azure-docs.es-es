---
title: Configuración de una red virtual existente
titleSuffix: Azure SQL Managed Instance
description: En este artículo se describe cómo configurar una red virtual y una subred existentes en las que se puede implementar Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 03/17/2020
ms.openlocfilehash: f2870aed0c44afa5f8bcbc6ea9ec5be6baececb7
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692689"
---
# <a name="configure-an-existing-virtual-network-for-azure-sql-managed-instance"></a>Configuración de una red virtual para Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Instancia administrada de Azure SQL se debe implementar dentro de una [red virtual](../../virtual-network/virtual-networks-overview.md) de Azure y la subred dedicadas solo a instancias administradas. Puede usar la red virtual y la subred existentes si se configuran de acuerdo con los [requisitos de red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md#network-requirements).

Si uno de los casos siguientes se aplica a usted, puede validar y modificar la red mediante el uso del script que se explica en este artículo:

- Tiene una nueva subred que aún no está configurada.
- No está seguro de que la subred cumple los [requisitos](connectivity-architecture-overview.md#network-requirements).
- Quiere comprobar si la subred cumple con los [requisitos de red](connectivity-architecture-overview.md#network-requirements) una vez realizados los cambios.

> [!Note]
> Solo puede crear una instancia administrada en las redes virtuales creadas mediante el modelo de implementación de Azure Resource Manager. No se admiten las redes virtuales de Azure creadas mediante el modelo de implementación clásica. Para calcular el tamaño de la subred, siga las instrucciones del artículo [Determinación del tamaño de subred para instancias administradas de SQL](vnet-subnet-determine-size.md). No se puede cambiar el tamaño de la subred después de implementar los recursos que contiene.
>
> Después de crear la instancia administrada, no se admite el traslado de la instancia o la red virtual a otro grupo de recursos o suscripción.

## <a name="validate-and-modify-an-existing-virtual-network"></a>Validación y modificación de una red virtual existente

Si quiere crear una instancia administrada dentro de una subred existente, es aconsejable usar el siguiente script de PowerShell para preparar la subred:

```powershell
$scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet'

$parameters = @{
    subscriptionId = '<subscriptionId>'
    resourceGroupName = '<resourceGroupName>'
    virtualNetworkName = '<virtualNetworkName>'
    subnetName = '<subnetName>'
    }

Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/delegateSubnet.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters
```

El script prepara la subred en tres pasos:

1. Validación: se validan la subred y la red virtual seleccionadas para garantizar el cumplimiento de los requisitos de red de Instancia administrada de SQL.
2. Confirmación: se muestra al usuario el conjunto de cambios que se deben realizar para preparar la subred para la implementación de Instancia administrada de SQL. También se solicitará su consentimiento.
3. Preparación: la subred y la red virtuales se configuran adecuadamente.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea [¿Qué es Instancia administrada de SQL?](sql-managed-instance-paas-overview.md)
- Para obtener un tutorial en el que se muestra cómo crear una red virtual y una instancia administrada, y restaurar una base de datos a partir de una copia de seguridad de base de datos, vea [Creación de una instancia administrada](instance-create-quickstart.md).
- Para ver información relativa a los distintos problemas de DNS, consulte [Configuración de un DNS personalizado](custom-dns-configure.md).
