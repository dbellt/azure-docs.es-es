---
title: Configuración de las redes de Azure Key Vault
description: Instrucciones detalladas paso a paso para configurar los firewalls y las redes virtuales de Azule Key Vault
services: key-vault
author: sebansal
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 5/11/2021
ms.author: mbaldwin
ms.custom: devx-track-azurecli
ms.openlocfilehash: d6035436a005012cf67a46302213f79b2dce1f59
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109850199"
---
# <a name="configure-azure-key-vault-networking-settings"></a>Configuración de las redes de Azure Key Vault

En este artículo se proporcionan instrucciones sobre cómo configurar las redes de Azure Key Vault para trabajar con otras aplicaciones y servicios de Azure. Puede obtener información detallada sobre las distintas configuraciones de seguridad de red [aquí](network-security.md).

Instrucciones paso a paso para configurar el firewall y las redes virtuales de Key Vault mediante Azure Portal, la CLI de Azure y Azure PowerShell

# <a name="portal"></a>[Portal](#tab/azure-portal)


1. Vaya al almacén de claves que quiere proteger.
2. Seleccione **Redes** y, luego, elija la pestaña **Firewalls and virtual networks** (Firewalls y redes virtuales).
3. Haga clic en **Redes seleccionadas** en **Permitir el acceso desde**.
4. Para agregar redes virtuales existentes a los firewall y las reglas de red virtual, haga clic en **+ Agregar redes virtuales existentes**.
5. En la nueva hoja que se abrirá, seleccione la suscripción, las redes virtuales y las subredes a las cuales desea otorgar acceso a este almacén de claves. Si las redes virtuales y subredes que seleccionó no tienen puntos de conexión de servicio habilitados, confirme que desea habilitar los puntos de conexión de servicio y seleccione **Habilitar**. Esta operación podría tardar hasta 15 minutos en surtir efecto.
6. En **Redes IP**, agregue rangos de direcciones IPv4; para ello, escriba los rangos de direcciones IPv4 en la [notación CIDR (enrutamiento de interdominios sin clases)](https://tools.ietf.org/html/rfc4632) o en las direcciones IP individuales.
7. Si desea permitir que los servicios de confianza de Microsoft omitan el firewall de Key Vault, seleccione "Sí". Para ver una lista completa de los servicios de confianza Key Vault actuales, consulte el siguiente vínculo. [Servicios de confianza de Azure Key Vault](./overview-vnet-service-endpoints.md#trusted-services)
7. Seleccione **Guardar**.

Asimismo, puede también agregar nuevas redes virtuales y subredes y habilitar los puntos de conexión de servicio de las redes virtuales y subredes recién creadas. Para ello, haga clic en **+ Agregar nueva red virtual**. A continuación, siga las indicaciones.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

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

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

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
---
## <a name="references"></a>Referencias
* Referencia de plantilla de ARM: [Referencia de la plantilla de ARM de Azure Key Vault](/azure/templates/Microsoft.KeyVault/vaults)
* Comandos de la CLI de Azure: [az keyvault network-rule](/cli/azure/keyvault/network-rule)
* Cmdlets de Azure PowerShell: [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>Pasos siguientes

* [Puntos de conexión de servicio de red virtual para Key Vault](overview-vnet-service-endpoints.md)
* [Introducción a la seguridad de Azure Key Vault](security-features.md)
