---
title: 'Tutorial: Vinculación de redes virtuales a circuitos ExpressRoute - Azure Portal'
description: En este tutorial se muestra cómo crear una conexión para vincular una red virtual con un circuito de Azure ExpressRoute mediante Azure Portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 88674255c98559a06c33bd5030aefba9184ada58
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111538743"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit-using-the-portal"></a>Tutorial: Conexión de una red virtual con un circuito de ExpressRoute mediante el portal

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [CLI de Azure](howto-linkvnet-cli.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-linkvnet-classic.md)
> 

Este tutorial le ayuda a crear una conexión para vincular una red virtual con un circuito de Azure ExpressRoute mediante Azure Portal. Las redes virtuales que se conectan al circuito de Azure ExpressRoute pueden estar en la misma suscripción o formar parte de una suscripción diferente.

En este tutorial, obtendrá información sobre cómo:
> [!div class="checklist"]
> - conectar una red virtual a un circuito de la misma suscripción;
> - conectar una red virtual a un circuito de otra suscripción;
> - eliminar el vínculo entre la red virtual y el circuito ExpressRoute.

## <a name="prerequisites"></a>Requisitos previos

* Revise los [requisitos previos](expressroute-prerequisites.md), los [requisitos de enrutamiento](expressroute-routing.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.

* Tiene que tener un circuito ExpressRoute activo.
  * Siga las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) y habilite el circuito mediante el proveedor de conectividad.
  * Asegúrese de que dispone de un emparejamiento privado de Azure configurado para el circuito. Consulte el artículo [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md) para conocer las instrucciones de emparejamiento y enrutamiento.
  * Asegúrese de que el emparejamiento privado de Azure se configure y establezca el emparejamiento BGP entre la red y Microsoft para la conectividad de un extremo a otro.
  * Asegúrese de que ha creado y aprovisionado totalmente una red virtual y una puerta de enlace de red virtual. Siga las instrucciones para [crear una puerta de enlace de red virtual en ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Una puerta de enlace de red virtual para ExpressRoute usa el GatewayType "ExpressRoute", no VPN.

* Es posible vincular hasta 10 redes virtuales a un circuito ExpressRoute estándar. Todas las redes virtuales deben pertenecer a la misma región geopolítica al utilizar un circuito de ExpressRoute estándar.

* Una única red virtual se puede vincular a 16 circuitos ExpressRoute como máximo. Use el procedimiento siguiente para crear un nuevo objeto de conexión para cada circuito ExpressRoute al que quiere conectarse. Los circuitos ExpressRoute pueden estar en la misma suscripción, en suscripciones diferentes o en una combinación de ambas.

* Si habilita el complemento prémium de ExpressRoute, puede vincular redes virtuales fuera de la región geopolítica del circuito ExpressRoute. El complemento Premium también le permitirá conectar más de 10 redes virtuales al circuito ExpressRoute en función del ancho de banda elegido. Consulte las [preguntas más frecuentes](expressroute-faqs.md) para obtener más detalles sobre el complemento premium.

* Para crear la conexión desde el circuito ExpressRoute hasta la puerta de enlace de red virtual de ExpressRoute de destino, el número de espacios de direcciones anunciados desde las redes virtuales locales o emparejadas debe ser igual o menor que **200**. Una vez creada correctamente la conexión, puede agregar hasta un máximo de 1000 espacios de direcciones adicionales a las redes virtuales locales o emparejadas.

* Puede [ver un vídeo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit) antes de comenzar para entender mejor los pasos.

## <a name="connect-a-vnet-to-a-circuit---same-subscription"></a>Conexión de una red virtual con un circuito: misma suscripción

> [!NOTE]
> La información de la configuración de BGP no aparecerá si el proveedor de nivel 3 configuró los emparejamientos. Si el circuito tiene un estado aprovisionado, ya puede crear conexiones.
>

### <a name="to-create-a-connection"></a>Para crear una conexión

1. Asegúrese de que el circuito ExpressRoute y el emparejamiento privado de Azure estén correctamente configurados. Siga las instrucciones de [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md) y de [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-arm.md). El circuito ExpressRoute debe tener un aspecto similar a la imagen que aparece a continuación:

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/express-route-circuit.png" alt-text="Captura de pantalla de un circuito de ExpressRoute":::

1. Ahora puede iniciar el aprovisionamiento de una conexión para vincular la puerta de enlace de red virtual con el circuito ExpressRoute. Seleccione **Conexión** > **Agregar** para abrir la página **Agregar conexión**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/add-connection.png" alt-text="Captura para agregar conexión":::

1. Escriba un nombre para la conexión y, a continuación, seleccione **Siguiente: Configuración >** .

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-basic.png" alt-text="Página básica de creación de conexión":::

1. Seleccione la puerta de enlace que pertenece a la red virtual que desea vincular al circuito y seleccione **Revisar y crear**. Una vez completada la validación, seleccione **Crear**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-settings.png" alt-text="Página de configuración de creación de conexión":::

1. Una vez que la conexión esté correctamente configurada, el objeto de conexión mostrará la información de la conexión.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-object.png" alt-text="Captura de pantalla de objeto de conexión":::

## <a name="connect-a-vnet-to-a-circuit---different-subscription"></a>Conexión de una red virtual con un circuito: otra suscripción

Puede compartir un circuito ExpressRoute entre varias suscripciones. En la ilustración siguiente se muestra un sencillo esquema de cómo funciona el uso compartido de circuitos ExpressRoute entre varias suscripciones.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/cross-subscription.png" alt-text="Conectividad entre suscripciones":::

Cada una de las nubes más pequeñas dentro de la nube de gran tamaño se usa para representar las suscripciones que pertenecen a diferentes departamentos dentro de una organización. Cada departamento de la organización usa su propia suscripción para implementar sus servicios, pero puede compartir un único circuito ExpressRoute para volver a conectarse a la red local. Un solo departamento (en este ejemplo: TI) puede ser el propietario de ExpressRoute. Otras suscripciones dentro de la organización pueden usar el circuito ExpressRoute.

  > [!NOTE]
  > Los cargos de conectividad y ancho de banda de un circuito dedicado recaerán en el propietario del circuito ExpressRoute. Todas las redes virtuales comparten el mismo ancho de banda.
  >

### <a name="administration---about-circuit-owners-and-circuit-users"></a>Administración: acerca de los propietarios y los usuarios del circuito

El "propietario del circuito" es un usuario avanzado autorizado del recurso de circuito ExpressRoute. Puede crear autorizaciones que los "propietarios del circuito", a su vez, pueden canjear. Los usuarios del circuito son propietarios de puertas de enlace de red virtual que no están en la misma suscripción que el circuito ExpressRoute. Los usuarios del circuito pueden canjear autorizaciones (una autorización por red virtual).

El propietario del circuito tiene la capacidad de modificar y revocar las autorizaciones en cualquier momento. La revocación de una autorización dará como resultado la eliminación de todas las conexiones de vínculos de la suscripción cuyo acceso se haya revocado.

### <a name="circuit-owner-operations"></a>Operaciones del propietario del circuito

**Creación de una autorización de conexión**

El propietario del circuito crea una autorización, que crea una clave de autorización que puede usar un usuario del circuito para conectar sus puertas de enlace de red virtual al circuito ExpressRoute. Una autorización solo es válida para una conexión.

> [!NOTE]
> Cada conexión requiere una autorización independiente.
>

1. En la página de ExpressRoute, seleccione **Autorizaciones**, escriba un **nombre** para la autorización y seleccione **Guardar**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization.png" alt-text="Autorizaciones":::

2. Una vez guardada la configuración, copie los valores de **Identificador de recurso** y **Clave de autorización**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization-key.png" alt-text="Clave de autorización":::

**Eliminación de una autorización de conexión**

Para eliminar una conexión, seleccione el icono **Eliminar** correspondiente a la clave de autorización de su conexión.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-authorization-key.png" alt-text="Eliminar clave de autorización":::

Si desea eliminar la conexión y conservar la clave de autorización, puede eliminar la conexión desde la página de conexión del circuito.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection-owning-circuit.png" alt-text="Eliminar conexión desde el circuito donde se encuentra":::

### <a name="circuit-user-operations"></a>Operaciones del usuario del circuito

El usuario del circuito necesita el identificador del recurso y una clave de autorización del propietario del circuito.

**Canje de una autorización de conexión**

1. Seleccione el botón **+ Crear un recurso**. Busque **Conexión** y seleccione **Crear**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-new-resources.png" alt-text="Creación de nuevos recursos":::

1. Asegúrese de que *Tipo de conexión* está establecido en **ExpressRoute**. Seleccione los valores de *Grupo de recursos* y *Ubicación* y, a continuación, seleccione **Aceptar** en la página de datos básicos.

    > [!NOTE]
    > La ubicación *debe* coincidir con la ubicación de la puerta de enlace de red virtual para la que está creando la conexión.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-basics.png" alt-text="Datos básicos":::

1. En la página **Configuración**, seleccione una opción en *Puerta de enlace de red virtual* y active la casilla **Canjear autorización**. Escriba un valor en *Clave de autorización* y en *Peer circuit URI* (Emparejar URI de circuito) y asigne un nombre a la conexión. Seleccione **Aceptar**. 
 
    > [!NOTE]
    > El *URI de circuito del mismo nivel* es el identificador de recurso del circuito ExpressRoute (que puede encontrar en el panel de configuración de las propiedades del circuito ExpressRoute).

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-settings.png" alt-text="Página de configuración":::

1. Revise la información de la página **Resumen** y seleccione **Aceptar**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-summary.png" alt-text="Página de resumen":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede eliminar una conexión y desvincular la red virtual de un circuito de ExpressRoute; para ello, seleccione el icono **Eliminar** en la página de la conexión.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection.png" alt-text="Eliminar conexión":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a conectar una red virtual a un circuito de la misma suscripción y de otra. Para obtener más información acerca de la puerta de enlace de ExpressRoute, consulte: 

> [!div class="nextstepaction"]
> [Acerca de las puertas de enlace de red virtual de ExpressRoute](expressroute-about-virtual-network-gateways.md)
