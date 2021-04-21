---
title: Actualización o eliminación de un equilibrador de carga existente usado por conjuntos de escalado de máquinas virtuales
titleSuffix: Update or delete an existing load balancer used by virtual machine scale sets
description: Con este artículo de procedimientos, podrá empezar a usar Azure Standard Load Balancer y conjuntos de escalado de máquinas virtuales.
services: load-balancer
documentationcenter: na
author: irenehua
ms.custom: seodec18, devx-track-azurecli
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/29/2020
ms.author: irenehua
ms.openlocfilehash: 2079eeb97ac935ba24c6ff0616a58fbb28a962f4
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107480093"
---
# <a name="update-or-delete-a-load-balancer-used-by-virtual-machine-scale-sets"></a>Actualización o eliminación de un equilibrador de carga usado por conjuntos de escalado de máquinas virtuales

Al trabajar con conjuntos de escalado de máquinas virtuales y una instancia de Azure Load Balancer, puede:

- Agregar, actualizar y eliminar reglas.
- Agregar configuraciones.
- Eliminar el equilibrador de carga.

## <a name="set-up-a-load-balancer-for-scaling-out-virtual-machine-scale-sets"></a>Configuración de un equilibrador de carga para el escalado horizontal de conjuntos de escalado de máquinas virtuales

Asegúrese de que la instancia de Azure Load Balancer tenga configurado un [grupo NAT de entrada](/cli/azure/network/lb/inbound-nat-pool) y que el conjunto de escalado de máquinas virtuales esté colocado en el grupo de back-end del equilibrador de carga. Load Balancer creará automáticamente nuevas reglas NAT de entrada en el grupo NAT de entrada cuando se agreguen nuevas instancias de máquina virtual al conjunto de escalado de máquinas virtuales.

Para comprobar si el grupo NAT de entrada está configurado correctamente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el menú izquierdo, seleccione **Todos los recursos**. A continuación, seleccione **MyLoadBalancer** en la lista de recursos.
1. En **Configuración**, seleccione **Reglas NAT de entrada**. En el panel derecho, si ve una lista de reglas creadas para cada instancia del conjunto de escalado de máquinas virtuales, significa que está todo listo para realizar el escalado vertical en cualquier momento.

## <a name="add-inbound-nat-rules"></a>Adición de reglas NAT de entrada

No se pueden agregar reglas NAT de entrada individuales. Pero se puede agregar un conjunto de reglas NAT de entrada con un intervalo de puertos de front-end y un puerto de back-end definidos para todas las instancias del conjunto de escalado de máquinas virtuales.

Para actualizar un conjunto completo de reglas NAT de entrada para los conjuntos de escalado de máquinas virtuales, primero actualice el grupo NAT de entrada en el equilibrador de carga. Después, haga referencia al grupo de NAT de entrada desde el perfil de red del conjunto de escalado de máquinas virtuales. A continuación se muestra un ejemplo completo que usa la CLI.

