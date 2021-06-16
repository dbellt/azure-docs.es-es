---
title: Traslado de un equilibrador de carga interno de Azure a otra región de Azure mediante Azure PowerShell
description: Use una plantilla de Azure Resource Manager para trasladar un equilibrador de carga interno de Azure de una región de Azure a otra mediante Azure PowerShell.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 048d6e423c063485705636cac3db0dfa6241874f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704889"
---
# <a name="move-azure-internal-load-balancer-to-another-region-using-powershell"></a>Traslado de un equilibrador de carga interno de Azure a otra región mediante PowerShell

Hay varios escenarios en los que quizá quiera trasladar su equilibrador de carga interno actual de una región a otra. Por ejemplo, puede que quiera crear un equilibrador de carga interno con la misma configuración para realizar pruebas. También puede que quiera trasladar un equilibrador de carga interno a otra región como parte del planeamiento para la recuperación ante desastres.

Los equilibradores de carga internos de Azure no se pueden trasladar de una región a otra. Sin embargo, puede usar una plantilla de Azure Resource Manager para exportar la configuración y la red virtual actuales de un equilibrador de carga interno.  Después, puede preparar el recurso para otra región al exportar el equilibrador de carga y la red virtual a una plantilla, modificar los parámetros para que coincidan con la región de destino y, a continuación, implementar la plantilla en la nueva región.  Para más información sobre Resource Manager y sus plantillas, consulte [Exportación de grupos de recursos a plantillas](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que el equilibrador de carga interno de Azure se encuentra en la región de Azure desde la que va a realizar el traslado.

- Los equilibradores de carga internos de Azure no se pueden trasladar entre regiones.  Tendrá que asociar el nuevo equilibrador de carga a los recursos de la región de destino.

- Necesitará contar con el rol de colaborador de red u otro superior para exportar la configuración de un equilibrador de carga interno e implementar una plantilla para crear un equilibrador de carga interno en otra región.
   
- Identifique el diseño de red de origen y todos los recursos que está usando actualmente. Este diseño incluye, pero no se limita a los equilibradores de carga, los grupos de seguridad de red, las máquinas virtuales y las redes virtuales.

- Compruebe que su suscripción a Azure permite crear equilibradores de carga internos en la región de destino que se usa. Para habilitar la cuota necesaria, póngase en contacto con el soporte técnico.

- Asegúrese de que la suscripción tiene suficientes recursos para admitir la adición de equilibradores de carga para este proceso.  Vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)


## <a name="prepare-and-move"></a>Preparación y traslado
En los pasos siguientes se muestra cómo preparar el equilibrador de carga interno para el traslado mediante una plantilla de Resource Manager y cómo trasladar la configuración del equilibrador de carga interno a la región de destino mediante Azure PowerShell.  Como parte de este proceso, la configuración de la red virtual del equilibrador de carga interno debe incluirse y debe crearse antes de trasladar el equilibrador de carga interno.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-virtual-network-template-and-deploy-from-azure-powershell"></a>Exportación de la plantilla de red virtual e implementación desde Azure PowerShell

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones de la pantalla:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
2.  Obtenga el identificador de recurso de la red virtual que quiere trasladar a la región de destino y colóquelo en una variable mediante [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):

    ```azurepowershell-interactive
    $sourceVNETID = (Get-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exporte la red virtual de origen a un archivo .json en el directorio donde se ejecuta el comando [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. El nombre del archivo descargado se asignará en función del grupo de recursos desde el que se exportó el recurso.  Busque el archivo que se exportó con el comando denominado **\<resource-group-name>.json** y ábralo en el editor que prefiera:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Para editar el parámetro del nombre de la red virtual, cambie la propiedad **defaultValue** del nombre de la red virtual de origen por el nombre de la red virtual de destino y asegúrese de que el nombre esté entre comillas:
    
    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentmyResourceGroupVNET.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_myVNET1_name": {
        "defaultValue": "<target-virtual-network-name>",
        "type": "String"
        }
    ```

