---
title: Restricción de la filtración de datos a Azure Storage - Azure PowerShell
description: En este artículo aprenderá a limitar y restringir la filtración de datos de red virtual a los recursos de Azure Storage con directivas de puntos de conexión de servicio de red virtual mediante Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: RDhillon
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 785cdcc2436486b69f9033bfa1a4fb0cb441d2a0
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110678741"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-azure-powershell"></a>Administración de la filtración de datos a cuentas de Azure Storage con directivas de punto de conexión de servicio de red virtual mediante Azure PowerShell

Las directivas de punto de conexión de servicio de red virtual permiten aplicar el control de acceso en cuentas de Azure Storage desde una red virtual a través de puntos de conexión de servicio. Se trata de una clave para proteger las cargas de trabajo, administrar qué cuentas de almacenamiento se permiten y dónde se permite la filtración de datos.
En este artículo aprenderá a:

* Cree una red virtual.
* Agregue una subred y habilite un punto de conexión de servicio para Azure Storage.
* Cree dos cuentas de Azure Storage y permita el acceso de red a estas desde la subred creada anteriormente.
* Crear una directiva de punto de conexión de servicio para permitir el acceso solo a una de las cuentas de almacenamiento.
* Implementar una máquina virtual (VM) en la subred.
* Confirmar el acceso a la cuenta de almacenamiento permitida desde la subred.
* Confirmar que se ha denegado el acceso a la cuenta de almacenamiento no permitida desde la subred.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 1.0.0 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Antes de crear una red virtual, cree un grupo de recursos para ella y los demás recursos que se crearon en este artículo. Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup*: 

```azurepowershell-interactive
New-AzResourceGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS
```

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada *myVirtualNetwork* con el prefijo de dirección *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

## <a name="enable-a-service-endpoint"></a>Habilitación de un punto de conexión de servicio

Cree una subred en la red virtual. En este ejemplo, se crea una subred denominada *Private* con un punto de conexión de servicio para *Microsoft.Storage*: 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-for-the-subnet"></a>Restricción del acceso de la red para la subred

Cree reglas de seguridad de grupo de seguridad de red con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). La siguiente regla permite el acceso de salida a las direcciones IP públicas asignadas al servicio Azure Storage: 

```azurepowershell-interactive
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

La siguiente regla deniega el acceso a todas las direcciones IP públicas. La regla anterior invalida esta regla porque su prioridad es más alta, lo que permite el acceso a las direcciones IP públicas de Azure Storage.

```azurepowershell-interactive
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name Deny-Internet-All `
  -Access Deny `
  -DestinationAddressPrefix Internet `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 110 -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

La siguiente regla permite que el tráfico de Protocolo de escritorio remoto (RDP) pueda entrar en la subred desde cualquier lugar. Se permiten las conexiones de Escritorio remoto a la subred, por lo que puede confirmar acceso a la red a un recurso en un paso posterior.

```azurepowershell-interactive
$rule3 = New-AzNetworkSecurityRuleConfig `
  -Name Allow-RDP-All `
  -Access Allow `
  -DestinationAddressPrefix VirtualNetwork `
  -DestinationPortRange 3389 `
  -Direction Inbound `
  -Priority 120 `
  -Protocol * `
  -SourceAddressPrefix * `
  -SourcePortRange *
```

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNsgPrivate*.

```azurepowershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myNsgPrivate `
  -SecurityRules $rule1,$rule2,$rule3
```

Asocie el grupo de seguridad de red a la subred *Private* con [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) y escriba la configuración de la subred en la red virtual. En el ejemplo siguiente se asocia el grupo de seguridad de red *myNsgPrivate* a la subred *Private*:

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="restrict-network-access-to-azure-storage-accounts"></a>Restricción del acceso de red a las cuentas de Azure Storage

Los pasos que deben seguirse para restringir el acceso de la red a los recursos creados con servicios de Azure habilitados para puntos de conexión de servicio varían en función del servicio. Consulte en la documentación de cada servicio concreto los pasos necesarios para dicho servicio. Como ejemplo, de aquí en adelante se explican los pasos necesarios para restringir el acceso de red en una cuenta de Azure Storage.

### <a name="create-two-storage-accounts"></a>Creación de dos cuentas de almacenamiento

Cree una cuenta de almacenamiento de Azure con [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

```azurepowershell-interactive
$storageAcctName1 = 'allowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName1 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Después de crear la cuenta de almacenamiento, recupere la clave para dicha cuenta en una variable con [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey):

```azurepowershell-interactive
$storageAcctKey1 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName1).Value[0]
```

