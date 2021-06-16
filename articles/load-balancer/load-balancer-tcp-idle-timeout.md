---
title: Configuración del tiempo de espera de inactividad y restablecimiento de TCP de Load Balancer
titleSuffix: Azure Load Balancer
description: En este artículo, aprenderá a configurar el tiempo de espera de inactividad y restablecimiento de TCP de Azure Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18, devx-track-azurepowershell
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2020
ms.author: allensu
ms.openlocfilehash: 2a360f5972e4a0b35be20af18d365a88db779c85
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695570"
---
# <a name="configure-tcp-reset-and-idle-timeout-for-azure-load-balancer"></a>Configuración del tiempo de espera de inactividad y restablecimiento de TCP de Azure Load Balancer

Azure Load Balancer tiene el siguiente intervalo de tiempo de espera de inactividad:

* De 4 a 100 minutos para las reglas de salida.
* De 4 a 30 minutos para reglas de Load Balancer y las reglas NAT de entrada.

De forma predeterminada, se establece en 4 minutos. Si un período de inactividad es mayor que el valor del tiempo de espera, no habrá ninguna garantía de que todavía exista la sesión TCP o HTTP entre el cliente y el servicio. 

En las secciones siguientes se describe cómo cambiar la configuración del tiempo de espera de inactividad y restablecimiento de los recursos de Load Balancer.

## <a name="set-tcp-reset-and-idle-timeout"></a>Configuración del tiempo de espera de inactividad y restablecimiento de TCP
---
# <a name="portal"></a>[**Portal**](#tab/tcp-reset-idle-portal)

Para establecer el tiempo de espera de inactividad y restablecimiento de una instancia de Load Balancer, edite la regla de carga equilibrada. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En el menú izquierdo, seleccione **Grupos de recursos**.

3. Seleccione el grupo de recursos del equilibrador de carga. En este ejemplo se crea un grupo de recursos denominado **myResourceGroup**.

4. Seleccione el equilibrador de carga. En este ejemplo, el equilibrador de carga se denomina **myLoadBalancer**.

5. En **Configuración**, seleccione **Reglas de equilibrio de carga**.

     :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules.png" alt-text="Edite las reglas del equilibrador de carga." border="true":::

6. Seleccione la regla de equilibrio de carga. En este ejemplo, la regla de equilibrio de carga se denomina **myLBrule**.

7. En la regla de equilibrio de carga, mueva el control deslizante hacia **Tiempo de espera de inactividad (minutos)** al valor de tiempo de espera.  

8. En **Restablecimiento de TCP**, seleccione **Habilitado**.

   :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules-tcp-reset.png" alt-text="Establezca el tiempo de espera de inactividad y el restablecimiento de TCP." border="true":::

9. Seleccione **Guardar**.

# <a name="powershell"></a>[**PowerShell**](#tab/tcp-reset-idle-powershell)

Para establecer el tiempo de espera de inactividad y el restablecimiento de TCP, establezca los valores de los siguientes parámetros de la regla de equilibrio de carga con [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer):

* **IdleTimeoutInMinutes**
* **EnableTcpReset**

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

Reemplace los siguientes ejemplos por los valores de los recursos:

* **myResourceGroup**
* **myLoadBalancer**

```azurepowershell
$lb = Get-AzLoadBalancer -Name "myLoadBalancer" -ResourceGroup "myResourceGroup"
$lb.LoadBalancingRules[0].IdleTimeoutInMinutes = '15'
$lb.LoadBalancingRules[0].EnableTcpReset = 'true'
Set-AzLoadBalancer -LoadBalancer $lb
```

# <a name="azure-cli"></a>[**CLI de Azure**](#tab/tcp-reset-idle-cli)

Para establecer el tiempo de espera de inactividad y el restablecimiento de TCP, use los parámetros del comando [az network lb rule update](/cli/azure/network/lb/rule?az_network_lb_rule_update):

* **--idle-timeout**
* **--enable-tcp-reset**

Valide el entorno antes de empezar:

* Inicie sesión en Azure Portal y compruebe que la suscripción está activa, para lo que debe ejecutar `az login`.
* Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar `az --version`. Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

Reemplace los siguientes ejemplos por los valores de los recursos:

* **myResourceGroup**
* **myLoadBalancer**
* **myLBrule**


```azurecli
az network lb rule update \
    --resource-group myResourceGroup \
    --name myLBrule \
    --lb-name myLoadBalancer \
    --idle-timeout 15 \
    --enable-tcp-reset true
```
---
## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el tiempo de espera de inactividad y el restablecimiento de TCP, consulte [Tiempo de espera de inactividad y restablecimiento de TCP de Load Balancer](load-balancer-tcp-reset.md).

Para obtener más información sobre cómo configurar el modo de distribución del equilibrador de carga, consulte [Configuración de un modo de distribución del equilibrador de carga](load-balancer-distribution-mode.md).
