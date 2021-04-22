---
title: 'Azure Policy: extensión de configuración de invitado'
description: Obtenga información sobre la extensión que se usa para auditar o configurar valores dentro de las máquinas virtuales.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: mgreenegit
ms.author: migreene
ms.date: 04/15/2021
ms.openlocfilehash: 2fda3cc2cf9adc3a734780209a0c9cc06a04e7cf
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368549"
---
# <a name="overview-of-the-azure-policy-guest-configuration-extension"></a>Información general sobre la extensión Guest Configuration de Azure Policy

La extensión Guest Configuration es un componente de Azure Policy que realiza operaciones de auditoría y configuración dentro de las máquinas virtuales.
Las directivas, como las definiciones de línea de base de seguridad para [Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffc9b3da7-8347-4380-8e70-0a0361d8dedd) y [Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F72650e9f-97bc-4b2a-ab5f-9781a9fcecbc), no pueden comprobar la configuración dentro de las máquinas hasta que se instale la extensión.

## <a name="prerequisites"></a>Requisitos previos

Para que la máquina se autentique en el servicio de configuración de invitado, esta debe tener una [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/overview.md).
El requisito de la identidad en una máquina virtual se cumple si se establece la propiedad siguiente.

  ```json
  "identity": {
    "type": "SystemAssigned"
  }
  ```

### <a name="operating-systems"></a>Sistemas operativos

La compatibilidad con la extensión Guest Configuration es la misma que con el sistema operativo [documentado para la solución de un extremo a otro.](../../governance/policy/concepts/guest-configuration.md#supported-client-types)

### <a name="internet-connectivity"></a>Conectividad de Internet

El agente instalado por la extensión Guest Configuration debe poder acceder a los paquetes de contenido que enumeran las asignaciones de Guest Configuration y notificar el estado al servicio de configuración de invitado.
La máquina puede conectarse mediante HTTPS saliente a través del puerto TCP 443 o mediante una conexión a través de redes privadas.
Para obtener más información sobre las redes privadas, consulte los artículos siguientes:

- [Guest Configuration, Comunicación a través de un vínculo privado en Azure](../../governance/policy/concepts/guest-configuration.md#communicate-over-private-link-in-azure)
- [Uso de puntos de conexión privados para Azure Storage](../../storage/common/storage-private-endpoints.md)

## <a name="how-can-i-install-the-extension"></a>¿Cómo puedo instalar la extensión?

Para implementar la última versión de la extensión a escala, incluidos los requisitos de identidad, asigne la directiva de Azure Policy destinada a [implementar los requisitos previos para habilitar las directivas de configuración de invitado en máquinas virtuales](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json).
En el caso de las máquinas individuales, la extensión puede implementarse mediante la CLI de Azure, PowerShell, las plantillas de Resource Manager o herramientas de terceros.

El nombre de instancia de la extensión debe establecerse en "AzurePolicyforWindows" o "AzurePolicyforLinux", ya que las directivas a las que se hace referencia anteriormente requieren estas cadenas específicas.

De forma predeterminada, todas las implementaciones se actualizan a la última versión. El valor predeterminado de la propiedad _autoUpgradeMinorVersion_ es "true", a menos que se especifique lo contrario. No es necesario preocuparse por actualizar el código cuando se publiquen nuevas versiones de la extensión.

### <a name="azure-cli"></a>Azure CLI

Para implementar la extensión en Linux:


```azurecli
az vm extension set  --publisher Microsoft.GuestConfiguration --name ConfigurationforLinux --extension-instance-name AzurePolicyforLinux --resource-group myResourceGroup --vm-name myVM
```

Para implementar la extensión en Windows:

```azurecli
az vm extension set  --publisher Microsoft.GuestConfiguration --name ConfigurationforWindows --extension-instance-name AzurePolicyforWindows --resource-group myResourceGroup --vm-name myVM
```

### <a name="powershell"></a>PowerShell

Para implementar la extensión en Linux:

```powershell
Set-AzVMExtension -Publisher 'Microsoft.GuestConfiguration' -Type 'ConfigurationforLinux' -Name 'AzurePolicyforLinux' -TypeHandlerVersion 1.0 -ResourceGroupName 'myResourceGroup' -Location 'myLocation' -VMName 'myVM'
```

Para implementar la extensión en Windows:

```powershell
Set-AzVMExtension -Publisher 'Microsoft.GuestConfiguration' -Type 'ConfigurationforWindows' -Name 'AzurePolicyforWindows' -TypeHandlerVersion 1.0 -ResourceGroupName 'myResourceGroup' -Location 'myLocation' -VMName 'myVM'
```

### <a name="resource-manager-template"></a>Plantilla de Resource Manager

Para implementar la extensión en Linux:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(parameters('VMName'), '/AzurePolicyforLinux')]",
  "apiVersion": "2019-07-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.GuestConfiguration",
    "type": "ConfigurationforLinux",
    "typeHandlerVersion": "1.0"
    "autoUpgradeMinorVersion": true,
    "settings": {},
    "protectedSettings": {}
  }
}
```

Para implementar la extensión en Windows:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(parameters('VMName'), '/AzurePolicyforWindows')]",
  "apiVersion": "2019-07-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.GuestConfiguration",
    "type": "ConfigurationforWindows",
    "typeHandlerVersion": "1.0"
    "autoUpgradeMinorVersion": true,
    "settings": {},
    "protectedSettings": {}
  }
}
```

### <a name="terraform"></a>Terraform

Para implementar la extensión en Linux:

```terraform
resource "azurerm_virtual_machine_extension" "gc" {
  name                  = "AzurePolicyforLinux"
  virtual_machine_id    = "myVMID"
  publisher             = "Microsoft.GuestConfiguration"
  type                  = "ConfigurationforLinux"
  type_handler_version  = "1.0"
}
```

Para implementar la extensión en Windows:

```terraform
resource "azurerm_virtual_machine_extension" "gc" {
  name                  = "AzurePolicyforWindows"
  virtual_machine_id    = "myVMID"
  publisher             = "Microsoft.GuestConfiguration"
  type                  = "ConfigurationforWindows"
  type_handler_version  = "1.0"
}
```

## <a name="settings"></a>Configuración

No es necesario incluir ninguna configuración ni propiedades de configuración protegida en la extensión.
El agente recupera toda esta información de los recursos de [asignación de configuración de invitado](/rest/api/guestconfiguration/guestconfigurationassignments). Por ejemplo, las propiedades [ConfigurationUri](/rest/api/guestconfiguration/guestconfigurationassignments/createorupdate#guestconfigurationnavigation), [Mode](/rest/api/guestconfiguration/guestconfigurationassignments/createorupdate#configurationmode) y [ConfigurationSetting](/rest/api/guestconfiguration/guestconfigurationassignments/createorupdate#configurationsetting) se administran por configuración, en lugar de en la extensión de VM.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre Guest Configuration de Azure Policy, consulte [Información sobre Guest Configuration de Azure Policy](../../governance/policy/concepts/guest-configuration.md).
* Para obtener más información sobre cómo funcionan las extensiones y el agente de Linux, vea [Características y extensiones de VM de Azure para Linux](features-linux.md).
* Para más información sobre cómo funcionan las extensiones de Windows y Windows Guest Agent, consulte [Características y extensiones de VM de Azure para Windows](features-windows.md).  
* Para instalar Windows Guest Agent, vea [Información general del agente de máquina virtual de Azure](agent-windows.md).  
* Para instalar el agente de Linux, vea [Información y uso del agente de Linux de Azure](agent-linux.md).  
