---
title: 'Inicio rápido: Creación y configuración de una instancia de Route Server mediante la CLI de Azure'
description: En esta guía de inicio rápido, aprenderá a crear y configurar una instancia de Route Server mediante la CLI de Azure.
services: route-server
author: duongau
ms.service: route-server
ms.topic: quickstart
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: dce3dd10843bc6abf2878600ad1aba17a4318c80
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108017944"
---
# <a name="quickstart-create-and-configure-route-server-using-azure-cli"></a>Inicio rápido: Creación y configuración de una instancia de Route Server mediante la CLI de Azure 

Este artículo le ayuda a configurar Azure Route Server para su emparejamiento con aplicaciones virtuales de red (NVA) en la red virtual mediante la CLI de Azure. Azure Route Server aprenderá las rutas de la NVA y las programará para las máquinas virtuales de la red virtual. También anunciará las rutas de la red virtual a la NVA. Para más información, consulte [Azure Route Server](overview.md).

:::image type="content" source="media/quickstart-configure-route-server-portal/environment-diagram.png" alt-text="Diagrama del entorno de implementación de Route Server mediante la CLI de Azure." border="false":::

> [!IMPORTANT]
> Azure Route Server (versión preliminar) está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

##  <a name="prerequisites"></a>Requisitos previos 

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
* Asegúrese de que tiene la versión de la CLI de Azure más reciente, o bien, puede usar Azure Cloud Shell en el portal. 
* Consulte [Límites de servicio de Azure Route Server](route-server-faq.md#limitations). 

##  <a name="create-a-route-server"></a>Creación de una instancia de Route Server 

###  <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Inicie sesión en la cuenta de Azure y seleccione su suscripción. 

Para empezar la configuración, inicie sesión en la cuenta de Azure. Si usa la opción "Pruébelo" de CloudShell, iniciará sesión de manera automática. Use los siguientes ejemplos para conectarse:

```azurecli-interactive
az login
```

Compruebe las suscripciones para la cuenta.

```azurecli-interactive
az account list
```

Seleccione la suscripción para la que desea crear un circuito ExpressRoute.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

### <a name="create-a-resource-group-and-virtual-network"></a>Creación de un grupo de recursos y red virtual 

Para poder crear una instancia de Azure Route Server, necesitará una red virtual para hospedar la implementación. Use el comando siguiente para crear un grupo de recursos y una red virtual. Si ya tiene una red virtual, puede ir directamente a la sección siguiente.

```azurecli-interactive
az group create -n "RouteServerRG" -l "westus" 
az network vnet create -g "RouteServerRG" -n "myVirtualNetwork" --address-prefix "10.0.0.0/16" 
``` 

### <a name="add-a-subnet"></a>Incorporación de una subred 

1. Agregue una subred llamada *RouteServerSubnet* para implementar la instancia de Azure Route Server en ella. Esta subred es una subred dedicada solo para Azure Route Server. La subred RouteServerSubnet debe ser /27 o un prefijo más corto, (como /26 o /25). De lo contrario, recibirá un mensaje de error al agregar la instancia de Azure Route Server.

    ```azurecli-interactive 
    az network vnet subnet create -g "RouteServerRG" --vnet-name "myVirtualNetwork" --name "RouteServerSubnet" --address-prefix "10.0.0.0/24"  
    ``` 

1. Obtenga el identificador de la subred RouteServerSubnet. Para ver el identificador de recurso de todas las subredes de la red virtual, use este comando: 

    ```azurecli-interactive 
    $subnet_id = $(az network vnet subnet show -n "RouteServerSubnet" --vnet-name "myVirtualNetwork" -g "RouteServerRG" --query id -o tsv) 
    ``` 

El identificador de RouteServerSubnet tiene un aspecto similar al siguiente: 

`/subscriptions/<subscriptionID>/resourceGroups/RouteServerRG/providers/Microsoft.Network/virtualNetworks/myVirtualNetwork/subnets/RouteServerSubnet`

## <a name="create-the-route-server"></a>Creación de la instancia de Route Server 

Cree la instancia de Route Server con este comando: 

```azurecli-interactive
az network routeserver create -n "myRouteServer" -g "RouteServerRG" --hosted-subnet $subnet_id  
``` 

La ubicación debe coincidir con la ubicación de la red virtual. HostedSubnet es el identificador de RouteServerSubnet que obtuvo en la sección anterior. 

## <a name="create-peering-with-an-nva"></a>Creación del emparejamiento con una NVA 

Use el siguiente comando para establecer el emparejamiento desde la instancia de Route Server a la NVA: 

```azurecli-interactive 

az network routeserver peering create --routeserver "myRouteServer" -g "RouteServerRG" --peer-ip "nva_ip" --peer-asn "nva_asn" -n "NVA1_name" 

``` 

"nva_ip" es la dirección IP de la red virtual asignada a la NVA. "nva_asn" es el número de sistema autónomo (ASN) configurado en la NVA. El ASN puede ser cualquier número de 16 bits que no se encuentre en el intervalo del 65515 al 65520. Este intervalo de ASN está reservado por Microsoft. 

Para configurar el emparejamiento con diferentes NVA u otra instancia de la misma NVA para la redundancia, use este comando:

```azurecli-interactive 

az network routeserver peering create --routeserver "myRouteServer" -g "RouteServerRG" --peer-ip "nva_ip" --peer-asn "nva_asn" -n "NVA2_name" 
``` 

## <a name="complete-the-configuration-on-the-nva"></a>Finalización de la configuración en la NVA 

Para completar la configuración en la NVA y habilitar las sesiones de BGP, necesita la dirección IP y el ASN de la instancia de Azure Route Server. Puede obtener esta información mediante este comando: 

```azurecli-interactive 
az network routeserver show -g "RouteServerRG" -n "myRouteServer" 
``` 

La salida tiene la siguiente información. 

```azurecli-interactive 
RouteServerAsn  : 65515 

RouteServerIps  : {10.5.10.4, 10.5.10.5}  "virtualRouterAsn": 65515, 

  "virtualRouterIps": [ 

    "10.0.0.4", 

    "10.0.0.5" 

  ], 

``` 

## <a name="configure-route-exchange"></a>Configuración del intercambio de rutas 

Si tiene una puerta de enlace de ExpressRoute y una instancia de Azure VPN Gateway en la misma red virtual y quiere que intercambien rutas, puede habilitar el intercambio de rutas en la instancia de Azure Route Server.

> [!IMPORTANT]
> En el caso de las implementaciones green field, asegúrese de crear la instancia de Azure VPN Gateway antes de crear la instancia de Azure Route Server; en caso contrario, se producirá un error en la implementación de Azure VPN Gateway.
> 

1. Para habilitar el intercambio de rutas entre Azure Route Server y las puertas de enlace, use este comando:

```azurecli-interactive 
az network routeserver update -g "RouteServerRG" -n "myRouteServer" --allow-b2b-traffic true 

``` 

2. Para deshabilitar el intercambio de rutas entre Azure Route Server y las puertas de enlace, use este comando:

```azurecli-interactive
az network routeserver update -g "RouteServerRG" -n "myRouteServer" --allow-b2b-traffic false 
``` 

## <a name="troubleshooting"></a>Solución de problemas 

Puede ver las rutas anunciadas y recibidas por Azure Route Server con este comando:

```azurecli-interactive 
az network routeserver peering list-advertised-routes -g RouteServerRG --vrouter-name myRouteServer -n NVA1_name 
az network routeserver peering list-learned-routes -g RouteServerRG --vrouter-name myRouteServer -n NVA1_name 
``` 

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita la instancia de Azure Route Server, use estos comandos para quitar el emparejamiento de BGP y, a continuación, quite la instancia de Route Server. 

1. Elimine el emparejamiento de BGP entre Azure Route Server y una NVA con este comando:

```azurecli-interactive
az network routeserver peering delete --routeserver "myRouteServer" -g "RouteServerRG" -n "NVA2_name" 
``` 

2. Elimine la instancia de Azure Route Server con este comando: 

```azurecli-interactive 
az network routeserver delete -n "myRouteServer" -g "RouteServerRG" 
``` 

## <a name="next-steps"></a>Pasos siguientes

Después de crear la instancia de Azure Route Server, continúe para obtener información sobre cómo interactúa Azure Route Server con las puertas de enlace de VPN y ExpressRoute: 

> [!div class="nextstepaction"]
> [Compatibilidad de Azure Route Server (versión preliminar) con ExpressRoute y VPN de Azure](expressroute-vpn-support.md)
 