El nuevo grupo NAT de entrada no debe tener un intervalo de puertos de front-end superpuestos con los grupos NAT de entrada existentes. Para ver los grupos NAT de entrada existentes configurados, use este [comando de la CLI](/cli/azure/network/lb/inbound-nat-pool#az_network_lb_inbound_nat_pool_list):
  
```azurecli-interactive
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool 
          --protocol Tcp 
          --frontend-port-range-start 80 
          --frontend-port-range-end 89 
          --backend-port 80 
          --frontend-ip-name MyFrontendIp
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool'}"
            
  az vmss update-instances
          -â€“instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS
```
## <a name="update-inbound-nat-rules"></a>Actualización de reglas NAT de entrada

No se pueden actualizar reglas NAT de entrada individuales. Pero se puede actualizar un conjunto de reglas NAT de entrada con un intervalo de puertos de front-end y un puerto de back-end definidos para todas las instancias del conjunto de escalado de máquinas virtuales.

Para actualizar un conjunto completo de reglas NAT de entrada para los conjuntos de escalado de máquinas virtuales, actualice el grupo NAT de entrada en el equilibrador de carga.
    
```azurecli-interactive
az network lb inbound-nat-pool update 
        -g MyResourceGroup 
        --lb-name MyLb 
        -n MyNatPool
        --protocol Tcp 
        --backend-port 8080
```

## <a name="delete-inbound-nat-rules"></a>Eliminación de reglas NAT de entrada

No se pueden eliminar reglas NAT de entrada individuales, pero puede eliminar todo el conjunto de reglas NAT de entrada mediante la eliminación del grupo NAT de entrada.

Para eliminar el grupo NAT, quítelo primero del conjunto de escalado. Aquí se muestra un ejemplo completo que usa la CLI:

```azurecli-interactive
    az vmss update
       --resource-group MyResourceGroup
       --name MyVMSS
       --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools
     az vmss update-instances 
       --instance-ids "*" 
       --resource-group MyResourceGroup
       --name MyVMSS
    az network lb inbound-nat-pool delete
       --resource-group MyResourceGroup
       -â€“lb-name MyLoadBalancer
       --name MyNatPool
```

## <a name="add-multiple-ip-configurations"></a>Adición de varias configuraciones de IP

Para agregar varias configuraciones de IP:

1. En el menú izquierdo, seleccione **Todos los recursos**. A continuación, seleccione **MyLoadBalancer** en la lista de recursos.
1. En **Configuración**, seleccione **Configuración de direcciones IP de front-end**. A continuación, seleccione **Agregar**.
1. En la página **Agregar dirección IP de front-end**, escriba los valores y seleccione **Aceptar**.
1. Siga el [paso 5](./load-balancer-multiple-ip.md#step-5-configure-the-health-probe) y el [paso 6](./load-balancer-multiple-ip.md#step-5-configure-the-health-probe) de este tutorial si se necesitan nuevas reglas de equilibrio de carga.
1. Si es necesario, cree un conjunto de reglas NAT de entrada con las configuraciones de IP de front-end recién creadas. En la sección anterior se puede encontrar un ejemplo.

## <a name="multiple-virtual-machine-scale-sets-behind-a-single-load-balancer"></a>Varias instancias de Virtual Machine Scale Sets detrás de un solo Load Balancer

Cree un grupo NAT de entrada en Load Balancer, haga referencia al grupo NAT de entrada en el perfil de red de un conjunto de escalado de máquinas virtuales y, por último, actualice las instancias para que los cambios surtan efecto. Repita los pasos para todas las instancias de Virtual Machine Scale Sets.

Asegúrese de crear grupos NAT de entrada independientes con intervalos de puertos de front-end no superpuestos.
  
```azurecli-interactive
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool 
          --protocol Tcp 
          --frontend-port-range-start 80 
          --frontend-port-range-end 89 
          --backend-port 80 
          --frontend-ip-name MyFrontendIpConfig
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool'}"
            
  az vmss update-instances
          -â€“instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS
          
  az network lb inbound-nat-pool create 
          -g MyResourceGroup 
          --lb-name MyLb
          -n MyNatPool2
          --protocol Tcp 
          --frontend-port-range-start 100 
          --frontend-port-range-end 109 
          --backend-port 80 
          --frontend-ip-name MyFrontendIpConfig2
  az vmss update 
          -g MyResourceGroup 
          -n myVMSS2 
          --add virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools "{'id':'/subscriptions/mySubscriptionId/resourceGroups/MyResourceGroup/providers/Microsoft.Network/loadBalancers/MyLb/inboundNatPools/MyNatPool2'}"
            
  az vmss update-instances
          -â€“instance-ids *
          --resource-group MyResourceGroup
          --name MyVMSS2
```

## <a name="delete-the-front-end-ip-configuration-used-by-the-virtual-machine-scale-set"></a>Eliminación de la configuración de IP de front-end usada por el conjunto de escalado de máquinas virtuales

Para eliminar la configuración de IP de front-end que usa el conjunto de escalado:

 1. En primer lugar, elimine el grupo NAT de entrada (el conjunto de reglas NAT de entrada) que hace referencia a la configuración de IP de front-end. En la sección anterior podrá encontrar instrucciones sobre cómo eliminar las reglas de entrada.
 1. Elimine la regla de equilibrio de carga que hace referencia a la configuración de IP de front-end.
 1. Elimine la configuración de IP de front-end.

## <a name="delete-a-load-balancer-used-by-a-virtual-machine-scale-set"></a>Eliminación de un equilibrador de carga usado por un conjunto de escalado de máquinas virtuales

Para eliminar la configuración de IP de front-end que usa el conjunto de escalado:

 1. En primer lugar, elimine el grupo NAT de entrada (el conjunto de reglas NAT de entrada) que hace referencia a la configuración de IP de front-end. En la sección anterior podrá encontrar instrucciones sobre cómo eliminar las reglas de entrada.
 1. Elimine la regla de equilibrio de carga que hace referencia al grupo de back-end que contiene el conjunto de escalado de máquinas virtuales.
 1. Quite la referencia `loadBalancerBackendAddressPool` del perfil de red del conjunto de escalado de máquinas virtuales.
 
 Aquí se muestra un ejemplo completo que usa la CLI:

```azurecli-interactive
    az vmss update
       --resource-group MyResourceGroup
       --name MyVMSS
       --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerBackendAddressPools
    az vmss update-instances 
       --instance-ids "*" 
       --resource-group MyResourceGroup
       --name MyVMSS
```
Por último, elimine el recurso del equilibrador de carga.
 
## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Azure Load Balancer y los conjuntos de escalado de máquinas virtuales, lea más sobre los conceptos.

> [Azure Load Balancer con conjuntos de escalado de máquinas virtuales](load-balancer-standard-virtual-machine-scale-sets.md)
