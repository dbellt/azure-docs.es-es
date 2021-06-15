---
title: PowerShell para puntos de conexión de red virtual y reglas para bases de datos únicas y agrupadas
description: Proporciona scripts de PowerShell para crear y administrar puntos de conexión de servicio virtual para su instancia de Azure SQL Database y Azure Synapse.
services: sql-database
ms.service: sql-database
ms.subservice: deployment-configuration
ms.devlang: PowerShell
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto, mathoma
ms.date: 04/17/2019
ms.custom: sqldbrb=1, devx-track-azurepowershell
tags: azure-synapse
ms.openlocfilehash: bc00adebe6b60f47dcd2fb5de50508ff4ada2ae7
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698988"
---
# <a name="powershell-create-a-virtual-service-endpoint-and-vnet-rule-for-azure-sql-database"></a>PowerShell: Creación de una regla de red virtual y un punto de conexión de servicio virtual para Azure SQL Database
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Las *reglas de red virtual* son una característica de firewall que controla si el [servidor SQL lógico](../logical-servers.md) de las bases de datos de [Azure SQL Database](../sql-database-paas-overview.md), así como los grupos elásticos o las bases de datos de [Azure Synapse](../../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) aceptan las comunicaciones que se envían desde subredes específicas de redes virtuales.

> [!IMPORTANT]
> Este artículo se aplica a Azure SQL Database, incluido Azure Synapse (anteriormente SQL SW). En pocas palabras, el término Azure SQL Database en este artículo se aplica a las bases de datos que pertenecen a Azure SQL Database o a Azure Synapse. En cambio, este artículo *no* se aplica a la instancia administrada de Azure SQL, ya que no tiene un punto de conexión de servicio asociado a ella.

En este artículo se muestra un script de PowerShell que realiza las siguientes acciones:

1. Crea un *punto de conexión de servicio virtual* de Microsoft Azure en la subred.
2. Agrega el punto de conexión al firewall de su servidor para crear una *regla de red virtual*.

Para obtener más contexto, consulte [Puntos de conexión de servicio de red virtual para Azure SQL Database][sql-db-vnet-service-endpoint-rule-overview-735r].