La clave se utiliza para crear un recurso compartido de archivos en un paso posterior. Especifique `$storageAcctKey` y anote el valor, ya que tendrá que escribirlo manualmente más adelante, cuando asigne el recurso compartido de archivos a una unidad de una máquina virtual.

Ahora repita los pasos anteriores para crear una segunda cuenta de almacenamiento.

```azurepowershell-interactive
$storageAcctName2 = 'notallowedaccount'

New-AzStorageAccount `
  -Location EastUS `
  -Name $storageAcctName2 `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

Recupere también la clave de la cuenta de almacenamiento de esta cuenta para usarla posteriormente para crear un recurso compartido de archivos.

```azurepowershell-interactive
$storageAcctKey2 = (Get-AzStorageAccountKey -ResourceGroupName myResourceGroup -AccountName $storageAcctName2).Value[0]
```

### <a name="create-a-file-share-in-each-of-the-storage-account"></a>Creación de un recurso compartido de archivos en cada cuenta de almacenamiento

Cree un contexto para la clave y la cuenta de almacenamiento con [New-AzStorageContext](/powershell/module/az.storage/new-AzStoragecontext). El contexto encapsula el nombre y la clave de la cuenta de almacenamiento:

```azurepowershell-interactive
$storageContext1 = New-AzStorageContext $storageAcctName1 $storageAcctKey1

$storageContext2 = New-AzStorageContext $storageAcctName2 $storageAcctKey2
```

Cree un recurso compartido de archivos con [New-AzStorageShare](/powershell/module/az.storage/new-azstorageshare):

```azurepowershell-interactive
$share1 = New-AzStorageShare my-file-share -Context $storageContext1

$share2 = New-AzStorageShare my-file-share -Context $storageContext2
```

### <a name="deny-all-network-access-to-a-storage-accounts"></a>Denegación de todo el acceso de red a una cuenta de almacenamiento

De forma predeterminada, las cuentas de almacenamiento aceptan conexiones de red procedentes de clientes de cualquier red. Para limitar el acceso a redes seleccionadas, cambie la acción predeterminada a *Deny* con [Update-AzStorageAccountNetworkRuleSet](/powershell/module/az.storage/update-azstorageaccountnetworkruleset). Una vez que se deniega el acceso a la red, no se puede acceder a la cuenta de almacenamiento desde ninguna red.

```azurepowershell-interactive
Update-AzStorageAccountNetworkRuleSet `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -DefaultAction Deny

Update-AzStorageAccountNetworkRuleSet  `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -DefaultAction Deny
```

### <a name="enable-network-access-only-from-the-vnet-subnet"></a>Habilitación del acceso de red solo desde la subred de red virtual

Recupere la red virtual creada con [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) y recupere el objeto de subred privada en una variable con [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig):

```azurepowershell-interactive
$privateSubnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Name myVirtualNetwork `
  | Get-AzVirtualNetworkSubnetConfig -Name Private
```

Permita el acceso a la red a la cuenta de almacenamiento desde la subred *Private* con [Add-AzStorageAccountNetworkRule](/powershell/module/az.network/add-aznetworksecurityruleconfig).

```azurepowershell-interactive
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName1 `
  -VirtualNetworkResourceId $privateSubnet.Id

Add-AzStorageAccountNetworkRule `
  -ResourceGroupName myresourcegroup `
  -Name $storageAcctName2 `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>Aplicación de la directiva para permitir el acceso a una cuenta de almacenamiento válida

Para asegurarse de que los usuarios de la red virtual solo puedan acceder a las cuentas de Azure Storage seguras y permitidas, puede crear una directiva de punto de conexión de servicio con la lista de cuentas de almacenamiento permitidas en la definición. Esta directiva se aplicará a la subred de la red virtual que está conectada al almacenamiento a través de los puntos de conexión de servicio.

### <a name="create-a-service-endpoint-policy"></a>Creación de una directiva de punto de conexión de servicio

En esta sección se crea la definición de directiva con la lista de recursos permitidos para el acceso a través del punto de conexión de servicio.

Recupere el identificador de recurso de la primera cuenta de almacenamiento (permitida). 

```azurepowershell-interactive
$resourceId = (Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name $storageAcctName1).id
```

Cree la definición de directiva para permitir el recurso anterior.

```azurepowershell-interactive
$policyDefinition = New-AzServiceEndpointPolicyDefinition -Name mypolicydefinition `
  -Description "Service Endpoint Policy Definition" `
  -Service "Microsoft.Storage" `
  -ServiceResource $resourceId
