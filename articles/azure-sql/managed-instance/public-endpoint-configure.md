---
title: 'Configuración de un punto de conexión público: Instancia administrada de Azure SQL'
description: Aprenda a configurar un punto de conexión público para Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto, sstein
ms.date: 02/08/2021
ms.openlocfilehash: a515352f4c4138ac5fa8fb829f8d483747bac323
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666436"
---
# <a name="configure-public-endpoint-in-azure-sql-managed-instance"></a>Configuración de un punto de conexión público en Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

El punto de conexión público para una [instancia administrada](./sql-managed-instance-paas-overview.md) habilita el acceso a datos de su instancia administrada desde fuera de la [red virtual](../../virtual-network/virtual-networks-overview.md). Puede acceder a su instancia administrada desde servicios de Azure de varios inquilinos, como Power BI, Azure App Service o una red local. Al usar el punto de conexión público en una instancia administrada, no es necesario que use una VPN, lo que puede contribuir a evitar problemas de rendimiento de la VPN.

En este artículo, aprenderá a:

> [!div class="checklist"]
>
> - Habilitar un punto de conexión público para su instancia administrada en Azure Portal.
> - Habilitar un punto de conexión público para su instancia administrada usando PowerShell.
> - Configurar un grupo de seguridad de red en la instancia administrada para permitir el tráfico al punto de conexión público de la instancia administrada.
> - Obtener la cadena de conexión del punto de conexión público en la instancia administrada.

## <a name="permissions"></a>Permisos

Debido a la confidencialidad de los datos de una instancia administrada, la configuración para habilitar el punto de conexión público de la instancia administrada requiere un proceso de dos pasos. Esta medida de seguridad se ajusta a la separación de tareas (SoD):

