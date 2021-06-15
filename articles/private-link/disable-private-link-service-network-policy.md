---
title: 'Deshabilitación de directivas de red para la dirección IP de origen del servicio de vínculo privado de Azure '
description: Aprenda a deshabilitar directivas de red del vínculo privado de Azure.
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 887cba48aaac9cd7bc4f1f1f88b61733f0fbb9cf
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696586"
---
# <a name="disable-network-policies-for-private-link-service-source-ip"></a>Deshabilitación de directivas de red para la dirección IP de origen del servicio de vínculo privado

Para elegir una dirección IP de origen para el servicio de vínculo privado, se requiere un valor `privateLinkServiceNetworkPolicies` de deshabilitación explícita en la subred. Esta configuración solo es aplicable a la dirección IP privada específica que se eligió como IP de origen del servicio de vínculo privado. Para otros recursos de la subred, el acceso se controla en función de la definición de reglas de seguridad de los grupos de seguridad de red (NSG). 
 
Al usar el portal para crear un servicio de Private Link, este valor se deshabilita automáticamente como parte del proceso de creación. Las implementaciones con cualquier cliente de Azure (PowerShell, CLI o plantillas), se requiere un paso adicional para cambiar esta propiedad. Puede deshabilitar la directiva mediante Cloud Shell desde Azure Portal, las instalaciones locales de Azure PowerShell o la CLI de Azure, o bien usar plantillas de Azure Resource Manager.  
 
Siga los pasos que se indican a continuación para deshabilitar las directivas de red del servicio de vínculo privado para una red virtual llamada *myVirtualNetwork* con una subred *predeterminada* hospedada en un grupo de recursos llamado *myResourceGroup*. 

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
En esta sección se describe cómo deshabilitar directivas de punto de conexión privado de subred mediante Azure PowerShell.
En el código, reemplace "default" por el nombre de la subred virtual.

```azurepowershell
$virtualSubnetName = "default"
$virtualNetwork= Get-AzVirtualNetwork `
  -Name "myVirtualNetwork" ` 
  -ResourceGroupName "myResourceGroup"
   
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq $virtualSubnetName} ).privateLinkServiceNetworkPolicies = "Disabled"  
 
$virtualNetwork | Set-AzVirtualNetwork 
```
## <a name="using-azure-cli"></a>Uso de la CLI de Azure
En esta sección se describe cómo deshabilitar directivas de punto de conexión privado de subred mediante la CLI de Azure.
```azurecli
az network vnet subnet update \ 
  --name default \ 
  --resource-group myResourceGroup \ 
  --vnet-name myVirtualNetwork \ 
  --disable-private-link-service-network-policies true 
```
## <a name="using-a-template"></a>Uso de una plantilla
En esta sección se describe cómo deshabilitar directivas de punto de conexión privado de subred mediante una plantilla de Azure Resource Manager.
```json
{ 
    "name": "myVirtualNetwork", 
    "type": "Microsoft.Network/virtualNetworks", 
    "apiVersion": "2019-04-01", 
    "location": "WestUS", 
    "properties": { 
        "addressSpace": { 
            "addressPrefixes": [ 
                "10.0.0.0/16" 
             ] 
        }, 
        "subnets": [ 
               { 
                 "name": "default", 
                 "properties": { 
                        "addressPrefix": "10.0.0.0/24", 
                        "privateLinkServiceNetworkPolicies": "Disabled" 
                    } 
                } 
        ] 
    } 
} 
 
```
## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el [punto de conexión privado de Azure](private-endpoint-overview.md).
 
