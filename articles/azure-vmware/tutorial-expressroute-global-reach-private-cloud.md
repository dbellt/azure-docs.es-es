---
title: Emparejamiento de entornos locales con Azure VMware Solution
description: Aprenda a emparejar ExpressRoute Global Reach con una nube privada en Azure VMware Solution.
ms.topic: tutorial
ms.custom: contperf-fy21q4
ms.date: 04/27/2021
ms.openlocfilehash: fb0d8bc8391c96449cdf260c543cfed1af1125f5
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176115"
---
# <a name="peer-on-premises-environments-to-azure-vmware-solution"></a>Emparejamiento de entornos locales con Azure VMware Solution

En este paso del inicio rápido, conectará Azure VMware Solution a un entorno local. Global Reach de ExpressRoute conecta su entorno local y la nubes privada de Azure VMware Solution. La conexión de Global Reach de ExpressRoute se establece entre el circuito ExpressRoute de la nube privada y una conexión existente de ExpressRoute con los entornos locales. 


>[!NOTE]
>Puede conectarse mediante VPN, pero ese tema no se detalla en este documento de inicio rápido.

En este tutorial se produce una conexión como se muestra en el diagrama.

:::image type="content" source="media/pre-deployment/azure-vmware-solution-on-premises-diagram.png" alt-text="Diagrama que muestra la conectividad de red local de ExpressRoute Global Reach." lightbox="media/pre-deployment/azure-vmware-solution-on-premises-diagram.png" border="false":::


## <a name="before-you-begin"></a>Antes de empezar

Antes de habilitar la conectividad entre dos circuitos ExpressRoute mediante Global Reach, consulte la documentación sobre cómo [habilitar la conectividad en distintas suscripciones de Azure](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions).  

## <a name="prerequisites"></a>Requisitos previos
- Un circuito de ExpressRoute independiente y funcional utilizado para conectar entornos locales con Azure, que es el _circuito 1_ a efectos de emparejamiento.
- Asegúrese de que todas las puertas de enlace, incluido el servicio del proveedor de ExpressRoute, admitan un número de sistema autónomo (ASN) de 4 bytes. Azure VMware Solution utiliza ASN públicos de 4 bytes para anunciar rutas.


## <a name="create-an-expressroute-authorization-key-in-the-on-premises-expressroute-circuit"></a>Creación de una clave de autorización de ExpressRoute en el circuito local de ExpressRoute

1. En la hoja **Circuitos de ExpressRoute**, en Configuración, seleccione **Autorizaciones**.

1. Escriba el nombre de la clave de autorización y seleccione **Guardar**.

   :::image type="content" source="media/expressroute-global-reach/start-request-auth-key-on-premises-expressroute.png" alt-text="Seleccione Autorizaciones y escriba el nombre de la clave de autorización.":::

   Una vez creada, la nueva clave aparece en la lista de claves de autorización del circuito.

1. Tome nota de la clave de autorización y el identificador de ExpressRoute. Los usará en el paso siguiente para completar el emparejamiento.

## <a name="peer-private-cloud-to-on-premises"></a>Emparejamiento de una nube privada con el entorno local 
Ahora que ha creado una clave de autorización para el circuito ExpressRoute de la nube privada, puede emparejarlo con el circuito ExpressRoute local. El emparejamiento se lleva a cabo desde el circuito local de ExpressRoute en **Azure Portal**. Utilizará el identificador de recurso (id. de circuito de ExpressRoute) y la clave de autorización del circuito ExpressRoute de la nube privada para finalizar el emparejamiento.

1. En la nube privada, en Administrar, seleccione **Conectividad** > **ExpressRoute Global Reach** > **Agregar**.

    :::image type="content" source="./media/expressroute-global-reach/expressroute-global-reach-tab.png" alt-text="Captura de pantalla que muestra la pestaña Global Reach de ExpressRoute en la nube privada de Azure VMware Solution.":::

1. Escriba el identificador de ExpressRoute y la clave de autorización creada en la sección anterior.

   :::image type="content" source="./media/expressroute-global-reach/on-premises-cloud-connections.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo para especificar información de conexión.":::   

1. Seleccione **Crear**. La nueva conexión se muestra en la lista de conexiones de la nube local.

>[!TIP]
>Para eliminar o desconectar una conexión de la lista, seleccione **Más**.  
>
>:::image type="content" source="./media/expressroute-global-reach/on-premises-connection-disconnect.png" alt-text="Desconexión o eliminación de una conexión local":::


## <a name="verify-on-premises-network-connectivity"></a>Comprobación de la conectividad de red local

Ahora debería ver en el **enrutador perimetral local** el lugar donde ExpressRoute conecta los segmentos de red NSX-T y los segmentos de administración de Azure VMware Solution.

>[!IMPORTANT]
>Todos los usuarios tienen un entorno diferente, aunque algunos deberán permitir que estas rutas se propaguen de nuevo a la red local.  

## <a name="next-steps"></a>Pasos siguientes
Continúe con el siguiente tutorial para aprender a implementar y configurar la solución VMware HCX para su nube privada de Azure VMware Solution.

> [!div class="nextstepaction"]
> [Implementación y configuración de VMware HCX](tutorial-deploy-vmware-hcx.md)


<!-- LINKS - external-->

<!-- LINKS - internal -->