6.  Para editar la región de destino a la que se va a trasladar la red virtual, cambie la propiedad **location** en resources:

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```
  
7. Para obtener los códigos de ubicación de la región, puede usar el cmdlet Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation) al ejecutar el siguiente comando:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8.  Si quiere, también puede cambiar otros parámetros del archivo **\<resource-group-name>.json**. Estos son opcionales según sus necesidades:

    * **Espacio de direcciones**: el espacio de direcciones de la red virtual se puede modificar antes de guardar al editar la sección **resources** > **addressSpace** y cambiar la propiedad **addressPrefixes** en el archivo **\<resource-group-name>.json**:

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },

        ```

    * **Subred**: es posible modificar o realizar adiciones al nombre de subred y al espacio de direcciones de subred si edita la sección **subnets** del archivo **\<resource-group-name>.json**. El nombre de la subred se puede cambiar si modifica la propiedad **name**. El espacio de direcciones de subred se puede cambiar si modifica la propiedad **addressPrefix** en el archivo **\<resource-group-name>.json**:

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

         Para cambiar el prefijo de dirección, debe editar el archivo **\<resource-group-name>.json** en dos lugares; primero en la sección mencionada anteriormente y después en la sección **type** a continuación.  Cambie la propiedad **addressPrefix** para que coincida con la anterior:

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

9.  Guarde el archivo **\<resource-group-name>.json**.

10. Cree un grupo de recursos en la región de destino para la red virtual de destino que se va a implementar mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
11. Implemente el archivo **\<resource-group-name>.json** editado en el grupo de recursos que creó en el paso anterior mediante [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```
12. Para comprobar que los recursos se crearon en la región de destino, use los comandos [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup) y [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzVirtualNetwork -Name <target-virtual-network-name> -ResourceGroupName <target-resource-group-name>

    ```
### <a name="export-the-internal-load-balancer-template-and-deploy-from-azure-powershell"></a>Exportación de la plantilla del equilibrador de carga interno e implementación desde Azure PowerShell

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones de la pantalla:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenga el identificador del recurso del equilibrador de carga interno que quiere trasladar a la región de destino y colóquelo en una variable mediante [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer):

    ```azurepowershell-interactive
    $sourceIntLBID = (Get-AzLoadBalancer -Name <source-internal-lb-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exporte la configuración del equilibrador de carga interno de origen a un archivo .json en el directorio donde se ejecuta el comando [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceIntLBID -IncludeParameterDefaultValue
   ```
4. El nombre del archivo descargado se asignará en función del grupo de recursos desde el que se exportó el recurso.  Busque el archivo que se exportó con el comando denominado **\<resource-group-name>.json** y ábralo en el editor que prefiera:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Para editar el parámetro del nombre del equilibrador de carga interno, cambie la propiedad **defaultValue** del nombre del equilibrador de carga interno de origen por el nombre del equilibrador de carga interno de destino. Asegúrese de que el nombre se encuentra entre comillas:

    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_externalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```
 
6. Para editar el valor de la red virtual de destino que se traslado anteriormente, primero debe obtener el Id. de recurso y después copiarlo y pegarlo en el archivo **\<resource-group-name>.json**.  Para obtener el identificador, use [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):
   
   ```azurepowershell-interactive
    $targetVNETID = (Get-AzVirtualNetwork -Name <target-vnet-name> -ResourceGroupName <target-resource-group-name>).Id
    ```
    Escriba la variable y presione ENTRAR para mostrar el identificador del recurso.  Resalte la ruta de acceso del identificador y cópiela en el portapapeles:

    ```powershell
    PS C:\> $targetVNETID
    /subscriptions/7668d659-17fc-4ffd-85ba-9de61fe977e8/resourceGroups/myResourceGroupVNET-Move/providers/Microsoft.Network/virtualNetworks/myVNET2-Move
    ```

7.  En el archivo **\<resource-group-name>.json**, pegue el **identificador del recurso** de la variable en lugar del valor **defaultValue** en el segundo parámetro del identificador de la red virtual de destino y asegúrese de escribir la ruta de acceso entre comillas:
   
    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_externalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```

8. Para editar la región de destino a la que se va a trasladar la configuración del equilibrador de carga interno, cambie la propiedad **location** en **resources** del archivo **\<resource-group-name>.json**:

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-internal-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

11. Para obtener los códigos de ubicación de la región, puede usar el cmdlet Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation) al ejecutar el siguiente comando:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
12. También puede cambiar otros parámetros de la plantilla si así lo desea; son opcionales según sus requisitos:
    
    * **SKU**: puede cambiar la SKU del equilibrador de carga interno en la configuración del nivel estándar al básico o viceversa si modifica la propiedad **sku** > **name** en el archivo **\<resource-group-name>.json**:

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-internal-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Para más información sobre las diferencias entre los equilibradores de carga de la SKU básica y estándar, consulte [Introducción a Azure Standard Load Balancer](./load-balancer-overview.md).

    * **Reglas de equilibrio de carga**: puede agregar o quitar reglas de equilibrio de carga en la configuración agregando o quitando entradas en la sección **loadBalancingRules** del archivo **\<resource-group-name>.json**:

        ```json
        "loadBalancingRules": [
                    {
                        "name": "myInboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false,
                            "loadDistribution": "Default",
                            "disableOutboundSnat": true,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolInbound')]"
                            },
                            "probe": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/probes/myHTTPProbe')]"
                            }
                        }
                    }
                ]
        ```
       Para más información sobre las reglas de equilibrio de carga, consulte [¿Qué es Azure Load Balancer?](./load-balancer-overview.md)

    * **Sondeos**: puede agregar o quitar sondeos para el equilibrador de carga en la configuración agregando o quitando entradas en la sección **probes** del archivo **\<resource-group-name>.json**:

        ```json
        "probes": [
                    {
                        "name": "myHTTPProbe",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "Http",
                            "port": 80,
                            "requestPath": "/",
                            "intervalInSeconds": 15,
                            "numberOfProbes": 2
                        }
                    }
                ],
        ```
       Para más información sobre los sondeos de estado de Azure Load Balancer, consulte [Sondeos de estado de Load Balancer](./load-balancer-custom-probe-overview.md).

    * **Reglas NAT de entrada**: puede agregar o quitar reglas NAT de entrada para el equilibrador de carga agregando o quitando entradas en la sección **inboundNatRules** del archivo **\<resource-group-name>.json**:

        ```json
        "inboundNatRules": [
                    {
                        "name": "myInboundNATRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 4422,
                            "backendPort": 3389,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false
                        }
                    }
                ]
        ```
        Para completar la adición o eliminación de una regla NAT de entrada, esta debe agregarse o quitarse como propiedad **type** al final del archivo **\<resource-group-name>.json**:

        ```json
        {
            "type": "Microsoft.Network/loadBalancers/inboundNatRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('loadBalancers_myLoadBalancer_name'), '/myInboundNATRule')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "frontendIPConfiguration": {
                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                },
                "frontendPort": 4422,
                "backendPort": 3389,
                "enableFloatingIP": false,
                "idleTimeoutInMinutes": 4,
                "protocol": "Tcp",
                "enableTcpReset": false
            }
        }
        ```
        Para más información sobre las reglas NAT de entrada, consulte [¿Qué es Azure Load Balancer?](./load-balancer-overview.md)
    
