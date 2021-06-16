---
title: 'Tutorial: Conexión a un servidor SQL de Azure mediante un punto de conexión privado de Azure: PowerShell'
description: Use este tutorial para aprender a crear un servidor SQL de Azure con un punto de conexión privado mediante Azure PowerShell.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 10/31/2020
ms.author: allensu
ms.custom: fasttrack-edit, devx-track-azurepowershell
ms.openlocfilehash: 8108a742cb738be7c52b78f0f7725d2b7f6744b9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693631"
---
# <a name="tutorial-connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-powershell"></a>Tutorial: Conexión a un servidor SQL de Azure mediante un punto de conexión privado de Azure: Azure PowerShell

Un punto de conexión privado de Azure es el bloque de creación fundamental para Private Link en Azure. Permite que los recursos de Azure, como las máquinas virtuales, se comuniquen con recursos de Private Link de manera privada.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una red virtual y un host bastión.
> * Cree una máquina virtual.
> * Crear un servidor SQL de Azure y un punto de conexión privado.
> * Probar la conectividad con el punto de conexión privado del servidor SQL.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreateSQLEndpointTutorial-rg' -Location 'eastus'
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Creación de una red virtual y un host bastión

En esta sección, creará una red virtual, una subred y un host bastión. 

El host bastión se utilizará para conectarse de forma segura a la máquina virtual a fin de probar el punto de conexión privado.

Creación de una red virtual y un host bastión con:

* [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)
* [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress)
* [New-AzBastion](/powershell/module/az.network/new-azbastion)

```azurepowershell-interactive
## Create backend subnet config. ##
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name myBackendSubnet -AddressPrefix 10.0.0.0/24

## Create Azure Bastion subnet. ##
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig -Name AzureBastionSubnet -AddressPrefix 10.0.1.0/24

## Create the virtual network. ##
$parameters1 = @{
    Name = 'MyVNet'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    AddressPrefix = '10.0.0.0/16'
    Subnet = $subnetConfig, $bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @parameters1

## Create public IP address for bastion host. ##
$parameters2 = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @parameters2

## Create bastion host ##
$parameters3 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @parameters3
```

El host de Azure Bastion tarda unos minutos en implementarse.

## <a name="create-test-virtual-machine"></a>Creación de una máquina virtual de prueba

En esta sección, creará una máquina virtual que se utilizará para probar el punto de conexión privado.

Cree la máquina virtual con:

  * [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential)
  * [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 
  * [New-AzVM](/powershell/module/az.compute/new-azvm)
  * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
  * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
  * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
  * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)


```azurepowershell-interactive
## Set credentials for server admin and password. ##
$cred = Get-Credential

## Command to get virtual network configuration. ##
$vnet = Get-AzVirtualNetwork -Name myVNet -ResourceGroupName CreateSQLEndpointTutorial-rg

## Command to create network interface for VM ##
$parameters1 = @{
    Name = 'myNicVM'
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
}
$nicVM = New-AzNetworkInterface @parameters1

## Create a virtual machine configuration.##
$parameters2 = @{
    VMName = 'myVM'
    VMSize = 'Standard_DS1_v2'
}
$parameters3 = @{
    ComputerName = 'myVM'
    Credential = $cred
}
$parameters4 = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'
}
$vmConfig = 
New-AzVMConfig @parameters2 | Set-AzVMOperatingSystem -Windows @parameters3 | Set-AzVMSourceImage @parameters4 | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Location 'eastus' -VM $vmConfig
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-an-azure-sql-server"></a>Creación de un servidor SQL de Azure

En esta sección, va a crear un servidor y una base de datos SQL mediante:

* [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver)
* [New-AzSQlDatabase](/powershell/module/az.sql/new-azsqldatabase)

Cree un servidor y una base de datos SQL. Reemplace **\<sql-server-name>** por el nombre de servidor único:

```azurepowershell-interactive
## Set and administrator and password for the SQL server. ##
$cred = Get-Credential

## Create SQL server ##
$parameters1 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ServerName = '<sql-server-name>'
    SqlAdministratorCredentials = $cred
    Location = 'eastus'
}
New-AzSqlServer @parameters1

## Create database. ##
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ServerName = '<sql-server-name>'
    DatabaseName = 'mysqldatabase'
    RequestedServiceObjectiveName = 'S0'
    SampleName = 'AdventureWorksLT'
}
New-AzSqlDatabase @parameters2
```

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección creará el punto de conexión privado y la conexión con:

* [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/New-AzPrivateLinkServiceConnection)
* [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint)

```azurepowershell-interactive
## Place SQL server into variable. Replace <sql-server-name> with your server name ##
$server = Get-AzSqlServer -ResourceGroupName CreateSQLEndpointTutorial-rg -ServerName <sql-server-name>

## Create private endpoint connection. ##
$parameters1 = @{
    Name = 'myConnection'
    PrivateLinkServiceId = $server.ResourceID
    GroupID = 'sqlserver'
}
$privateEndpointConnection = New-AzPrivateLinkServiceConnection @parameters1

## Place virtual network into variable. ##
$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Name 'myVNet'

