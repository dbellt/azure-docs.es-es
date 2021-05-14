---
title: Configuración del modo de distribución de Azure Load Balancer
titleSuffix: Azure Load Balancer
description: En este artículo, empezará a configurar el modo de distribución de Azure Load Balancer para admitir la afinidad de IP de origen.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2021
ms.author: allensu
ms.openlocfilehash: 44c56b91a02d53126f00ad64a1404ad4664c6a7f
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107886629"
---
# <a name="configure-the-distribution-mode-for-azure-load-balancer"></a>Configuración del modo de distribución de Azure Load Balancer

Azure Load Balancer admite dos modos de distribución para distribuir el tráfico a las aplicaciones:

* Basado en hash
* Modo de afinidad de IP de origen

En este artículo, verá cómo configurar el modo de distribución de Azure Load Balancer.


## <a name="configure-distribution-mode"></a>Configuración del modo de distribución

---

# <a name="azure-portal"></a>[**Azure Portal**](#tab/azure-portal)

Para cambiar la configuración del modo de distribución, modifique la regla de equilibrio de carga en el portal.

1. Inicie sesión en Azure Portal y busque el grupo de recursos que contiene el equilibrador de carga que desea cambiar. Para ello, debe hacer clic en **Grupos de recursos**.
2. En la pantalla de información general del equilibrador de carga, seleccione **Reglas de equilibrio de carga**  en **Configuración**.
3. En la pantalla de reglas de equilibrio de carga, seleccione la regla de equilibrio de carga para la que desee cambiar el modo de distribución.
4. En la regla, para cambiar el modo de distribución hay que cambiar el cuadro desplegable **Persistencia de la sesión**. 

Están disponibles las opciones siguientes: 

* **Ninguno (basado en hash)**: especifica que cualquier máquina virtual puede controlar las solicitudes sucesivas del mismo cliente.
* **IP de cliente (tupla de dos elementos de afinidad de IP de origen)** : especifica que la misma máquina virtual controlará las solicitudes sucesivas de la misma dirección IP de cliente.
* **IP de cliente y protocolo (tupla de tres elementos de afinidad de IP de origen)** : especifica que la misma máquina virtual controlará las solicitudes sucesivas de la combinación de la misma dirección IP de cliente y protocolo.

5. Elija el modo de distribución y seleccione **Guardar**.

:::image type="content" source="./media/load-balancer-distribution-mode/session-persistence.png" alt-text="Cambie la persistencia de la sesión en la regla del equilibrador de carga." border="true":::


# <a name="powershell"></a>[**PowerShell**](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Use PowerShell para cambiar la configuración de distribución del equilibrador de carga en una regla de equilibrio de carga existente. El siguiente comando actualiza el modo de distribución: 

```azurepowershell-interactive
$lb = Get-AzLoadBalancer -Name MyLoadBalancer -ResourceGroupName MyResourceGroupLB
$lb.LoadBalancingRules[0].LoadDistribution = 'default'
Set-AzLoadBalancer -LoadBalancer $lb
```

Establezca el valor del elemento `LoadDistribution` para el tipo necesario de equilibrio de carga. 

* Especifique **sourceIP** para el equilibrio de carga con tupla de dos elementos (IP de origen e IP de destino). 

* Especifique **sourceIPProtocol** para el equilibrio de carga con tupla de tres elementos (IP de origen, IP de destino y tipo de protocolo). 

* Especifique **Default** para el comportamiento predeterminado de equilibrio de carga con tupla de cinco elementos.

# <a name="cli"></a>[**CLI**](#tab/azure-cli)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

Use la CLI de Azure para cambiar la configuración de distribución del equilibrador de carga en una regla de equilibrio de carga existente.  El siguiente comando actualiza el modo de distribución:

```azurecli-interactive
az network lb rule update \
    --lb-name myLoadBalancer \
    --load-distribution Default \
    --name myHTTPRule \
    --resource-group myResourceGroupLB 
```
Establezca el valor de `--load-distribution` para el tipo necesario de equilibrio de carga.

* Especifique **sourceIP** para el equilibrio de carga con tupla de dos elementos (IP de origen e IP de destino). 

* Especifique **sourceIPProtocol** para el equilibrio de carga con tupla de tres elementos (IP de origen, IP de destino y tipo de protocolo). 

* Especifique **Default** para el comportamiento predeterminado de equilibrio de carga con tupla de cinco elementos.

Para obtener más información sobre el comando que se usa en este artículo, consulte [az network lb rule update](/cli/azure/network/lb/rule#az_network_lb_rule_update).

---

## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre Azure Load Balancer](load-balancer-overview.md)
* [Introducción a la configuración de un equilibrador de carga accesible desde Internet](quickstart-load-balancer-standard-public-powershell.md)
* [Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)