13. Guarde el archivo **\<resource-group-name>.json**.
    
10. Cree un grupo de recursos en la región de destino para el equilibrador de carga interno de destino que se va a implementar mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). El grupo de recursos existente anterior también se puede reutilizar como parte de este proceso:
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Implemente el archivo **\<resource-group-name>.json** editado en el grupo de recursos que creó en el paso anterior mediante [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Para comprobar que los recursos se crearon en la región de destino, use los comandos [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup) y [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzLoadBalancer -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

## <a name="discard"></a>Discard (Descartar) 

Después de la implementación, si quiere empezar de nuevo o descartar la red virtual y el equilibrador de carga en el destino, elimine el grupo de recursos que se creó en el destino y se eliminará la red virtual y el equilibrador de carga trasladados.  Para quitar el grupo de recursos, use [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

## <a name="clean-up"></a>Limpieza

Para confirmar los cambios y completar el traslado del NSG, elimine el NSG o el grupo de recursos de origen mediante [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) o [Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork) y [Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer)

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

``` azurepowershell-interactive

Remove-AzLoadBalancer -name <load-balancer> -ResourceGroupName <resource-group-name>

Remove-AzVirtualNetwork -Name <virtual-network-name> -ResourceGroupName <resource-group-name>


```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, migró un equilibrador de carga interno de Azure de una región a otra y limpió los recursos de origen.  Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:


- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Traslado de máquinas virtuales de Azure a otra región](../site-recovery/azure-to-azure-tutorial-migrate.md)
