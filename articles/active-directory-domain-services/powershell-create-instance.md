---
title: Habilitación de Azure AD Domain Services mediante PowerShell | Microsoft Docs
description: Aprenda a configurar y habilitar Azure Active Directory Domain Services con Azure AD PowerShell y Azure PowerShell.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: d4bc5583-6537-4cd9-bc4b-7712fdd9272a
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 05/19/2021
ms.author: justinha
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f2977e0321a5ba447e2304440ed69be539ad4c52
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467367"
---
# <a name="enable-azure-active-directory-domain-services-using-powershell"></a>Habilitación de Azure Active Directory Domain Services mediante PowerShell

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, LDAP o autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory. Estos servicios de dominio se usan sin necesidad de implementar, administrar ni aplicar revisiones a los controladores de dominio. Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión con sus credenciales corporativas y, además, le permite usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos.

En este artículo se muestra cómo habilitar Azure AD DS con PowerShell.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesita los siguientes recursos:

* Instale y configure Azure PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure PowerShell y conectarse a la suscripción de Azure](/powershell/azure/install-az-ps).
    * Asegúrese de que inicia sesión en su suscripción de Azure con el cmdlet [Connect-AzAccount][Connect-AzAccount].
* Instale y configure Azure AD PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure AD PowerShell y conectarse a Azure AD](/powershell/azure/active-directory/install-adv2).
    * Asegúrese de iniciar sesión en el inquilino de Azure AD mediante el cmdlet [Connect-AzureAD][Connect-AzureAD].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para habilitar Azure AD DS.
* Necesita privilegios de *colaborador* en la suscripción de Azure para crear los recursos de Azure AD DS necesarios.

  > [!IMPORTANT]
  > Aunque el módulo **Az.ADDomainServices** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`.

  ```azurepowershell-interactive
  Install-Module -Name Az.ADDomainServices
  ```

## <a name="create-required-azure-ad-resources"></a>Creación de los recursos de Azure AD necesarios

Azure AD DS necesita una entidad de servicio para autenticarse y comunicarse, así como un grupo de Azure AD para definir qué usuarios tienen permisos administrativos en el dominio administrado.

En primer lugar, cree una entidad de servicio de Azure AD utilizando un identificador de aplicación específico llamado *Servicios de controlador de dominio*. En Public Azure, el valor del identificador es *2565bd9d-da50-47d4-8b85-4c97f669dc36*. En otras nubes, el valor es *6ba9a5d4-8456-4118-b521-9c5ca10cdf84*. No cambie este identificador de aplicación.

Cree una entidad de servicio de Azure AD con el cmdlet [New-AzureADServicePrincipal][New-AzureADServicePrincipal]:

```powershell
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
```

Ahora, cree un grupo de Azure AD denominado *Administradores de controlador de dominio de AAD*. A los usuarios agregados a este grupo se les conceden permisos para realizar tareas de administración en el dominio administrado.

Primero, obtenga el identificador de objeto del grupo *Administradores de DC de AAD* mediante el cmdlet [Get-AzureADGroup][Get-AzureADGroup]. Si el grupo no existe, debe crearlo con el grupo *Administradores de DC de AAD* mediante el cmdlet [New-AzureADGroup][New-AzureADGroup]:

```powershell
# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# If the group doesn't exist, create it
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}
```

Después de crear el grupo *Administradores de DC de AAD*, obtenga el identificador de objeto del usuario deseado mediante el cmdlet [Get-AzureADUser][Get-AzureADUser] y, luego, agregue el usuario al grupo mediante el cmdlet [Add-AzureADGroupMember][Add-AzureADGroupMember].

En el ejemplo siguiente, el identificador de objeto de usuario de la cuenta con un UPN de `admin@contoso.onmicrosoft.com`. Reemplace esta cuenta de usuario por el UPN del usuario que quiere agregar al grupo *Administradores de controlador de dominio de AAD*:

```powershell
# Retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

## <a name="create-network-resources"></a>Crear recursos de red

En primer lugar, registre el proveedor de recursos de Azure AD Domain Services mediante el cmdlet [Register-AzResourceProvider][Register-AzResourceProvider]:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

A continuación, cree un grupo de recursos con el cmdlet [New-AzResourceGroup][New-AzResourceGroup]. En el ejemplo siguiente, el grupo de recursos se denomina *myResourceGroup* y se crea en la región *westus*. Use su propio nombre y la región deseada:

```azurepowershell-interactive
$ResourceGroupName = "myResourceGroup"
$AzureLocation = "westus"

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation
```

