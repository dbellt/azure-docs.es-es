---
title: Traslado de un equilibrador de carga externo de Azure a otra región de Azure mediante Azure PowerShell
description: Use una plantilla de Azure Resource Manager para trasladar un equilibrador de carga externo de Azure de una región de Azure a otra mediante Azure PowerShell.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: dcc1b584956bb4cee2865cd264b91b389ace19e4
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689338"
---
# <a name="move-azure-external-load-balancer-to-another-region-using-azure-powershell"></a>Traslado de un equilibrador de carga externo de Azure a otra región mediante Azure PowerShell

Hay varios escenarios en los que quizá quiera trasladar su equilibrador de carga externo actual de una región a otra. Por ejemplo, puede que quiera crear un equilibrador de carga externo con la misma configuración para realizar pruebas. También puede que quiera trasladar un equilibrador de carga externo a otra región como parte del planeamiento para la recuperación ante desastres.

Los equilibradores de carga externos de Azure no se pueden trasladar de una región a otra. Sin embargo, puede usar una plantilla de Azure Resource Manager para exportar la configuración y dirección IP pública actuales de un equilibrador de carga externo.  Después, puede preparar el recurso para otra región al exportar el equilibrador de carga y la dirección IP pública a una plantilla, modificar los parámetros para que coincidan con la región de destino y, a continuación, implementar la plantilla en la nueva región.  Para más información sobre Resource Manager y sus plantillas, consulte [Exportación de grupos de recursos a plantillas](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que el equilibrador de carga externo de Azure se encuentra en la región de Azure desde la que va a realizar el traslado.

- Los equilibradores de carga externos de Azure no se pueden trasladar entre regiones.  Tendrá que asociar el nuevo equilibrador de carga a los recursos de la región de destino.

- Necesitará contar con el rol de colaborador de red u otro superior para exportar la configuración de un equilibrador de carga externo e implementar una plantilla para crear un equilibrador de carga externo en otra región.
   
- Identifique el diseño de red de origen y todos los recursos que está usando actualmente. Este diseño incluye, entre otros, equilibradores de carga, grupos de seguridad de red (NSG), direcciones IP públicas y redes virtuales.

- Compruebe que la suscripción a Azure permite crear equilibradores de carga externos en la región de destino que se usa. Para habilitar la cuota necesaria, póngase en contacto con el soporte técnico.

- Asegúrese de que la suscripción tiene suficientes recursos para admitir la adición de equilibradores de carga para este proceso.  Vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)


## <a name="prepare-and-move"></a>Preparación y traslado
En los pasos siguientes se muestra cómo preparar el equilibrador de carga externo para el traslado mediante una plantilla de Resource Manager y cómo trasladar la configuración del equilibrador de carga externo a la región de destino mediante Azure PowerShell.  Como parte de este proceso, debe incluirse y crearse la configuración de la dirección IP pública del equilibrador de carga externo antes de trasladarlo.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-public-ip-template-and-deploy-from-azure-powershell"></a>Exportación de la plantilla de la dirección IP pública e implementación desde Azure PowerShell

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones de la pantalla:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
2. Obtenga el Id. de recurso de la dirección IP pública que quiere trasladar a la región de destino y colóquelo en una variable mediante [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):

    ```azurepowershell-interactive
    $sourcePubIPID = (Get-AzPublicIPaddress -Name <source-public-ip-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exporte la IP pública de origen a un archivo .json en el directorio donde se ejecuta el comando [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. El nombre del archivo descargado se asignará en función del grupo de recursos desde el que se exportó el recurso.  Busque el archivo que se exportó con el comando denominado **\<resource-group-name>.json** y ábralo en el editor que prefiera:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Para editar el parámetro del nombre de la dirección IP pública, cambie el valor **defaultValue** de la propiedad del nombre de la IP pública de origen por el nombre de la dirección IP pública de destino. Asegúrese de que el nombre se encuentra entre comillas:
    
    ```json
        {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_myVM1pubIP_name": {
        "defaultValue": "<target-publicip-name>",
        "type": "String"
        }
    }

    ```

6. Para editar la región de destino a la que se va a trasladar la dirección IP pública, cambie la propiedad **location** en resources:

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
               }
               }
             ]             
    ```
  