> [!TIP]
> Si todo lo que necesita es evaluar o agregar *nombre de tipo* del punto de conexión del servicio virtual para Azure SQL Database a la subred, puede ir directamente a nuestro [script de PowerShell más directo](#a-verify-subnet-is-endpoint-ps-100).

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para los [cmdlets de `Az.Sql`](/powershell/module/az.sql). Para ver el módulo anterior, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

## <a name="major-cmdlets"></a>Cmdlets principales

En este artículo se resalta el cmdlet [**New-AzSqlServerVirtualNetworkRule**](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule), que agrega el punto de conexión de la subred a la lista de control de acceso (ACL) de su servidor, lo que crea una regla.

En la lista siguiente se muestra la secuencia de otros cmdlets *principales* que debe ejecutar para prepararse para la llamada a **New-AzSqlServerVirtualNetworkRule**. En este artículo, estas llamadas se producen en el [script 3: "regla de red virtual"](#a-script-30):

1. [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig): crea un objeto de subred.
2. [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork): crea su red virtual y le proporciona la subred.
3. [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Set-azVirtualNetworkSubnetConfig): asigna un punto de conexión de servicio de red virtual a la subred.
4. [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork): conserva las actualizaciones realizadas en la red virtual.
5. [New-AzSqlServerVirtualNetworkRule](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule): una vez que la subred sea un punto de conexión, agrega la subred como una regla de red virtual en la ACL de su servidor.
   - Este cmdlet ofrece el parámetro **-IgnoreMissingVnetServiceEndpoint**, a partir del módulo de Azure RM PowerShell versión 5.1.1.

## <a name="prerequisites-for-running-powershell"></a>Requisitos previos para ejecutar PowerShell

- Ya puede iniciar sesión en Azure, por ejemplo, en [Azure Portal][http-azure-portal-link-ref-477t].
- Ya puede ejecutar scripts de PowerShell.

> [!NOTE]
> Asegúrese de que los puntos de conexión de servicio están activados para la red virtual o subred que quiere agregar al servidor. De lo contrario, se producirá un error al crear la regla de firewall de red virtual del servidor.

## <a name="one-script-divided-into-four-chunks"></a>Un script dividido en cuatro fragmentos

Nuestro script de PowerShell de demostración se divide en una secuencia de scripts más pequeños. La división simplifica el aprendizaje y proporciona flexibilidad. Los scripts se deben ejecutar en su secuencia indicada. Si ahora no tiene tiempo de ejecutar los scripts, se muestra el resultado de la prueba real después del script 4.

<a name="a-script-10"></a>

### <a name="script-1-variables"></a>Script 1: variables

Este primer script de PowerShell asigna valores a variables. Los scripts posteriores dependen de estas variables.

> [!IMPORTANT]
> Antes de ejecutar este script, puede editar los valores, si quiere. Por ejemplo, si ya tiene un grupo de recursos, le recomendamos que edite el nombre del grupo de recursos como el valor asignado.
>
> El nombre de la suscripción debe editarse en el script.

### <a name="powershell-script-1-source-code"></a>Código fuente del script 1 de PowerShell

```powershell
######### Script 1 ########################################
##   LOG into to your Azure account.                     ##
##   (Needed only one time per powershell.exe session.)  ##
###########################################################

$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]'
if ('yes' -eq $yesno) { Connect-AzAccount }

###########################################################
##  Assignments to variables used by the later scripts.  ##
###########################################################

# You can edit these values, if necessary.
$SubscriptionName = 'yourSubscriptionName'
Select-AzSubscription -SubscriptionName $SubscriptionName

$ResourceGroupName = 'RG-YourNameHere'
$Region = 'westcentralus'

$VNetName = 'myVNet'
$SubnetName = 'mySubnet'
$VNetAddressPrefix = '10.1.0.0/16'
$SubnetAddressPrefix = '10.1.1.0/24'
$VNetRuleName = 'myFirstVNetRule-ForAcl'

$SqlDbServerName = 'mysqldbserver-forvnet'
$SqlDbAdminLoginName = 'ServerAdmin'
$SqlDbAdminLoginPassword = 'ChangeYourAdminPassword1'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql'  # Official type name.

Write-Host 'Completed script 1, the "Variables".'
```

<a name="a-script-20"></a>

### <a name="script-2-prerequisites"></a>Script 2: Requisitos previos

Este script se prepara para el siguiente script, donde se encuentra la acción del punto de conexión. Este script crea automáticamente los siguientes elementos de lista, pero solo si aún no existen. Puede omitir el script 2 si está seguro de que estos elementos ya existen:

- Grupo de recursos de Azure
- Servidor SQL lógico

### <a name="powershell-script-2-source-code"></a>Código fuente del script 2 de PowerShell

```powershell
######### Script 2 ########################################
##   Ensure your Resource Group already exists.          ##
###########################################################

Write-Host "Check whether your Resource Group already exists."

$gottenResourceGroup = $null
$gottenResourceGroup = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue

if ($null -eq $gottenResourceGroup) {
    Write-Host "Creating your missing Resource Group - $ResourceGroupName."
    New-AzResourceGroup -Name $ResourceGroupName -Location $Region
} else {
    Write-Host "Good, your Resource Group already exists - $ResourceGroupName."
}

$gottenResourceGroup = $null

###########################################################
## Ensure your server already exists. ##
###########################################################

Write-Host "Check whether your server already exists."

$sqlDbServer = $null
$azSqlParams = @{
    ResourceGroupName = $ResourceGroupName
    ServerName        = $SqlDbServerName
    ErrorAction       = 'SilentlyContinue'
}
$sqlDbServer = Get-AzSqlServer @azSqlParams

if ($null -eq $sqlDbServer) {
    Write-Host "Creating the missing server - $SqlDbServerName."
    Write-Host "Gather the credentials necessary to next create a server."

    $sqlAdministratorCredentials = [pscredential]::new($SqlDbAdminLoginName,(ConvertTo-SecureString -String $SqlDbAdminLoginPassword -AsPlainText -Force))

    if ($null -eq $sqlAdministratorCredentials) {
        Write-Host "ERROR, unable to create SQL administrator credentials.  Now ending."
        return
    }

    Write-Host "Create your server."

    $sqlSrvParams = @{
        ResourceGroupName           = $ResourceGroupName
        ServerName                  = $SqlDbServerName
        Location                    = $Region
        SqlAdministratorCredentials = $sqlAdministratorCredentials
    }
    New-AzSqlServer @sqlSrvParams
} else {
    Write-Host "Good, your server already exists - $SqlDbServerName."
}

$sqlAdministratorCredentials = $null
$sqlDbServer = $null

Write-Host 'Completed script 2, the "Prerequisites".'
```

<a name="a-script-30"></a>

## <a name="script-3-create-an-endpoint-and-a-rule"></a>Script 3: Creación de un punto de conexión y una regla

Este script crea una red virtual con una subred. A continuación, el script asigna el tipo de punto de conexión **Microsoft.Sql** a la subred. Por último, el script agrega su subred a la lista de control de acceso (ACL), de modo que crea una regla.

### <a name="powershell-script-3-source-code"></a>Código fuente del script 3 de PowerShell

```powershell
######### Script 3 ########################################
##   Create your virtual network, and give it a subnet.  ##
###########################################################

Write-Host "Define a subnet '$SubnetName', to be given soon to a virtual network."

$subnetParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$subnet = New-AzVirtualNetworkSubnetConfig @subnetParams

Write-Host "Create a virtual network '$VNetName'.`nGive the subnet to the virtual network that we created."

$vnetParams = @{
    Name              = $VNetName
    AddressPrefix     = $VNetAddressPrefix
    Subnet            = $subnet
    ResourceGroupName = $ResourceGroupName
    Location          = $Region
}
$vnet = New-AzVirtualNetwork @vnetParams

###########################################################
##   Create a Virtual Service endpoint on the subnet.    ##
###########################################################

Write-Host "Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet."

$vnetSubParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    VirtualNetwork  = $vnet
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$vnet = Set-AzVirtualNetworkSubnetConfig @vnetSubParams

Write-Host "Persist the updates made to the virtual network > subnet."

$vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet

$vnet.Subnets[0].ServiceEndpoints  # Display the first endpoint.

###########################################################
##   Add the Virtual Service endpoint Id as a rule,      ##
##   into SQL Database ACLs.                             ##
###########################################################

Write-Host "Get the subnet object."

$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

$subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vnet

Write-Host "Add the subnet .Id as a rule, into the ACLs for your server."

$ruleParams = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
    VirtualNetworkSubnetId = $subnet.Id
}
New-AzSqlServerVirtualNetworkRule @ruleParams 

Write-Host "Verify that the rule is in the SQL Database ACL."

$rule2Params = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
}
Get-AzSqlServerVirtualNetworkRule @rule2Params