- El administrador de la instancia administrada debe habilitar el punto de conexión público en la instancia administrada. Dicho administrador se puede encontrar en la página **Introducción** del recurso de la instancia administrada.
- Un administrador de red debe permitir el tráfico usando un grupo de seguridad de red. Para más información, vea los [permisos para el grupo de seguridad de red](../../virtual-network/manage-network-security-group.md#permissions).

## <a name="enabling-public-endpoint-for-a-managed-instance-in-the-azure-portal"></a>Habilitar un punto de conexión público para una instancia administrada en Azure Portal

1. Inicie Azure Portal en <https://portal.azure.com/.>.
1. Abra el grupo de recursos con la instancia administrada y seleccione la **instancia administrada de SQL** en la que quiera configurar el punto de conexión público.
1. En la configuración de **seguridad**, seleccione la pestaña **Red virtual**.
1. En la página de configuración de la red virtual, seleccione **Habilitar** y después el icono **Guardar** para actualizar la configuración.

![Captura de pantalla que muestra una página Red virtual de SQL Managed Instance con el punto de conexión público habilitado.](./media/public-endpoint-configure/mi-vnet-config.png)

## <a name="enabling-public-endpoint-for-a-managed-instance-using-powershell"></a>Habilitar un punto de conexión público para una instancia administrada usando PowerShell

### <a name="enable-public-endpoint"></a>Habilitar el punto de conexión público

Ejecute los comandos de PowerShell siguientes. Reemplace **subscription-id** por su identificador de suscripción. Reemplace también **rg-name** por el grupo de recursos de la instancia administrada y **mi-name** por el nombre de la instancia administrada.

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi = $mi | Set-AzSqlInstance -PublicDataEndpointEnabled $true -force
```

### <a name="disable-public-endpoint"></a>Deshabilitar el punto de conexión público

Para deshabilitar el punto de conexión público con PowerShell, deberá ejecutar el comando siguiente (además, no olvide cerrar el NSG para el puerto de entrada 3342 si lo tiene configurado):

```powershell
Set-AzSqlInstance -PublicDataEndpointEnabled $false -force
```

## <a name="allow-public-endpoint-traffic-on-the-network-security-group"></a>Permitir el tráfico del punto de conexión público en el grupo de seguridad de red

1. Si todavía tiene abierta la página de configuración de la instancia administrada, vaya a la pestaña **Introducción**. En caso contrario, vuelva a su recurso **Instancia administrada de SQL**. Seleccione el vínculo **Red virtual/subred**, que le llevará a la página de configuración de la red virtual.

    ![Captura de pantalla que muestra la página Configuración de red virtual, donde puede encontrar el valor de red virtual o subred.](./media/public-endpoint-configure/mi-overview.png)

1. Seleccione la pestaña **Subredes** en el panel de configuración izquierdo de la red virtual y anote el **GRUPO DE SEGURIDAD** de su instancia administrada.

    ![Captura de pantalla que muestra la pestaña Subred, donde puede obtener el grupo de seguridad de la instancia administrada.](./media/public-endpoint-configure/mi-vnet-subnet.png)

1. Vuelva al grupo de recursos que contiene la instancia administrada. Debería ver el nombre del **grupo de seguridad de red** anotado anteriormente. Seleccione el nombre que va a introducir en la página de configuración del grupo de seguridad de red.

1. Seleccione la pestaña **Reglas de seguridad de entrada** y **agregue** una regla que tenga mayor prioridad que la regla **deny_all_inbound** con la siguiente configuración: </br> </br>

    |Configuración  |Valor sugerido  |Descripción  |
    |---------|---------|---------|
    |**Origen**     |Cualquier dirección IP o etiqueta de servicio         |<ul><li>En servicios de Azure como Power BI, seleccione la etiqueta de servicio en la nube de Azure.</li> <li>Para el equipo o la máquina virtual de Azure, use la dirección IP de NAT.</li></ul> |
    |**Intervalos de puertos de origen**     |* |Deje esta opción en * (cualquiera), puesto que los puertos de origen suelen asignarse de forma dinámica y, por lo tanto, son impredecibles. |
    |**Destino**     |Any         |Deje la opción Destino en Cualquiera para permitir el tráfico en la subred de la instancia administrada. |
    |**Intervalos de puertos de destino**     |3342         |Establezca el puerto de destino en 3342, que es el punto de conexión TDS público de la instancia administrada. |
    |**Protocolo**     |TCP         |Instancia administrada de SQL utiliza el protocolo TCP para TDS. |
    |**Acción**     |Allow         |Permite el tráfico entrante en la instancia administrada mediante el punto de conexión público. |
    |**Prioridad**     |1300         |Asegúrese de que esta regla tiene más prioridad que la regla **deny_all_inbound**. |

    ![Captura de pantalla que muestra las reglas de seguridad de entrada con la nueva regla public_endpoint_inbound encima de la regla deny_all_inbound.](./media/public-endpoint-configure/mi-nsg-rules.png)

    > [!NOTE]
    > El puerto 3342 se usa para las conexiones del punto de conexión público a la instancia administrada y no se puede cambiar en este momento.

## <a name="obtaining-the-managed-instance-public-endpoint-connection-string"></a>Obtener la cadena de conexión del punto de conexión público en la instancia administrada

1. Vaya a la página de configuración de la instancia administrada que se ha habilitado para el punto de conexión público. Seleccione la pestaña **Cadenas de conexión** en **Configuración**.
1. El nombre de host del punto de conexión público debe tener el formato <mi_name>.**public**.<dns_zone>.database.windows.net y el puerto usado para la conexión debe ser 3342. A continuación se muestra un ejemplo de un valor de servidor de la cadena de conexión que indica el puerto del punto de conexión público que se puede usar en las conexiones de SQL Server Management Studio o Azure Data Studio: `<mi_name>.public.<dns_zone>.database.windows.net,3342`

    ![Captura de pantalla que muestra las cadenas de conexión de los puntos de conexión públicos y privados.](./media/public-endpoint-configure/mi-public-endpoint-conn-string.png)

## <a name="next-steps"></a>Pasos siguientes

Información acerca del uso de [Instancia administrada de Azure SQL de forma segura con puntos de conexión públicos](public-endpoint-overview.md).
