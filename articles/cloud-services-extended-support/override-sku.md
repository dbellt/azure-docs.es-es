---
title: Invalidación de la información de SKU sobre CSCFG/CSDEF para Azure Cloud Services (soporte extendido)
description: Invalidación de la información de SKU sobre CSCFG/CSDEF para Azure Cloud Services (soporte extendido)
ms.topic: how-to
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachandw
ms.date: 04/05/2021
ms.custom: ''
ms.openlocfilehash: d5dfae4b5cfee8f61e11e418a05e86017d119410
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739267"
---
# <a name="override-sku-information-over-cscfgcsdef-in-cloud-services-extended-support"></a>Invalidación de la información de SKU sobre CSCFG/CSDEF en Cloud Services (soporte extendido) 

Esta característica permite al usuario actualizar el tamaño de rol y el recuento de instancias en su instancia de Cloud Services mediante la propiedad **allowModelOverride** sin tener que actualizar los archivos de configuración de servicio y de definición del servicio, lo que permite que el servicio en la nube se escale y reduzca vertical y horizontalmente, y lo haga sin necesidad de volver a empaquetar e implementar.

## <a name="set-allowmodeloverride-property"></a>Definición de la propiedad allowModelOverride
La propiedad allowModelOverride se puede establecer de las maneras siguientes:
* Cuando allowModelOverride = true, la llamada API actualizará el tamaño de rol y el recuento de instancias para el servicio en la nube sin validar los valores con los archivos csdef y cscfg. 
> [!Note]
> El archivo cscfg se actualizará para reflejar el recuento de instancias de rol, pero el archivo csdef (dentro de cspkg) conservará los valores anteriores.
* Cuando allowModelOverride = false, la llamada API produciría un error cuando los valores de tamaño de rol y recuento de instancias no coincidan con los de los archivos csdef y cscfg, respectivamente.

El valor predeterminado está establecido en false. Si la propiedad se restablece en false luego haber estado definida en true, se volverían a comprobar los archivos csdef y cscfg con fines de validación.

Consulte los ejemplos siguientes para aplicar la propiedad en PowerShell, la plantilla y el SDK.

### <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure
Si la propiedad "allowModelOverride" se establece en true, se actualizará el servicio en la nube con las propiedades de rol definidas en la sección roleProfile.
```json
"properties": {
        "packageUrl": "[parameters('packageSasUri')]",
        "configurationUrl": "[parameters('configurationSasUri')]",
        "upgradeMode": "[parameters('upgradeMode')]",
        “allowModelOverride” : true,
        "roleProfile": {
          "roles": [
            {
              "name": "WebRole1",
              "sku": {
                "name": "Standard_D1_v2",
                "capacity": "1"
              }
            },
            {
              "name": "WorkerRole1",
              "sku": {
                "name": "Standard_D1_v2",
                "capacity": "1"
              }
            }
          ]
        },

```
### <a name="powershell"></a>PowerShell
Si se establece el modificador "AllowModelOverride" en el nuevo cmdlet New-AzCloudService, se actualizará el servicio en la nube con las propiedades del SKU definidas en RoleProfile.
```powershell
New-AzCloudService ` 
-Name “ContosoCS” ` 
-ResourceGroupName “ContosOrg” ` 
-Location “East US” `
-AllowModelOverride  ` 
-PackageUrl $cspkgUrl ` 
-ConfigurationUrl $cscfgUrl ` 
-UpgradeMode 'Auto' ` 
-RoleProfile $roleProfile ` 
-NetworkProfile $networkProfile  ` 
-ExtensionProfile $extensionProfile ` 
-OSProfile $osProfile `
-Tag $tag
```
### <a name="sdk"></a>SDK
Si la variable AllowModelOverride se establece en true, se actualizará el servicio en la nube con las propiedades del SKU definidas en RoleProfile.

```csharp
CloudService cloudService = new CloudService
    {
        Properties = new CloudServiceProperties
            {
                RoleProfile = cloudServiceRoleProfile
                Configuration = < Add Cscfg xml content here>
                PackageUrl = <Add cspkg SAS url here>,
                ExtensionProfile = cloudServiceExtensionProfile,
                OsProfile= cloudServiceOsProfile,
                NetworkProfile = cloudServiceNetworkProfile,
                UpgradeMode = 'Auto',
                AllowModelOverride = true
            },
                Location = m_location
            };
CloudService createOrUpdateResponse = m_CrpClient.CloudServices.CreateOrUpdate(“ContosOrg”, “ContosoCS”, cloudService);
```
### <a name="azure-portal"></a>Azure Portal
El portal no permite que la propiedad anterior invalide el tamaño de rol y el recuento de instancias en csdef y cscfg. 


## <a name="next-steps"></a>Pasos siguientes 
- Revise los [requisitos previos de implementación](deploy-prerequisite.md) de Cloud Services (soporte extendido).
- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