Write-Host 'Completed script 3, the "Virtual-Network-Rule".'
```

<a name="a-script-40"></a>

## <a name="script-4-clean-up"></a>Script 4: Limpieza

Este script final elimina los recursos que crearon los scripts anteriores para la demostración. Sin embargo, el script pide confirmación antes de eliminar lo siguiente:

- Servidor SQL lógico
- Grupo de recursos de Azure

Puede ejecutar el script 4 en cualquier momento una vez se complete el script 1.

### <a name="powershell-script-4-source-code"></a>Código fuente del script 4 de PowerShell

```powershell
######### Script 4 ########################################
##   Clean-up phase A:  Unconditional deletes.           ##
##                                                       ##
##   1. The test rule is deleted from SQL Database ACL.        ##
##   2. The test endpoint is deleted from the subnet.    ##
##   3. The test virtual network is deleted.             ##
###########################################################

Write-Host "Delete the rule from the SQL Database ACL."

$removeParams = @{
    ResourceGroupName      = $ResourceGroupName
    ServerName             = $SqlDbServerName
    VirtualNetworkRuleName = $VNetRuleName
    ErrorAction            = 'SilentlyContinue'
}
Remove-AzSqlServerVirtualNetworkRule @removeParams

Write-Host "Delete the endpoint from the subnet."

$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

Remove-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vnet

Write-Host "Delete the virtual network (thus also deletes the subnet)."

$removeParams = @{
    Name              = $VNetName
    ResourceGroupName = $ResourceGroupName
    ErrorAction       = 'SilentlyContinue'
}
Remove-AzVirtualNetwork @removeParams

###########################################################
##   Clean-up phase B:  Conditional deletes.             ##
##                                                       ##
##   These might have already existed, so user might     ##
##   want to keep.                                       ##
##                                                       ##
##   1. Logical SQL server                        ##
##   2. Azure resource group                             ##
###########################################################

$yesno = Read-Host 'CAUTION !: Do you want to DELETE your server AND your resource group?  [yes/no]'
if ('yes' -eq $yesno) {
    Write-Host "Remove the server."

    $removeParams = @{
        ServerName        = $SqlDbServerName
        ResourceGroupName = $ResourceGroupName
        ErrorAction       = 'SilentlyContinue'
    }
    Remove-AzSqlServer @removeParams

    Write-Host "Remove the Azure Resource Group."
    
    Remove-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
} else {
    Write-Host "Skipped over the DELETE of SQL Database and resource group."
}

