---
title: Uso de Azure Application Gateway con máquinas virtuales de VMware
description: Se describe cómo usar la instancia de Azure Application Gateway para administrar el tráfico web entrante de los servidores web que se ejecutan en máquinas virtuales de VMware en el entorno de nube privada de CloudSimple
author: shortpatti
ms.author: v-patsho
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: d6a189aad8766d5e4a5e5d603c4e43eb685dacc0
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177861"
---
# <a name="use-azure-application-gateway-with-vmware-virtual-machines-in-the-cloudsimple-private-cloud-environment"></a>Uso de Azure Application Gateway con máquinas virtuales de VMware en el entorno de nube privada de CloudSimple

Puede usar Azure Application Gateway para administrar el tráfico web entrante de los servidores web que se ejecutan en máquinas virtuales de VMware en el entorno de nube privada de CloudSimple.

Aproveche Azure Application Gateway en una implementación híbrida público-privado para administrar el tráfico web a las aplicaciones, proporcionar un front-end seguro y descargar el procesamiento TLS en los servicios que se ejecutan en el entorno de VMware. Azure Application Gateway enruta el tráfico web que entra en las instancias del grupo back-end que residen en entornos de VMware según las reglas configuradas y los sondeos de estado.

Para usar la solución Azure Application Gateway, es necesario cumplir los siguientes requisitos:

* Tener una suscripción a Azure.
* Crear y configurar una red virtual de Azure y una subred dentro de la red virtual
* Crear y configurar reglas de grupo de seguridad de red (NSG) y emparejar la red virtual a la nube privada de CloudSimple mediante ExpressRoute
* Crear y configurar la nube privada
* Crear y configurar la instancia de Azure Application Gateway

## <a name="azure-application-gateway-deployment-scenario"></a>Escenario de implementación de Azure Application Gateway

En este escenario, Azure Application Gateway se ejecuta en la red virtual de Azure. La red virtual está conectada a la nube privada mediante un circuito ExpressRoute. Todas las subredes de la nube privada son accesibles mediante IP desde las subredes de la red virtual.

![Equilibrador de carga de Azure en la red virtual de Azure](media/load-balancer-use-case.png)

## <a name="how-to-deploy-the-solution"></a>Implementación de la solución

El proceso de implementación consta de las siguientes tareas:

1. [Comprobación de que se cumplen los requisitos previos](#1-verify-prerequisites)
2. [Realizar la conexión virtual de Azure a la nube privada](#2-connect-your-azure-virtual-network-to-your-private-cloud)
3. [Implementar una instancia de Azure Application Gateway](#3-deploy-an-azure-application-gateway)
4. [Crear y configurar un grupo de máquinas virtuales de servidor web en la nube privada](#4-create-and-configure-a-web-server-vm-pool-in-your-private-cloud)

## <a name="1-verify-prerequisites"></a>1. Verificar los requisitos previos

Compruebe que se cumplen estos requisitos previos:

* Ya se ha creado una instancia de Azure Resource Manager y una red virtual.
* Ya se ha creado una subred dedicada (para Application Gateway) dentro de la red virtual de Azure.
* Ya se ha creado una nube privada de CloudSimple.
* No hay ningún conflicto de IP entre las subredes IP de la red virtual y las subredes de la nube privada.

## <a name="2-connect-your-azure-virtual-network-to-your-private-cloud"></a>2. Conectar la red virtual de Azure a la nube privada

Para conectar la red virtual de Azure a la nube privada, siga este proceso.

1. [En el portal de CloudSimple, copie la información de emparejamiento de ExpressRoute](virtual-network-connection.md).

2. [Configure una puerta de enlace de red virtual para la red virtual de Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).

3. [Vincule la red virtual al circuito ExpressRoute de CloudSimple](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription).

4. [Use la información de emparejamiento que copió para vincular la red virtual al circuito ExpressRoute](virtual-network-connection.md).

## <a name="3-deploy-an-azure-application-gateway"></a>3. Implementar una instancia de Azure Application Gateway

Las instrucciones detalladas para este procedimiento están disponibles en [Creación de una puerta de enlace de aplicaciones con reglas de enrutamiento basadas en rutas de acceso mediante Azure Portal](../application-gateway/create-url-route-portal.md). A continuación, se muestra un resumen de los pasos necesarios:

1. Cree una red virtual en la suscripción y grupo de recursos.
2. Cree una subred (para usarla como subred dedicada) dentro de la red virtual.
3. Cree una instancia estándar de Application Gateway (opcionalmente, puede habilitar WAF): en la página principal de Azure Portal, haga clic en **Recursos** > **Redes** > **Application Gateway** en la parte superior izquierda de la página. Seleccione el tamaño y la SKU estándar y proporcione la información de la suscripción, el grupo de recursos y la ubicación de Azure. Si es necesario, cree una dirección IP pública para esta puerta de enlace de aplicaciones y proporcione detalles sobre la red virtual y la subred dedicada de dicha puerta de enlace.
4. Agregue un grupo back-end con máquinas virtuales e incorpórelo a la puerta de enlace de aplicaciones.

## <a name="4-create-and-configure-a-web-server-vm-pool-in-your-private-cloud"></a>4. Crear y configurar un grupo de máquinas virtuales de servidor web en la nube privada

En vCenter, cree máquinas virtuales con el sistema operativo y el servidor web que prefiera (como Windows/IIS o Linux/Apache). Elija una subred o VLAN designada para el nivel web en la nube privada. Compruebe que al menos una vNIC de las máquinas virtuales de servidor web se encuentra en la subred del nivel web.
