---
title: Deshabilitar las directivas de red de puntos de conexión privados en Azure
description: Obtenga información sobre cómo deshabilitar las directivas de red de puntos de conexión privados.
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6014867ed30901f0b6470d843dd7b3761bc1ba71
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110696612"
---
# <a name="disable-network-policies-for-private-endpoints"></a>Deshabilitar las directivas de red de puntos de conexión privados

Las directivas de red como, por ejemplo, los grupos de seguridad de red (NSG) no se admiten para los puntos de conexión privados. Para implementar un punto de conexión privado en una subred especificada, es necesario un valor de deshabilitación explícito en dicha subred. Este valor solo es aplicable para el punto de conexión privado. Para otros recursos de la subred, el acceso se controla en función de la definición de reglas de seguridad de los grupos de seguridad de red (NSG). 
 
Al usar el portal para crear un punto de conexión privado, este valor se deshabilita automáticamente como parte del proceso de creación. La implementación mediante otros clientes requiere un paso adicional para cambiar este valor. Puede deshabilitar el valor mediante Cloud Shell desde Azure Portal, las instalaciones locales de Azure PowerShell y la CLI de Azure, o bien usar plantillas de Azure Resource Manager.  
 
En los siguientes ejemplos se describe cómo deshabilitar `PrivateEndpointNetworkPolicies` para una red virtual llamada *myVirtualNetwork* con una subred *predeterminada* hospedada en un grupo de recursos llamado *myResourceGroup*.

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell
En esta sección se describe cómo deshabilitar directivas de punto de conexión privado de subred mediante Azure PowerShell.

```azurepowershell
$virtualNetwork= Get-AzVirtualNetwork `
  -Name "myVirtualNetwork" ` 
  -ResourceGroupName "myResourceGroup"  
   
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq 'default'} ).PrivateEndpointNetworkPolicies = "Disabled" 
 
$virtualNetwork | Set-AzVirtualNetwork 
```
## <a name="using-azure-cli"></a>Uso de la CLI de Azure
En esta sección se describe cómo deshabilitar directivas de punto de conexión privado de subred mediante la CLI de Azure.
```azurecli
az network vnet subnet update \ 
  --name default \ 
  --resource-group myResourceGroup \ 
  --vnet-name myVirtualNetwork \ 
  --disable-private-endpoint-network-policies true
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
                                    "privateEndpointNetworkPolicies": "Disabled" 
                                 } 
                         } 
                  ] 
          } 
} 
```
## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el [punto de conexión privado de Azure](private-endpoint-overview.md)
 
