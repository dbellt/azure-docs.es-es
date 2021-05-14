---
title: 'Azure VMware Solution by CloudSimple: conexión local mediante ExpressRoute'
description: En este artículo se describe cómo solicitar una conexión local mediante ExpressRoute desde la red de la región de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: a710000a27a0b41079a1cba7482b3fdcaa966edc
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108185331"
---
# <a name="connect-from-on-premises-to-cloudsimple-using-expressroute"></a>Conexión desde una red local a CloudSimple mediante ExpressRoute

Si ya tiene una conexión de Azure ExpressRoute desde una ubicación externa (por ejemplo, local) a Azure, puede conectarla a su entorno de CloudSimple. Puede hacerlo a través de una característica de Azure que permite que dos circuitos ExpressRoute se conecten entre sí. Este método establece entre los dos entornos una conexión segura, privada, de ancho de banda alto y baja latencia.

[![Conexión de ExpressRoute local - Global Reach](media/cloudsimple-global-reach-connection.png)](media/cloudsimple-global-reach-connection.png)

## <a name="before-you-begin"></a>Antes de empezar

Se requiere un bloque de direcciones de red **/29** para establecer la conexión de Global Reach desde el entorno local.  El espacio de direcciones /29 se usa para la red de tránsito entre circuitos de ExpressRoute.  La red de tránsito no debe superponerse con ninguna de las redes virtuales de Azure, redes locales o redes de nube privada de CloudSimple.

## <a name="prerequisites"></a>Requisitos previos

* Se requiere un circuito Azure ExpressRoute para poder establecer la conexión entre el circuito y las redes de nube privada de CloudSimple.
* Se requiere un usuario con privilegios para crear claves de autorización en un circuito ExpressRoute.

## <a name="scenarios"></a>Escenarios

El hecho de conectar la red local con la red de nube privada permite usar la nube privada de varias maneras, entre las que se incluyen los siguientes escenarios:

* Acceso a la red de nube privada sin crear una conexión VPN de sitio a sitio.
* Uso de la instancia local de Active Directory como origen de identidades en la nube privada.
* Migración de máquinas virtuales que se ejecutan de forma local a la nube privada.
* Uso de la nube privada como parte de una solución de recuperación ante desastres.
* Consumo de recursos locales en las máquinas virtuales de carga de trabajo de la nube privada.

## <a name="connecting-expressroute-circuits"></a>Conexión de circuitos ExpressRoute

Para establecer la conexión de ExpressRoute, debe crear una autorización en el circuito ExpressRoute y proporcionar la información de autorización a CloudSimple.


### <a name="create-expressroute-authorization"></a>Creación de la autorización de ExpressRoute

1. Inicie sesión en Azure Portal.

2. En la barra de búsqueda superior, busque **Circuito ExpressRoute** y haga clic en **Circuitos ExpressRoute** en **Servicios**.
    [![Circuitos ExpressRoute](media/azure-expressroute-transit-search.png)](media/azure-expressroute-transit-search.png)

3. Seleccione el circuito ExpressRoute que quiere conectar a la red de CloudSimple.

4. En la página de ExpressRoute, haga clic en **Autorizaciones**, escriba el nombre de la autorización y haga clic en **Guardar**.
    [![Autorización de circuito ExpressRoute](media/azure-expressroute-transit-authorizations.png)](media/azure-expressroute-transit-authorizations.png)

5. Para copiar el identificador de recurso y la clave de autorización, haga clic en el icono de copia. Pegue el identificador y la clave en un archivo de texto.
    [![Copia de la autorización de circuito ExpressRoute](media/azure-expressroute-transit-authorization-copy.png)](media/azure-expressroute-transit-authorization-copy.png)

    > [!IMPORTANT]
    > El **identificador de recurso** debe copiarse de la interfaz de usuario y debe tener el formato ```/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/expressRouteCircuits/<express-route-circuit-name>``` cuando se proporcione al servicio de soporte técnico.

6. Abra una incidencia de <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">soporte técnico</a> para la conexión que se va a crear.
    * Tipo de problema: **técnico**
    * Suscripción: **la suscripción en la que se implementa CloudSimple**
    * Servicio: **VMware Solution by CloudSimple**
    * Tipo de problema: **solicitud de servicio**
    * Subtipo de problema: **Creación de una conexión de ExpressRoute en un entorno local**
    * Proporcione el identificador de recurso y la clave de autorización que copió y guardó en el panel de detalles.
    * Proporcione un espacio de direcciones de red /29 para la red de tránsito.
    * ¿Está enviando la ruta predeterminada a través de ExpressRoute?
    * ¿Debe usar el tráfico de nube privada la ruta predeterminada enviada a través de ExpressRoute?

    > [!IMPORTANT]
    > El envío de la ruta predeterminada le permite enviar todo el tráfico de Internet desde la nube privada mediante su conexión a Internet local.  Para deshabilitar la ruta predeterminada configurada en la nube privada y usar la ruta predeterminada de la conexión local, proporcione los detalles en la incidencia de soporte técnico.

## <a name="next-steps"></a>Pasos siguientes

* [Obtener más información acerca de las conexiones de red de Azure](cloudsimple-azure-network-connection.md)  
