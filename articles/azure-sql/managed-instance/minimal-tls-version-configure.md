---
title: 'Configuración de la versión mínima de TLS: Instancia administrada'
description: Obtenga información para configurar la versión mínima de TLS para Instancia administrada
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: devx-track-azurepowershell
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: ''
ms.date: 05/25/2020
ms.openlocfilehash: 1e87c1d658792662893b86213324b8367960e9e5
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666381"
---
# <a name="configure-minimal-tls-version-in-azure-sql-managed-instance"></a>Configuración de la versión mínima de TLS en Instancia administrada de Azure SQL
La opción de versión mínima de [Seguridad de la capa de transporte (TLS)](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server) permite a los clientes controlar la versión de TLS que usa Instancia administrada de Azure SQL.

Actualmente, se admiten TLS 1.0, 1.1 y 1.2. La opción de versión mínima de TLS garantiza que se admitan las versiones posteriores más recientes de TLS. Por ejemplo, elegir una versión de TLS superior a 1.1. significa que solo se aceptan conexiones con TLS 1.1 y 1.2, y se rechaza TLS 1.0. Después de realizar las pruebas para confirmar que las aplicaciones son compatibles, se recomienda establecer la versión mínima de TLS en 1.2, ya que incluye correcciones de vulnerabilidades detectadas en versiones anteriores, y es la versión superior de TLS admitida en Instancia administrada de Azure SQL.

En el caso de los clientes con aplicaciones que se basan en versiones anteriores de TLS, se recomienda establecer la versión mínima de TLS según los requisitos de las aplicaciones. En el caso de los clientes que se basan en aplicaciones para conectarse mediante una conexión sin cifrar, se recomienda no establecer ninguna versión mínima de TLS. 

Para obtener más información, consulte [Consideraciones de TLS para la conectividad de SQL Database](../database/connect-query-content-reference-guide.md#tls-considerations-for-database-connectivity).

Después de establecer la versión mínima de TLS, se producirá un error en los intentos de inicio de sesión de los clientes que usen una versión de TLS inferior a la versión mínima de TLS del servidor, con el siguiente error:

```output
Error 47072
Login failed with invalid TLS version
```

## <a name="set-minimal-tls-version-via-powershell"></a>Establecimiento de la versión mínima de TLS mediante PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. El script siguiente requiere el [módulo de Azure PowerShell](/powershell/azure/install-az-ps).

El siguiente script de PowerShell muestra cómo `Get` y `Set` la propiedad **Versión mínima de TLS** en el nivel de instancia:

```powershell
#Get the Minimal TLS Version property
(Get-AzSqlInstance -Name sql-instance-name -ResourceGroupName resource-group).MinimalTlsVersion

# Update Minimal TLS Version Property
Set-AzSqlInstance -Name sql-instance-name -ResourceGroupName resource-group -MinimalTlsVersion "1.2"
```

## <a name="set-minimal-tls-version-via-azure-cli"></a>Establecimiento de la versión mínima de TLS mediante la CLI de Azure

> [!IMPORTANT]
> Todos los scripts de esta sección requieren la [CLI de Azure](/cli/azure/install-azure-cli).

### <a name="azure-cli-in-a-bash-shell"></a>CLI de Azure en un shell de Bash

El siguiente script de la CLI muestra cómo cambiar la opción **Minimal TLS Version** (Versión mínima de TLS) en un shell de Bash:

```azurecli-interactive
# Get current setting for Minimal TLS Version
az sql mi show -n sql-instance-name -g resource-group --query "minimalTlsVersion"

# Update setting for Minimal TLS Version
az sql mi update -n sql-instance-name -g resource-group --set minimalTlsVersion="1.2"
```