```

Cree la directiva de punto de conexión de servicio mediante la definición de directiva creada anteriormente.

```azurepowershell-interactive
$sepolicy = New-AzServiceEndpointPolicy -ResourceGroupName myresourcegroup `
  -Name mysepolicy -Location EastUS
  -ServiceEndpointPolicyDefinition $policyDefinition
```

### <a name="associate-the-service-endpoint-policy-to-the-virtual-network-subnet"></a>Asociación de la directiva de punto de conexión de servicio a la subred de la red virtual

Después de crear la directiva de punto de conexión de servicio, deberá asociarla a la subred de destino con la configuración del punto de conexión de servicio de Azure Storage.

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.0.0/24 `
  -NetworkSecurityGroup $nsg `
  -ServiceEndpoint Microsoft.Storage `
  -ServiceEndpointPolicy $sepolicy

$virtualNetwork | Set-AzVirtualNetwork
```
## <a name="validate-access-restriction-to-azure-storage-accounts"></a>Validación de la restricción de acceso a las cuentas de Azure Storage

### <a name="deploy-the-virtual-machine"></a>Implementación de la máquina virtual

Para probar el acceso de red a una cuenta de almacenamiento, implemente una VM en la subred.

Cree una máquina virtual en la subred *Private* con [New-AzVM](/powershell/module/az.compute/new-azvm). Cuando ejecute el comando siguiente, se le solicitarán las credenciales. Los valores que especifique se configuran como el nombre de usuario y la contraseña de la máquina virtual. La opción `-AsJob` crea la máquina virtual en segundo plano, así que puede continuar con el siguiente paso.

```azurepowershell-interactive
New-AzVm -ResourceGroupName myresourcegroup `
  -Location "East US" `
  -VirtualNetworkName myVirtualNetwork `
  -SubnetName Private `
  -Name "myVMPrivate" -AsJob
```

Se devuelve una salida similar a la del siguiente ejemplo:

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```

### <a name="confirm-access-to-the-allowed-storage-account"></a>Confirmación del acceso a la cuenta de almacenamiento *permitida*

Use [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para devolver la dirección IP pública de una máquina virtual. En el siguiente ejemplo se devuelve la dirección IP pública de la máquina virtual *myVmPrivate*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Reemplace `<publicIpAddress>` en el siguiente comando, por la dirección IP pública devuelta por el comando anterior y, a continuación, escriba el siguiente comando:

```powershell
mstsc /v:<publicIpAddress>
```

Se crea un archivo de Protocolo de Escritorio remoto (.rdp) y se descarga en su equipo. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente), para especificar las credenciales que escribió al crear la máquina virtual. Seleccione **Aceptar**. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.

En la VM *myVmPrivate*, asigne el recurso compartido de archivos de Azure de la cuenta de almacenamiento permitida a la unidad Z con PowerShell. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList ("Azure\allowedaccount"), $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\allowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

PowerShell devuelve una salida similar a la del ejemplo siguiente:

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\allowedaccount.file.core.windows.net\my-f...
```

El recurso compartido de archivos de Azure se ha asignado correctamente a la unidad Z.

Cierre la sesión de Escritorio remoto a la máquina virtual *myVmPrivate*.

### <a name="confirm-access-is-denied-to-non-allowed-storage-account"></a>Comprobación del acceso denegado a la cuenta de almacenamiento *no permitida*

En la misma VM *myVmPrivate*, intente asignar el recurso compartido de archivos de Azure a la unidad X. 

```powershell
$acctKey = ConvertTo-SecureString -String $storageAcctKey1 -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\notallowedaccount", $acctKey
New-PSDrive -Name X -PSProvider FileSystem -Root "\\notallowedaccount.file.core.windows.net\my-file-share" -Credential $credential
```

El acceso al recurso compartido se deniega y recibe el error `New-PSDrive : Access is denied`. Se denegó el acceso porque la cuenta de almacenamiento *notallowedaccount* no está en la lista de recursos permitidos en la directiva de punto de conexión de servicio. 

Cierre la sesión de Escritorio remoto a la máquina virtual *myVmPublic*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene:

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aplicado una directiva de punto de conexión de servicio a través de un punto de conexión de servicio de Azure Virtual Network para Azure Storage. Ha creado cuentas de Azure Storage y limitado el acceso de red solo a determinadas cuentas de almacenamiento (por tanto, denegado a otras) desde una subred de la red virtual. Para más información sobre las directivas de punto de conexión de servicio, consulte [Información general sobre las directivas de punto de conexión de servicio](virtual-network-service-endpoint-policies-overview.md).