7. Para obtener los códigos de ubicación de la región, puede usar el cmdlet Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation) al ejecutar el siguiente comando:

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8. También puede cambiar otros parámetros de la plantilla si así lo desea; son opcionales según sus requisitos:

    * **SKU**: puede cambiar la SKU de la dirección IP pública de la configuración de estándar a básica o viceversa modificando la propiedad **sku** > **name** en el archivo **\<resource-group-name>.json**:

         ```json
            "resources": [
                   {
                    "type": "Microsoft.Network/publicIPAddresses",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                    "location": "<target-region>",
                    "sku": {
                        "name": "Basic",
                        "tier": "Regional"
                    },
         ```

         Para más información sobre las diferencias entre las IP públicas de la SKU básica y estándar, consulte [Creación, modificación o eliminación de una dirección IP pública](../virtual-network/virtual-network-public-ip-address.md).

    * **Método de asignación de IP pública** y **tiempo de espera de inactividad**: puede cambiar estas dos opciones en la plantilla si cambia la propiedad **publicIPAllocationMethod** de **Dynamic** a **Static** o bien de **Static** a **Dynamic**. El tiempo de espera de inactividad se puede cambiar modificando la propiedad **idleTimeoutInMinutes** con la cantidad deseada.  El valor predeterminado es **4**:

         ```json
         "resources": [
                {
                "type": "Microsoft.Network/publicIPAddresses",
                "apiVersion": "2019-06-01",
                "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                "location": "<target-region>",
                  "sku": {
                  "name": "Basic",
                  "tier": "Regional"
                 },
                "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
                   }
                }            
         ```

        Para más información sobre los métodos de asignación y los valores de tiempo de espera de inactividad, consulte [Creación, modificación o eliminación de una dirección IP pública](../virtual-network/virtual-network-public-ip-address.md).


9. Guarde el archivo **\<resource-group-name>.json**.