Cree la red virtual y las subredes para Azure AD Domain Services. Se crean dos subredes: una para *DomainServices* y otra para *Cargas de trabajo*. Azure AD DS se implementa en la subred dedicada *DomainServices*. No implemente otras aplicaciones o cargas de trabajo en esta subred. Use la subred *Cargas de trabajo* u otras subredes con el resto de las máquinas virtuales.

Cree las subredes mediante el cmdlet [New-AzVirtualNetworkSubnetConfig][New-AzVirtualNetworkSubnetConfig] y, luego, cree la red virtual mediante el cmdlet [New-AzVirtualNetwork][New-AzVirtualNetwork].

```azurepowershell-interactive
$VnetName = "myVnet"

# Create the dedicated subnet for Azure AD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -AddressPrefix 10.0.0.0/24

# Create an additional subnet for your own VM workloads
$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet= New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location westus `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
```

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Azure AD DS necesita un grupo de seguridad de red para proteger los puertos necesarios para el dominio administrado y bloquear el resto del tráfico entrante. Un [grupo de seguridad de red (NSG)][nsg-overview] contiene una lista de reglas que permiten o deniegan el tráfico de red al tráfico de una red virtual de Azure. En Azure AD DS, el grupo de seguridad de red actúa como un nivel de protección adicional para bloquear el acceso al dominio administrado. Para ver los puertos necesarios, consulte [Grupos de seguridad de red y puertos necesarios][network-ports].

Los siguientes cmdlets de PowerShell usan [New-AzNetworkSecurityRuleConfig][New-AzNetworkSecurityRuleConfig] para crear las reglas y, a continuación, [New-AzNetworkSecurityGroup][New-AzNetworkSecurityGroup] para crear el grupo de seguridad de red. Después, el grupo de seguridad de red y las reglas se asocian a la subred de la red virtual mediante el cmdlet [Set-AzVirtualNetworkSubnetConfig][Set-AzVirtualNetworkSubnetConfig].

```azurepowershell-interactive
$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

## <a name="create-a-managed-domain"></a>Creación de un dominio administrado

Ahora vamos a crear un dominio administrado. Establezca el identificador de la suscripción de Azure y luego proporcione un nombre para el dominio administrado, como *aaddscontoso.com*. Puede obtener el identificador de la suscripción con el cmdlet [Get-AzSubscription][Get-AzSubscription].

Si elige una región que admite zonas de disponibilidad, los recursos de Azure AD DS se distribuyen entre las zonas para conseguir redundancia adicional.

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas.

No es necesario realizar ninguna configuración para que Azure AD DS se distribuya entre zonas. La plataforma Azure controla automáticamente la distribución en zonas de los recursos. Para más información y ver la disponibilidad regional, consulte [¿Qué son las zonas de disponibilidad en Azure?][availability-zones]

```azurepowershell-interactive
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Enable Azure AD Domain Services for the directory.
$replicaSetParams = @{
  Location = $AzureLocation
  SubnetId = "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"
}
$replicaSet = New-AzADDomainServiceReplicaSet @replicaSetParams

$domainServiceParams = @{
  Name = $ManagedDomainName
  ResourceGroupName = $ResourceGroupName
  DomainName = $ManagedDomainName
  ReplicaSet = $replicaSet
}
New-AzADDomainService @domainServiceParams
```

Tarda unos minutos hasta que se crea el recurso y se devuelve el control al símbolo del sistema de PowerShell. El dominio administrado se sigue aprovisionando en segundo plano y puede tardar hasta una hora en finalizar la implementación. En Azure Portal, la página **Información general** del dominio administrado muestra el estado actual durante esta fase de implementación.

Cuando en Azure Portal se muestra que el dominio administrado ha terminado de aprovisionarse, es necesario realizar las siguientes tareas:

* Actualice la configuración de DNS para la red virtual de manera que las máquinas virtuales puedan encontrar el dominio administrado para la unión o autenticación de dominios.
    * Para configurar DNS, seleccione el dominio administrado en el portal. En la ventana **Información general**, se le pedirá que configure automáticamente estos valores de DNS.
* [Habilite la sincronización de contraseñas en Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) de manera que los usuarios finales puedan iniciar sesión en el dominio administrado mediante sus credenciales corporativas.

## <a name="complete-powershell-script"></a>Script completo de PowerShell

El siguiente script completo de PowerShell combina todas las tareas que se muestran en este artículo. Copie el script y guárdelo en un archivo con la extensión `.ps1`. Ejecute el script en una consola local de PowerShell o en [Azure Cloud Shell][cloud-shell].

> [!NOTE]
> Para habilitar Azure AD DS, debe ser administrador global del inquilino de Azure AD. También necesita al menos privilegios de *colaborador* en la suscripción de Azure.

