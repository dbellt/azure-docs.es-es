---
title: Configuración de firewalls y redes virtuales de Azure Key Vault
description: Instrucciones detalladas paso a paso para configurar los firewalls y las redes virtuales de Azule Key Vault
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: 91cba45bc38bddc32aae036a029006c5004da058
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108140648"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>Configuración de firewalls y redes virtuales de Azure Key Vault

En este artículo se proporcionan instrucciones sobre cómo configurar el firewall de Azure Key Vault. En este documento se describen las distintas configuraciones del firewall de Key Vault en detalle y se proporcionan instrucciones paso a paso sobre cómo configurar Azure Key Vault para que funcione con otras aplicaciones y servicios de Azure.

Para más información, consulte [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="firewall-settings"></a>Configuración del firewall

En esta sección se tratan las distintas formas en que se puede configurar el firewall de Azure Key Vault.

### <a name="key-vault-firewall-disabled-default"></a>Firewall de Key Vault Firewall deshabilitado (opción predeterminada)

De forma predeterminada, cuando se crea un nuevo almacén de claves, el firewall de Azure Key Vault está deshabilitado. Todas las aplicaciones y los servicios de Azure pueden acceder al almacén de claves y enviarle solicitudes. Tenga en cuenta que esta configuración no significa que cualquier usuario podrá realizar operaciones en el almacén de claves. El almacén de claves todavía se restringe a los secretos, las claves y los certificados almacenados en él al requerir permisos de directiva de acceso y autenticación de Azure Active Directory. Para comprender con más detalle la autenticación del almacén de claves, consulte [aquí](./authentication-fundamentals.md) el documento sobre los aspectos básicos de la autenticación del almacén de claves. Para más información, consulte [Acceso a Azure Key Vault desde detrás de un firewall](./access-behind-firewall.md).

### <a name="key-vault-firewall-enabled-trusted-services-only"></a>Firewall de Key Vault habilitado (solo servicios de confianza)

Al habilitar el firewall de Key Vault, se le ofrecerá la opción de permitir que los servicios de confianza de Microsoft omitan este firewall. La lista de servicios de confianza no cubre todos los servicios de Azure. Por ejemplo, Azure DevOps no se encuentra en la lista de servicios de confianza. **Esto no implica que los servicios que no aparecen en la lista de servicios de confianza no sean de confianza o no sean seguros.** La lista de servicios de confianza abarca los servicios en los que Microsoft controla todo el código que se ejecuta en el servicio. Dado que los usuarios pueden escribir código personalizado en servicios de Azure como Azure DevOps, Microsoft no ofrece la opción de crear una aprobación global para el servicio. Además, que un servicio aparezca en la lista de servicios de confianza no significa que se permita en todos los escenarios. 

Para determinar si un servicio que está intentando usar está en la lista de servicios de confianza, consulte [aquí](./overview-vnet-service-endpoints.md#trusted-services) el siguiente documento.
Para obtener una guía paso a paso, siga las instrucciones que se proporcionan aquí para [Azure Portal, la CLI de Azure y PowerShell](#use-the-azure-portal)

### <a name="key-vault-firewall-enabled-ipv4-addresses-and-ranges---static-ips"></a>Firewall de Key Vault habilitado (intervalos y direcciones IPv4: direcciones IP estáticas)

Si desea autorizar el acceso de un servicio determinado al almacén de claves a través del firewall de Key Vault, puede agregar su dirección IP a la lista de permitidos del firewall del almacén de claves. Esta configuración es la mejor para los servicios que usan direcciones IP estáticas o intervalos conocidos. En este caso, hay un límite de 1000 intervalos de CIDR.

Para permitir una dirección IP o un intervalo de direcciones de un recurso de Azure, como una aplicación web o una aplicación lógica, realice los pasos siguientes.

1. Iniciar sesión en Azure Portal
1. Seleccione el recurso (instancia específica del servicio).
1. Haga clic en la hoja "Propiedades" en "Configuración".
1. Busque el campo "Dirección IP".
1. Copie este valor o intervalo y escríbalo en la lista de permitidos del firewall del almacén de claves.

Para permitir un servicio completo de Azure a través del firewall de Key Vault, use la lista de direcciones IP de centros de datos documentadas públicamente para Azure que se encuentra [aquí](https://www.microsoft.com/download/details.aspx?id=41653). Busque las direcciones IP asociadas con el servicio que desea en la región que quiera y agregue esas direcciones IP al firewall del almacén de claves mediante los pasos anteriores.

### <a name="key-vault-firewall-enabled-virtual-networks---dynamic-ips"></a>Firewall de Key Vault habilitado (redes virtuales: direcciones IP dinámicas)

Si está intentando permitir un recurso de Azure como una máquina virtual a través de Key Vault, es posible que no pueda usar direcciones IP estáticas y que no desee permitir que todas las direcciones IP de Azure Virtual Machines tengan acceso al almacén de claves.

En este caso, debe crear el recurso en una red virtual y, a continuación, permitir que el tráfico de la red virtual y la subred concretas acceda al almacén de claves. Para ello, siga estos pasos.

1. Iniciar sesión en Azure Portal
1. Seleccione el almacén de claves que desee configurar.
1. Seleccione la hoja "Redes".
1. Seleccione "+Agregar red virtual existente".
1. Seleccione la red virtual y la subred que desee permitir a través del firewall del almacén de claves.

### <a name="key-vault-firewall-enabled-private-link"></a>Firewall de Key Vault habilitado (Private Link)

Para saber cómo configurar una conexión de vínculo privado en el almacén de claves, consulte este [documento](./private-link-service.md).

> [!IMPORTANT]
> Una vez que las reglas del firewall están en vigor, los usuarios solo pueden realizar operaciones del [plano de datos](security-features.md#privileged-access) de Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IPv4 permitidos. Esto también se aplica al acceso de Key Vault desde Azure Portal. Aunque los usuarios pueden ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Esto también afecta al selector de Key Vault de otros servicios de Azure. Los usuarios podrían ver la lista de almacenes de claves, pero no enumerar las claves si las reglas del firewall limitan su equipo cliente.

> [!NOTE]
> Tenga en cuenta las siguientes limitaciones de configuración:
> * Se permite un máximo de 127 reglas de red virtual y 127 reglas de IPv4. 
> * Las reglas de red IP solo se permiten para direcciones IP públicas. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en RFC 1918) en las reglas IP. Las redes privadas incluyen direcciones que comienzan por **10.** , **172.16-31** y **192.168.** . 
> * Solo se admiten direcciones IPV4 en este momento.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

Así es cómo puede configurar firewalls y redes virtuales de Key Vault mediante Azure Portal:

1. Vaya al almacén de claves que quiere proteger.
2. Seleccione **Redes** y, luego, elija la pestaña **Firewalls and virtual networks** (Firewalls y redes virtuales).
3. Haga clic en **Redes seleccionadas** en **Permitir el acceso desde**.
4. Para agregar redes virtuales existentes a los firewall y las reglas de red virtual, haga clic en **+ Agregar redes virtuales existentes**.
5. En la nueva hoja que se abrirá, seleccione la suscripción, las redes virtuales y las subredes a las cuales desea otorgar acceso a este almacén de claves. Si las redes virtuales y subredes que seleccionó no tienen puntos de conexión de servicio habilitados, confirme que desea habilitar los puntos de conexión de servicio y seleccione **Habilitar**. Esta operación podría tardar hasta 15 minutos en surtir efecto.
6. En **Redes IP**, agregue rangos de direcciones IPv4; para ello, escriba los rangos de direcciones IPv4 en la [notación CIDR (enrutamiento de interdominios sin clases)](https://tools.ietf.org/html/rfc4632) o en las direcciones IP individuales.
7. Si desea permitir que los servicios de confianza de Microsoft omitan el firewall de Key Vault, seleccione "Sí". Para ver una lista completa de los servicios de confianza Key Vault actuales, consulte el siguiente vínculo. [Servicios de confianza de Azure Key Vault](./overview-vnet-service-endpoints.md#trusted-services)
7. Seleccione **Guardar**.

Asimismo, puede también agregar nuevas redes virtuales y subredes y habilitar los puntos de conexión de servicio de las redes virtuales y subredes recién creadas. Para ello, haga clic en **+ Agregar nueva red virtual**. A continuación, siga las indicaciones.

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure 

Estos son los pasos para configurar firewalls y redes virtuales de Key Vault mediante la CLI de Azure

1. [Instale la CLI de Azure](/cli/azure/install-azure-cli) e [inicie sesión](/cli/azure/authenticate-azure-cli).

2. Se muestra la lista de reglas de red virtual disponibles. Si no ha establecido ninguna regla para este almacén de claves, la lista estará vacía.
   ```azurecli
   az keyvault network-rule list --resource-group myresourcegroup --name mykeyvault
   ```

3. Habilite un punto de conexión de servicio para Key Vault en una red virtual y subred existentes.
   ```azurecli
   az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.KeyVault"
   ```

4. Agregue una regla de red para una red virtual y subred.
   ```azurecli
   subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
   az keyvault network-rule add --resource-group "demo9311" --name "demo9311premium" --subnet $subnetid
   ```

5. Agregue un rango de direcciones IP desde el que se va a permitir el tráfico.
   ```azurecli
   az keyvault network-rule add --resource-group "myresourcegroup" --name "mykeyvault" --ip-address "191.10.18.0/24"
   ```

6. Si cualquier servicio de confianza necesita obtener acceso a este almacén de claves, establezca la opción `bypass` en `AzureServices`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mykeyvault" --bypass AzureServices
   ```

7. Active las reglas de red estableciendo la acción predeterminada en `Deny`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mekeyvault" --default-action Deny
   ```

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Estas son las instrucciones para configurar los firewalls y las redes virtuales de Key Vault mediante PowerShell:

1. Instale la última versión de [Azure PowerShell](/powershell/azure/install-az-ps) e [inicie sesión](/powershell/azure/authenticate-azureps).

2. Se muestra la lista de reglas de red virtual disponibles. Si no ha establecido ninguna regla para este almacén de claves, la lista estará vacía.
   ```powershell
   (Get-AzKeyVault -VaultName "mykeyvault").NetworkAcls
   ```

3. Habilite el punto de conexión de servicio para Key Vault en una red virtual y subred existentes.
   ```powershell
   Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
   ```

4. Agregue una regla de red para una red virtual y subred.
   ```powershell
   $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -VirtualNetworkResourceId $subnet.Id
   ```

5. Agregue un rango de direcciones IP desde el que se va a permitir el tráfico.
   ```powershell
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -IpAddressRange "16.17.18.0/24"
   ```

6. Si cualquier servicio de confianza necesita obtener acceso a este almacén de claves, establezca la opción `bypass` en `AzureServices`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -Bypass AzureServices
   ```

7. Active las reglas de red estableciendo la acción predeterminada en `Deny`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -DefaultAction Deny
   ```

## <a name="references"></a>Referencias
* Referencia de plantilla de ARM: [Referencia de la plantilla de ARM de Azure Key Vault](/azure/templates/Microsoft.KeyVault/vaults)
* Comandos de la CLI de Azure: [az keyvault network-rule](/cli/azure/keyvault/network-rule)
* Cmdlets de Azure PowerShell: [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>Pasos siguientes

* [Puntos de conexión de servicio de red virtual para Key Vault](overview-vnet-service-endpoints.md)
* [Introducción a la seguridad de Azure Key Vault](security-features.md)