10. Cree un grupo de recursos en la región de destino para la dirección IP pública de destino que se va a implementar mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Implemente el archivo **\<resource-group-name>.json** editado en el grupo de recursos que creó en el paso anterior mediante [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Para comprobar que los recursos se crearon en la región de destino, use los comandos [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup) y [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzPublicIPAddress -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

### <a name="export-the-external-load-balancer-template-and-deploy-from-azure-powershell"></a>Exportación de la plantilla del equilibrador de carga externo e implementación desde Azure PowerShell

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones de la pantalla:
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenga el identificador del recurso del equilibrador de carga externo que quiere trasladar a la región de destino y colóquelo en una variable mediante [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer):

    ```azurepowershell-interactive
    $sourceExtLBID = (Get-AzLoadBalancer -Name <source-external-lb-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exporte la configuración del equilibrador de carga externo de origen a un archivo .json en el directorio donde se ejecuta el comando [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup):
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceExtLBID -IncludeParameterDefaultValue
   ```
4. El nombre del archivo descargado se asignará en función del grupo de recursos desde el que se exportó el recurso.  Busque el archivo que se exportó con el comando denominado **\<resource-group-name>.json** y ábralo en el editor que prefiera:
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Para editar el parámetro del nombre del equilibrador de carga externo, cambie la propiedad **defaultValue** del nombre del equilibrador de carga externo de origen por el nombre del equilibrador de carga externo de destino. Asegúrese de que el nombre se encuentra entre comillas:

    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "loadBalancers_myLoadbalancer_ext_name": {
        "defaultValue": "<target-external-lb-name>",
        "type": "String"
            },
        "publicIPAddresses_myPubIP_in_externalid": {
        "defaultValue": "<target-publicIP-resource-ID>",
        "type": "String"
            },

    ```

6.  Para modificar el valor de la dirección IP pública de destino que se ha trasladado antes, primero debe obtener el identificador de recurso y, a continuación, copiarlo y pegarlo en el archivo **\<resource-group-name>.json**.  Para obtener el identificador, use [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress):

    ```azurepowershell-interactive
    $targetPubIPID = (Get-AzPublicIPaddress -Name <target-public-ip-name> -ResourceGroupName <target-resource-group-name>).Id
    ```
    Escriba la variable y presione ENTRAR para mostrar el identificador del recurso.  Resalte la ruta de acceso del identificador y cópiela en el portapapeles:

    ```powershell
    PS C:\> $targetPubIPID
    /subscriptions/7668d659-17fc-4ffd-85ba-9de61fe977e8/resourceGroups/myResourceGroupLB-Move/providers/Microsoft.Network/publicIPAddresses/myPubIP-in-move
    ```

7.  En el archivo **\<resource-group-name>.json**, pegue el **identificador del recurso** de la variable en lugar del valor **defaultValue** en el segundo parámetro del identificador externo de la IP pública y, después, asegúrese de escribir la ruta de acceso entre comillas:

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            "loadBalancers_myLoadbalancer_ext_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
            },
            "publicIPAddresses_myPubIP_in_externalid": {
            "defaultValue": "<target-publicIP-resource-ID>",
            "type": "String"
            },

    ```

8.  Si ha configurado las reglas de salida y NAT de salida para el equilibrador de carga, en este archivo habrá una tercera entrada para el identificador externo de la IP pública saliente.  Repita los pasos anteriores en la **región de destino** para obtener el identificador de la dirección IP pública de salida y pegue esa entrada en el archivo **\<resource-group-name>.json**:

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "loadBalancers_myLoadbalancer_ext_name": {
                "defaultValue": "<target-external-lb-name>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_in_externalid": {
                "defaultValue": "<target-publicIP-resource-ID>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_out_externalid": {
                "defaultValue": "<target-publicIP-outbound-resource-ID>",
                "type": "String"
            }
        },
    ```

10. Para editar la región de destino a la que se va a trasladar la configuración del equilibrador de carga externo, cambie la propiedad **location** en **resources** del archivo **\<resource-group-name>.json**:

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-external-lb-region>",
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
    
    * **SKU**: puede cambiar la SKU del equilibrador de carga externo en la configuración del nivel estándar al básico o viceversa si modifica la propiedad **sku** > **name** en el archivo **\<resource-group-name>.json**:

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-external-lb-region>",
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

    * **Reglas de salida**: puede agregar o quitar reglas de salida en la configuración mediante la edición de la propiedad **outboundRules** en el archivo **\<resource-group-name>.json** :

        ```json
        "outboundRules": [
                    {
                        "name": "myOutboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "allocatedOutboundPorts": 10000,
                            "protocol": "All",
                            "enableTcpReset": false,
                            "idleTimeoutInMinutes": 15,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolOutbound')]"
                            },
                            "frontendIPConfigurations": [
                                {
                                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPoutbound')]"
                                }
                            ]
                        }
                    }
                ]
        ```

         Para más información sobre las reglas de salida, consulte [Reglas de salida de Load Balancer](./load-balancer-outbound-connections.md#outboundrules).

13. Guarde el archivo **\<resource-group-name>.json**.
    
10. Cree un grupo de recursos en la región de destino para el equilibrador de carga externo de destino que se va a implementar mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). El grupo de recursos existente anterior también se puede reutilizar como parte de este proceso:
    
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

Después de la implementación, si quiere empezar de nuevo o descartar la IP pública y el equilibrador de carga en el destino, elimine el grupo de recursos que se creó en el destino y se eliminará la IP pública y el equilibrador de carga trasladados.  Para quitar el grupo de recursos, use [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

## <a name="clean-up"></a>Limpieza

Para confirmar los cambios y completar el traslado del NSG, elimine el NSG o el grupo de recursos de origen mediante [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) o [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress) y [Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer)

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

``` azurepowershell-interactive

Remove-AzLoadBalancer -name <load-balancer> -ResourceGroupName <resource-group-name>

Remove-AzPublicIpAddress -Name <public-ip> -ResourceGroupName <resource-group-name>


```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha movido un grupo de seguridad de red de Azure de una región a otra y ha limpiado los recursos de origen.  Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:


- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Traslado de máquinas virtuales de Azure a otra región](../site-recovery/azure-to-azure-tutorial-migrate.md)