## Disable private endpoint network policy ##
$vnet.Subnets[0].PrivateEndpointNetworkPolicies = "Disabled"
$vnet | Set-AzVirtualNetwork

## Create private endpoint
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'myPrivateEndpoint'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    PrivateLinkServiceConnection = $privateEndpointConnection
}
New-AzPrivateEndpoint @parameters2
```
## <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

En esta sección creará y configurará la zona DNS privada con:

* [New-AzPrivateDnsZone](/powershell/module/az.privatedns/new-azprivatednszone)
* [New-AzPrivateDnsVirtualNetworkLink](/powershell/module/az.privatedns/new-azprivatednsvirtualnetworklink)
* [New-AzPrivateDnsZoneConfig](/powershell/module/az.network/new-azprivatednszoneconfig)
* [New-AzPrivateDnsZoneGroup](/powershell/module/az.network/new-azprivatednszonegroup)

```azurepowershell-interactive
## Place virtual network into variable. ##
$vnet = Get-AzVirtualNetwork -ResourceGroupName 'CreateSQLEndpointTutorial-rg' -Name 'myVNet'

## Create private dns zone. ##
$parameters1 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    Name = 'privatelink.database.windows.net'
}
$zone = New-AzPrivateDnsZone @parameters1

## Create dns network link. ##
$parameters2 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    ZoneName = 'privatelink.database.windows.net'
    Name = 'myLink'
    VirtualNetworkId = $vnet.Id
}
$link = New-AzPrivateDnsVirtualNetworkLink @parameters2

## Create DNS configuration ##
$parameters3 = @{
    Name = 'privatelink.database.windows.net'
    PrivateDnsZoneId = $zone.ResourceId
}
$config = New-AzPrivateDnsZoneConfig @parameters3

## Create DNS zone group. ##
$parameters4 = @{
    ResourceGroupName = 'CreateSQLEndpointTutorial-rg'
    PrivateEndpointName = 'myPrivateEndpoint'
    Name = 'myZoneGroup'
    PrivateDnsZoneConfig = $config
}
New-AzPrivateDnsZoneGroup @parameters4
```

## <a name="test-connectivity-to-private-endpoint"></a>Prueba de la conectividad con el punto de conexión privado

En esta sección, utilizará la máquina virtual creada en el paso anterior para conectarse al servidor SQL mediante el punto de conexión privado.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com) 
 
2. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.

3. Seleccione **CreateSQLEndpointTutorial-rg**.

4. Seleccione **myVM**.

5. En la página de información general para **myVM**, seleccione **Conectar** y, luego, **Bastion**.

6. Seleccione el botón **Usar bastión** azul.

7. Especifique el nombre de usuario y la contraseña proporcionados durante la creación de la máquina virtual.

8. Abra Windows PowerShell en el servidor después de conectarse.

9. Escriba `nslookup <sqlserver-name>.database.windows.net`. Reemplace **\<sqlserver-name>** por el nombre del servidor SQL que creó en los pasos anteriores.  Recibirá un mensaje similar al que se muestra a continuación:

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Se devuelve la dirección IP privada **10.0.0.5** del nombre del servidor SQL.  Esta dirección se encuentra en la subred de la red virtual que creó anteriormente.


10. Instale [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) en **myVM**.

11. Abra **SQL Server Management Studio**.

12. En **Conectar con el servidor**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Tipo de servidor | Seleccione **Motor de base de datos**.|
    | Nombre de servidor | Escriba **\<sql-server-name>.database.windows.net**. |
    | Autenticación | Seleccione **Autenticación de SQL Server**. |
    | Nombre de usuario | Escriba el nombre de usuario que especificó durante la creación del servidor. |
    | Contraseña | Escriba la contraseña que especificó durante la creación del servidor. |
    | Recordar contraseña | Seleccione **Sí**. |

13. Seleccione **Conectar**.

14. Examine las bases de datos en el menú izquierdo.

15. (Opcionalmente) Cree o consulte información de **mysqldatabase**.

16. Cierre la conexión bastión a **myVM**. 

## <a name="clean-up-resources"></a>Limpieza de recursos 
Cuando haya terminado mediante el punto de conexión privado, el servidor SQL y la máquina virtual, elimine el grupo de recursos y todos los recursos que contiene: 

1. Escriba **CreateSQLEndpointTutorial-rg** en el cuadro **Buscar** situado en la parte superior del portal y seleccione **CreateSQLEndpointTutorial-rg** en los resultados de la búsqueda. 

2. Seleccione **Eliminar grupo de recursos**. 

3. Escriba **CreateSQLEndpointTutorial-rg** en **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado:

* Una red virtual y un host bastión.
* Una máquina virtual.
* Un servidor SQL de Azure con un punto de conexión privado.

Ha utilizado la máquina virtual para probar la conectividad de forma segura al servidor SQL mediante el punto de conexión privado.

Como paso siguiente, puede que también le interese el escenario de la arquitectura de **aplicación web con conectividad privada a Azure SQL Database**, que conecta una aplicación web situada fuera de la red virtual con el punto de conexión privado de una base de datos.
> [!div class="nextstepaction"]
> [Conectividad privada de una aplicación web a Azure SQL Database](/azure/architecture/example-scenario/private-web-app/private-web-app)