```azurepowershell-interactive
# Change the following values to match your deployment.
$AaddsAdminUserUpn = "admin@contoso.onmicrosoft.com"
$ResourceGroupName = "myResourceGroup"
$VnetName = "myVnet"
$AzureLocation = "westus"
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Connect to your Azure AD directory.
Connect-AzureAD

# Login to your Azure subscription.
Connect-AzAccount

# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"

# First, retrieve the object ID of the 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Create the delegated administration group for Azure AD Domain Services if it doesn't already exist.
if (!$GroupObjectId) {
  $GroupObjectId = New-AzureADGroup -DisplayName "AAD DC Administrators" `
    -Description "Delegated group to administer Azure AD Domain Services" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "AADDCAdministrators"
  }
else {
  Write-Output "Admin group already exists."
}

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq '$AaddsAdminUserUpn'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId

# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation

# Create the dedicated subnet for AAD Domain Services.
$SubnetName = "DomainServices"
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $AzureLocation `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet

$NSGName = "aaddsNSG"

# Create a rule to allow inbound TCP port 3389 traffic from Microsoft secure access workstations for troubleshooting
$nsg201 = New-AzNetworkSecurityRuleConfig -Name AllowRD `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 201 `
    -SourceAddressPrefix CorpNetSaw `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389

# Create a rule to allow TCP port 5986 traffic for PowerShell remote management
$nsg301 = New-AzNetworkSecurityRuleConfig -Name AllowPSRemoting `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 301 `
    -SourceAddressPrefix AzureActiveDirectoryDomainServices `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 5986

# Create the network security group and rules
$nsg = New-AzNetworkSecurityGroup -Name $NSGName `
    -ResourceGroupName $ResourceGroupName `
    -Location $AzureLocation `
    -SecurityRules $nsg201,$nsg301

# Get the existing virtual network resource objects and information
$vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $ResourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $SubnetName
$addressPrefix = $subnet.AddressPrefix

# Associate the network security group with the virtual network subnet
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName `
    -VirtualNetwork $vnet `
    -AddressPrefix $addressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork

# Enable Azure AD Domain Services for the directory.
$replicaSetParams = @{
  Location = $AzureLocation
  SubnetId = "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"
}
$replicaSet = New-AzADDomainServiceReplicaSet @replicaSetParams

$domainServiceParams = @{
  Name = $ManagedDomainName
  ResourceGroupName = $ResourceGroupName
  DomainName = $ManagedDomainName
  ReplicaSet = $replicaSet
}
New-AzADDomainService @domainServiceParams
```

Tarda unos minutos hasta que se crea el recurso y se devuelve el control al símbolo del sistema de PowerShell. El dominio administrado se sigue aprovisionando en segundo plano y puede tardar hasta una hora en finalizar la implementación. En Azure Portal, la página **Información general** del dominio administrado muestra el estado actual durante esta fase de implementación.

Cuando en Azure Portal se muestra que el dominio administrado ha terminado de aprovisionarse, es necesario realizar las siguientes tareas:

* Actualice la configuración de DNS para la red virtual de manera que las máquinas virtuales puedan encontrar el dominio administrado para la unión o autenticación de dominios.
    * Para configurar DNS, seleccione el dominio administrado en el portal. En la ventana **Información general**, se le pedirá que configure automáticamente estos valores de DNS.
* [Habilite la sincronización de contraseñas en Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) de manera que los usuarios finales puedan iniciar sesión en el dominio administrado mediante sus credenciales corporativas.

## <a name="next-steps"></a>Pasos siguientes

Para ver el dominio administrado en acción, puede [unir a un dominio una máquina virtual Windows][windows-join], [configurar LDAP seguro][tutorial-ldaps] y [configurar la sincronización de hash de contraseñas][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[nsg-overview]: ../virtual-network/network-security-groups-overview.md
[network-ports]: network-considerations.md#network-security-groups-and-required-ports

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[New-AzVirtualNetworkSubnetConfig]: /powershell/module/Az.Network/New-AzVirtualNetworkSubnetConfig
[New-AzVirtualNetwork]: /powershell/module/Az.Network/New-AzVirtualNetwork
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: ../cloud-shell/cloud-shell-windows-users.md
[availability-zones]: ../availability-zones/az-overview.md
[New-AzNetworkSecurityRuleConfig]: /powershell/module/az.network/new-aznetworksecurityruleconfig
[New-AzNetworkSecurityGroup]: /powershell/module/az.network/new-aznetworksecuritygroup
[Set-AzVirtualNetworkSubnetConfig]: /powershell/module/az.network/set-azvirtualnetworksubnetconfig