Write-Host 'Completed script 4, the "Clean-Up".'
```

<a name="a-actual-output"></a>

<a name="a-verify-subnet-is-endpoint-ps-100"></a>

## <a name="verify-your-subnet-is-an-endpoint"></a>Compruebe que la subred es un punto de conexión

Es posible que tenga una subred a la que ya se haya asignado el nombre de tipo **Microsoft.Sql**, lo que significa que ya es un punto de conexión de servicio virtual. Puede usar [Azure Portal][http-azure-portal-link-ref-477t] para crear una regla de red virtual desde el punto de conexión.

O bien, es posible que no sepa con seguridad si su subred tiene el nombre de tipo **Microsoft.Sql**. Puede ejecutar el siguiente script de PowerShell para realizar estas acciones:

1. Compruebe que su subred tenga el nombre de tipo **Microsoft.Sql**.
2. En caso de que falte, puede asignar el nombre de tipo.
    - El script le pedirá su *confirmación* antes de aplicar el nombre de tipo que falta.

### <a name="phases-of-the-script"></a>Fases del script

A continuación, se muestran las fases del script de PowerShell:

1. Inicie sesión en su cuenta de Azure (solo debe hacerlo una vez por sesión de PS).  Asigne las variables.
2. Busque la red virtual y, a continuación, su subred.
3. ¿Su subred está etiquetada como tipo de servidor de punto de conexión de **Microsoft.Sql**?
4. Agregue un punto de conexión del servicio virtual con el nombre de tipo **Microsoft.Sql** en su subred.

> [!IMPORTANT]
> Antes de ejecutar este script, debe editar los valores asignados a las variables de $, cerca de la parte superior del script.

### <a name="direct-powershell-source-code"></a>Código fuente directo de PowerShell

Este script de PowerShell no actualiza nada, a menos que responda Sí si le pide confirmación. El script puede agregar el nombre de tipo **Microsoft.Sql** a su subred. No obstante, el script solo intenta la adición si falta el nombre de tipo a la subred.

```powershell
### 1. LOG into to your Azure account, needed only once per PS session.  Assign variables.
$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]'
if ('yes' -eq $yesno) { Connect-AzAccount }

# Assignments to variables used by the later scripts.
# You can EDIT these values, if necessary.

$SubscriptionName = 'yourSubscriptionName'
Select-AzSubscription -SubscriptionName "$SubscriptionName"

$ResourceGroupName = 'yourRGName'
$VNetName = 'yourVNetName'
$SubnetName = 'yourSubnetName'
$SubnetAddressPrefix = 'Obtain this value from the Azure portal.' # Looks roughly like: '10.0.0.0/24'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql'  # Do NOT edit. Is official value.

### 2. Search for your virtual network, and then for your subnet.
# Search for the virtual network.
$vnet = $null
$vnet = Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Name $VNetName

if ($vnet -eq $null) {
    Write-Host "Caution: No virtual network found by the name '$VNetName'."
    return
}

$subnet = $null
for ($nn = 0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $subnet = $vnet.Subnets[$nn]
    if ($subnet.Name -eq $SubnetName) { break }
    $subnet = $null
}

if ($null -eq $subnet) {
    Write-Host "Caution: No subnet found by the name '$SubnetName'"
    Return
}

### 3. Is your subnet tagged as 'Microsoft.Sql' endpoint server type?
$endpointMsSql = $null
for ($nn = 0; $nn -lt $subnet.ServiceEndpoints.Count; $nn++) {
    $endpointMsSql = $subnet.ServiceEndpoints[$nn]
    if ($endpointMsSql.Service -eq $ServiceEndpointTypeName_SqlDb) {
        $endpointMsSql
        break
    }
    $endpointMsSql = $null
}

if ($null -eq $endpointMsSql) {
    Write-Host "Good: Subnet found, and is already tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'."
    return
} else {
    Write-Host "Caution: Subnet found, but not yet tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'."

    # Ask the user for confirmation.
    $yesno = Read-Host 'Do you want the PS script to apply the endpoint type name to your subnet?  [yes/no]'
    if ('no' -eq $yesno) { return }
}

### 4. Add a Virtual Service endpoint of type name 'Microsoft.Sql', on your subnet.
$setParams = @{
    Name            = $SubnetName
    AddressPrefix   = $SubnetAddressPrefix
    VirtualNetwork  = $vnet
    ServiceEndpoint = $ServiceEndpointTypeName_SqlDb
}
$vnet = Set-AzVirtualNetworkSubnetConfig @setParams

# Persist the subnet update.
$vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet

for ($nn = 0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $vnet.Subnets[0].ServiceEndpoints # Display.
}
```

<!-- Link references: -->
[sql-db-vnet-service-endpoint-rule-overview-735r]:../vnet-service-endpoint-rule-overview.md
[http-azure-portal-link-ref-477t]: https://portal.azure.com/
